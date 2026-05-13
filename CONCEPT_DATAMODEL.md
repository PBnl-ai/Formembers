# PORTAL — Conceptueel Datamodel & UX-flow

*Werknotitie. Geen implementatie, alleen de denkstructuur waar we op verder bouwen.*

---

## 1. Het mentale model in één zin

**Eén soort bouwsteen (Entry) + drie soorten containers (Place, Route, List), met AI als redactionele assistent en de gebruiker als curator van z'n eigen atlas.**

Alles wat op de site verschijnt is óf een Entry, óf een verzameling van Entries. Verzamelingen zijn op drie manieren mogelijk: geografisch (Place), thematisch-langs-een-lijn (Route), of persoonlijk (List). Die scheiding is belangrijk omdat dezelfde entry in alle drie tegelijk kan zitten zonder duplicatie.

---

## 2. Entiteiten

### 2.1 Entry — de bouwsteen

Eén adres, één plek, één entry. Hotel, restaurant, winkel, museum, galerie, designwinkel, antiekmarkt, bakkerij die het verdient.

Velden (kern):

- `id`, `slug`, `name`
- `category` — sanctuary / table / shop / culture / experience (uitbreidbaar; bewust kort gehouden)
- `intro` — NL, 400–700 chars, redactioneel; AI mag genereren maar Birgit reviewt
- `seoText` — langer, meer feiten
- `lat`, `lng` — *verplicht*, het anker waar alles op draait
- `address` — straat / postcode / stad / regio / land (afgeleid via Google Places, niet handmatig)
- `placeRefs` — referenties naar de Place-hiërarchie (zie 2.2); afgeleid uit lat/lng, niet handmatig gezet
- `tags[]` — max 6, vrij; AI suggereert
- `priceLevel`, `rating`, `website`, `reserveUrl`, `phone`, `openingHours`
- `imageUrl`, `gallery[]` — hero + extra beelden
- `isOffline` — concept/draft; default true zoals in het bestaande auto-import-systeem
- `language`, `originalEntryId`, `isTranslation` — bilingual NL/EN
- `featured`, `featuredInSlider` — redactionele knoppen

### 2.2 Place — de geografische container (dynamisch)

Geen vaste hiërarchie maar een **afgeleide**. Eén entry weet z'n coördinaten; daaruit volgt automatisch in welke containers hij valt:

```
Country (altijd, vanaf 1 entry)
  └─ Region (handmatig of clustering — bv. Provence, Cotswolds, Lake Como)
       └─ City (administratief, uit Google Places "locality")
            └─ District (later; bv. Le Marais, Shibuya)
```

**Promotion-regels** (instelbaar, niet nu in beton gieten):

- City → eigen pagina zodra ≥ 5 entries *én* ≥ 2 categorieën vertegenwoordigd. Onder die drempel: entry verschijnt alleen onder Country.
- Region → handmatig aangemaakt door Birgit (Provence, Côte d'Azur, Lake District) óf voorgesteld door AI wanneer een dichte cluster buiten één stad valt.
- District → pas relevant als één City > 20 entries heeft; dan splitsen.

De promotion is geen migratie maar een *zichtbaarheidsregel*: de entry verandert nooit, alleen welke pagina's hem als "kind" tonen.

### 2.3 Route — de cross-cutting container

Een Route is een geordende reeks Entries plus optioneel ankerpunten (steden, tussenstops). Routes negeren de Place-hiërarchie volledig.

Voorbeelden:

- **Route du Soleil** — Nederland → Zuid-Frankrijk, hotels en restaurants langs de A6/A7
- **Modernist Italy** — een week designhotels van Milaan tot Como
- **Champagneroute** — Reims → Épernay → Aÿ
- **Antiekroute Provence** — vier dorpen, drie markten, twee galeries

Velden: `id`, `slug`, `name`, `intro`, `entries[]` (geordend), `waypoints[]` (vrije ankerpunten met lat/lng + optionele tekst), `coverImage`, `durationDays`, `season` (wanneer mooist).

Routes zijn redactioneel, maar de UI laat de gebruiker er één afspelen op een kaart (auto-zoom langs de stops) en converteren naar een eigen Trip (zie 2.4).

### 2.4 User, List, Trip, Note — de personalisatie-laag

**User** — `id`, `email`, `name`, `createdAt`, `homeCountry`, `preferences{}`.

**List** — een named verzameling Entries die een user samenstelt. Eén user kan meerdere lists hebben ("Bruiloft Toscane 2027", "Weekend Parijs", "Bucket Japan"). Veld `visibility`: private / link-only / public. Public lists krijgen een eigen URL — een lijst kan dus zelf inhoud worden op de site, met attributie aan de samensteller.

**Trip** — een List met datums en volgorde. Genoeg metadata om er een mini-reisplan van te maken (datum heen, datum terug, nachten per stop).

**Note** — privé tekst van een user bij één Entry. "Hier was ik in mei 2025, vraag naar tafel 14." Onzichtbaar voor anderen.

**Status per Entry per User** — `wishlist` / `visited` / `favorite`. Drie booleans, niet één enum, want een plek kan tegelijk visited én favorite zijn.

---

## 3. Paginatypes

| Pagina | Wanneer | Wat erop |
|---|---|---|
| **Home** | altijd | Editorial overzicht, dynamisch op basis van seizoen + recent toegevoegd |
| **Country** | vanaf 1 entry | Big hero, AI-intro over het land, lijst sub-Regions/Cities die de drempel halen, "losse" entries onderaan |
| **Region** | handmatig of AI-suggestie | Big hero (landschap), AI-intro, Cities + losse entries, kaart |
| **City** | bij promotie (≥5 + ≥2 categorieën) | Big hero (skyline / signature beeld), AI-intro, entries per categorie, kaart, "in de buurt" verwijzingen |
| **Entry detail** | per entry | Hero + gallery, intro, feiten, kaart, "in de buurt" (3–5 entries binnen straal), pairing-suggesties via AI |
| **Route** | redactioneel | Lange-form scroll met geordende entries, embedded map, "start deze route" knop (→ Trip) |
| **Discover / Category** | per categorie | bestaand patroon (ontdek.html) |
| **My Atlas** | ingelogd | Persoonlijk dashboard — zie sectie 5 |

**Wat nu nog ontbreekt in de HTML:** kruimelpad (Country › Region › City › Entry), de map-component, de geolocation-prompt, het account-menu. De huidige HTML is puur stijlreferentie.

---

## 4. De landing op een plek — het "grote beeld"

Elke Country/Region/City/Route opent met een full-bleed hero (60–80vh). Drie lagen erover:

1. **Het beeld zelf** — één gecureerd signature-shot. Voor City idealiter geen cliché-skyline maar iets dat de *toon* zet (een binnenplaats, een straat, een interieur).
2. **Een minimaal infopaneel** — naam van de plek, aantal entries, koppels naar de categorieën die er zijn. Géén weerwidget en géén toeristische feitjes; dat past niet bij de toon.
3. **Een AI-geschreven intro** — 3–5 zinnen direct onder de hero, redactioneel van toon, geen marketingtaal. Birgit kan overschrijven; AI-versie blijft als fallback.

Op City-pagina's is de hero contextgevoelig: in juli zie je een ander beeld dan in december (seizoens-gallery, mits beschikbaar). Klein detail, voelt levend.

---

## 5. Map & locatie

**Op detail- en city-pagina's:** een Google Maps embed met alle relevante entries als pin. Klik op pin → mini-card → "naar de pagina". Default styling: light, weinig labels, past bij stille luxe.

**"Waar ben je nu" — een Compass-mode.** Eén knop bovenin: *Wat is er om me heen?* Bij toestemming pakt de site de geo-locatie en opent een kaart met de dichtstbijzijnde entries binnen 10/25/50 km — instelbaar met een slider. Werkt voor de bezoeker die toevallig al ergens is én voor de reisplanner die thuis zit te kijken (in dat geval val je terug op de homecountry van het profiel).

**Voor ingelogde members:** Compass-mode laat ook *jouw saved entries* zien als afzonderlijke laag — handig op reis.

**Routes op kaart:** een Route speel je af: de map zoomt sequentieel langs stops, met een tijdbalk eronder. Bedoeling is "trailer", niet GPS-navigatie. Voor echte navigatie linkt PORTAL door naar Google Maps.

---

## 6. Account & personalisatie — wat een member vanaf dag 1 kan

**Sign-up is bewust laagdrempelig:** naam, e-mail, wachtwoord (of magic link). Geen telefoonnummer, geen profielfoto verplicht. Eén overtuigend argument waarom: *"Bewaar adressen, bouw je eigen atlas."*

**Day-one features (MVP-personalisatie):**

- **Save** — elke entry heeft een save-knop. Save zonder lijst gaat naar "Mijn Atlas" als algemene bewaarbak.
- **Lists** — named lijsten aanmaken en entries verplaatsen/kopiëren.
- **Status-toggle** — wishlist / visited / favorite per entry.
- **Notes** — privé tekstveld per entry. Onzichtbaar voor anderen.
- **My Atlas** — persoonlijke dashboard-pagina met (a) wereldkaart waarop al je saved entries als pin verschijnen — *de kaart vult zich naarmate je rondkijkt* — en (b) je lists, status-overzicht, recent gekeken entries.

**Day-soon features (V1.5, conceptueel meenemen):**

- **Co-curatie** — deel een lijst met je partner met bewerkingsrechten. Beide kunnen toevoegen, beide zien notes.
- **Public lists** — zet een lijst op publiek; krijgt een eigen URL, kan op de site verschijnen als "Birgit's Tokyo edit".
- **Trip-modus** — een List krijgt datums + volgorde; AI vult gaten ("je hebt geen lunch op dag 3, hier zijn 2 opties tussen Aix en Avignon").
- **Seizoens-attendering** — "Je hebt 4 plekken in Provence opgeslagen, beste maanden zijn mei–juni en september."
- **Geofence-notificaties** — push als je binnen 2 km van een saved plek komt (alleen mobile/app, optionele toestemming).
- **Visit log** — markeer "I was here" met datum en optioneel notitie. Achteraf: jaaroverzicht.

**Inspiratie / referentievoorbeelden (uiteenlopend, voor toon en mechaniek):**

- *Letterboxd* — public lists als sociaal object, zonder de site sociaal te laten voelen. Curatie wordt content.
- *Strava heatmap* — visualisatie van waar je geweest bent als persoonlijke beloning, niet als prestatie.
- *Apple Maps Guides* — hoe je een lijst van plekken verpakt als publiceerbare gids.
- *Soho House app* — de "for members" toon: rustig, informatief, geen toeters.
- *Aman / Belmond apps* — luxury personalisatie zonder gimmick.
- *Notion* — de manier waarop genest persoonlijk eigendom voelt zonder zwaar te zijn.

Géén punten/badges/streaks. Past niet bij de positionering.

---

## 7. AI als redactioneel zenuwstelsel

Drie rollen, scherp afgebakend:

1. **Schrijver van basisteksten** — Country-, Region-, City-intro's en "in de buurt"-paragraafjes op detailpagina's. Genereer bij creatie/promotie, sla op als veld, Birgit kan overschrijven. AI-versie nooit live tonen zonder review-mogelijkheid.
2. **Verbinder** — bij elke detailpagina: pairing-suggesties ("dit hotel + dit restaurant op loopafstand"), nearby-blok (3–5 entries binnen 2 km of binnen dezelfde city).
3. **Redactionele assistent voor Birgit** — gat-signalering ("Lyon heeft hotels en eten, maar geen design/winkels; overweeg X, Y, Z"), seizoens-suggesties op de homepage, en kandidaat-lijsten via discovery (bestaande systeem uit `RESTAURANTS_AUTO_IMPORT.md` kan grotendeels worden hergebruikt).

AI raakt nooit *user-content*: notes, lists, journal blijven privé en worden niet als trainingsinput of contextbron gebruikt.

---

## 8. Wat we nu vastleggen vs. later beslissen

**Nu in steen:**

- Eén Entry-type, lat/lng verplicht, Place is afgeleid niet handmatig.
- Place-hiërarchie Country → Region → City → District, dynamisch zichtbaar.
- Routes en Lists zijn aparte cross-cutting containers, geen sub-types van Place.
- User-personalisatie krijgt 4 dingen vanaf dag 1: Save, Lists, Status, Notes.
- AI is altijd reviewbaar; auto-publicatie alleen voor "in de buurt"-snippets, niet voor hoofdteksten.

**Later beslissen (komt vanzelf bij eerste echte content):**

- Exacte drempel voor City-promotion (start: 5 entries + 2 categorieën, evalueer bij 10 echte steden).
- Of District een eigen pagina-type krijgt of een sectie op de City-pagina blijft.
- Hoe diep de Trip-modus gaat (alleen volgorde + datum, of inclusief routing/transport).
- Of public lists een editorial check krijgen of vrij gepubliceerd kunnen.
- Definitief vocabulaire NL/EN voor "Save / Wishlist / Visited / Favorite".

---

## 9. Volgende stap

Twee mogelijke richtingen:

- **A. Wireframe-flow** voor één concrete case: een City-pagina (bv. Lyon zodra de drempel valt) inclusief breadcrumb, hero, secties, "in de buurt", en kaart.
- **B. Concrete schema-schets** in Drizzle/Postgres-stijl, voortbouwend op het bestaande restaurants-schema uit het Recepten-project — zodat het CMS straks dezelfde patronen volgt.

A geeft het meest visuele gevoel, B geeft Birgit (of een ontwikkelaar) iets om mee verder te bouwen. Beide kunnen, in deze volgorde.
