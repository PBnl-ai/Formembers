# Restaurants Auto-Import — Volledige Spec

**Project van herkomst:** recepten.pb.nl (Next.js + Drizzle + Neon Postgres)
**Doel van dit document:** een complete, op zichzelf staande beschrijving van hoe restaurants automatisch worden geïmporteerd in Recepten — geschikt om als blueprint te gebruiken voor een ander project (zoals "voor members" / Portal).

---

## 1. Doel & overzicht

Restaurants worden niet handmatig één voor één ingevoerd. Het systeem ondersteunt drie invoer-routes:

| Route | Bedoeling | Endpoint |
|---|---|---|
| **Discovery** | Bronpagina (bv. Lekker500 / Michelin) afstropen → lijst van kandidaten | `POST /api/restaurants/discover` |
| **Auto-import** | Eén restaurant volledig opbouwen vanuit naam + stad + optionele hints | `POST /api/restaurants/auto-import` |
| **Manual import** | Klaar payload van een externe bron (bv. ChatGPT-custom-GPT) inslikken | `POST /api/restaurants/import` |

De typische flow in het CMS:
1. Admin opent `/cms/restaurants/import`
2. Klik op een bron (Lekker500, Gault&Millau, Michelin per provincie, of custom URL) → **Discovery** geeft lijst
3. Admin selecteert restaurants (alle nieuwe / volgende 10 / per stuk)
4. Voor elke selectie → **Auto-import**: Google Places + scraping + GPT-4o vult alle velden, schrijft NL+EN copy, upload hero image naar R2
5. Restaurant wordt als concept (`isOffline: true`) opgeslagen — admin kan nog reviewen voor publicatie

Alle ingebouwde dedup checkt fuzzy op naam (lowercase + alleen alphanumeriek, "&" → "en") om bv. "Brut 172" en "Brut172" te matchen.

---

## 2. Data model (Drizzle / Postgres)

Tabel `restaurants`:

```ts
{
  id: varchar (uuid, primary key, default gen_random_uuid())
  name: text NOT NULL
  slug: text NOT NULL UNIQUE
  city: text NOT NULL
  intro: text NOT NULL          // 400-700 tekens, persoonlijke beschrijving
  rating: integer NOT NULL DEFAULT 45  // 45 = 4.5 sterren (x10 schaal)
  price: text NOT NULL DEFAULT "€€€"
  cuisine: text NOT NULL        // enum uit ALLOWED_CUISINES
  tags: text[] NOT NULL DEFAULT '{}'  // max 6
  imageUrl: text                // hoofdbeeld (R2 hosted)
  imageUrl2: text               // optioneel tweede beeld
  website: text                 // eigen restaurant-website
  reserveUrl: text              // alleen erkende platforms (zie §5)
  phone: text                   // internationaal formaat (+31..., +44..., +1...)
  openingHours: text            // JSON string: [{dayOfWeek, opens, closes}] × 7
  addressStreet: text
  addressZipcode: text
  addressCity: text
  addressMaps: text             // google maps link
  country: text DEFAULT "NL"
  featuredInSlider: boolean DEFAULT false
  province: text
  seoText: text                 // 3-5 zinnen voor SEO
  isOffline: boolean NOT NULL DEFAULT false  // true = concept/draft
  // bilingual
  language: text NOT NULL DEFAULT "nl"  // 'nl' | 'en'
  originalRestaurantId: varchar        // EN versie linkt naar NL origineel
  isTranslation: boolean NOT NULL DEFAULT false
  autoSyncEnglish: boolean NOT NULL DEFAULT true  // auto-vertaal bij save
  createdAt: timestamp DEFAULT now()
  updatedAt: timestamp DEFAULT now()
}
```

Indexen: `slug`, `city`, `language`.

### Aangeleverde cuisine-enum
Vaste lijst van ~50 waarden (`ALLOWED_CUISINES` in `app/api/restaurants/auto-import/route.ts`). Als GPT iets retourneert dat niet in de lijst staat → fallback naar `"Overig"`. Voorbeelden:
- Europees: Italiaans, Frans, Spaans, Grieks, Portugees, Brits, Scandinavisch
- Aziatisch: Japans, Chinees, Thais, Indiaas, Koreaans, Vietnamees, Indonesisch
- Stijl/type: Fine Dining, Bistro, Brasserie, Steakhouse, Pizzeria, Tapas, Omakase, Sushi, Robata, Yakitori, Ramen, BBQ, Smokehouse
- Dieet: Vegetarisch, Veganistisch, Biologisch, Farm-to-table
- Fallback: `Overig`

---

## 3. Discovery API — `POST /api/restaurants/discover`

**Input:**
```json
{ "url": "https://lekker.nl/restaurant/klassering" }
```

**Output:**
```json
{
  "source": "Lekker500",
  "total": 500,
  "new_count": 412,
  "existing_count": 88,
  "restaurants": [
    {
      "name": "De Nieuwe Winkel",
      "city": "Nijmegen",
      "website": "",
      "source_url": "https://lekker.nl/restaurant/de-nieuwe-winkel",
      "cuisine": "",
      "price": "",
      "description": "",
      "exists": false
    }
  ]
}
```

### Parsing-strategieën per type bron

Het endpoint kiest een strategie op basis van de URL:

| Detectie | Strategie | GPT nodig? |
|---|---|---|
| `lekker.nl` | Parse Inertia.js `data-page` attribute uit HTML (fallback: aparte `X-Inertia` JSON request) | nee |
| `gault-millau.nl` | Tegel geografisch API: NL bounding box (51.3-53.55 lat, 3.35-7.25 lng) opgedeeld in 0.4°×0.5° gridcellen, parallel batches van 10 | nee |
| `guide.michelin.com` | Regex op `card__menu-content--title` blokken, auto-paginatie tot 10 pages (48/page) | nee |
| anders | **GPT-4o extract** met embedded-JSON detection (Wix `warmupData`, `__NEXT_DATA__`, generieke JSON blocks) | ja |

### Overview-page detection (alleen GPT-pad)
Als ≥50% van de gevonden "restaurants" namen heeft die matchen op `/best|top|beste|restaurants in|guide to/i` → de pagina is een overzicht. Het systeem volgt dan de sub-page links (max 15 parallel in batches van 5) en parsest elke onderpagina apart. Werkt voor lijsten als "Best Restaurants in [city]" met links naar stadspagina's.

### Fuzzy dedup
Voor elk gevonden restaurant:
```ts
const norm = (s) => s.toLowerCase().replace(/[&]/g, "en").replace(/[^a-z0-9]/g, "")
exists = bestaande NL restaurants .find(r => norm(r.name) === norm(found.name))
```

### Auth
`verifyAuth(request)` — 401 als geen geldige token. Zie `app/lib/auth.ts`.

### Timeout
`export const maxDuration = 120` — Vercel function timeout op 120 sec (grote Michelin-pages + sub-page fan-out kan lang duren).

---

## 4. Auto-import API — `POST /api/restaurants/auto-import`

Hét endpoint dat 1 restaurant volledig opbouwt uit minimale input.

**Input:**
```json
{
  "name": "De Nieuwe Winkel",       // required
  "city": "Nijmegen",                // optional (wordt anders afgeleid)
  "website": "https://...",          // optional hint
  "source_url": "https://lekker.nl/restaurant/de-nieuwe-winkel",  // optional
  "cuisine_hint": "Fine Dining",     // optional
  "price_hint": "€€€€",              // optional
  "description": "..."               // optional context voor GPT
}
```

**Output (succes):**
```json
{
  "success": true,
  "restaurant": {
    "id": "...", "name": "...", "slug": "...", "city": "...",
    "cuisine": "...", "imageUrl": "...",
    "hasImage": true, "hasPhone": true, "hasAddress": true, "hasHours": true
  }
}
```

**Output (duplicate):** 409 status met `{ duplicate: true, duplicateId, error }`.

### Pipeline (volgorde belangrijk!)

1. **Fuzzy dedup** op naam — als bestaat: 409.
2. **Google Places API** — `places:searchText` met `"${name} restaurant ${city}"` → place_id, dan `places/{id}` voor details. Geeft: `phone`, `addressStreet`, `addressZipcode`, `addressCity`, `country`, `website`, `openingHours`, `rating`. **Meest betrouwbare bron voor feitelijke data.**
3. **Fetch source page** (`source_url`) — bv. de Lekker500 detail-pagina.
4. **Vind restaurant website** — priority: body.website > Google Places > regex match in source HTML (zoekt links die niet naar agency/social/gids zelf wijzen).
5. **Fetch restaurant website HTML** (15s timeout, browser-style User-Agent).
6. **OG image** — extract `og:image` uit source/website HTML → download → check ≥ 200×200 px en > 5KB → Sharp JPEG q80 compress → upload naar Cloudflare R2 als `restaurants/{slug}/hero.jpg`.
7. **GPT-4o extract & generate** (`extractAndGenerate`) — één call, model `gpt-4o`, temperature `0.4`, `response_format: json_object`. Krijgt ge-cleande HTML van source + website + Google + description hint. Retourneert:
   - feitelijk: `city`, `phone`, `addressStreet`, `addressZipcode`, `addressCity`, `openingHours`, `reserveUrl`
   - redactioneel: `intro` (NL, 400-700 chars), `seoText` (3-5 zinnen NL), `cuisine` (uit `ALLOWED_CUISINES`), `tags` (max 6 specifiek)
8. **Merge** Google Places > GPT-4o voor feiten (`phone || `, `addressStreet || ` etc.).
9. **Race-condition slug check** — opnieuw kijken of slug ondertussen bestaat (door parallelle import).
10. **INSERT** als `isOffline: true, language: "nl"`.
11. **Translate to EN** — aparte GPT-4o call, vertaalt `name, intro, cuisine, tags, seoText`. Insert tweede record met `slug + "-en"`, `language: "en"`, `isTranslation: true`, `originalRestaurantId`.
12. **Revalidate Next.js cache** voor de slug.

### Hardcoded redactionele regels (system prompt)

GPT krijgt strikte instructies om generieke marketingtaal te vermijden:

**Verboden zinnen in `intro`:**
- "passie" / "met passie"
- "de beste ingrediënten"
- "je voelt je thuis"
- "keer op keer terug"
- "vriendelijk en attent"
- "gezellige maaltijd"
- "modern en uitnodigend"
- "zorgvuldig samengesteld"
- "genieten van"
- elke zin die op ELK restaurant zou kunnen slaan

**Verplicht in intro:** wat dit restaurant UNIEK maakt, signature dishes, sfeer SPECIFIEK ("industrieel interieur met open keuken en vinyl op de achtergrond" ipv "gezellig"), de locatie als bijzonder.

### URL-filters (anti-spam / anti-agency)

```ts
const BLOCKED_DOMAINS = /\.(agency|marketing|design|studio|consulting|reclame|media)\b/i
const BLOCKED_KEYWORDS = /flyingelephant|wordpress\.com|squarespace\.com|wix\.com/i
```

Het `website`-veld wordt door `cleanWebsiteUrl()` gefilterd — als de URL match → set op `null`. Voorkomt dat het systeem reclamebureau-portfolios als "restaurant website" opslaat.

### Reservation URL whitelist

```ts
const reservePlatforms = /couverts|thefork|formitable|quandoo|resengo|tablemanager|opentable|iens|seatme|booking/i
```

GPT-4o vult vaak een gewone website als `reserveUrl`. Als de URL **niet** matcht met deze platforms → wordt overgeheveld naar `website` (en `reserveUrl` wordt `null`).

### HTML cleaning vóór GPT
Voor zowel discover (long-page) als auto-import:
- Verwijder `<script>`, `<style>`, `<noscript>`, `<svg>`, `<img>`, `<header>`, `<footer>`, `<nav>`, `<iframe>`, comments
- Strip `class=`, `style=`, `data-*` attributen
- Collapse witruimte
- Truncate naar 40k chars (auto-import) / 60k chars (discover, 30k als embedded JSON aanwezig)

### Random rating
Bij INSERT: `rating = 36 + floor(random() * 9)` (= 3.6 t/m 4.4 sterren). Idee: kandidaten beginnen redelijk, admin verfijnt handmatig.

---

## 5. Manual import API — `POST /api/restaurants/import`

Bedoeld voor externe leveranciers van data (typisch een custom GPT die JSON teruggeeft). Geen scraping, geen Google Places, geen image upload — gewoon: payload → INSERT.

**Input** (alle velden optioneel behalve `name`/`city`/`intro`/`cuisine`):
```json
{
  "name": "...",
  "city": "..." | { "address": { "city": "..." } },
  "intro": "..." | "description": "...",
  "cuisine": "..." | ["..."] | "type",
  "rating": 45 | { "average": 4.5, "scale": 5 },
  "price": "€€€" | "priceLevel": 3,
  "tags": ["..."],
  "openingHours": [{ "dayOfWeek": "Monday", "opens": "12:00", "closes": "22:00" }],
  "addressStreet": "...",   "address.street": "...",
  "addressZipcode": "...",  "address.postalCode": "...",
  "addressCity": "...",
  "addressMaps": "...",     "googleMapsUrl": "...",
  "website": "...",
  "reserveUrl": "..." | "reservationUrl": "...",
  "phone": "...",
  "imageUrl": "...",
  "province": "...",
  "country": "...",
  "seoText": "..." | "sfeer_vibe": "..." | "seo_text": "..."
}
```

Accepteert ook een outer wrapper: `{ "restaurant": { ... } }` (ChatGPT-stijl).

**Field mapping** (zie `app/api/restaurants/import/route.ts` voor exact gedrag):
- Rating: `number` → as-is. `{ average }` → `round(average * 10)`.
- Price: `string` → as-is. `{ priceLevel: N }` → `"€".repeat(N)`.
- OpeningHours: array → `JSON.stringify`.
- Cuisine: array → eerste element. String → as-is. Niets → `""`.

Altijd `isOffline: true`. Dedup op naam (case-insensitive). Slug auto-generated, met `-{timestamp}` suffix bij collision.

---

## 6. Check duplicate API — `POST /api/restaurants/check-duplicate`

Lichtgewicht batch check.

```json
// in
{ "names": ["De Nieuwe Winkel", "Brut 172"] }
// out
{ "De Nieuwe Winkel": true, "Brut 172": false }
```

Gebruikt `ilike` (case-insensitive), alleen NL records.

---

## 7. CMS UI patroon — `/cms/restaurants/import`

### Layout
- **Bronnen-grid** (boven): default bronnen Lekker500 + Gault&Millau, plus 12 Michelin-provincies in een uitklapbare lijst. Cards tonen cached counts: `nieuw/totaal`.
- **Custom URL form**: naam + URL → discover + opgeslagen in `localStorage` als custom bron.
- **Resultaten-tabel**: kolommen `selecteer | naam | stad | cuisine | status | actie`. Per rij:
  - `exists: true` → grijs + "bestaat al"
  - `importStatus`: idle → importing (spinner) → done (checkmark) → error (rode tekst)

### State management
- `localStorage["restaurant-import-sources"]` — custom bronnen (naast defaults)
- `sessionStorage["restaurant-import-cache"]` — discover-resultaten per bron-URL (overleeft refresh)
- `sessionStorage["restaurant-import-active"]` — laatste actieve bron

### Bulk select acties
- "Selecteer alle nieuwe"
- "Selecteer volgende 10" (alleen niet-bestaand + idle + nog niet geselecteerd)
- "Selecteer niets"

### Sequentiële import
Geselecteerde restaurants worden **één voor één** geïmporteerd via `auto-import`. Reden: elke call is 30-90 sec (Google Places + 2× HTML fetch + 2× GPT-4o + R2 upload). Parallelliseren zou Vercel functions OOM-en.

Per call: 115 sec client-timeout (server is 120). Bij duplicate (409) → markeer als `exists: true` in ALLE gecachte bronnen (via `markExistsEverywhere` fuzzy match).

### Revalidatie
Na elke succesvolle import roept het backend `revalidateRestaurant(slug)` aan — die invalideert de Next.js ISR cache voor de detail-pagina én de lijst.

---

## 8. External dependencies & ENV vars

### NPM packages
- `openai` — GPT-4o calls
- `@aws-sdk/client-s3` — R2 upload (R2 is S3-compatible)
- `sharp` — image dimension check + JPEG compression
- `drizzle-orm`, `next` — basis stack

### Environment variables
```bash
OPENAI_API_KEY=sk-...
GOOGLE_PLACES_API_KEY=...

R2_ACCOUNT_ID=...
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=...
R2_BUCKET=recepten-pbnl
R2_PUBLIC_URL=https://pub-xxx.r2.dev
```

Als `OPENAI_API_KEY` ontbreekt → auto-import faalt (verplicht).
Als `GOOGLE_PLACES_API_KEY` ontbreekt → graceful fallback: alleen GPT-4o extraction (minder betrouwbaar voor feiten).
Als één van de R2 vars ontbreekt → graceful fallback: geen image upload, `imageUrl: null` (admin moet handmatig).

---

## 9. Belangrijke regels / fallbacks / edge cases

### Opening hours: lunch + diner consolidation
Een restaurant met 2 perioden per dag (12:00-14:30 + 17:30-22:30) wordt geconsolideerd tot één periode: vroegste opens, laatste closes (12:00-22:30). Speciaal geval: closing tijd vóór 06:00 wordt behandeld als "next day" zodat 23:00 < 01:00 (next day) correct sorteert.

### Day mapping (Google Places periods)
`period.open.day` is 0-6 met 0 = Sunday (Google's conventie). Wordt gemapt naar:
```ts
["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"]
```

### HTML entities
Restaurant-namen uit scraped HTML bevatten vaak `&#39;`, `&amp;`, `&quot;` — voor INSERT wordt alles gedecodeerd via `decodeHtmlEntities()` (in beide routes geïmplementeerd).

### Race-condition slug check
Bij parallelle imports kan dezelfde slug door 2 calls tegelijk worden geclaimd. Auto-import doet **twee** checks: één bij start (fuzzy op naam) én één vlak voor INSERT (exact op slug). Bij collision: 409.

### Image rejection criteria
OG image wordt afgewezen als:
- Download fail of < 5 KB raw
- Sharp metadata fail of < 200×200 px
- Compressed JPEG < 10 KB (waarschijnlijk lege placeholder)

### Always offline by default
Elke import zet `isOffline: true`. Restaurants verschijnen niet op de publieke site totdat een admin ze in het CMS handmatig op `isOffline: false` zet. Geeft tijd om de intro te reviseren.

### Auto-translate kan falen zonder INSERT te falen
De `translateToEnglish` call is in `try/catch`. Als die faalt → log, NL record blijft. Admin kan later handmatig vertalen.

---

## 10. Quick reference — file paths

```
shared/schema.ts:206-252         restaurants table + insert schema
app/api/restaurants/
  discover/route.ts              POST: bronpagina parsen → lijst kandidaten (Lekker500/Gault&Millau/Michelin/GPT)
  auto-import/route.ts           POST: 1 restaurant volledig opbouwen
  import/route.ts                POST: manual import (extern payload)
  check-duplicate/route.ts       POST: batch dedup check
  route.ts                       GET: list + filters (publiek)
  [id]/route.ts                  GET/PATCH/DELETE: single restaurant
app/cms/restaurants/
  page.tsx                       overzicht
  import/page.tsx                de auto-import UI (660 regels)
  new/page.tsx                   handmatig toevoegen
  edit/[id]/page.tsx             bewerken
app/components/
  CMSRestaurantForm.tsx          shared form NEW + EDIT
  RestaurantsClient.tsx          publieke lijst
  RestaurantDetailClient.tsx     publieke detail
  HeroSliderRestaurants.tsx      slider met featuredInSlider=true
app/lib/
  auth.ts                        verifyAuth(request) → bool
  revalidate.ts                  revalidateRestaurant(slug)
```

---

## 11. Wat NIET in dit systeem zit

- Geen "voorgestelde restaurants" / discovery zonder admin-trigger
- Geen automatische re-fetch (data wordt 1x opgehaald, blijft tot admin herimporteert)
- Geen image-fallback chain (alleen OG image — als die ontbreekt: handmatig)
- Geen bulk-edit van bestaande records via import (alleen INSERT, geen UPDATE)
- Geen monitoring/alerts bij bron-pagina structuurwijzigingen (Michelin/Lekker kunnen breken → handmatig fixen)

---

## 12. Patronen die je sowieso wilt overnemen bij een hergebruik

1. **Driedeling discover / auto-import / manual import** is goud waard — elk dekt een ander invoer-scenario zonder over te lappen.
2. **`isOffline` (draft-flag) bij ALLE imports** voorkomt prutsige content op de publieke site.
3. **Fuzzy dedup** (`normalize = lowercase + alphanum only + "&" → "en"`) — voorkom 95% van duplicate-frustraties.
4. **Google Places > GPT-4o voor feiten, GPT-4o > Google voor copy** — combineer hun sterke kanten.
5. **System prompt met verboden marketingtaal-lijst** — drukt GPT-output naar bruikbaar niveau zonder elke generatie handmatig te herschrijven.
6. **Embedded-JSON detection** (Wix `warmupData`, `__NEXT_DATA__`) — bespaart vaak het hele GPT-call, en is betrouwbaarder.
7. **Overview-page sub-page volgen** — laat één discover-call een hele "Best Restaurants in [country]" lijst uitkleden over 15 sub-pagina's.
8. **Race-condition guard** met dubbele slug-check + sequentiële import in UI — geen "duplicate key" errors meer.
9. **Reservation-URL whitelist** — voorkomt dat GPT je `reserveUrl`-veld misbruikt als 2e website-veld.
10. **Auto-translate in achtergrond na INSERT** — onafhankelijk van originele insert, faalt veilig.

---

**Versie:** opgesteld op basis van repo-state 13 mei 2026 (recepten.pb.nl `main` branch).
