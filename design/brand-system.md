# Radar Brand System

The visual system that runs across every Radar surface — the platform, the Decode landing page, internal documents, and outbound assets. Documented here because the system itself is the artifact: typography is the hero, the absence of SaaS chrome is intentional, and the constraints listed below are what produce a consistent operator-brief feel across pages built at different times by different methods.

## Posture

Radar is a brief, not a marketing site. The visual posture should read like an internal operations document — terse, dense, signal-forward. The buyer's first impression should be that someone serious built this, not that someone marketed this.

This means:

- **Typography is the hero.** Display weight comes from type, not from imagery, color blocks, or motion.
- **No stock photos.** Of any kind. People, abstract gradients, geometric backgrounds — none of it.
- **No icons or emojis.** Including, but not limited to: checkmarks next to feature lists, brand-colored Lucide/Heroicons, decorative arrows, decorative bullets. The middle dot `·` is the one allowed separator.
- **No badges, no chips, no tags.** No "NEW" labels, no rounded pill containers, no progress chips.
- **No animation that isn't functional.** Hover state brightening on a CTA is functional. Scroll-triggered fade-ins are not.

The negative space — what's deliberately absent — is as load-bearing as what's present.

## Color

Four colors. Period.

```
--navy:  #0D1B2A   background
--gold:  #C9A84C   accent, primary CTA, eyebrow callouts
--white: #FFFFFF   display type (H1s)
--cream: #E8E3D6   body type, italic emphasis, lower-priority text
```

There is no secondary palette, no semantic color (red for errors, green for success), no muted variations. Everything renders against navy. White carries display weight. Cream carries body weight. Gold is a single attention layer used sparingly: the primary CTA fill, accent rules, eyebrows, the 3px left border on stat blocks.

Accessibility: every text color hits WCAG AA against the navy background. Cream-on-navy is the lowest contrast pair at ~12:1 — well above the 4.5:1 floor. Gold-on-navy is ~7:1.

## Typography

Three families. No fallback to anything outside this set except the system fallback chain.

```
Bebas Neue              display, H1s, CTAs, eyebrow numerics
Cormorant Garamond      body copy, italic emphasis, deliverable text
DM Mono                 eyebrows, mono labels, footers, version slugs
```

**Bebas Neue** carries display weight. It's used at three scales: ~64px for H1s on desktop / ~48px on mobile, ~36px for section sub-heads (e.g. "MARQUISE JONES" in the Written By section), and ~18px on primary CTAs with letter-spacing. It is never used for body copy. It is never used at small sizes without significant letter-spacing.

**Cormorant Garamond** carries body weight. Italic for emphasis (subheads, the deliverable description, the "written by me, not a quiz" callouts). Roman for paragraph copy. Never used for buttons, never used in all-caps.

**DM Mono** carries metadata. Eyebrows ("OPERATOR BRIEF · $47"), section labels ("WHAT YOU GET", "HOW IT WORKS"), footers ("© RADAR 2026"), version slugs ("DECODE / 001"). Always uppercase, always letter-spaced 2-3px, always at small sizes (10-13px).

The combination is intentional: a structural display face, a humanist serif for narrative, a precise mono for instrumentation. No sans-serif body face, by design — most platforms reach for one and Radar deliberately doesn't.

## Components

### V39.1 stat block

The block treatment used for "WHAT YOU GET" lists, the HOW IT WORKS steps, and any callout where structured items need to read with operational weight rather than as a UI list.

```
3px gold left border
20px left padding
navy background
Cormorant Garamond white 18px text
22px bottom margin between blocks
```

No bullets, no numbers (except where numbered explicitly via a small DM Mono prefix above the line), no chevrons, no leading icons. The gold border is the only visual marker. The block is unstyled aside from typography — no card, no background variation, no rounded corners.

This treatment is named V39.1 and is referenced in code as `.stat-block`. The numbered variant adding a small DM Mono `01/02/03` prefix above each line is `.step-block`.

### Brand bar

A two-element flex row at the top of every page, pinned above a 1px gold-at-25%-opacity divider:

```
left:   "WORKFORCE RADAR"   DM Mono gold 13px ls 3px
right:  "DECODE / 001"      DM Mono gold 10px ls 1.5px
```

The right slot is page-specific — version slugs, document IDs, section markers. The left slot is constant. The size relationship (primary mark larger than the secondary slug) is mandatory; reversing it inverts the hierarchy and reads as wrong.

### Primary CTA

```
gold background, navy text
Bebas Neue 18px ls 2px uppercase
16px vertical / 28px horizontal padding
min 48px tap target
hover: filter brightness(1.08)
active: 1px translateY
```

The CTA brightens on hover, never darkens. Darkening gold on a navy background reads as "deactivated" — exactly the wrong signal for a button the buyer should commit to. Brightening reads as active and ready.

The button is always one of two specific copy patterns: the action-plus-price form (`DECODE MY PROFILE · $47`) or the pure-action form (`SUBMIT FOR DECODE`). Never the marketing-y forms ("Get Started", "Learn More", "Continue").

### Reassurance line

A small DM Mono gold 10px ls 2px line that sits directly under each primary CTA, listing the operational reality of what the buyer is committing to:

```
48-HOUR TURNAROUND  ·  PDF DELIVERY  ·  ONE PAGE
ENCRYPTED SUBMISSION  ·  YOUR FILE IS PRIVATE
```

The middle dot `·` is the separator, with two non-breaking spaces on each side. Em-dashes are not used as separators in this system; they read as marketing-y and the dot reads as procedural.

## Surfaces using this system

- `workforceradar.com` — main platform, all four portals
- `radar-decode.netlify.app` — the Decode standalone product
- Outbound documents (Decode PDFs, internal briefs, frameworks delivered to Full System buyers)
- Profile and repo READMEs

Any new surface inherits the system without modification. Variations are not allowed without an explicit version bump (V39.1 → V40 → V41) documented in the build log.

## What this system rejects

- The "modern SaaS landing page" template (gradient backgrounds, illustrated heroes, three-column feature grids with icons)
- Cardboard-cutout testimonials with photos and starbursts
- Pricing pages with feature comparison tables full of green checkmarks
- Toast notifications for non-error events
- Skeleton loaders styled to match the brand
- "Powered by" footer logos
- Cookie consent modals styled as trust signals
- Any visual treatment that signals "this is a SaaS product" rather than "this is operational intelligence"

The rejection is the brand. A buyer who lands on a Radar page and notices what isn't there — that's the read.
