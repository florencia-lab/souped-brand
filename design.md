# Souped Onboarding — Design System

Design reference for the question-game format. Covers tokens, components, layout rules, and cascade gotchas. Use this when adding new steps or building sibling game flows (app-game, brand-game, etc.).

---

## 1. Colour Tokens

All colours live in `@layer tokens` as CSS custom properties on `:root`. Never use raw hex or rgba inside step HTML — always use a token or a Tailwind class that maps to one.

### Light (default)

| Token | Value | Role |
|---|---|---|
| `--bg` | `#fff8f0` | Page background (warm cream) |
| `--bg-mid` | `#f3ebe0` | Muted surface |
| `--bg-card` | `#f3ebe1` | Card background |
| `--ink` | `#080025` | Primary text |
| `--ink-70` | `rgba(8,0,37,.72)` | Strong secondary text |
| `--ink-60` | `rgba(8,0,37,.6)` | Secondary text |
| `--ink-50` | `rgba(8,0,37,.5)` | Placeholder / label |
| `--ink-30` | `rgba(8,0,37,.3)` | Muted text / borders |
| `--ink-15` | `rgba(8,0,37,.15)` | Card borders, dividers |
| `--ink-08` | `rgba(8,0,37,.08)` | Subtle fills |
| `--ink-05` | `rgba(8,0,37,.05)` | Faintest tint |
| `--orange` | `#ff6b35` | Brand accent, CTAs |
| `--orange-dk` | `#e03328` | Orange hover |
| `--orange-soft` | `rgba(255,107,53,.12)` | Orange tint background |
| `--amber` | `#ea580c` | Pill / warning text |
| `--violet` | `#7b68ee` | Brand purple |
| `--sage` | `#0cbc4d` | Success / keep |
| `--success-bg` | `#ecfdf3` | Success surface |
| `--success-border` | `rgba(12,188,77,.25)` | Success border |
| `--success-ink` | `#228b54` | Success text |
| `--border-input` | `#e5e5e5` | Input border |
| `--border-muted` | `#c4b5e3` | Violet-tinted border |

### Dark overrides

Applied via `[data-theme="dark"]` on `<html>`. Key flips:

| Token | Dark value |
|---|---|
| `--bg` | `#080025` |
| `--bg-card` | `#1c1c38` |
| `--ink` | `#fff8f0` |
| `--ink-*` | Same opacity, white base |
| `--success-bg` | `rgba(12,188,77,.18)` |
| `--success-ink` | `#5eebac` |

---

## 2. Typography

### Fonts

| Role | Family | Load |
|---|---|---|
| Display / headings | `neighbor` (variable) | Adobe Fonts `vdd7qlu` |
| Body | `DM Sans` | Google Fonts |
| Mono | `JetBrains Mono` | Google Fonts |

```html
<!-- Head links (required) -->
<link rel="stylesheet" href="https://use.typekit.net/vdd7qlu.css">
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,400;0,9..40,500;0,9..40,600;0,9..40,700;1,9..40,400&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
```

### Scale (inside steps)

All `.step h1` and `.step h2` are overridden by the compact layout rule:

```css
.step h1, .step h2 {
  font-size: clamp(1.5rem, 4vw, 2.5rem) !important;
  line-height: 1.1 !important;
}
```

Subtitle paragraphs (direct children of `.step`):

```css
.step > p {
  font-size: 15px !important;
  margin-bottom: 0.3rem !important;
  max-width: 34rem;
  margin-inline: auto !important;
  text-align: center !important;
  text-wrap: balance;
}
.step > p:last-of-type { margin-bottom: 2.25rem !important; }
```

### Display class

Apply `font-display` to any heading that needs the neighbor typeface + OpenType features:

```html
<h2 class="font-display font-bold tracking-tight">...</h2>
```

---

## 3. Background

Fixed dot-grid on `.mesh-bg` (one element per page, `aria-hidden`):

```html
<div class="mesh-bg" aria-hidden="true"></div>
```

Light: 24 × 24px grid of 1px ink-12 dots.  
Dark: same grid, white-08 dots at 35% opacity.  
Do not change to a gradient — the flat + dot pattern is intentional.

---

## 4. Components

### 4.1 Buttons

Buttons must be **unlayered** (not inside `@layer`) because Tailwind Play CDN's Preflight sets `button { background-color: transparent }` as unlayered CSS, which beats anything inside `@layer`.

#### Base class: `.btn`

```css
.btn {
  display: inline-flex; align-items: center; justify-content: center; gap: 6px;
  padding: 10px 24px; border-radius: 8px;
  font-family: 'DM Sans', sans-serif; font-weight: 600; font-size: 12.8px;
  white-space: nowrap; cursor: pointer; border: 1px solid transparent;
  transition: background-color 150ms ease, color 150ms ease,
              box-shadow 150ms ease, transform 150ms ease, border-color 150ms ease;
}
.btn:active:not(:disabled) { transform: scale(.98); }
.btn:disabled { opacity: .35; cursor: not-allowed; }
```

#### Variants

| Class | Use |
|---|---|
| `.btn-primary` | Main CTA — solid orange |
| `.btn-secondary` | Back / cancel — outlined |
| `.btn-ghost` | Inline link-style |
| `.btn-destructive` | Destructive confirm |
| `.btn-destructive-outline` | Destructive outlined |
| `.btn-block` | Full-width modifier |

```html
<!-- Forward action -->
<button class="btn btn-primary" onclick="nextStep()">
  Continue <svg class="hi"><use href="#hi-arrow-right"/></svg>
</button>

<!-- Back action -->
<button class="btn btn-secondary" onclick="goBack()">
  <svg class="hi"><use href="#hi-arrow-left"/></svg> Back
</button>
```

Icons inside buttons use `.hi` (16 × 16px, `flex-shrink:0`).

### 4.2 Game Cards

```css
.game-card {
  background: var(--bg-card);
  border: 1px solid var(--ink-15) !important;  /* !important beats Tailwind Preflight */
  border-radius: 14px;
  padding: 20px 24px;
}
.game-card:has(> textarea) { padding: 12px; }          /* textarea cards: slim */
.game-card:has(> textarea) > textarea { border: none !important; } /* card = only border */
```

Inputs / selects / textareas inside `.step` always get `background: var(--bg)` — the surface/background token, not the card token.

### 4.3 Spec Cards

Larger reveal/summary cards:

```css
.spec-card {
  background: var(--bg-card);
  border: 1px solid var(--ink-15) !important;
  border-radius: 24px;
  padding: 24px 32px;
}
```

### 4.4 Scenario Cards (pick-a-card)

```css
.scenario-card {
  background: var(--bg-card);
  border: 1px solid var(--ink-15);
  border-radius: 24px; padding: 28px;
  cursor: pointer;
}
.scenario-card:hover  { border-color: var(--ink-30); }
.scenario-card.picked { background: rgba(255,107,53,.08); border-color: var(--orange); }
```

### 4.5 This-or-That Options

```css
.tot-option {
  flex: 1; padding: 28px 24px; border-radius: 14px;
  border: 1px solid transparent; background: var(--bg-card);
  cursor: pointer; text-align: center;
}
.tot-option:hover  { background: var(--bg-mid); border-color: var(--border-muted); }
.tot-option.chosen { background: rgba(255,107,53,.08); border-color: var(--orange); }
```

### 4.6 Pills

```html
<!-- Coloured outline pill (intro screen) -->
<span class="pill-label text-orange/70 border-orange/30">15 Questions</span>

<!-- Muted status pill -->
<span class="pill-muted">Draft</span>
```

### 4.7 Mad Lib Blanks

Inline `contenteditable` spans:

```html
<span contenteditable="true" class="madlib-blank"
      id="q1-who" data-placeholder="who?" role="textbox"></span>
```

CSS: orange bottom-border, turns full orange on focus. Placeholder via `::before` + `attr(data-placeholder)`.

### 4.8 Selects / Dropdowns

```html
<select class="mt-2 w-full bg-transparent border border-white/10 rounded-lg
               py-3 pl-3 pr-10 text-white/70 text-sm
               focus:border-orange/50 focus:outline-none">
  <option value="" class="bg-dark">Pick one...</option>
</select>
```

- `pr-10` gives the native dropdown arrow room to breathe on the right — equal to the left padding visually.
- Options use `class="bg-dark"` so Chrome's option list stays legible on dark backgrounds.

### 4.9 Tabs

```html
<div class="tabs">
  <button class="tab is-active">Option A</button>
  <button class="tab">Option B</button>
</div>
```

Active tab gets `background: var(--orange); color: #fff`.

### 4.10 Progress Strip

Thin orange bar with glow dot + phase labels. Hidden on intro/intake, shown from step 1.

```html
<div class="progress-strip px-6 max-w-4xl mx-auto w-full" data-visible="false">
  <div class="phase-bar">
    <div class="phase-fill" id="progress-fill" style="width:0%"></div>
  </div>
  <div class="phase-labels">
    <span data-phase="0" id="pl-0">Problem</span>
    <span data-phase="1" id="pl-1">User</span>
    <!-- … -->
  </div>
</div>
```

JS drives `width` on `#progress-fill` and adds `.is-active` / `.is-done` / `.just-activated` to the label spans.

---

## 5. Step Layout

Every question screen is a `.step` div inside `#game-container`. All steps are `position:absolute` at all times — switching is done via opacity + transform, never by showing/hiding with `display`.

### Anatomy of a question step

```html
<div class="step" id="step-N">
  <!-- Phase badge — hidden via CSS; progress strip shows phase instead -->
  <div class="phase-badge mb-3">Phase 2 · The User</div>

  <!-- Heading — font-display, centered -->
  <h2 class="font-display font-bold text-[clamp(2rem,6vw,56px)] leading-[1.05] tracking-tight mb-3">
    Question Title
  </h2>

  <!-- Subtitle — direct child <p>, gets centered + balanced by CSS -->
  <p class="text-white/40 text-sm mb-8 text-center">
    One sentence prompt or framing.
  </p>

  <!-- Interactive card -->
  <div class="game-card">
    <!-- inputs, mad libs, scenario cards, etc. -->
  </div>

  <!-- Action row -->
  <div class="flex gap-3 mt-8">
    <button class="btn btn-secondary" onclick="goBack()">
      <svg class="hi"><use href="#hi-arrow-left"/></svg> Back
    </button>
    <button class="btn btn-primary" id="qN-next" onclick="nextStep()" disabled>
      Continue <svg class="hi"><use href="#hi-arrow-right"/></svg>
    </button>
  </div>
</div>
```

### Key rules

- **Heading**: Always `font-display font-bold tracking-tight`. The compact CSS rule caps it at `2.5rem` with `!important`.
- **Subtitle**: Use a single `<p>` as a direct child of `.step`. Centering, max-width (34rem), and `text-wrap:balance` are applied automatically. Do not split into two `<p>` tags.
- **Card**: Use `.game-card` for the interactive area. Inputs inside inherit `background:var(--bg)`.
- **Action row**: `<div class="flex gap-3 mt-8">` with Back on the left, Continue on the right. Continue starts `disabled` and is enabled by JS once the user fills something in.
- **Phase badge**: Include the `<div class="phase-badge">` div in the HTML (for reference), but CSS hides it — the progress strip is the source of truth for phase context.

### Transitions

Forward → previous step swipes LEFT, new step fades in (after 420ms delay).  
Back → previous step swipes RIGHT, prior step fades in.

Managed by `go(stepId)` in JS. Never manipulate `.active` directly.

---

## 6. Tailwind White-Alpha Class Overrides

The utilities layer remaps all `text-white/XX`, `border-white/XX`, and `bg-white/XX` Tailwind classes to ink-based tokens in light mode. This means HTML written for dark mode (white text on dark) works identically in light mode (ink text on cream) without touching the HTML.

**Use Tailwind classes for color, not inline `rgba()` values.** Inline styles bypass the override system and will look broken in light mode.

| Tailwind class | Light-mode result |
|---|---|
| `text-white/40` | `var(--ink-60)` |
| `text-white/50` | `var(--ink-60)` |
| `text-white/70` | `var(--ink-70)` |
| `text-white/80` | `var(--ink)` |
| `border-white/10` | `var(--ink-08)` |
| `border-white/20` | `var(--ink-15)` |
| `bg-white/5` | `var(--ink-05)` |
| `bg-white/10` | `var(--ink-08)` |

---

## 7. Cascade Rules & Gotchas

### Unlayered CSS beats @layer

Tailwind Play CDN injects its own unlayered CSS (Preflight). Anything inside `@layer components` or `@layer utilities` loses to unlayered CSS per the cascade spec.

**What must be unlayered:**
- `.btn` and all variants
- `.theme-switch` and its children
- `.tabs` / `.tab`
- Subtitle centering: `.step > p { margin-inline: auto !important; text-align: center !important; }`

**What can stay in @layer:**
- `.game-card`, `.scenario-card`, `.tot-option`, `.madlib-blank`, `.progress-strip`, etc.
- Any rule that doesn't need to beat Tailwind Preflight.

### !important for margin and borders on cards

`margin-inline: auto` on subtitle `<p>` elements needs `!important` because Tailwind's Preflight sets `* { margin: 0 }` as unlayered CSS.  
`border` on `.game-card` needs `!important` because Tailwind's Preflight resets `border-width: 0`.

### Theme initialization

The `initTheme()` IIFE at the top of the `<script>` block sets `[data-theme]` on `<html>` before first paint (defaults to `"light"` unless localStorage has `"dark"`). Every sibling game must include this IIFE.

---

## 8. Icons

Inline SVG sprite (`style="display:none"`) in `<body>`. Reference with:

```html
<svg class="hi"><use href="#hi-arrow-right"/></svg>
```

The `.hi` class sizes the icon to `1em × 1em` and aligns it to the text baseline. Use `style="width:1.5em;height:1.5em"` to override size when needed.

Available symbols: `hi-arrow-right`, `hi-arrow-left`, `hi-check`, `hi-bolt`, `hi-sparkles`, `hi-user`, `hi-document`, `hi-paper-clip`, `hi-envelope`, `hi-x-mark`, `hi-sun`, `hi-moon`, `hi-lock-closed`, and more — see the sprite block in the HTML.

---

## 9. Adding a New Question Step

Checklist:

1. Add `<div class="step" id="step-N">` inside `#game-container`.
2. Include the phase badge div (even though it's hidden by CSS).
3. Use a single subtitle `<p>` as a direct child of `.step` — the centering/sizing CSS applies automatically.
4. Wrap the interactive area in `.game-card`.
5. End with the Back / Continue action row (`flex gap-3 mt-8`).
6. Add `N` to `stepOrder` array in JS.
7. Add `N` to the correct phase in `phaseSteps` / `phaseMap`.
8. Wire up a `watch()` call (or equivalent) to enable the Continue button once input is valid.
9. Collect the answer in `generateSpec()` / `generateBrand()`.

---

## 10. File Locations

| File | Description |
|---|---|
| `app-game/index.html` | App Discovery game — 15 questions, 5 phases |
| `brand-game/index.html` | Brand Studio game — 18 questions, 7 phases |
| `design.md` | This file |
| `favicon.svg` / `favicon.png` | Shared favicon |
