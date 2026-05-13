# L'INDEX — Project Instruction for Claude Code

## Project overview

**Project name:** portal (MVP codename)  
**Site name:** PORTAL.  
**Purpose:** A curated luxury link directory — hotels, gastronomy, design, destinations. Managed by Birgit.  
**Stack:** Flat HTML + Tailwind CSS (CDN) + Iconify icons. No framework yet. Future: Next.js.  
**Status:** MVP — clickable, navigable, no backend.

---

## File structure

```
portal/
├── index.html              ← Homepage (editorial overview)
├── ontdek.html             ← Category/collection page
├── [new pages here]
├── assets/
│   └── style.css           ← Custom CSS variables + any non-Tailwind overrides
└── CLAUDE.md               ← This file
```

Keep it flat. No subdirectories for HTML pages at this stage.

---

## Design system

### Color tokens

Never use raw hex values inline in HTML. All colors are defined via Tailwind config (in a script block at the top of each page) and/or as CSS variables in assets/style.css.

**Current palette:**

| Token name   | Value     | Usage                         |
|--------------|-----------|-------------------------------|
| alabaster    | #F8F8F7   | Page background, nav text     |
| taupe        | #D1BCAF   | Accent / steun kleur — italic accent in headings ("curated guide", "& Retreats", etc.) |
| stone-900    | Tailwind  | Primary text, dark sections   |
| stone-600    | Tailwind  | Body text, descriptions       |
| stone-500    | Tailwind  | Secondary text, labels        |
| stone-400    | Tailwind  | Tertiary, placeholders        |
| stone-300    | Tailwind  | Subtle text                   |
| stone-200    | Tailwind  | Borders, dividers             |
| stone-100    | Tailwind  | Footer background             |
| stone-800    | Tailwind  | Image placeholders (dark)     |

Define the custom color in each page's Tailwind config block:

```html
<script>
  tailwind.config = {
    theme: {
      extend: {
        colors: {
          alabaster: '#F8F8F7',
        }
      }
    }
  }
</script>
```

Then use bg-alabaster instead of bg-[#F8F8F7] throughout.

---

## Typography

| Role               | Classes                                                                                   |
|--------------------|-------------------------------------------------------------------------------------------|
| Hero heading H1    | font-serif text-5xl md:text-7xl lg:text-8xl tracking-tight font-medium leading-[1.1]     |
| Section heading    | font-serif text-4xl md:text-5xl tracking-tight font-medium                               |
| Card heading       | font-serif text-2xl tracking-tight font-medium                                           |
| Small card heading | font-serif text-sm sm:text-base md:text-2xl tracking-tight font-medium                  |
| Body text          | text-base md:text-lg text-stone-600 font-light leading-relaxed                           |
| Label / overline   | text-xs font-semibold uppercase tracking-widest text-stone-400                           |
| Nav items          | text-sm font-medium tracking-wide uppercase                                               |
| Logo               | font-sans font-semibold tracking-tighter text-lg uppercase                               |

Italic accent in headings: use a span with text-taupe italic font-normal

---

## Shared HTML patterns

### Head block (use on every page)

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Page Title] | L'INDEX.</title>
  <script>
    tailwind.config = {
      theme: {
        extend: {
          colors: {
            alabaster: '#F8F8F7',
          }
        }
      }
    }
  </script>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://code.iconify.design/iconify-icon/1.0.7/iconify-icon.min.js"></script>
  <link rel="stylesheet" href="assets/style.css">
</head>
```

### Body opening

```html
<body class="bg-alabaster text-stone-900 font-sans antialiased selection:bg-stone-200 overflow-x-hidden">
```

### Navigation

Copy the nav + fullscreen menu overlay from ontdek.html exactly. It has the correct structure: hamburger + logo on the left, nav links on the right, mix-blend-difference for the invert effect, and a working open/close script at the bottom. Reuse unchanged on all pages. Update active link state as needed.

### Hero section pattern

```html
<header class="min-h-[70vh] flex flex-col justify-end p-6 lg:p-10 pb-16 lg:pb-24 max-w-[1600px] mx-auto">
  <div class="grid grid-cols-1 lg:grid-cols-12 gap-12 lg:gap-6 items-end">
    <div class="lg:col-span-8">
      <h1 class="font-serif text-5xl md:text-7xl lg:text-8xl tracking-tight font-medium leading-[1.1] text-stone-900">
        [Heading]
      </h1>
    </div>
    <div class="lg:col-span-4 flex flex-col gap-8">
      <p class="text-base md:text-lg text-stone-600 font-light leading-relaxed">[Intro text]</p>
    </div>
  </div>
</header>
```

### Section wrapper

```html
<section class="border-t border-stone-200">
  <div class="max-w-[1600px] mx-auto p-6 lg:p-10 py-24 lg:py-40">
    <!-- content -->
  </div>
</section>
```

### Link card (grid item)

```html
<a href="[url]" target="_blank" rel="noopener noreferrer" class="group flex flex-col gap-3 md:gap-5">
  <div class="relative w-full aspect-square overflow-hidden bg-stone-100">
    <img src="[url]" alt="[name]" class="object-cover w-full h-full group-hover:scale-105 transition-transform duration-700 ease-in-out">
  </div>
  <div class="flex flex-col gap-1 md:gap-1.5">
    <div class="flex justify-between items-start">
      <h3 class="font-serif text-sm sm:text-base md:text-2xl tracking-tight font-medium text-stone-900 pr-2">[Name]</h3>
      <iconify-icon icon="solar:arrow-right-up-linear" stroke-width="1.5" class="text-sm md:text-xl text-stone-400 group-hover:text-stone-900 group-hover:translate-x-1 group-hover:-translate-y-1 transition-all flex-shrink-0"></iconify-icon>
    </div>
    <span class="text-[10px] md:text-xs font-semibold uppercase tracking-widest text-stone-400">[Location or subtitle]</span>
  </div>
</a>
```

### Category section header

```html
<div class="flex flex-col md:flex-row justify-between items-start md:items-end mb-12 lg:mb-20 gap-8">
  <div>
    <div class="text-xs font-semibold uppercase tracking-widest text-stone-400 mb-3">[Category label]</div>
    <h2 class="font-serif text-4xl md:text-5xl tracking-tight font-medium">[Section title]</h2>
  </div>
  <a href="[url]" class="text-sm font-medium text-stone-400 hover:text-stone-900 transition-colors">Alles bekijken</a>
</div>
```

### Footer

Copy footer from start.html. Keep unchanged across all pages.

---

## Grid conventions

| Use case          | Grid classes                                      |
|-------------------|---------------------------------------------------|
| 4-column cards    | grid grid-cols-2 md:grid-cols-4 gap-6 lg:gap-10  |
| Asymmetric hero   | grid grid-cols-1 md:grid-cols-12 gap-6 lg:gap-10 |
| 2-column category | grid grid-cols-1 md:grid-cols-2 gap-4 lg:gap-8   |
| Content + sidebar | grid grid-cols-1 lg:grid-cols-12 (8+4 split)     |

Max content width: always max-w-[1600px] mx-auto.
Page padding: always p-6 lg:p-10.

---

## Icons

Iconify Solar line icons only. Always stroke-width="1.5".

| Usage          | Icon                                   |
|----------------|----------------------------------------|
| Arrow right    | solar:arrow-right-linear               |
| Arrow right-up | solar:arrow-right-up-linear            |
| Arrow down     | solar:arrow-down-linear                |
| Hamburger      | solar:hamburger-menu-linear            |
| Close          | solar:close-linear                     |

---

## Rules

1. No hardcoded hex values in HTML. Use Tailwind tokens (alabaster, stone-*).
2. No inline styles. Everything via Tailwind classes.
3. No extra dependencies. Only Tailwind CDN and Iconify CDN.
4. Keep JavaScript minimal. Only what's needed (menu toggle). No frameworks.
5. All external links open in target="_blank" rel="noopener noreferrer".
6. Images: use object-cover with aspect-square or aspect-[4/3]. Always include alt text.
7. Section vertical padding: py-24 lg:py-40. Don't deviate without reason.
8. Consistency over creativity. New pages follow existing patterns exactly. Introduce new patterns only when the content genuinely requires it.

---

## Pages in this MVP

| File              | Status    | Description                                         |
|-------------------|-----------|-----------------------------------------------------|
| index.html        | build     | Homepage: hero + editorial grid + category cards    |
| ontdek.html       | existing  | All categories with link grids                      |
| sanctuary.html    | build     | Single category page (Hotels & Retreats)            |
| [category].html   | later     | Other single category pages                         |

---

## What this MVP needs to demonstrate

- Clear visual identity: stille luxe, editorial feel, serif/sans contrast
- Working navigation between all pages
- Clickable external links in each category
- Content simple enough that Birgit can edit HTML directly

---

## What to ignore for now

- CMS or database
- Next.js migration
- Authentication
- Search or filter functionality
- Newsletter backend
- Performance optimization
