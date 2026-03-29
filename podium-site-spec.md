# Podium — podium.build
## Cursor Spec · v1

> **Instruction to agent:** This is a greenfield marketing site. Do not reference, import from, or modify any existing Podium repos. Read this spec in full before writing a single line. Complete phases in strict order. All copy is final unless marked `[COPY TBD]`. All font files will be provided directly in Cursor — do not attempt to import from Google Fonts or CDN.

---

## 0. Overview

**Purpose:** Standalone marketing site at podium.build  
**Stack:** React 18 + TypeScript + Vite 6 + Tailwind CSS v4  
**Single page:** One scrollable page, no internal routing  
**Animations:** Framer Motion (scroll-triggered fade/slide in only — no scrolljacking)  
**Fonts:** Display serif (to be provided) + Suisse Int'l or PP Neue Montreal (to be provided)  
**No external UI libraries**

### File structure

```
src/
  index.css              ← design tokens, font-face declarations
  main.tsx               ← Vite entry
  App.tsx                ← single page shell
  components/
    Nav.tsx
    sections/
      Hero.tsx           ← Section 01
      Idea.tsx           ← Section 02
      Platform.tsx       ← Section 03
      Intelligence.tsx   ← Section 04
      Built.tsx          ← Section 05
      Developers.tsx     ← Section 06
      Close.tsx          ← Section 07
    Footer.tsx
  hooks/
    useInView.ts         ← lightweight scroll observer hook
```

---

## 1. Phase 0 — Design Tokens

**File:** `src/index.css`

```css
@font-face {
  font-family: 'DisplaySerif';
  src: url('/fonts/display-serif.woff2') format('woff2');
  font-weight: 400 700;
  font-style: normal italic;
}

@font-face {
  font-family: 'SansBody';
  src: url('/fonts/sans-body.woff2') format('woff2');
  font-weight: 300 500;
  font-style: normal;
}

:root {
  /* Color */
  --color-ground:     #F5F2ED;   /* warm off-white, primary bg */
  --color-ink:        #0F0E0C;   /* near-black, primary type */
  --color-ink-muted:  #6B6760;   /* secondary type, labels */
  --color-ink-ghost:  #C8C4BE;   /* ghost type, decorative */
  --color-cobalt:     #1A35E8;   /* signature accent, use sparingly */
  --color-dark:       #0D1117;   /* dark section ground */
  --color-dark-card:  #161B22;   /* dark card surface */

  /* Typography scale */
  --font-display:     'DisplaySerif', Georgia, serif;
  --font-body:        'SansBody', system-ui, sans-serif;

  --text-hero:        clamp(3.5rem, 7vw, 7rem);
  --text-display:     clamp(2.5rem, 5vw, 5rem);
  --text-list-lg:     clamp(2rem, 3.5vw, 3.5rem);
  --text-list-sm:     clamp(1.25rem, 2vw, 2rem);
  --text-body:        clamp(1rem, 1.2vw, 1.2rem);
  --text-label:       0.75rem;
  --text-code:        0.875rem;

  /* Spacing */
  --section-pad-y:    clamp(6rem, 10vw, 12rem);
  --section-pad-x:    clamp(1.5rem, 6vw, 8rem);
  --max-w:            1440px;

  /* Motion */
  --ease-out:         cubic-bezier(0.16, 1, 0.3, 1);
  --duration-enter:   0.7s;
}

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

html { background: var(--color-ground); color: var(--color-ink); }

body {
  font-family: var(--font-body);
  font-size: var(--text-body);
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}
```

---

## 2. Phase 1 — useInView Hook

**File:** `src/hooks/useInView.ts`

Lightweight IntersectionObserver hook. Returns `inView: boolean` once element has entered viewport. Does not toggle back off.

```typescript
import { useEffect, useRef, useState } from 'react'

export function useInView(threshold = 0.15) {
  const ref = useRef<HTMLElement>(null)
  const [inView, setInView] = useState(false)

  useEffect(() => {
    const el = ref.current
    if (!el) return
    const obs = new IntersectionObserver(
      ([entry]) => { if (entry.isIntersecting) { setInView(true); obs.disconnect() } },
      { threshold }
    )
    obs.observe(el)
    return () => obs.disconnect()
  }, [threshold])

  return { ref, inView }
}
```

---

## 3. Phase 2 — Nav

**File:** `src/components/Nav.tsx`

Fixed top. Full width. Transparent background with a subtle blur on scroll (add `backdrop-filter: blur(12px)` + `background: rgba(245,242,237,0.85)` once scrollY > 40).

Left: Podium wordmark in `var(--font-body)` weight 500, `var(--color-ink)`. No logo image — wordmark only for now.  
Right: Single CTA — "Get API Key" — in `var(--font-body)`, small, `var(--color-cobalt)`, no background, underline on hover.

Height: 56px. z-index: 100.

```tsx
// Nav is minimal — wordmark + single text CTA
// Scroll listener adds backdrop blur after 40px
// No hamburger menu, no nav links
```

---

## 4. Phase 3 — Section 01: Hero

**File:** `src/components/sections/Hero.tsx`

**Layout:** Full viewport height (`100svh`). Centered content vertically. Horizontal padding from `--section-pad-x`. Max width `--max-w`, centered.

**Background:** Subtle atmospheric gradient wash — not a photograph, not a mesh. A soft radial bloom of warm amber/cream tones, very low opacity, that gives the ground just enough temperature to feel alive. Implemented as a CSS radial gradient layered behind the type. Does not move or animate.

**Headline:** `var(--font-display)`, `var(--text-hero)`, `var(--color-ink)`, line-height 1.05. Two lines. The word or phrase in cobalt is wrapped in a `<span style="color: var(--color-cobalt)">`. Italic on the cobalt word only.

**Copy:**
```
Commerce infrastructure
for the *agentic* economy.
```
*(cobalt + italic on "agentic")*

**Subline:** Below the headline, 3rem gap. `var(--font-body)`, `var(--text-body)`, `var(--color-ink-muted)`. Max width 480px.

```
The building blocks to create personal, proactive agents and
applications that earn trust, act on a user's behalf,
and prove what they did.
```

**CTA:** Below subline, 2rem gap. Two elements inline:
- Primary: "Get API Key →" — `var(--color-ink)`, `var(--font-body)` weight 500, no background, bottom border 1px `var(--color-ink)`, padding-bottom 2px. Hover: border-color transitions to `var(--color-cobalt)`, color transitions to `var(--color-cobalt)`.
- Secondary: "Read the docs" — `var(--color-ink-muted)`, no decoration. Hover: `var(--color-ink)`.
- Gap between: 2rem.

**Entrance animation:** Headline fades up from 20px offset, 0.7s, `var(--ease-out)`. Subline and CTA follow with 120ms stagger. Plays once on mount — no scroll trigger needed.

---

## 5. Phase 4 — Section 02: The Idea

**File:** `src/components/sections/Idea.tsx`

**Layout:** `var(--section-pad-y)` top/bottom. `var(--section-pad-x)` horizontal. Max width `--max-w`. Content max-width: 720px, left-aligned (not centered).

**Ground:** `var(--color-ground)` — continuous from hero.

**Label:** Small all-caps, `var(--text-label)`, `var(--color-ink-muted)`, `var(--font-body)` weight 500, letter-spacing 0.12em. Text: `"THE IDEA"`. Margin-bottom 2.5rem.

**Body copy:** `var(--font-body)`, `var(--text-body)` scaled up slightly — 1.25rem / 1.8 line-height. `var(--color-ink)`. Two paragraphs, 1.5rem gap between.

```
The agentic economy is not a feature. It's a shift in how
software relates to people — from tools that wait to be used,
to agents that understand, act, and earn trust over time.

Podium is the infrastructure layer for that shift. A composable
set of primitives — intelligence, commerce, memory, settlement —
that developers combine into agents and applications with a
depth of understanding that compounds with every interaction.
Not just on one surface. Everywhere a user encounters something
built on Podium, the understanding carries forward.
```

**Entrance:** `useInView` hook. Section fades up 24px, 0.7s, `var(--ease-out)` when entering viewport.

---

## 6. Phase 5 — Section 03: The Platform

**File:** `src/components/sections/Platform.tsx`

**Layout:** `var(--section-pad-y)` top/bottom. `var(--section-pad-x)` horizontal. Max width `--max-w`.

**Label:** Same treatment as Section 02. Text: `"THE PLATFORM"`.

**Intro line:** `var(--font-display)`, `clamp(1.75rem, 3vw, 2.5rem)`, `var(--color-ink)`, max-width 600px, margin-bottom 4rem.

```
Eleven primitives. Endlessly composed.
```

**Cascading list:** The eleven Podium primitives rendered as a stacked typographic list. Inspired by image 2 (Immothep). Each item is one line. Type size and opacity fade from top to bottom — the first four are full weight and full opacity, the middle three are medium weight and ~70% opacity, the final four are lighter weight and ~40% opacity. Visual metaphor: there is more depth here than you can hold at once.

Each list item has:
- Left: primitive name in `var(--font-display)`, size stepping from `var(--text-list-lg)` down to `var(--text-list-sm)`
- Right: a one-phrase descriptor in `var(--font-body)`, `var(--text-label)`, `var(--color-ink-muted)`, vertically aligned to baseline of the name

```
Companion            Personal agent profile and preference state
Task Pool            On-chain USDC bounties, escrow, settlement
Enrichment Pipeline  Continuous multi-source product intelligence
Campaign Engine      Intent capture — swipe, vote, survey, UGC
For-You Feed         Personalized product and creator discovery
Points Ledger        Programmable earn and spend
Rewards              On-chain minting, six reward types
Commerce             Full transaction pipeline, Stripe and x402
Agentic Feed         Agent-native product discovery, no auth required
Search               Product, creator, and campaign discovery
x402 Rails           Machine-native USDC payment over HTTP
```

**Opacity + size mapping:**
- Items 1–4: opacity 1.0, size `var(--text-list-lg)`
- Items 5–7: opacity 0.65, size interpolated
- Items 8–11: opacity 0.35, size `var(--text-list-sm)`

**Entrance animation:** Items stagger in sequentially, 60ms between each, fading up 16px. Triggered by `useInView` on the section.

---

## 7. Phase 6 — Section 04: Intelligence

**File:** `src/components/sections/Intelligence.tsx`

**Layout:** Dark section. Ground: `var(--color-dark)`. Full width. `var(--section-pad-y)` top/bottom. `var(--section-pad-x)` horizontal. Max width `--max-w`.

**Ghost type:** Large display serif text — `var(--font-display)`, `clamp(4rem, 8vw, 9rem)`, color `rgba(255,255,255,0.04)`, absolutely positioned, filling the section background. Text (two lines, centered):

```
Understands.
Remembers.
```

**Floating card:** Positioned over the ghost type, slightly left-of-center, mild rotation (-1.5deg). Dark card surface `var(--color-dark-card)`, border `1px solid rgba(255,255,255,0.08)`, border-radius 4px, padding 2.5rem, max-width 480px. Contains:

- Small label: `var(--text-label)`, `var(--color-cobalt)`, weight 500, letter-spacing 0.1em. Text: `"INTENT PROVENANCE"`
- Code block: monospace, `var(--text-code)`, `rgba(255,255,255,0.75)`, line-height 1.6. Renders the provenance JSON snippet:

```json
{
  "intentScore": 0.84,
  "intentProvenance": {
    "USER_DECLARED": {
      "voteCount": 47,
      "weight": 0.82
    },
    "MARKET_DERIVED": {
      "signalCount": 312,
      "weight": 0.18
    }
  }
}
```

- Below the code block, 1.5rem gap: a single line in `var(--font-body)`, small, `rgba(255,255,255,0.4)`. Text: `"Every recommendation, sourced and scored."`

**Statement:** To the right of or below the card (responsive), in `var(--font-display)`, `clamp(1.5rem, 2.5vw, 2.25rem)`, `rgba(255,255,255,0.9)`, max-width 420px:

```
The same understanding,
on every surface
a user encounters.
```

Below statement, 1rem gap: `var(--font-body)`, `var(--text-body)`, `rgba(255,255,255,0.5)`, max-width 380px:

```
Intent profiles, interaction history, conversation memory —
persisted, synthesized, and available wherever someone
engages with something built on Podium.
```

**Entrance:** Card fades up and in with slight scale (0.97 → 1) on `useInView`. Statement fades in with 200ms delay.

---

## 8. Phase 7 — Section 05: What Gets Built

**File:** `src/components/sections/Built.tsx`

**Layout:** Returns to `var(--color-ground)`. `var(--section-pad-y)` top/bottom. `var(--section-pad-x)` horizontal. Max width `--max-w`.

**Label:** `"WHAT GETS BUILT"`

**Intro line:** `var(--font-display)`, `clamp(1.75rem, 3vw, 2.5rem)`:

```
The same primitives.
Radically different outputs.
```

**Experience cards:** Four cards in a 2×2 grid on desktop, single column on mobile. Each card is minimal — no images, no icons, no shadows. A thin top border in `var(--color-cobalt)` (2px), the rest of the card border in `rgba(0,0,0,0.08)`. Padding 2rem. Background `var(--color-ground)`.

Card anatomy:
- Surface tag: `var(--text-label)`, `var(--color-cobalt)`, letter-spacing 0.1em — e.g. `"WEB + TELEGRAM"`
- Name: `var(--font-display)`, `clamp(1.25rem, 2vw, 1.75rem)`, `var(--color-ink)`, margin-top 0.75rem
- Description: `var(--font-body)`, `var(--text-body)`, `var(--color-ink-muted)`, margin-top 0.5rem
- Primitives used: small, `var(--text-label)`, `var(--color-ink-ghost)`, margin-top 1.25rem. Label: `"BUILT WITH"` then comma-separated primitive names

**Four cards:**

```
Surface: WEB + TELEGRAM
Name: Beauty Companion
Desc: A personal skincare agent that builds a taste profile
      through conversation, surfaces personalized products,
      and purchases on a user's behalf.
Primitives: Companion, Enrichment Pipeline, Campaign Engine,
            Commerce, x402 Rails

Surface: WEB + NATIVE
Name: Task Bounty Board
Desc: An open marketplace where brands post USDC-backed tasks,
      solvers claim and submit proof, and settlement happens
      on-chain with transparent reputation tracking.
Primitives: Task Pool, SolverRegistry, RewardPool, Points Ledger

Surface: ANY AGENT RUNTIME
Name: Agentic Commerce Node
Desc: A Podium-powered node exposable to Claude, Cursor,
      LangChain, or any MCP-compatible runtime — giving agents
      the ability to discover products, check intent profiles,
      and execute purchases through typed tool calls.
Primitives: Agentic Feed, Companion, x402 Rails, Search

Surface: NATIVE MOBILE
Name: Brand Discovery App
Desc: A swipe-native experience where every interaction is a
      structured intent signal. Users earn rewards for engaging.
      Brands get a real-time understanding of what their audience
      actually wants.
Primitives: Campaign Engine, For-You Feed, Points Ledger,
            Rewards, Enrichment Pipeline
```

**Entrance:** Cards stagger in 80ms apart on `useInView`.

---

## 9. Phase 8 — Section 06: For Developers

**File:** `src/components/sections/Developers.tsx`

**Layout:** `var(--section-pad-y)` top/bottom. `var(--section-pad-x)` horizontal. Max width `--max-w`. Two-column on desktop: left is copy + CTAs, right is code block.

**Label:** `"FOR DEVELOPERS"`

**Headline:** `var(--font-display)`, `clamp(2rem, 3.5vw, 3rem)`, `var(--color-ink)`:

```
Start building
in minutes.
```

**Body:** `var(--font-body)`, `var(--text-body)`, `var(--color-ink-muted)`, max-width 380px, margin-top 1.5rem:

```
Podium works with the tools you already use —
Cursor, Claude Code, LangChain, Vercel AI SDK.
One API key. A TypeScript SDK. And infrastructure
that scales with the intelligence you build on it.
```

**CTAs:** Two, stacked, margin-top 2.5rem:
- Primary: "Get API Key →" — same treatment as hero CTA
- Secondary: "Read the docs" — muted, text-only

**Code block (right column):** Dark card, same surface as Intelligence section card. Monospace, `var(--text-code)`, syntax-highlighted manually (no library). Shows:

```typescript
import { createPodiumClient } from '@podiumcommerce/node-sdk'

const client = createPodiumClient({
  apiKey: process.env.PODIUM_API_KEY
})

// Build an intent profile
const profile = await client.companion.createProfile({
  userId,
  requestBody: {
    concerns: ['hydration', 'anti-aging'],
    priceRange: { min: 25, max: 100 },
    avoidances: ['fragrance', 'parabens'],
  }
})

// Get personalized recommendations
const recs = await client.companion.listRecommendations({
  userId, count: 5
})
```

Syntax coloring (manual, inline styles):
- Keywords (`import`, `from`, `const`, `await`, `new`): `var(--color-cobalt)`
- Strings: `#E06C75` (muted red)
- Numbers + booleans: `#D19A66` (amber)
- Comments: `rgba(255,255,255,0.3)`
- Default: `rgba(255,255,255,0.85)`

**Entrance:** Left column fades up, right column fades up with 150ms delay.

---

## 10. Phase 9 — Section 07: Close

**File:** `src/components/sections/Close.tsx`

**Layout:** Full width. `var(--color-ground)`. Padding top `clamp(8rem, 14vw, 16rem)`, bottom `clamp(6rem, 10vw, 10rem)`. Centered content.

**Headline:** `var(--font-display)`, `var(--text-hero)`, centered, `var(--color-ink)`. One cobalt word, italic:

```
Build something
*worth* trusting.
```
*(cobalt + italic on "worth")*

**CTA:** "Get API Key →" — same primary treatment, centered, margin-top 3rem.

**Entrance:** Fades up from 32px offset on `useInView`, 0.9s duration.

---

## 11. Phase 10 — Footer

**File:** `src/components/Footer.tsx`

Minimal. `var(--color-ground)`. Border-top `1px solid rgba(0,0,0,0.08)`. Padding 2rem `var(--section-pad-x)`. Two rows:

Row 1 — horizontal flex, space-between:
- Left: "Podium" wordmark, `var(--font-body)` weight 500, small
- Right: "Docs" and "X / Twitter" as plain text links, `var(--color-ink-muted)`, `var(--text-label)`

Row 2 — `var(--text-label)`, `var(--color-ink-ghost)`, margin-top 1rem:
- "© 2026 KIKI World Inc."

---

## 12. Phase 11 — App.tsx Shell

**File:** `src/App.tsx`

Compose all sections in order. Wrap in a single `<main>` with no overflow-x. Import and mount:

```tsx
<>
  <Nav />
  <main>
    <Hero />
    <Idea />
    <Platform />
    <Intelligence />
    <Built />
    <Developers />
    <Close />
  </main>
  <Footer />
</>
```

---

## 13. Open Questions

1. **Font files** — provide display serif and sans body woff2 files directly in Cursor before execution. Names in `@font-face` declarations above are placeholders.
2. **Primary CTA destination** — "Get API Key →" currently points to `#` placeholder. Confirm URL before execution (likely `https://podium.build/onboarding` or the developer portal).
3. **Docs URL** — confirm the path where docs will live (`podium.build/docs` or separate subdomain).
4. **X / Twitter handle** — confirm `@podaboratory` or updated handle for footer link.
5. **KIKI World Inc.** — confirm legal entity name for copyright line.
6. **Atmospheric hero treatment** — spec uses a CSS radial gradient. If a real atmospheric image (DARC-direction, licensed) becomes available, it slots in as a `background-image` layer behind the gradient. No code change needed — just an asset swap.

---

## Smoke Tests

- [ ] Nav becomes frosted at scroll > 40px, transparent at top
- [ ] Hero headline renders at full `--text-hero` size on 1440px, scales gracefully to 375px mobile
- [ ] Platform list: first 4 items full weight/opacity, items 8–11 clearly lighter — visual depth is legible
- [ ] Intelligence section: dark ground, ghost type behind floating card, card has mild rotation
- [ ] Cobalt accent appears in: hero headline, close headline, nav CTA, card borders, code keywords — and nowhere else
- [ ] No horizontal scroll at any viewport width
- [ ] All entrance animations play once and do not replay
- [ ] Footer copyright reads "© 2026 KIKI World Inc."
