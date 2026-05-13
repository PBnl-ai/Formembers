# For Members — Project Constitution

> **Lees dit voor je iets bouwt.** Dit is het werkdocument dat de richting bepaalt. Wat hier staat wint van wat ik in een losse prompt zeg. Als iets onduidelijk is, vraag terug — niet improviseren.

---

## Imports — altijd meeladen

@HUISSTIJL.html — De visuele bron van waarheid. Kleur, typografie, componenten. Géén Tailwind-klasse, kleurtoken of font hardcoden zonder dit te raadplegen.

@CONCEPT_DATAMODEL.md — De productlogica. Wat is een Entry, hoe werkt de Place-hiërarchie, wat doen Routes, wat krijgt een member vanaf dag 1.

---

## Wat For Members is

Een private editorial atlas voor stille luxe. Hotels, restaurants, winkels, musea, design — wereldwijd, met dichtheid-gedreven stadspagina's, AI-gegenereerde context, en personalisatie voor ingelogde members. Onderdeel van Studio PB.NL.

**Naam — definitief:** *For Members.* (mét punt). Werknaam "PORTAL" is op 13 mei 2026 afgevoerd. Het Vercel-project heet intern nog `portal` omdat de project-ID stabiel moet blijven, maar overal in copy, GitHub-repo, domein en titels is het For Members.

**Toon:** ingetogen, editorial, niet sjiek. We schrijven als magazine, niet als directory.

**Niet:** een reisgids, een marktplaats, een Trustpilot-kloon. Géén punten, badges, of streaks. Géén "luxueus" of "onvergetelijk" in copy.

---

## De besluiten die al genomen zijn

Niet opnieuw ter discussie stellen, niet "verbeteren", niet "moderner maken". Vraag terug als je iets wil afwijken.

### Huisstijl

- **Light-first** — `#f4f2ef` is de pagina-achtergrond. Dark mode beschikbaar via toggle (rechtsboven), maar niet de default.
- **Theme via CSS-variabelen** — exact zoals in `HUISSTIJL.html`. `data-theme="light|dark"` op `<html>`, anti-flash script in `<head>`, `localStorage` key `fm-theme`.
- **Eén accentkleur: taupe `#afa297`.** Géén accent-rood, géén success-groen. Status communiceren met copy, niet met kleur.
- **PB-neutralen** — `--bg`, `--bg-deep`, `--surface`, `--surface-alt`, `--text`, `--text-sec`, `--text-muted`, `--text-faint`, `--text-dim`, `--text-ghost`, plus de border-familie. Allemaal in `HUISSTIJL.html`.
- **Fonts: Fraunces + Inter Tight.** Niets anders. Géén Bebas, géén monospace.
- **Border-radius: 0** op alle UI-componenten.
- **Easing: `cubic-bezier(0.16, 1, 0.3, 1)`** — de signature.
- **Iconen: Iconify Solar Linear, stroke-width 1.5.** Géén Lucide, géén gevulde varianten.

### Wat For Members expliciet NIET overneemt uit PB-familie

Glass-morfisme, shimmer, scanlines, heartbeat glow, flashlight, corner marks, monospace UI-labels. Die horen bij DMM/PerfectMoods/Studio. Hier niet.

### Productlogica (zie `CONCEPT_DATAMODEL.md` voor details)

- **Eén Entry-type** met verplichte lat/lng. Place-hiërarchie (Country → Region → City → District) is afgeleid uit coördinaten, niet handmatig.
- **City-promotie:** standaard ≥ 5 entries én ≥ 2 categorieën. Instelbaar.
- **Routes en Lists** zijn cross-cutting containers, geen sub-types van Place.
- **Member day-1 features:** Save, Lists, Status (wishlist/visited/favorite), Notes. Méér mag opgevoerd worden in v1.5, niet nu.
- **AI** schrijft Country/Region/City-intro's en "in de buurt"-blokken. Reviewable, niet auto-publiceren zonder veld in CMS.

---

## Status — MVP-demo v0.1 (13 mei 2026)

Gepubliceerd op **formembers.nl** via Vercel · git-tag `v0.1-mvp-demo` op GitHub PBnl-ai/Formembers.

**12 klikbare pagina's, allemaal in dezelfde stack:**

| Pagina | Doel |
|---|---|
| `index.html` | Home — hero (klikbaar naar Country), concept-blokken, landen, categorie-overzicht, steden-scroll, nieuw-section |
| `country.html` | Frankrijk — hero, AI-intro, Lyon prominent + 3 teaser-steden, 4 categorie-secties |
| `city.html` | Lyon — hero, intro, entries per categorie |
| `entry.html` | Cour des Loges Lyon — hero, gallery, feiten, in de buurt |
| `atlas.html` | Mijn Atlas — saved entries dashboard, lists, status |
| `route.html` | Routes — Route du Soleil als voorbeeld |
| `auth.html` | Inloggen / Aanmelden |
| `coming-soon.html` | Volgt — placeholder voor unbuilt categorieën |
| `ontdek.html`, `sanctuary.html`, `sanctuary-aman-tokyo.html`, `shop.html` | Categorie-pagina's, vroege stijlreferentie (gemigreerd naar pb-* tokens, consistent met de rest) |
| `HUISSTIJL.html` | Design-systeem reference, niet productie |

**Wat overal consistent is geregeld (verwacht het, breek het niet):**
- Fraunces + Inter Tight, taupe-accent `#afa297`, border-radius 0
- pb-* / fm-* CSS-vars + tokens — géén stone-* of hex-codes
- Fixed `nav.fm-nav-bar` met: aurora drift achter de blur (22s loop, GPU-accelerated), search-dropdown als sibling binnen `<nav>`, theme-toggle in fullscreen menu footer (níet meer fixed top-right)
- Anti-flash script + `data-theme="light|dark"` + `localStorage.fm-theme`
- `<meta robots="noindex,nofollow">` op alle 12 pagina's — demo blijft uit Google
- Page titles consistent als `[Pagina] | For Members.` (NL)
- Hero-foto's lokaal in `assets/` als WebP (frankrijk.webp 256KB, lyon.webp 426KB) of geoptimaliseerd JPG — géén kapotte Unsplash-URLs meer
- Op mobiel zijn de concept-uitleg blokken op de homepage `fm-collapsible` (titel + pijl, body uitvouwbaar)

**Niet meer aanwezig (regressies hier opnieuw introduceren = fout):**
- Geen `client/` of `src/` directories, geen build-step
- Geen `position: relative` op `nav.fm-nav-bar` (overrulet `position: fixed` van Tailwind — top-balk verdwijnt bij scrollen)
- Geen Engelstalige page titles of UI-copy
- Geen werknaam "PORTAL" of "L'INDEX" in user-zichtbare tekst

---

## Volgende stappen na MVP-demo

Niet meer "één pagina afmaken" — die fase is voorbij. Nu meer thematisch:

1. **Echte kaart** op `atlas.html` + `entry.html` (Google Maps embed of Mapbox) — vervangt de huidige mock-SVGs.
2. **Tweede land** (bv. Japan met Tokyo + Aman Tokyo) om te bewijzen dat het Country/City/Entry patroon schaalt zonder copy-paste-rot.
3. **CMS-laag** — entries staan nu hardcoded in HTML. Beslissingsmoment: Notion-driven, simpele JSON + static-gen, of vooruit naar Next.js + Drizzle/Neon (zoals Recepten/PerfectMoods).
4. **Member-laag** — Save/Lists/Status werkt nu via `localStorage`. Voor echte gebruikers: auth + database.

Bouwvolgorde voor de toekomst: probeer steeds één van bovenstaande thema's af te ronden voor je aan de volgende begint.

---

## Bestaande HTMLs — ALTIJD eerst lezen

Sinds v0.1-mvp-demo zijn alle 12 pagina's gemigreerd naar dezelfde stack — er zijn dus géén "oude" pagina's meer. Maar voor je iets bouwt:

**REGEL — niet onderhandelbaar:** open eerst `index.html` (rijkste page) en `country.html` (het Country/City pattern), bestudeer header, footer, nav, card-patronen, breakpoints, manier waarop overlines + serif-koppen + meta-labels gestapeld worden. Hergebruik die structuur — niet vanaf nul beginnen, niet "moderner" maken.

**Bij twijfel:** `HUISSTIJL.html` wint — dat is het design-systeem reference. De productie-pagina's volgen het, maar `HUISSTIJL.html` is autoritatief over kleuren, fonts, schaal, componenten.

---

## Werkafspraken

### Codestijl

- **Tailwind CDN.** Geen build-step, geen PostCSS, geen extractie naar CSS-files. Inline classes.
- **Tailwind config** in een `<script>`-blok bovenin elke pagina, identiek aan `HUISSTIJL.html`. Géén verschillende configs per pagina.
- **CSS-variabelen** voor thema-tokens, in een `<style>`-blok onder de Tailwind config. Klassen verwijzen naar `var(--bg)`, niet naar hardcoded hex.
- **Iconify** via `<iconify-icon>`-tag, niet via SVG-inline (tenzij het logo).
- **Image filter in dark mode:** `contrast(1.05) brightness(0.97)` op `<img>` met `transition: filter 0.7s`. Reset op hover.

### Bestandsstructuur

```
/
  index.html           # home (laatste in volgorde)
  city.html            # template — gebruik Lyon als demo
  entry.html           # template — gebruik één hotel uit Lyon
  country.html         # template — Frankrijk
  route.html           # Route du Soleil
  atlas.html           # My Atlas (logged-in)
  auth.html            # sign-up / login
  HUISSTIJL.html       # bron van waarheid — niet wijzigen zonder overleg
  CONCEPT_DATAMODEL.md # productlogica
  CLAUDE.md            # dit bestand
```

Géén `/assets/`, géén `/css/`, géén `/components/` map. Alles inline. Pas wanneer er drie pagina's dezelfde header hebben, hardcoded copy-paste — refactoring komt later met een echte build-step.

### Wat NIET te doen

- ❌ Géén nieuwe fonts laden, géén Inter, géén Roboto, géén "moderne sans".
- ❌ Géén shadcn/ui, géén Radix, géén React. Plain HTML.
- ❌ Géén kleur toevoegen aan het palet zonder overleg. Eén accent.
- ❌ Géén `rounded-md` / `rounded-lg` / shadows. We doen scherpe hoeken.
- ❌ Géén toeristische clichés in copy ("onvergetelijk", "luxueus", "5-sterren", "perfect voor de veeleisende reiziger"). Zie sectie Toon in `HUISSTIJL.html`.
- ❌ Géén lifestyle stock photo's — Unsplash mag, maar kies architectuur/materie/licht, géén lachende mensen.
- ❌ Géén drone-zonsondergangen, géén HDR-grading, géén tekst-over-beeld overlays.
- ❌ Géén gradient hero's. Wel: gerichte hero met groot beeld + minimaal infopaneel + AI-intro daaronder.

### Wat WEL te doen

- ✅ Eerst de structuur, dan de styling, dan de content.
- ✅ Bij elke pagina: theme-toggle werkend, kruimelpad zichtbaar (Country › Region › City › Entry), kaart-placeholder (Google Maps embed mag mock zijn voor nu — gewoon een grijs vlak met pinnetjes).
- ✅ Een pagina moet ook in dark mode kloppen. Klik 'm na het bouwen, zorg dat geen tekst onleesbaar wordt.
- ✅ Mobile-first responsive — alle bestaande HTMLs werken al goed mobile, houd dat patroon vast.
- ✅ Bij elke nieuwe pagina: korte changelog-comment bovenaan met datum + wat erin zit. Zoals `<!-- v0.1 — 13 mei 2026 — eerste klikbare structuur, header + hero + entry-grid -->`.

---

## Open vragen — kom hier niet zelf op uit, vraag terug

- City-promotion drempel: 5+2 categorieën — exact zo aanhouden of instelbaar maken in CMS?
- Co-curatie (partner-account met edit-rechten) — day-1 of v1.5?
- Public lists ("Birgit's Tokyo edit") — vrij publiceren of redactionele review?

Bij elk van deze: vraag terug aan Peter voor je iets bouwt dat ervan uitgaat.

### Beslist sinds v0.1-mvp-demo

- ✅ **Naamgeving** — *For Members.* (mét punt). PORTAL is afgevoerd.
- ✅ **Theme-toggle locatie** — in de fullscreen menu footer met "Light mode" / "Dark mode" label. Niet meer als fixed knop top-right.

---

## Hoe te starten in een nieuwe sessie

1. Lees dit bestand (gebeurt automatisch).
2. Lees `HUISSTIJL.html` en `CONCEPT_DATAMODEL.md` als context nog ontbreekt.
3. Kijk welke pagina aan de beurt is volgens de bouwvolgorde hierboven.
4. Bouw één pagina af tot statisch klikbaar — geen halve dingen achterlaten.
5. Toon resultaat, vraag review voordat je doorgaat.

---

*Werkdocument · v0.2 · 13 mei 2026 · Studio PB.NL*
