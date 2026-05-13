# For Members — Project Constitution

> **Lees dit voor je iets bouwt.** Dit is het werkdocument dat de richting bepaalt. Wat hier staat wint van wat ik in een losse prompt zeg. Als iets onduidelijk is, vraag terug — niet improviseren.

---

## Imports — altijd meeladen

@HUISSTIJL.html — De visuele bron van waarheid. Kleur, typografie, componenten. Géén Tailwind-klasse, kleurtoken of font hardcoden zonder dit te raadplegen.

@CONCEPT_DATAMODEL.md — De productlogica. Wat is een Entry, hoe werkt de Place-hiërarchie, wat doen Routes, wat krijgt een member vanaf dag 1.

---

## Wat For Members is

Een private editorial atlas voor stille luxe. Hotels, restaurants, winkels, musea, design — wereldwijd, met dichtheid-gedreven stadspagina's, AI-gegenereerde context, en personalisatie voor ingelogde members. Onderdeel van Studio PB.NL.

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

## Bouwvolgorde

We bouwen statisch klikbaar — geen backend, geen build-step, gewoon HTML met Tailwind CDN zoals de bestaande pagina's. Volgorde:

1. **City-pagina** (bv. Lyon) — bevat alle nieuwe concepten: breadcrumb, hero, secties per categorie, kaart, "in de buurt", member-toggles. Begin hier.
2. **Entry-detail** (één hotel of restaurant binnen Lyon) — verdieping van City.
3. **Country-pagina** (Frankrijk, met Lyon als sub) — vereenvoudiging van City.
4. **My Atlas** — ingelogd dashboard met saved entries op kaart, lists, notes.
5. **Home** — als laatste, etalage van 1–4.
6. **Route-pagina** (Route du Soleil) — losse cross-cutter.
7. **Auth** (sign-up, login, settings) — minimaal, statisch.

Niet zes pagina's tegelijk half-af. Eerst City *écht* af, dan pas door.

---

## Bestaande HTMLs — ALTIJD eerst lezen

De repo bevat al:

- `index.html` — home (oude versie, plat design, géén concepten toegepast)
- `shop.html`, `sanctuary.html`, `sanctuary-aman-tokyo.html`, `ontdek.html` — stijlreferentie

**REGEL — niet onderhandelbaar:** voor je *één regel code schrijft* voor een nieuwe pagina, open je eerst `index.html` en `sanctuary.html`. Bestudeer header, footer, navigatie, card-patronen, breakpoints, manier waarop overlines + serif-koppen + meta-labels gestapeld worden. Die structuur hergebruiken — niet vanaf nul beginnen, niet "moderner" maken.

**Bij twijfel:** `HUISSTIJL.html` wint van de bestaande HTMLs (want die HTMLs gebruiken nog géén CSS-variabelen, géén theme-toggle, géén Fraunces, géén taupe-accent). Maar de *structuur* uit de bestaande pagina's neem je over.

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

- Naamgeving definitief: "For Members." of "PORTAL."? (Werknaam in dit doc: For Members.)
- City-promotion drempel: 5+2 categorieën — exact zo aanhouden of instelbaar maken in CMS?
- Co-curatie (partner-account met edit-rechten) — day-1 of v1.5?
- Public lists ("Birgit's Tokyo edit") — vrij publiceren of redactionele review?
- Theme-toggle in nav zichtbaar of alleen in account-settings?

Bij elk van deze: vraag terug aan Bart voor je iets bouwt dat ervan uitgaat.

---

## Hoe te starten in een nieuwe sessie

1. Lees dit bestand (gebeurt automatisch).
2. Lees `HUISSTIJL.html` en `CONCEPT_DATAMODEL.md` als context nog ontbreekt.
3. Kijk welke pagina aan de beurt is volgens de bouwvolgorde hierboven.
4. Bouw één pagina af tot statisch klikbaar — geen halve dingen achterlaten.
5. Toon resultaat, vraag review voordat je doorgaat.

---

*Werkdocument · v0.1 · 13 mei 2026 · Studio PB.NL*
