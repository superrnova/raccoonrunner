# PRD: Rowdy's Last Call
**A browser-based mini game celebrating Topanga's acquisition of Raccoon Eyes**

---

## 1. Overview

A single-page microsite featuring a fully playable browser game. The game mechanic mirrors the product's core value proposition — plate waste visibility — by making the player *feel* the problem of missed data before landing them on the solution. No login, no leaderboard, no backend. A single self-contained HTML file, deployable to Netlify or Vercel by drag-and-drop.

**Target audience:** College & university dining operators, sustainability leads, food service directors — primarily reached via LinkedIn share and direct email link.

---

## 2. Microsite Structure

One HTML file. Top to bottom:

```
┌─────────────────────────────────────────┐
│  [Topanga wordmark or logo — top left]  │
├─────────────────────────────────────────┤
│                                         │
│   Plate waste has always been a         │
│   blind spot. Not anymore.              │
│                                         │
│   Can you catch it all? Our new         │
│   tracking system already does.         │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│            [ GAME CANVAS ]              │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│       [ See the real thing → ]          │
│                                         │
└─────────────────────────────────────────┘
```

### Page `<title>`
```html
<title>Rowdy's Last Call</title>
```
Controls the browser tab label and link preview text when shared on LinkedIn or via email. Do not leave as "index.html" or "Untitled."

### Page styling
- **Background color:** `#072524`
- **Font:** Geist — load via `https://fonts.googleapis.com/css2?family=Geist:wght@400;600;700&display=swap`
- **All page-level text color:** `#FFFFFF`
- **Headline:** Geist 700, ~48px desktop / 28px mobile
- **Subtitle:** Geist 400, ~18px desktop / 15px mobile, opacity 0.75
- **CTA button:** Geist Mono 600, all caps, label "SEE THE REAL THING →", background `#2463EB`, text `#FFFFFF`, border-radius `999px` (fully rounded pill), padding 14px 28px, no border
- **CTA link:** `https://topanga.io/solution/streamline-plate-waste`, opens in new tab
- Page max-width: 960px, centered, padding 40px 24px

---

## 3. Game Spec: Rowdy's Last Call

### 3.1 Concept
Plates loaded with food scraps slide off a dining hall conveyor belt from the top of the screen. Rowdy stands at the bottom holding a collection bin. The player moves Rowdy left and right to catch plates before they hit the trash. Every missed plate is a blind spot. Every caught plate is tracked data.

### 3.2 Canvas Dimensions
- **Desktop:** 800px wide × 500px tall
- **Mobile:** Full viewport width × 460px tall
- Centered on page, border: `1.5px solid rgba(226,248,156,0.25)`

### 3.3 Background Layer
- **Asset:** `GameBackground.png` (2752×1536px, provided)
- Render as full-canvas image fill (`drawImage`, cover/crop to fit canvas)
- **Dark overlay on top:** `rgba(4, 15, 15, 0.48)` — dims the illustration so game elements pop without obscuring the environment
- **Atmosphere on top of overlay:**
  - Vignette: `radial-gradient(ellipse at center, transparent 40%, rgba(0,0,0,0.25) 72%, rgba(0,0,0,0.55) 100%)` rendered as a canvas overlay
  - Subtle film grain: SVG `feTurbulence` noise layer, `opacity: 0.07`, animated at 10fps with random translate offsets
  - No flicker effect

**Conveyor belt strip (drawn in canvas, top edge):**
Draw a fixed decorative strip across the full canvas width at `y = 0`, height `28px`, to visually explain where plates are coming from:
- Base fill: `#3a2a10` (dark wood/belt tone)
- Repeating belt segments: alternating `#4a3618` and `#3a2a10` rectangles, each 30px wide
- Thin top highlight line: `rgba(255,200,100,0.18)`, 2px, at `y = 0`
- Bottom edge border: `#E2F89C` at 1.5px opacity 0.4, at `y = 28`
- Plates spawn from `y = 28` (just below the belt), so they appear to slide off it

### 3.4 Character: Rowdy

**Assets:** Three individual pre-cut PNG files with transparent backgrounds. No cropping or background removal required — load and draw directly.

| File | Expression | When used |
|---|---|---|
| `StandardRaccoon.png` | Neutral / confident, hands on hips | Default while moving |
| `HeartRaccoon.png` | Heart-eyes, arms raised in celebration | Flash for 400ms on successful catch |
| `SadRaccoon.png` | Sad, arms crossed, droopy eyes | Flash for 400ms on miss; shown on game over screen |

**Rendering:**
- Render Rowdy at **100×100px** on canvas, positioned at bottom: `y = canvasHeight - 115`
- Rowdy's horizontal center follows the player's position (`playerX`)
- All sprites are portrait-oriented — scale to fit within the 100×100px bounding box while preserving aspect ratio (Rowdy will be slightly narrower than 100px)
- Expression swap is instantaneous (swap image source, no tween)
- No flip needed for left/right movement — Rowdy always faces forward

**⚠️ Important — HeartRaccoon.png size normalization:**
`HeartRaccoon.png` is 5998×7004px — nearly 3× larger than the other two sprites — because the image canvas includes floating hearts surrounding Rowdy's body. Do NOT scale this image by its full bounding box or Rowdy will appear much smaller than in the other states. Instead, visually estimate that Rowdy's body occupies approximately the center 60% of the image height. Scale HeartRaccoon so that Rowdy's body height matches the rendered height of StandardRaccoon and SadRaccoon (i.e., scale HeartRaccoon to roughly 167px tall and draw it so Rowdy's body center aligns with the other sprites, letting the hearts bleed outside the normal bounding box). The hearts extending beyond the character bounds are fine and add a nice catch celebration effect.

**Collection bin:**
- A simple rounded rectangle drawn below Rowdy in `#072524` with `#E2F89C` border, ~80px wide × 20px tall
- This is the actual catch hitbox

### 3.5 Plates

Each plate is a canvas-drawn object:

**Plate visual:**
- Outer circle: `r=28px`, fill `radial-gradient` from `#ffffff` at 35% to `#ede4d0` at 100%
- Inner rim stroke: `rgba(0,0,0,0.07)`, lineWidth 8
- Food emoji centered on plate, font-size `22px`

**Food emoji pool** (pick randomly per plate):
`🥦 🍝 🍗 🥕 🍞 🥗 🌽 🍜 🥩 🍲 🫛 🥔`

**Plate types:**
- **Waste plate** (90% of spawns): has a food emoji, worth +10 points on catch, counts toward tracked total
- **Clean plate** (10% of spawns): empty (no emoji, just the plate), catching it does nothing / missing it does nothing — a visual decoy that adds tension without penalizing the player

### 3.6 Game Loop

**Spawning:**
- Plates spawn from the top of the canvas at randomized x positions (keep at least 60px from edges)
- Minimum horizontal gap between simultaneous plates: 80px
- Initial spawn rate: 1 plate every 1.8 seconds
- Initial fall speed: 2.8px per frame (60fps target)

**Difficulty ramp** (every 18 seconds):
| Stage | Fall speed | Spawn rate | Max simultaneous plates |
|---|---|---|---|
| 1 (0–18s) | 2.8 | 1.8s | 1 |
| 2 (18–36s) | 3.4 | 1.5s | 2 |
| 3 (36–54s) | 4.1 | 1.2s | 2 |
| 4 (54–72s) | 4.9 | 1.0s | 3 |
| 5 (72s+) | 5.8 | 0.8s | 3 |

**Rush hour event:** Every 45 seconds, trigger a 5-second burst where 4–5 plates appear in rapid succession. Show a brief text flash: `"RUSH HOUR"` in `#E2F89C` at canvas center, Geist 700 24px, fades out in 800ms.

### 3.7 Player Controls

**Desktop:**
- `←` `→` arrow keys: move Rowdy 18px per keydown event
- Smooth movement: hold key = continuous movement at 5px/frame

**Mobile:**
- Touch drag: Rowdy's x-position follows touch x directly
- No tap-to-move — drag only, so it feels like physically sliding him

### 3.8 Scoring & Lives

**Score tracking:**
- `caughtPlates` counter (waste plates only)
- `totalPlates` counter (waste plates spawned, regardless of caught/missed)
- **Displayed score:** `Math.round((caughtPlates / totalPlates) * 100)` = "TRACKED: XX%"
- Secondary display: raw points (10 per catch, streak multiplier below)

**Streak multiplier:**
- 3 catches in a row: `×1.5`
- 6 catches in a row: `×2.0`
- Miss resets streak to 1×
- Show multiplier as small badge next to score when active: `×1.5` in `#E2F89C`

**Lives:** 3 misses allowed. On miss:
- One life icon changes from filled ♥ to outlined ♡ in the HUD — filled `♥` in `#E2F89C` for remaining lives, outlined `♡` (stroke only, no fill) for lost lives
- Rowdy flashes SadRaccoon expression (400ms)
- A `?` bubble appears over the trash can for 600ms
- On 3rd miss: game over

### 3.9 HUD (in-canvas UI)

All HUD elements use `#072524` fill + `1.5px #E2F89C` border + `#E2F89C` text, Geist Mono or monospace fallback.

```
┌──────────────────────────────────────────────┐
│  TRACKED: 72%  ·  SCORE: 1,240  ·  ×1.5     │   ← top center bar
│                                        ♥♥♡   │   ← lives top right (3 icons)
└──────────────────────────────────────────────┘
```

**Catch feedback (on successful catch):**
- Small tag pops up from the caught plate and floats up 30px then fades out over 700ms
- Tag content: `✓  47g · [food name]` — food name corresponds to emoji (see mapping below)
- Tag style: `#072524` fill, `#E2F89C` border + text, Geist Mono 8px

**Emoji → food name mapping (for data tags):**
```
🥦 → Broccoli       🍝 → Pasta          🍗 → Roast chicken
🥕 → Carrots        🍞 → Bread roll     🥗 → Garden salad
🌽 → Corn           🍜 → Noodle soup    🥩 → Steak
🍲 → Stew           🫛 → Green beans    🥔 → Mashed potato
```

**Weight values** (randomized within range per food, shown in tag):
Each plate gets a random weight in its food's range: e.g. broccoli 15–40g, pasta 80–180g, steak 60–140g. Specific ranges can be approximate — just make them feel realistic for a dining hall plate waste scenario.

**Missed plate behavior:**
- A missed waste plate does NOT teleport or disappear — it continues falling past the collection bin and exits the bottom of the canvas naturally over ~0.3 seconds, then is removed from the object pool
- As it passes the bin level, trigger: Rowdy flashes SadRaccoon (400ms), one life dims, and the ? bubble appears over the trash can for 600ms then fades
- Clean plates (decoys) that are missed simply fall off the bottom silently — no feedback, no life lost

**Miss feedback:**
- A `?` in a circle (`#072524` fill, `#E2F89C` border, 38px diameter) appears over the trash can at bottom-right of canvas for 600ms then fades out

**Score edge case — NaN guard:**
- `totalPlates` starts at 0; before any plate has spawned, `caughtPlates / totalPlates` would produce `NaN`
- Guard: if `totalPlates === 0`, display `TRACKED: —%` instead of calculating. Once the first waste plate spawns, begin showing the live percentage normally

### 3.10 Game States

**STATE 1: Start screen (pre-game)**

The game canvas is fully rendered but dimmed — background at reduced opacity, StandardRaccoon.png visible and centered at bottom — giving the player a preview of the environment before they begin. A dark overlay sits on top (`rgba(4, 15, 15, 0.65)`) to create the low-light effect.

Centered on the canvas, over the overlay:
```
┌─────────────────────────────────┐
│      ROWDY'S LAST CALL          │   Geist 700, 28px, #E2F89C
│                                 │
│   ← → to begin                  │   desktop: white, Geist 400, 15px
│   Tap to begin                  │   mobile: white, Geist 400, 15px
│                                 │
│   (StandardRaccoon already      │
│    visible in background,       │
│    dimmed with everything else) │
└─────────────────────────────────┘
```

- Detect desktop vs mobile: if `window.matchMedia('(pointer: coarse)')` is true, show "Tap to begin"; otherwise show "← → to begin"
- **Trigger:** First arrow key press (desktop) or first touch (mobile) starts the game — no button click required
- **Transition:** On trigger, the dark overlay fades out over 400ms, bringing the canvas up to full normal brightness. The game loop begins immediately as the fade completes.

**In-game tutorial tooltip (appears once, at game start):**

Immediately after the start transition, show a brief instructional message inside the canvas at the top:

```
┌──────────────────────────────────────────────────┐
│  Move Rowdy to capture plates and score points.  │
└──────────────────────────────────────────────────┘
```
- Style: white Geist 400 text, 12px, on a `#072524` background box with `1.5px #E2F89C` border, centered horizontally, `y = 44px` (below HUD)
- Stays visible for **2.5 seconds**, then fades out over 500ms
- Never shown again once dismissed (just a one-time game-start hint)

**STATE 2: Playing**
Full game loop as described above.

**STATE 3A: Game over (standard — score < 100%)**
```
┌────────────────────────────┐
│      GAME OVER             │   Geist 700, 28px, #E2F89C
│                            │
│  [SadRaccoon.png]          │   centered, 100×100px
│                            │
│  You tracked               │
│       72%                  │   large, 48px, #E2F89C
│  of plate waste.           │
│                            │
│  "Plate waste has always   │
│  been a blind spot.        │
│  Not anymore."             │   italic, 13px, white 80%
│                            │
│  [ Play again ]  [ See the real thing → ] │
└────────────────────────────┘
```

**STATE 3B: Game over (perfect score — 100% tracked)**

Triggered only if `caughtPlates === totalPlates` and `totalPlates >= 5` (at least 5 waste plates must have spawned so a 100% isn't trivially earned in the first 3 seconds):
```
┌────────────────────────────┐
│   100% TRACKED.            │   Geist 700, 28px, #E2F89C
│   ROWDY'S IMPRESSED.       │   Geist 700, 28px, white
│                            │
│  [HeartRaccoon.png]        │   centered, scaled per §3.4 rules
│                            │
│  "Plate waste has always   │
│  been a blind spot.        │
│  Not anymore."             │   italic, 13px, white 80%
│                            │
│  [ Play again ]  [ See the real thing → ] │
└────────────────────────────┘
```
- No "You tracked X% of plate waste" line needed — the headline IS the score
- Same button behavior as State 3A

**Both game over states:**
- "Play again" resets all state and returns to State 1 (start screen)
- "See the real thing →" opens `https://topanga.io/solution/streamline-plate-waste` in new tab
- Both buttons: pill-shaped (`border-radius: 999px`), Geist Mono 600, all caps, consistent padding (12px 24px)
- Primary ("SEE THE REAL THING →"): `#2463EB` fill, `#FFFFFF` text
- Secondary ("PLAY AGAIN"): `#072524` fill, `#E2F89C` border, `#E2F89C` text

---

## 4. Technical Stack

| Concern | Approach |
|---|---|
| Language | Vanilla HTML/CSS/JS — no framework, no bundler |
| Rendering | HTML5 Canvas API for all game elements |
| Assets | Both PNGs embedded as Base64 at build time (no separate asset hosting needed) |
| Font | Geist via Google Fonts CDN (page-level), monospace fallback in canvas |
| Background removal | Not needed — all three Rowdy sprites ship with transparent backgrounds |
| Deployment | Single `.html` file drag-and-drop to Netlify |
| Mobile | Touch events (`touchstart`, `touchmove`) mirroring mouse handler |
| Accessibility | `prefers-reduced-motion` respected — if set, remove grain animation and rush hour flash |

**File structure (single file):**
```
index.html
  └── <style>         Page + HUD CSS
  └── <canvas>        Game canvas
  └── <div>           Page chrome (headline, subtitle, CTA)
  └── <script>        All game logic inline
      ├── Asset loading (Base64 embedded, no removal needed)
      ├── Game state machine (START / PLAYING / GAMEOVER)
      ├── Plate spawner
      ├── Physics loop (requestAnimationFrame)
      ├── Input handlers (keyboard + touch)
      ├── Collision detection
      ├── HUD renderer
      └── Atmosphere effects (vignette, grain)
```

---

## 5. Assets to Provide Claude Code

Place all files in the same directory as `index.html` before starting, or tell Claude Code to embed them as Base64:

| File | Dimensions | Mode | Source |
|---|---|---|---|
| `GameBackground.png` | 2752×1536px | RGBA | Nano Banana–generated cafeteria background |
| `StandardRaccoon.png` | 2306×3502px | RGBA | Rowdy default state — pre-cut transparent background |
| `HeartRaccoon.png` | 5998×7004px | RGBA | Rowdy excited/catch state — pre-cut transparent background |
| `SadRaccoon.png` | 2417×3530px | RGBA | Rowdy sad/miss/game over state — pre-cut transparent background |

All three Rowdy sprites already have transparent backgrounds — **no background removal step needed.** Load them directly as image assets.

When prompting Claude Code, say: *"Embed all four images as Base64 data URIs in the HTML file so it works as a single self-contained file with no external asset dependencies."*

---

## 6. Prompting Strategy for Claude Code

Start with this prompt, then iterate:

> "Build a single self-contained HTML file called `index.html` for a browser game called Rowdy's Last Call. Read `rowdys-last-call-PRD.md` in this directory fully before writing any code — it is the complete specification. Embed all four images (`GameBackground.png`, `StandardRaccoon.png`, `HeartRaccoon.png`, `SadRaccoon.png`) as Base64 data URIs so the file is fully self-contained. Start with just the core game loop: plates falling from the conveyor belt, Rowdy moving left and right, collision detection, lives, and score tracking. No visual polish yet — get the mechanics working first. Use HTML5 Canvas for all game rendering."

After the first build, test the core loop before asking for visual polish. Suggested iteration order:
1. ✅ Core loop working (plates fall, Rowdy moves, collision detection, lives, score)
2. ✅ Background + all three Rowdy sprites rendering correctly (transparent PNGs, no removal needed)
3. ✅ HUD styling (#E2F89C / #072524)
4. ✅ Catch/miss feedback animations (data tag float, ? bubble)
5. ✅ Expression state swaps (Standard → HeartRaccoon on catch, Standard → SadRaccoon on miss)
6. ✅ Start screen + game over screen
7. ✅ Atmosphere (vignette + grain)
8. ✅ Mobile touch support
9. ✅ Page chrome (headline, subtitle, CTA button)
10. ✅ Final polish (rush hour event, streak multiplier)

---

## 7. Out of Scope

- User accounts / login
- Leaderboard or score persistence
- Sound / audio
- Animations beyond sprite swaps and floating tags
- Server-side anything
- Analytics (can add a simple `gtag` snippet later if desired)
