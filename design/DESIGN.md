---
name: Clinical Precision
colors:
  surface: '#f8f9ff'
  surface-dim: '#cbdbf5'
  surface-bright: '#f8f9ff'
  surface-container-lowest: '#ffffff'
  surface-container-low: '#eff4ff'
  surface-container: '#e5eeff'
  surface-container-high: '#dce9ff'
  surface-container-highest: '#d3e4fe'
  on-surface: '#0b1c30'
  on-surface-variant: '#46464f'
  inverse-surface: '#213145'
  inverse-on-surface: '#eaf1ff'
  outline: '#767680'
  outline-variant: '#c6c5d0'
  surface-tint: '#525b90'
  primary: '#141e50'
  on-primary: '#ffffff'
  primary-container: '#2b3467'
  on-primary-container: '#959ed8'
  inverse-primary: '#bac3ff'
  secondary: '#476271'
  on-secondary: '#ffffff'
  secondary-container: '#c9e7f9'
  on-secondary-container: '#4c6878'
  tertiary: '#4e0012'
  on-tertiary: '#ffffff'
  tertiary-container: '#760020'
  on-tertiary-container: '#ff7684'
  error: '#ba1a1a'
  on-error: '#ffffff'
  error-container: '#ffdad6'
  on-error-container: '#93000a'
  primary-fixed: '#dee0ff'
  primary-fixed-dim: '#bac3ff'
  on-primary-fixed: '#0c1649'
  on-primary-fixed-variant: '#3a4377'
  secondary-fixed: '#c9e7f9'
  secondary-fixed-dim: '#aecbdc'
  on-secondary-fixed: '#001e2b'
  on-secondary-fixed-variant: '#2f4a59'
  tertiary-fixed: '#ffdadb'
  tertiary-fixed-dim: '#ffb2b7'
  on-tertiary-fixed: '#40000d'
  on-tertiary-fixed-variant: '#92002a'
  background: '#f8f9ff'
  on-background: '#0b1c30'
  surface-variant: '#d3e4fe'
  surface-ivory: '#FCFFE7'
  clinical-red: '#EB455F'
  navy-deep: '#2B3467'
  powder-blue: '#BAD7E9'
  border-subtle: '#E2E8F0'
  surface-subtle: '#F8FAFC'
typography:
  display:
    fontFamily: Plus Jakarta Sans
    fontSize: 40px
    fontWeight: '700'
    lineHeight: 48px
    letterSpacing: -0.02em
  display-mobile:
    fontFamily: Plus Jakarta Sans
    fontSize: 32px
    fontWeight: '700'
    lineHeight: 40px
    letterSpacing: -0.015em
  headline-lg:
    fontFamily: Plus Jakarta Sans
    fontSize: 30px
    fontWeight: '600'
    lineHeight: 38px
    letterSpacing: -0.015em
  headline-lg-mobile:
    fontFamily: Plus Jakarta Sans
    fontSize: 24px
    fontWeight: '600'
    lineHeight: 32px
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Plus Jakarta Sans
    fontSize: 22px
    fontWeight: '600'
    lineHeight: 28px
    letterSpacing: -0.01em
  headline-sm:
    fontFamily: Plus Jakarta Sans
    fontSize: 18px
    fontWeight: '600'
    lineHeight: 24px
  body-lg:
    fontFamily: Inter
    fontSize: 16px
    fontWeight: '400'
    lineHeight: 26px
  body-md:
    fontFamily: Inter
    fontSize: 14px
    fontWeight: '400'
    lineHeight: 22px
  body-sm:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '400'
    lineHeight: 18px
  label-md:
    fontFamily: Inter
    fontSize: 13px
    fontWeight: '500'
    lineHeight: 18px
  label-sm:
    fontFamily: Inter
    fontSize: 11px
    fontWeight: '500'
    lineHeight: 16px
  code-sm:
    fontFamily: JetBrains Mono
    fontSize: 11px
    fontWeight: '500'
    lineHeight: 16px
    letterSpacing: 0.02em
rounded:
  sm: 0.125rem
  DEFAULT: 0.25rem
  md: 0.375rem
  lg: 0.5rem
  xl: 0.75rem
  full: 9999px
spacing:
  gutter: 1.5rem
  gutter-sm: 1rem
  gutter-lg: 2rem
  margin: 1.5rem
  margin-sm: 1rem
  margin-lg: 3rem
  space-xs: 0.25rem
  space-sm: 0.5rem
  space-md: 1rem
  space-lg: 1.5rem
  space-xl: 2.5rem
---

## Brand & Style

This design system is tailored for a professional medical supplies and healthcare equipment platform. The brand identity fuses clinical rigor and technical reliability with accessible e-commerce clarity. The interface speaks to hospital procurement officers, clinical specialists, private practice doctors, and patients seeking certified home-care supplies.

The aesthetic follows modern clinical minimalism inspired by the structural restraint and micro-precision of shadcn/ui. The design avoids unnecessary decorative flourishes, leaning instead into meticulous layout boundaries, hairline dividers, confident type contrast, and surgical color coding. The visual atmosphere must evoke immediate trust, diagnostic precision, hygienic clarity, and rapid legibility during critical decision-making.

## Colors

The palette establishes an immediate clinical hierarchy rooted in authority, hygiene, and emergency responsiveness:

- **Primary (`#2B3467` - Deep Navy/Indigo):** Anchors brand authority, high-order navigation, regulatory certification badges, major headers, and primary transactional actions. Evokes medical professionalism and institutional stability.
- **Secondary (`#BAD7E9` - Powder Sky Blue):** Functions as a hygienic, calming accent for soft backgrounds, active badge containers, hover states, contextual medical banners, and progress indicators.
- **Tertiary (`#EB455F` - Clinical Coral/Red-Pink):** Reserved strictly for high-urgency interactions, emergency supply tags, alert banners, critical stock depletion warnings, and selected high-conversion callout triggers.
- **Neutral (`#64748B` - Slate):** Powers structural borders, muted metadata, technical spec labels, and secondary UI scaffolding without visual fatigue.
- **Named Surfaces:** `#FCFFE7` acts as a warm ivory accent for high-priority notification callouts, clinical advisory panels, and specialized product certificates, while `#F8FAFC` and `#FFFFFF` govern primary clinical surfaces.

## Typography

The typography hierarchy pairs contemporary geometric clarity with technical precision:

- **Headlines (Plus Jakarta Sans):** Balances humanist warmth with clean structural clarity. It keeps large-format medical categorizations, product names, and institutional headers approachable yet authoritative.
- **Body Text (Inter):** Deploys high x-height and neutral clarity to guarantee maximum legibility in complex technical specifications, clinical indications, dosing warnings, and regulatory notices.
- **Technical & Metric Labels (JetBrains Mono):** Applied deliberately to SKU numbers, batch IDs, ISO certifications, expiry timestamps, and dimensional specifications, reinforcing the feeling of precision engineering.

## Layout & Spacing

The layout is built on a responsive 12-column grid system paired with strict 8pt rhythm multiples (4px base units for tight component internals):

- **Desktop (1200px+):** 12 columns, `margin-lg` (3rem) outer boundary, `gutter-lg` (2rem) between columns. Product catalogs support multi-facet filter sidebars alongside 3-to-4 item catalog grids.
- **Tablet (768px - 1199px):** 8 columns, `margin` (1.5rem) canvas margins, `gutter` (1.5rem) grid spacing. Catalogs condense to 2-column or structured row layouts.
- **Mobile (<768px):** 4 columns, `margin-sm` (1rem) edge boundaries, `gutter-sm` (1rem) spacing. Complex tabular specs convert into vertical key-value definition cards.
- **Rhythm & Padding:** `space-xs` and `space-sm` govern internal component packing (button padding, input margins, chip gaps). `space-md` and `space-lg` structure section breaks and card body padding.

## Elevation & Depth

This design system embraces shadcn/ui-inspired surface hierarchy, prioritizing crisp micro-borders and subtle tonal steps over dramatic drop shadows:

- **Base Layer:** `#FFFFFF` main canvases with `#F8FAFC` container tiers for section segregation and catalog filters.
- **Borders & Dividers:** Subtle 1px solid borders (`#E2E8F0`) define cards, table cells, and input surfaces, creating clean delineation without visual noise.
- **Interactive Depth:** Resting cards utilize flat borders without shadows. On hover, elements elevate with an ambient, diffused shadow: `0 4px 20px -2px rgba(43, 52, 103, 0.06), 0 2px 6px -1px rgba(43, 52, 103, 0.04)`.
- **Modals & Overlays:** Overlays utilize a frosted neutral backdrop (`rgba(43, 52, 103, 0.4)` with `backdrop-filter: blur(4px)`), elevating critical dialogs via `0 20px 25px -5px rgba(43, 52, 103, 0.12), 0 8px 10px -6px rgba(43, 52, 103, 0.08)`.

## Shapes

The design system employs a disciplined, subtle corner radius (`roundedness: 1`, mapping to `0.25rem` / 4px base radius, `0.5rem` / 8px for containers, and `0.75rem` / 12px for modals).

This architectural, near-sharp geometry communicates surgical precision, clinical cleanliness, and enterprise dependability, avoiding the overly playful feel of heavily rounded consumer interfaces. High-utility badges and status indicators may use full-pill styling (`rounded-full`) solely to distinguish dynamic operational status (e.g., "In Stock", "Sterile", "FDA Approved").

## Components

- **Buttons:**
  - *Primary:* Solid Deep Navy (`#2B3467`) background with white text, 1px border (`#2B3467`), 8px border-radius, subtle active depression.
  - *Destructive / Urgent:* Vibrant Coral (`#EB455F`) background with white text for emergency orders, order cancellations, or hazardous warnings.
  - *Secondary / Outline:* Pure white background, `#E2E8F0` border, `#2B3467` text; hovers transition to Powder Blue tint (`#BAD7E9` at 20% opacity).
  - *Ghost:* Transparent surface, text in `#2B3467`, hovers activate `#F8FAFC`.

- **Badges & Chips:**
  - *Certification Badge:* `#F8FAFC` background, 1px border in `#BAD7E9`, text in `#2B3467`, accompanied by JetBrains Mono cert IDs.
  - *Urgent / Biohazard / Out of Stock Badge:* Soft coral tint background (`rgba(235, 69, 95, 0.1)`), text in `#EB455F`, 1px border in `rgba(235, 69, 95, 0.3)`.
  - *Advisory / Sterile Badge:* Warm ivory surface (`#FCFFE7`), text in `#2B3467`, 1px border in `#BAD7E9`.

- **Input Fields & Form Controls:**
  - Background `#FFFFFF`, 1px border in `#CBD5E1`, 6px border-radius.
  - Focus state features a crisp 2px focus ring in `#2B3467` with an offset of 2px.
  - Invalid states use a 1px border and ring in `#EB455F` with clear assistive error text in `Inter` 12px.

- **Checkboxes & Radios:**
  - Crisp 16px square/circle with 1px border (`#CBD5E1`). Checked state fills with `#2B3467` and white check icon. Focus exhibits standard keyboard-accessible outline.

- **Product & Equipment Cards:**
  - Structured on a 1px `#E2E8F0` border frame, pure white surface, 8px corner radius.
  - Includes a dedicated upper metadata zone (SKU, brand, regulatory standard), crisp 1:1 image canvas on `#F8FAFC`, mid-tier specification matrix, and bottom price/action block.

- **Medical Alert & Notice Banners:**
  - Full-width or inline rounded callouts framed with a 1px border.
  - Informational panels leverage `#BAD7E9` at 15% opacity with Deep Navy body.
  - Critical recall or safety notifications use `#EB455F` 10% fill with `#EB455F` border and deep charcoal text.