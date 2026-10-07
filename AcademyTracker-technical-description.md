# AcademyTracker — Technical Description

**Product:** AcademyTracker (concept MVP / playable demo)
**Artefact described:** `index.html` — one self-contained file, hosted as a static page (GitHub Pages)
**Purpose of this document:** let a developer who has never seen the app rebuild it, screen for screen and rule for rule, from this document alone.

Where this document says **"verbatim"**, the appendix contains the exact source text, which should be copied as is. Where it describes behaviour in prose, the prose is derived from the shipped code, not from intent.

---

## 1. What the app is (and is not)

AcademyTracker is a front-end-only concept demo of a football (soccer) analytics product for youth/academy players. The pitch to the user: *film a match on any phone; get a match report, season trends, a personalised training plan, a shareable player card, and fixtures/league table.*

**What it is not.** There is no computer-vision pipeline, no backend, no database, no login, no persistence and no network calls other than loading two Google Fonts. Every number on screen is hard-coded sample data. "Analysis" is a timed animation that ends by showing pre-written data. The interface says so in four places (see §6.2).

**Stack.** Plain HTML + CSS + vanilla JavaScript in one file. No framework, no build step, no dependencies. Fonts come from Google Fonts (`Archivo Black`, `Archivo` 400/500/600/700); if they fail to load, the CSS fallbacks apply.

**State.** All state lives in JavaScript variables and the DOM and is lost on refresh: current clip, current half, current season metric, self-rating answers, selected position, ticked drills.

---

## 2. Application shell

The page is: **header → sticky nav → `<main>` containing five views (only one visible at a time) → footer.**

### 2.1 Header
- Dark green band (`--turf-deep`), chalk text, padding `26px 20px 22px`. Inner container max-width 1040px, centred, flex row, gap 16px, wraps.
- **Logo:** an `<h1 class="logo">` reading `Academy` + `<span>Tracker</span>`. Archivo Black, `clamp(1.5rem, 4vw, 2.1rem)`, **not** upper-cased, letter-spacing 0. The word "Tracker" is amber (`--amber`). The logo is text only: no image, no icon, no background pattern.
- **Tagline** under the logo: `Match insights from any phone camera. No special kit.` (0.95rem, `#CFE3D6`).
- **Demo pill** on the right: `CONCEPT DEMO · SIMULATED DATA` — mono, 0.72rem, letter-spacing 0.08em, dashed 1.5px amber border, amber text, fully rounded. On screens ≤560px it drops to the left (`margin-left:0`).
- Browser tab title: `AcademyTracker — Concept MVP`.

### 2.2 Navigation bar (`nav.appnav`)
- White (`--card`) bar, 1.5px bottom border `#D8D4C6`, **sticky at `top:0`, z-index 5**. Measured height ≈ 47px (the sticky sample label depends on this; see §2.4).
- Inner row: max-width 1040, padding `0 20px`, flex, gap 2px, `overflow-x:auto` (tabs scroll sideways on phones).
- Five tab buttons, in this order, labels exactly as shown (emoji included):

| Order | Label | `data-view` | Shows |
|---|---|---|---|
| 1 | ⚽ Analyse a match | `view-match` | default view on load |
| 2 | 📈 My season | `view-season` | |
| 3 | 🏋️ Training plan | `view-train` | |
| 4 | 🃏 Player card | `view-card` | |
| 5 | 📅 Fixtures & table | `view-fixtures` | |

- Button style: no border/background, padding `14px 16px 11px`, bold 0.9rem, muted colour, 3px transparent bottom border, no wrapping. Active tab (`aria-current="true"`): text `--turf-deep`, bottom border `--amber`. Hover: text `--turf-deep`.
- **Behaviour on click:** set `aria-current` true on the clicked button and false on the others; set `hidden` on every view except the target; scroll to top; if the target is `view-season`, re-render the season chart for the currently selected metric.
- **"Try the real app" button** (sixth element, right-aligned): an `<a class="nav-cta" href="#">`. Wrapped in `.nav-cta-wrap`, which is `position:sticky; right:0`, `margin-left:auto`, `flex:none`, white background, left padding 12px and a soft left shadow, so the button stays visible when the tab row scrolls on a phone. Style: amber background, ink text, bold 0.82rem, pill shape, 1.5px `--amber-deep` border, hover background `--amber-deep`. **It currently points to `#`** (does nothing). To activate it, replace `href="#"` with the real app URL; an HTML comment marks the line. It does not change the active tab.

### 2.3 Footer
`AcademyTracker · concept MVP for the Imperial EDGE accelerator. This demo shows the intended experience; the computer-vision pipeline is not built yet.` Max-width 1040, 0.8rem, muted.

### 2.4 Persistent "sample data" label
Component `.sample-label`, text: **`Sample data`** (bold) `— figures shown are for demonstration only.` Appears on exactly two screens: the match report (step 3) directly under the report subtitle, and the My season view directly under its intro paragraph. Style: mono 0.7rem, `#FBF3D6` background, 1.5px **dashed** `--amber-deep` border, radius 10px, padding `5px 12px`, `width:fit-content`, `role="note"`. It is `position:sticky; top:47px; z-index:4` — it sticks just under the nav (below the nav in stacking order, so it never covers it) and so stays visible while scrolling.

### 2.5 Main container
`main`: max-width 1040px, centred, padding `28px 20px 60px`. Views are `<div id="view-match">` and `<section id="view-season|view-train|view-card|view-fixtures">`; all but `view-match` start `hidden`.

---

## 3. Pages (views)

### 3.1 View 1 — Analyse a match (`view-match`)
A three-step flow. Only one step is visible at a time: `#step-upload` → `#step-processing` → `#dashboard`.

#### Step 1 — Footage (`#step-upload`)
- Step label (mono, 0.72rem, uppercase, letter-spaced): **`STEP 1 / 3 · FOOTAGE`**.
- H2: `Pick a match recording`.
- Lede: `Film from the sideline with a normal phone. AcademyTracker finds the players, tracks the ball, and turns 60 shaky minutes into numbers you can act on.`
- **Upload tile** (a `<label for="video-upload">` wrapping a visually hidden `<input type="file" accept="video/*">`): dark green card, dashed amber border, upload icon (46px rounded square holding an arrow-up-into-tray SVG), title `Upload your own match footage`, hint `Any phone video works — tap to choose a file from your camera roll.` (`#upload-hint`).
  - **On file chosen:** add class `has-file` (solid border, amber icon tile), set the hint to `Selected: <file name> (<size in MB, 1 d.p.>) — tap a sample below or wait, analysing now…`, then immediately start the simulated analysis using a **copy of the first sample clip** (Rovers) with `id:"custom-upload"`, `title:"Your upload · <file name without extension>"`, `meta:"Your phone footage · analysis simulated for this demo"`, `sub:"Uploaded <today, e.g. 4 Oct 2026>"`. The file itself is never read or uploaded. Restarting does not reset the hint text.
- Divider: `OR TRY A SAMPLE MATCH` (mono, uppercase, hairlines either side).
- **Sample clip cards** (`#clip-list`, CSS grid `repeat(auto-fit, minmax(250px, 1fr))`, gap 16px). One `<button class="clip">` per entry in `CLIPS` (3 cards). Each card: a 118px green gradient thumbnail showing the pitch markings with four small randomly placed dots (1 amber, 2 chalk, 1 ink "ball" — positions are randomised on every page load), a translucent `● PHONE FOOTAGE` badge top-left; below it the title (bold 1rem), the `meta` line (muted 0.82rem) and the call to action `Analyse this match →`. Hover: lift 3px, green border, shadow.
- Note beneath (**the existing simulated-analysis note, must be kept**): `📱 The upload button above accepts a real video file — but since the CV pipeline isn't built yet, it runs the same simulated analysis as the samples below.`

#### Step 2 — Analysis (`#step-processing`, `aria-live="polite"`)
- Label `STEP 2 / 3 · ANALYSIS`, H2 `AcademyTracker is watching the match…`, lede `Slow and steady: every frame gets scanned, every player gets an ID, every touch gets logged.`
- Green panel containing: (a) a pitch SVG with four dots animating via SVG `<animate>` (two chalk, one amber "player" with a dashed amber bounding box, one small ink "ball") and a vertical amber **scan line** (3px, glow) sweeping left↔right on a 2.6s linear alternate CSS animation; (b) an 8px progress bar (amber fill, width transition 0.5s); (c) a monospace log area (min-height 120px) that appends lines.
- **Timeline** (normal motion; step = 850ms): the log lines appear at `850·n` ms for n = 1…6, each prefixed `▸ `, and set the bar width:

| n | Log text | Bar % |
|---|---|---|
| 1 | `Loading footage · <clip title in lower case>` | 8 |
| 2 | `Stabilising shaky camera… done` | 22 |
| 3 | `Detecting players… 22 found` | 40 |
| 4 | `Locking onto #7 A. Khan (that's you)` | 58 |
| 5 | `Tracking ball · logging touches, passes, sprints…` | 78 |
| 6 | `Building your match report… ready` | 100 |

  (`done`, `22 found`, `#7 A. Khan`, `ready` are amber `.ok` spans.) 700ms after line 6 the report renders. Total ≈ 5.8 s. With `prefers-reduced-motion: reduce`, the step is 60ms and the final delay 100ms, and all animations/transitions are disabled.
- Starting a new analysis clears any pending timers first.

#### Step 3 — Match report (`#dashboard`)
Header: label `STEP 3 / 3 · INSIGHTS`; H2 `Your match report`; subtitle = `<clip.title> · <clip.sub>`; then the **sample-data label** (§2.4).

**Jersey card** (210px wide, dark green, min-height 220px; full width ≤560px): top-left a **touches badge** (amber number + `TOUCHES`), top-right a **match rating** (`ovr` in chalk + `MATCH RATING`), then the shirt number (4.4rem, amber, Archivo Black), player name (uppercase) and position (mono, letter-spaced). Data: number 7, "A. Khan", position per clip, rating per clip, touches taken from the stat labelled `Touches`.

**Key-stats grid** (`auto-fit, minmax(130px,1fr)`, gap 12px): one white card per **visible** stat of the clip — big green number (Archivo Black 1.5rem), label, and a small mono green delta line. The `Touches` card gets the highlight style (amber-deep border, inset amber ring, amber-deep number). Visible stats per clip = the clip's `stats` minus those whose label begins `Top speed` or `Distance` (§6.3). So **four tiles**: Touches, Pass accuracy, Shots (…), Take-ons (…).

**Six tabs** (pill buttons, `role="tablist"`, selected = dark green fill): Heatmap, Sprints, Passing, Shot map, Attributes, Team shape. Rendering the report always resets to the first tab (Heatmap) and the first half (see §8 for a stale-button quirk). Only the selected panel is visible (`hidden` on the rest). Each panel is a white card (14px radius, 20px padding) with an H3, a muted sub-line, the graphic, and a **coach note** (cream `#FBF3D6` box with amber border) containing the clip-specific insight.

1. **Heatmap** — "Where you played". Sub: `Every second on the ball and off it, mapped onto the pitch. Attacking left → right.` Half toggle (`1st half` / `2nd half`, amber when pressed) picks `heat.h1` or `heat.h2`. Each entry `[x, y, r]` draws a circle at (x,y) radius r filled with a radial gradient (`#F2B705` at 85% opacity at the centre → `#E0641E` at 45% at 55% → `#E0641E` at 0% at the edge). Circle opacity = `0.9 − 0.08·index`.
2. **Sprints** — "Sprint map". Sub: `High-intensity runs per 5-minute block. Hover or tap a bar for detail.` A bar chart, one bar per 5-minute block (§4.1 `sprints`): SVG viewBox 100×56; baseline at y=48; bar width `0.7·(100/n)`, left offset `0.15·(100/n)`; bar height `v/max·40` (max floored at 1); bars green, amber on hover/focus; value printed above each bar (blank if 0); x-axis labels `0'`, `5'`, `10'`… at y=53.5. Each bar is focusable with an aria-label such as `Minutes 25 to 30: 0 sprints`.
3. **Passing** — "Pass map". Sub: `Completed passes in chalk, missed in amber. Attacking left → right.` Arrows from (x1,y1) to (x2,y2): completed = solid chalk `#F7F5EC`; missed = dashed amber `#F2B705` (`1.4 1` dash); stroke 0.55; triangular arrowheads (3×3.6 markers) in matching colour. Legend: `Completed` (solid chalk line) and `Missed` (dashed amber line).
4. **Shot map** — Sub: `Every shot, where it came from and how it ended. Attacking left → right.` Each shot = a dot (r=1) at the origin (x,y) and a line to the target (tx,ty), stroke 0.6, arrowhead. Goal = solid amber; saved/on target = solid chalk; off target = dashed amber. Targets beyond x=100 are intentionally outside the pitch (wide/over). Legend: `Goal` (amber), `On target` (chalk), `Off target` (dashed amber).
5. **Attributes** — "Match attributes". Sub: `Your performance this match, scored 0–99 against players in your age group. Not a permanent rating — it moves every game.` Six horizontal bars (Pace, Shooting, Passing, Dribbling, Defending, Physical) with the value right-aligned in mono; bar fill is green and animates from 0 to value% over 0.8s each time the panel is shown (set via `data-w`, applied after two animation frames). Beside it a fixed coach note: `How this works: each score compares this match to a benchmark of players in your age group. One bad game won't sink you — one great game won't crown you. Trends over 5+ matches are what matter.`
6. **Team shape** — Sub: `Average positions across the match. You are the amber dot.` Eleven (or five) dots on the pitch: teammates chalk r=2, you amber r=2.6, ink outline 0.4; the formation string (e.g. `4-3-3`) is printed top-right in Archivo Black size 4. Team attacks left → right, so the goalkeeper is at low x.

**Footer row:** button `← Analyse another match` (returns to step 1 and scrolls to top; does not clear anything else) and mono note `All numbers simulated for concept validation — no real CV yet.`

**Pitch graphic (shared by every pitch panel and the thumbnails).** SVG `viewBox="0 0 100 64"`, `preserveAspectRatio="xMidYMid meet"`, width 100%, on a `--turf` background. Coordinates in the data are in these units (x 0–100 along the pitch, y 0–64 across it; attack is toward x=100). The exact markup is in Appendix B.3.

### 3.2 View 2 — My season (`view-season`)
- Label `MY SEASON · 8 MATCHES ANALYSED`; H2 `Are you actually improving?`; lede `One match is noise. A season is a signal. AcademyTracker tracks every analysed match so you can see the trend, not just the highlight.`; then the **sample-data label** (§2.4).
- **Metric toggle** (three buttons, amber when pressed): `Match rating` (default), `Touches`, `Pass %`. *(There is deliberately no top-speed button; see §6.3.)*
- **Line chart** in a white card: SVG viewBox 100×56. Plot area x from 8 to 96 (`X(i) = 8 + i·88/7`), y from 46 (bottom) up to 8: `Y(v) = 46 − ((v − lo)/(hi − lo))·38` where `lo = min − pad`, `hi = max + pad`, `pad = 0.25·(max − min)` (or 1 if max = min). Green polyline (1.1 wide, round joins); a dot per match (r=1.8, green, white outline), the latest match's dot is amber; the value is printed above each dot (mono, size 3) and `M1`…`M8` below the axis (y=52); baseline from x=6 to 98 at y=46. Each dot group is focusable with an aria-label `vs <opponent>: <value><unit>`.
- **Three summary tiles** under the chart: `Latest (vs Rovers)` = last value; `Change since match 1` = last − first, rounded to one decimal, prefixed `+` when positive; `Season best` = maximum. Values carry the metric's unit (`%` for pass accuracy, none otherwise).
- A coach note per metric (`METRICS[metric].note`, Appendix B.4).
- Data: the 8-match `SEASON` table (Appendix B.4). Each time the view is opened the chart re-renders for the current metric; the chart is also rendered once on load for `rating`.

### 3.3 View 3 — Training plan (`view-train`)
- Label `TRAINING PLAN · WEEK OF 20 JUL` (static text); H2 `This week's homework`.
- **Plan source toggle** (two buttons): `From my match data` (default) and `From self-rating`. Switching shows one block and hides the other.

**A. From my match data (`#plan-match`)**
- Lede: `AcademyTracker picks three drills from your weakest match data — not a generic programme. Tick them off as you go; next match shows whether they worked.`
- Progress text `0 of 3 drills done this week`, updating on each tick to `N of 3 drills done this week`, and `3 of 3 drills done this week — full house` when all three are ticked.
- Three fixed drill cards from the `DRILLS` array (Appendix B.5). **No scoring or selection happens here:** the "personalisation" is pre-written copy that refers to the sample match data. Each card: checkbox (22px, green), drill name (a `<label>` for the checkbox), a "why" line (with a bold amber-deep highlight), and a mono duration/equipment line. Ticking toggles the `done` class (green border, struck-through, muted title). Ticks are not stored.

**B. From self-rating (`#plan-self`) — the quiz and plan generator**
- Lede: `No footage yet? Rate yourself 1–5 in each area and AcademyTracker builds a plan on the spot — more drills where you're weakest, fewer where you're already strong.`
- **Form** (white card, rows separated by hairlines):
  1. Row `Position` with five pill buttons: Goalkeeper, Defender, Midfielder, Winger, Striker (single-select, `aria-pressed`).
  2. Five rows, one per category, each with five round buttons `1`–`5` (single-select per row): `Cardio & fitness`, `Shooting`, `Passing`, `Dribbling`, `Defending`.
- **Generate my plan** button — disabled until a position is chosen **and** all five categories rated (greyed at 45% opacity). A hint beside it reads `Rate all 5 areas and pick a position to generate your plan (N left)` where N = unanswered categories + 1 if no position; when complete it reads `Ready — generate whenever you like.` Initial text before any interaction: `Rate all 5 areas and pick a position to generate your plan`.
- On click, the plan is generated (§6.5) and rendered below: a progress line `0 of <plan length> drills done this week` (→ `N of M …`, plus ` — full house` when N = M), then one card per drill with a small green **category chip** (`Cardio & fitness`, `Shooting`, …), the drill name, the "why" text and the duration line. Clicking Generate again rebuilds the list from scratch (ticks reset). Answers persist in memory while the page is open, so changing one rating and regenerating works.
- Static coach note at the foot of the view: `Honesty check: drills only count if next match's numbers move. AcademyTracker will compare your take-on success and shot placement before vs after this training block.`

### 3.4 View 4 — Player card (`view-card`)
- Label `PLAYER CARD · SHARE YOUR MATCH`; H2 `Your match, one card`; lede `A stats card built from real match data — made for the group chat, not just the coach's laptop.` (This wording is original and was left unchanged.)
- Two-column layout (`minmax(250px,330px) 1fr`, gap 24px; single column ≤640px).
- **Card** (`#pcard`, `role="img"`): 3px amber border, 18px radius, diagonal gradient from `--turf-deep` to `#0F2C1C`. Top row: large amber rating (2.9rem) with `MATCH RATING` beneath, and on the right the shirt number as `#7` at 35% opacity. Then the player name (uppercase, 1.3rem), the match line `<TITLE> · <SUB>` upper-cased (mono 0.7rem), a two-column list of the clip's **visible** stats capped at six (label with any bracketed qualifier stripped via `/\s*\(.*\)/`, value in mono), and the footer `ACADEMYTRACKER · VERIFIED FROM FOOTAGE` (amber, mono, letter-spaced). With the hidden stats removed, each card shows four stat rows: Touches, Pass accuracy, Shots, Take-ons.
- **Side panel:** a `<select>` (`Build card from`) listing the three clips as `<title> · rating <ovr>`; changing it re-renders the card.
- **Copy caption for socials:** copies to the clipboard (`navigator.clipboard.writeText`) the text `Match rating <ovr> · <touches> touches · <pass accuracy> passing — <clip title>. Tracked by AcademyTracker, straight from phone footage.` (values looked up by stat **label**, not array position), then shows `Caption copied — paste it anywhere.` for 6s. If the clipboard is unavailable/denied, it prints the caption itself in that status area instead.
- **Save as image:** a demo stub. It produces no file; it shows `In the real product this exports a PNG for stories/group chats. (Demo only.)` for 5s.
- Static coach note: `Why this matters: "verified from footage" is the difference between this and typing your own stats into a bio. If scouts and academies trust the badge, the card becomes your CV.`

### 3.5 View 5 — Fixtures & table (`view-fixtures`)
- Label `FIXTURES & TABLE · OAKFIELD COLTS U17`; H2 `The season so far`; lede `Where the team stands, what's coming up, and every result that got you here. Demo data — hook this up to your league's real fixture feed later.`
- Two columns (`1fr 1fr`, one column ≤720px): **Upcoming fixtures** — one card per fixture (mono date, `vs <opponent>`, venue/kick-off line). **Past results** — most-recent first (the data array is oldest-first and is reversed for display); each card shows `vs <opponent>`, `Home`/`Away` (from `H`/`A`), the score as `gf–ga` (en dash, Archivo Black) and a round result badge: **W** green (`--turf`), **D** amber-deep, **L** red (`--loss`), white mono letter.
- **League table:** columns `# · Team · P · W · D · L · GD · Pts`, inside a bordered, horizontally scrollable wrapper (table min-width 420px). The home team's row ("Oakfield Colts U17") is highlighted (pale green `#E1EFE4`, bold green text). GD is printed with a `+` when positive. Rows are shown in the array order (already sorted by points) with position numbers 1…9.
- A coach note: `AcademyTracker spotted: your last 4 results (Albion, Athletic, Grove, Rovers) are unbeaten, and line up with the same 4-match rating climb shown in My Season — the table's finally catching up to the numbers.`
- All data is static (Appendix B.6). The table is **not** computed from the results list (see §8).

---

## 4. Data model

The prototype has no database. The "tables" below are JavaScript constants. Types are given as the values appear in the code. Appendix B contains every row verbatim.

### 4.1 `CLIPS` — analysed matches (3 rows)
The central entity. One row per analysed session; drives the match report, the player card and the clip cards.

| Field | Type | Meaning / format |
|---|---|---|
| `id` | string | `"rovers"`, `"cup"`, `"training"`; an upload uses `"custom-upload"` |
| `title` | string | Display title, e.g. `U17 League · vs Rovers` |
| `meta` | string | Card sub-line: camera setup · duration · result |
| `sub` | string | Date and venue line shown under the report title |
| `player` | object | `{num:int, name:string, pos:string (upper-case), ovr:int 0–99}` — the one tracked player, always #7 "A. Khan", RIGHT WING |
| `stats` | array of `Stat` | See below. Always six entries in the same order: Touches, Top speed (km/h), Pass accuracy, Distance (km), Shots (…), Take-ons (…) |
| `attrs` | array of `[name:string, value:int 0–99]` | Always six, in order Pace, Shooting, Passing, Dribbling, Defending, Physical |
| `heat` | `{h1:Blob[], h2:Blob[]}` | `Blob = [x, y, r]` pitch coordinates and radius; five blobs per half |
| `heatNote` | string (HTML) | Coach note for the heatmap |
| `sprints` | int[] | Sprint count per consecutive 5-minute block (14, 15 and 6 entries) |
| `sprintNote` | string (HTML) | |
| `passes` | `Pass[]` | `{x1,y1,x2,y2: number, ok: boolean}` |
| `passNote` | string (HTML) | |
| `shots` | `Shot[]` | `{x,y: origin, tx,ty: target, result: "goal"|"saved"|"off"}` |
| `shotNote` | string (HTML) | |
| `team` | `{shape:string, dots: Dot[]}` | `Dot = [x, y]` or `[x, y, true]` where `true` marks the tracked player |
| `teamNote` | string (HTML) | |

**`Stat`**: `{v: string (display value, e.g. "81%" or "28.4"), l: string (label), d: string (delta/context line)}`. Values are strings, not numbers. Two stats are **stored but never displayed**: those whose label starts `Top speed` or `Distance` (§6.3).

Key facts per clip: Rovers — rating 82, 54 touches, won 3–1; County Cup — rating 69, 38 touches, lost 1–2; Training — rating 77, 61 touches, 26-minute session.

### 4.2 `SEASON` — per-match trend series (8 rows, oldest first)
| Field | Type | Notes |
|---|---|---|
| `opp` | string | Opponent with venue, e.g. `United (A)`, `Rovers (H)` |
| `rating` | int | Match rating |
| `touches` | int | |
| `pass` | int | Pass accuracy % |
| `speed` | number | Top speed km/h — **kept in the data, never charted or shown** |

The last row (Rovers) equals the Rovers clip (82 / 54 / 81 / 28.4); the sixth (Athletic) equals the County Cup clip (69 / 38 / 72 / 26.1).

### 4.3 `METRICS` — chart metadata (4 rows keyed by metric)
`rating`, `touches`, `pass`, `speed`; each `{label:string, unit:string, note:string (HTML)}`. Units: `""`, `""`, `"%"`, `" km/h"`. Only the first three are reachable from the UI.

### 4.4 `DRILLS` — fixed match-based plan (3 rows)
`{name, why (HTML), dur}` — see Appendix B.5.

### 4.5 `DRILL_POOL` — the drill library (5 categories × 4–5 drills = 21 drills)
Object keyed by category (`cardio`, `shooting`, `passing`, `dribbling`, `defending`), each an array of:

| Field | Type | Notes |
|---|---|---|
| `name` | string | |
| `why` | string | Reason shown on the card |
| `dur` | string | `"<minutes> min · needs: …"` |
| `positions` | string[] (optional) | If present, the drill is position-specific; contains values from `POSITIONS` |

Counts: cardio 4, shooting 4, passing 4, dribbling 4, defending 5. Position-specific drills (one each): cardio→midfielder, shooting→striker, passing→midfielder, dribbling→winger, defending→goalkeeper **and** defender (two separate drills).

### 4.6 Lookup/constant tables
| Name | Value |
|---|---|
| `CATEGORIES` | `["cardio","shooting","passing","dribbling","defending"]` (order matters; it is the tie-break order) |
| `CATEGORY_LABEL` | cardio→`Cardio & fitness`, shooting→`Shooting`, passing→`Passing`, dribbling→`Dribbling`, defending→`Defending` |
| `POSITIONS` | `["goalkeeper","defender","midfielder","winger","striker"]` |
| `POSITION_LABEL` | title-cased versions of the above |
| `POSITION_PRIMARY` | goalkeeper→defending, defender→defending, midfielder→passing, winger→dribbling, striker→shooting |
| `WEIGHT` | `{1:3, 2:2, 3:1, 4:1, 5:0}` — drills allotted per rating |
| `PLAN_CAP` | `6` — maximum drills in a generated plan |
| `HIDDEN_STATS` | regex `/^(Top speed\|Distance)/` — labels never displayed |

### 4.7 Runtime state (not persisted)
| Variable | Type | Initial | Changed by |
|---|---|---|---|
| `currentClip` | CLIPS row or null | null | starting/finishing an analysis |
| `currentHalf` | `"h1"`/`"h2"` | `"h1"` | half toggle; reset to `"h1"` on each report |
| `currentMetric` | `"rating"`/`"touches"`/`"pass"` | `"rating"` | season toggle |
| `ratings` | `{category: 1–5}` | `{}` | quiz buttons |
| `selfPosition` | position string or null | null | position buttons |
| `timers` | timeout ids | `[]` | analysis animation |

### 4.8 League & fixtures tables
- **`UPCOMING`** (4 rows): `{date:string, opp:string, venue:string}` — e.g. `Sat 2 Aug`, `Kingsmead`, `Away · KO 10:00`.
- **`RESULTS`** (8 rows, oldest first): `{opp:string, venue:"H"|"A", gf:int, ga:int}`.
- **`LEAGUE`** (9 rows, pre-sorted): `{team, p, w, d, l, gd, pts: ints, us?: true}`; `us:true` marks Oakfield Colts U17.

---

## 5. Relationships between tables

All links are **by value/position in code, not by foreign keys.** A rebuild with a real database should turn them into keys (see Appendix A).

```
CLIPS (3)  ──< used by >── clip cards (1 each), player-card <select> (1 option each),
   │                         match report (currentClip), upload (copy of CLIPS[0])
   │ id/opponent
   ├── CLIPS[rovers]   ≙ SEASON[7]   (Rovers: 82 / 54 / 81 / 28.4)      match by opponent name
   ├── CLIPS[cup]      ≙ SEASON[5]   (Athletic: 69 / 38 / 72 / 26.1)    match by opponent name
   └── CLIPS[training]   no SEASON row (a training session is not a league match)

SEASON (8) ── opp ──≈── RESULTS (8)   same eight opponents, same order (name only, venue letter repeated)
RESULTS (8) ── opp ──≈── LEAGUE (9)   opponent names appear as league teams (+ home team "Oakfield Colts U17" with us:true)
SEASON.rating series ── referenced in prose by ── METRICS.rating.note, fx-note ("4-match rating climb")

METRICS (4) ── key ── SEASON field names (rating, touches, pass, speed)  1:1 by property name
DRILL_POOL (5 categories) ── key ── CATEGORIES / CATEGORY_LABEL / WEIGHT keys / POSITION_PRIMARY values
DRILL_POOL[*].positions ⊂ POSITIONS;  POSITION_PRIMARY: POSITIONS → CATEGORIES
ratings (1..5 per category) ──WEIGHT──> drill count per category ──DRILL_POOL──> plan drills (each tagged with its category)
DRILLS (3) ── standalone; text refers to Rovers/Athletic sample data (no key)
```

Cardinalities: one player (#7) → many matches (`CLIPS`, `SEASON`); one match → one set of heat/sprint/pass/shot/attribute/team records (embedded in the clip, not separate tables); one category → 4–5 drills; one drill → 0 or more positions it is specific to (always 0 or 1 position in the data; defending has two drills each with one position).

---

## 6. Business rules

### 6.1 Navigation and view rules
1. Exactly one view is visible; the first load shows "Analyse a match" step 1.
2. Switching tabs never resets the in-progress state of other views (e.g. the quiz answers and ticked drills stay; the report stays on screen). Returning to "Analyse a match" shows whichever step was last showing.
3. Opening "My season" always re-renders the chart.

### 6.2 What is labelled as simulated or sample (must stay)
1. Header pill: `CONCEPT DEMO · SIMULATED DATA`.
2. Upload step note about the simulated analysis (§3.1).
3. Sample-data label on the match report and on the season screen (§2.4).
4. Report footer note `All numbers simulated for concept validation — no real CV yet.` and the page footer.
Also: the fixtures lede says "Demo data".

### 6.3 Statistics that must never be displayed
Top speed and distance covered cannot be measured from phone footage, so the demo must not claim them.
1. The values stay in `CLIPS[*].stats` (`Top speed (km/h)`, `Distance (km)`), `SEASON[*].speed` and `METRICS.speed`.
2. A single filter, `visibleStats(stats) = stats.filter(s => !HIDDEN_STATS.test(s.l))` with `HIDDEN_STATS = /^(Top speed|Distance)/`, is applied wherever stats are rendered: the key-stats grid and the player card. Nothing else may print `s.l`/`s.v` directly.
3. The season screen has no top-speed button.
4. No on-screen copy may mention top speed, distance, km or km/h. Where original sample copy did, it was rewritten (clip-2 sprint note; the repeat-sprint drill text). The share caption looks values up by label and never includes them.
5. Rendered stat count is therefore 4 per clip (Touches, Pass accuracy, Shots, Take-ons). **Touches is the headline stat** (highlighted tile, and the badge on the jersey).

### 6.4 Match-analysis rules
1. Choosing a sample clip or uploading any file starts the same timed simulation (§3.1); no file content is inspected.
2. An upload always produces the Rovers data under an upload-specific title/subtitle.
3. The report always opens on the Heatmap tab, first half.
4. "Analyse another match" (only present on the report) returns to step 1; it does not clear the upload tile's "selected" state.

### 6.5 Self-rating plan generator — drill scoring logic

**Inputs:** `selfPosition` (one of five) and `ratings[category]` for the five categories, each an integer 1–5 (1 = weakest).

**Constants:** `WEIGHT = {1:3, 2:2, 3:1, 4:1, 5:0}`; `PLAN_CAP = 6`; `CATEGORIES` order = cardio, shooting, passing, dribbling, defending.

**Algorithm (this is exactly what the code does):**

1. **Rank categories weakest-first.** Copy `CATEGORIES` and sort ascending by `ratings[cat]`. JavaScript's sort is stable, so categories with equal ratings stay in `CATEGORIES` order (cardio before shooting before passing …).
2. **Build each category's ordered drill pool.** Take `DRILL_POOL[cat]`; place first the drills whose `positions` array includes the user's position (position-specific), then all the rest in their original array order. *(Drills specific to a different position are in the "rest" group, in their original place at the end of the array. Because at most 3 drills are ever taken per category and every pool starts with at least three position-neutral drills, other positions' drills are never reached.)*
3. **Allocate.** Walk the ranked categories. For each, `need = WEIGHT[rating]`; append drills `pool[0], pool[1], …, pool[need-1]` (index wraps with `i % pool.length`, which cannot trigger because every pool has ≥4 drills and `need` ≤ 3). Each appended item is a copy of the drill plus `cat` (the category key). **Stop immediately once the plan has 6 drills** — this can cut a category's allocation short, and later (stronger) categories then get none.
4. **All-5s fallback.** If the plan is still empty (the user rated every category 5), substitute two maintenance drills: `DRILL_POOL.cardio[0]` (tagged `cardio`) and `DRILL_POOL.passing[0]` (tagged `passing`), and set the note `You rated yourself strong (5/5) across the board — nice. Here are two light maintenance drills to stay sharp rather than nothing at all.` The note is prepended to the first drill's "why" text. Note that the fallback uses index 0 of the **raw** pools (not position-ordered).
5. **Position bonus.** Let `primary = POSITION_PRIMARY[position]`. If the plan has **fewer than 6** drills **and** no drill in the plan has `cat === primary`, find the first drill in `DRILL_POOL[primary]` whose `positions` includes the user's position; if found, append it flagged `bonus:true`. It is rendered with the prefix **`Because you play <Position>:`** (bold) before its "why" text. (This is how a player who rated their main skill 5 still gets one position-relevant drill. The bonus is applied after the fallback, so it also applies to the all-5s plan: e.g. a striker gets a third drill, the near-post/far-post one; a midfielder already has passing in the plan so gets none.)
6. **Render** the plan in the order built (weakest category's drills first, bonus drill last).

**Properties that follow:**
- Allocation produces 0–6 drills. An empty result (all 5s) becomes the 2-drill fallback; the bonus adds at most 1 and only if the length is below 6. So a plan has between 1 and 6 drills. The smallest case is a single category rated 3 or 4 with everything else rated 5, where that category is the position's primary one (the Winger example below); if it is not the primary category, the bonus lifts the plan to 2.
- A category rated 5 gets nothing (unless it is the fallback/bonus); rated 4 or 3 gets one drill; rated 2 gets two; rated 1 gets three.
- When position-specific drills exist for a category, they always come first for that position, so a Winger's dribbling plan starts with "1v1 out wide · beat your full-back"; a Goalkeeper's defending plan starts with "Shot-stopping reactions · reflex saves"; a Defender's with "Last-ditch tackle timing".
- Position affects ordering within a category and the bonus drill only; it does not change how many drills a category gets.

**Worked examples** (Appendix B.5 has the drill names)
- Midfielder; cardio 1, shooting 3, passing 2, dribbling 5, defending 4. Ranked: cardio(1), passing(2), shooting(3), defending(4), dribbling(5). Cardio: 3 drills → *Box-to-box shuttle circuit* (midfielder-specific, first), *Repeat sprint blocks*, *Shuttle runs*. Passing: 2 → *Progressive line-breaking passes* (specific), *One-touch rondo*. Shooting: 1 → *Weak-side finishing*. That is 3 + 2 + 1 = **6** drills, so the cap is reached and the loop stops: defending (which would have added 1) is cut and dribbling gets 0. The bonus step is skipped because the plan is already at 6. Final plan = those six drills in that order.
- Striker; all ratings 5 → fallback: *Repeat sprint blocks*, *One-touch rondo* (note shown). Primary category shooting is absent and length < 6 → bonus *Near-post far-post finishing* appended with "Because you play Striker:". Final plan length 3.
- Winger; dribbling 4, all others 5. Ranked: dribbling(4) first (the only non-5), giving 1 drill — *1v1 out wide · beat your full-back*; no bonus (dribbling already in plan). Plan length 1.

### 6.6 Match-based plan
Fixed three drills; progress text `N of 3 drills done this week` with ` — full house` at 3 (no emoji). No scoring.

### 6.7 Season chart rules
- Y-scale is auto-fitted per metric (padding 25% of the range) so small changes are visible; it does **not** start at zero.
- "Change since match 1" is `last − first` with one-decimal rounding and an explicit `+` for positive.

### 6.8 Share-card rules
- Stat rows = visible stats, first six, label stripped of bracketed qualifiers.
- Caption values are found by stat label (`Touches`, `Pass accuracy`) — never by index — so removing or reordering other stats cannot break it.
- Card contents come from the *selected* clip only.

### 6.9 Result badges
`gf > ga` → W (green); `gf = ga` → D (amber-deep); `gf < ga` → L (red).

### 6.10 Accessibility rules
- Tabs/toggles use `aria-selected` / `aria-pressed` / `aria-current`; the report tabs have `role="tablist"/"tab"/"tabpanel"`.
- Charts are SVGs with `role="img"` and aria-labels; sprint bars and season dots are keyboard-focusable.
- Focus ring: 3px amber outline, offset 2px, on every `:focus-visible`.
- Upload input is visually hidden but present; the tile is a `<label>`, so it is operable by keyboard and tap.
- `prefers-reduced-motion: reduce` disables all animation/transition, makes log lines appear immediately and shortens the simulation to under a second.

---

## 7. Design system

### 7.1 Colour tokens (CSS custom properties on `:root`)
| Token | Hex | Used for |
|---|---|---|
| `--turf` | `#2E7D4B` | Pitch fill, thumbnails, positive deltas, chart line/bars, W badge, checkbox accent |
| `--turf-deep` | `#1F5A36` | Header, jersey, upload tile, active tab/nav text, buttons, headline numbers |
| `--turf-line` | `#3C935D` | Declared; pitch line colour in practice (hard-coded `#3C935D` in the SVG) |
| `--chalk` | `#F7F5EC` | Text on dark green, pass/shot lines, teammates' dots |
| `--amber` | `#F2B705` | Accent: logo "Tracker", CTA, active toggles, focus ring, "me" dot, goal lines, missed passes |
| `--amber-deep` | `#C79400` | CTA border/hover, headline stat, draw badge, "why" highlight |
| `--loss` | `#B23A2E` | L badge |
| `--ink` | `#15251C` | Body text, ball dot, text on amber |
| `--paper` | `#EDEAE0` | Page background |
| `--card` | `#FFFFFF` | Cards, nav, table |
| `--muted` | `#5E6E64` | Secondary text |

Other literal colours used: borders `#D8D4C6` (cards/controls) and `#E4E0D2` (hairlines, bar tracks, notes panel); light text on dark `#CFE3D6`; cream coach-note/sample-label fill `#FBF3D6`; pale green `#E1EFE4` (category chip, "us" table row); upload-tile hover `#254E37`; card gradient end `#0F2C1C`; record dot `#FF5A47`; heat-map orange `#E0641E`.

### 7.2 Typography
| Token | Stack | Use |
|---|---|---|
| `--display` | `'Archivo Black','Arial Black',sans-serif` | Logo, H1/H2/H3, big numbers (ratings, shirt number, scores) |
| `--body` | `'Archivo',-apple-system,'Segoe UI',sans-serif` | Body, buttons, nav (weights 400/500/600/700 loaded) |
| `--mono` | `ui-monospace,'SF Mono',Menlo,Consolas,monospace` | Labels, steps, deltas, pills, timings, table headers |

Scale: H1 `clamp(1.5rem,4vw,2.1rem)`; H2 `clamp(1.15rem,3vw,1.5rem)` uppercase; panel H3 1rem uppercase; body 1rem / line-height 1.5, anti-aliased; small text 0.75–0.9rem; mono labels 0.62–0.78rem with 0.05–0.14em letter-spacing, usually uppercase.

### 7.3 Layout and spacing
- Content width: max 1040px, centred, 20px side gutters on header, nav and main.
- Radius: `--radius` 14px for cards/panels; 10–12px for small cards/notes/buttons; 999px for pills and chips; 18px for the player card.
- Borders: 1.5px solid `#D8D4C6` is the standard card outline.
- Spacing: grid gaps 12px (stats), 14–16px (drills, clips), 20–24px (columns).
- Reset: `* {box-sizing:border-box; margin:0; padding:0}`; `html {scroll-behavior:smooth}`.
- Breakpoints: **720px** (fixtures grid → one column), **640px** (share layout → one column), **560px** (jersey full width; demo pill left-aligned). Other responsiveness comes from `auto-fit/minmax` grids, `flex-wrap`, and a horizontally scrolling nav/table.
- Mobile requirements verified: no horizontal page overflow at 390px on any screen; the CTA stays visible; the sample label sits under the nav.

### 7.4 Components (all styling in Appendix B.1)
Header band · logo · demo pill · sticky nav + tabs + CTA · step label · H2 + lede · clip card · upload tile · "or" divider · phone note · processing panel (scanbox, scan line, progress bar, log) · jersey card · stat tile (+ headline variant) · pill tabs · panel · half/metric/plan toggle (small rounded-rectangle buttons; amber when pressed) · pitch box · legend · attribute bar · coach note · restart button · sample label · season chart card + tiles · rating form (round 32px number buttons, pill position buttons; pressed = dark green) · drill card (+ category chip, + `done` state) · player card · select · secondary button (`.btn2`: white, green border, fills green on hover; disabled 45%) · copy-status line · fixture card · result badge · league table.

### 7.5 Imagery and motion
- No raster images. Everything is inline SVG or CSS. Pitch: 100×64 units, four faint vertical mowing stripes (white 3% alpha, 12.5 units wide at x = 0, 25, 50, 75), an inset 1.5 border, halfway line, centre circle r=7.5, penalty areas 13×28, six-yard boxes 5×13, line colour `#3C935D`, stroke 0.7.
- Motion: card hover lift (0.15s), progress bar (0.5s), attribute bars (0.8s), scan line (2.6s alternate), log lines fade in (0.4s), SMIL dot movement in the processing pitch (4–6s loops). All disabled under reduced motion.

---

## 8. Known quirks and sample-data inconsistencies (reproduce or fix deliberately)

These exist in the shipped file. A rebuild that must be "exactly the same" should keep them; a rebuild that wants to improve should decide on purpose.

1. **Toggle styling bleed:** the half-toggle handler sets `aria-pressed` on *every* `.half-toggle button` on the page, so clicking 1st/2nd half un-presses the season-metric and plan-source buttons' visual state (their behaviour is unaffected, only their pressed styling). Scope the selector to the heatmap group to fix.
2. **Stale half button:** a new report resets `currentHalf` to `"h1"` but does not reset which half button looks pressed.
3. **Randomised thumbnails:** the dots on sample-clip thumbnails change on every page load.
4. **Result/league data are independent:** the league table is hard-coded, not computed from `RESULTS`. Oakfield's row does agree with the results list (3 W, 3 D, 2 L, 12 pts, GD +2), but: the Rovers clip says "won 3–1" while the results list shows 2–1; the other teams' rows do not balance (31 wins in the table against 24 losses; goal differences sum to +14, not 0); and the `METRICS.rating` note says ratings have trended up "for 4 straight matches" while the series is 64, 70, 66, 74, 71, 69, 78, 82 (only the last two matches are consecutive rises).
5. **Dates:** sample dates (July/August) and the "WEEK OF 20 JUL" label are static text.
6. **Attribute names** "Pace" and "Physical" remain even though speed/distance are hidden; they are 0–99 scores, not measurements.
7. **Upload tile** is not reset on "Analyse another match", and re-selecting the identical file may not fire the browser `change` event.
8. **Save as image** is a placeholder.
9. **Persistence:** none — every refresh resets ticks, ratings and position.

---

## 9. Rebuild checklist and acceptance tests

Build order that works: (1) tokens + reset + fonts; (2) header, nav (with CTA), footer; (3) pitch SVG helpers; (4) `CLIPS`/data constants (copy Appendix B verbatim); (5) step 1 with upload + clip cards; (6) processing animation; (7) report with six panels; (8) season view; (9) training view (fixed plan, then quiz); (10) player card; (11) fixtures & table; (12) responsive pass; (13) host `index.html` at the root of the GitHub Pages branch.

Acceptance tests:
1. Page title is `AcademyTracker — Concept MVP`; the text "Turtle" appears nowhere; the name is always one word, "AcademyTracker".
2. No on-screen text or accessible label contains "top speed", "distance", "km" or "km/h", on any screen or report tab.
3. Choosing each sample clip (or uploading any file) shows the 6-line log and the report after ≈5.8 s; the report shows four stat tiles with Touches highlighted.
4. All six report tabs render; half toggle changes the heat map; the Attributes bars animate each time the tab opens.
5. Season: three metric buttons; the chart has 8 points, last point amber; tiles show Latest/Change/Best; sample label present.
6. Quiz: Generate is disabled until position + 5 ratings are set; for every one of the 5 positions × 5 rating levels the plan follows §6.5 (verify at least: all-1s → 6 drills, cardio ×3 then shooting ×3, no bonus; all-5s → the two fallback drills (cardio, passing) plus the position bonus for every position except Midfielder, whose primary category (passing) is already in the plan; ties keep category order; cap 6). The three worked examples in §6.5 plus "Goalkeeper, shooting 3, rest 5 → Weak-side finishing then Shot-stopping reactions (bonus)" were run against the shipped code and match.
7. Player card: 4 stat rows; select switches clips; copy-caption puts the exact sentence in §3.4 on the clipboard; save shows the demo message.
8. Fixtures: 4 upcoming, 8 results (newest first, W/D/L badges), 9-row table with the home team highlighted.
9. Sample label is visible on report and season screens, under the nav, while scrolling; upload-step simulated note present.
10. "Try the real app" is visible in the nav on a 390px screen, points to `#` (or the supplied URL), and does not change the active tab.
11. At 390px width there is no horizontal page scrolling on any screen.

---

## Appendix A — Suggested normalised schema (not in the prototype)

This is a design suggestion for a real backend, inferred from the structures in §4; it is not implemented anywhere in the demo.

```
player(id, display_name, shirt_number, position, team_id)
team(id, name, is_home_team)
match(id, season_id, kind['league','cup','training'], title, played_on, venue, opponent_team_id,
      home_away['H','A'], goals_for, goals_against, duration_min, camera_setup, video_url)
match_stat(match_id, player_id, key['touches','pass_accuracy','shots','shots_on_target','goals',
           'take_ons','take_ons_won', ...], value_num, delta_text)
        -- speed/distance are not measurable from phone footage and are not stored
match_attribute(match_id, player_id, name['Pace','Shooting','Passing','Dribbling','Defending','Physical'], score 0..99)
heat_blob(match_id, player_id, half 1|2, seq, x, y, r)
sprint_block(match_id, player_id, block_index, count)          -- 5-minute blocks
pass_event(match_id, player_id, x1, y1, x2, y2, completed)
shot_event(match_id, player_id, x, y, target_x, target_y, result['goal','saved','off'])
team_shape(match_id, formation); team_shape_dot(match_id, seq, x, y, is_player)
insight(match_id, panel['heat','sprint','pass','shot','team'], text)
season_point(season_id, player_id, match_id, rating, touches, pass_accuracy)
league_row(season_id, team_id, played, won, drawn, lost, goal_diff, points)       -- derive from match
fixture(id, season_id, team_id, opponent_team_id, kickoff_at, venue, home_away)
drill(id, category['cardio','shooting','passing','dribbling','defending'], name, why, duration_min, equipment, position[nullable])
self_rating(id, player_id, position, cardio, shooting, passing, dribbling, defending, created_at)
plan(id, player_id, source['match','self'], created_at); plan_drill(plan_id, seq, drill_id, is_bonus, done)
config(weight_by_rating {1:3,2:2,3:1,4:1,5:0}, plan_cap 6, position_primary map)
```

Notes: `league_row` should be computed from `match` rather than stored if consistency is wanted (see §8.4). `drill.position` would need to be many-to-many if one drill should serve several positions.

---

## Appendix B — Verbatim source

The following blocks are copied directly from the shipped `index.html`.

### B.1 Complete stylesheet

```css
  :root{
    --turf:#2E7D4B;
    --turf-deep:#1F5A36;
    --turf-line:#3C935D;
    --chalk:#F7F5EC;
    --amber:#F2B705;
    --amber-deep:#C79400;
    --loss:#B23A2E;
    --ink:#15251C;
    --paper:#EDEAE0;
    --card:#FFFFFF;
    --muted:#5E6E64;
    --radius:14px;
    --display:'Archivo Black','Arial Black',sans-serif;
    --body:'Archivo',-apple-system,'Segoe UI',sans-serif;
    --mono:ui-monospace,'SF Mono',Menlo,Consolas,monospace;
  }
  *{box-sizing:border-box;margin:0;padding:0}
  html{scroll-behavior:smooth}
  body{font-family:var(--body);background:var(--paper);color:var(--ink);line-height:1.5;-webkit-font-smoothing:antialiased}
  button{font-family:inherit;cursor:pointer}
  :focus-visible{outline:3px solid var(--amber);outline-offset:2px;border-radius:4px}

  /* ---------- header ---------- */
  header{
    background:var(--turf-deep);
    color:var(--chalk);
    padding:26px 20px 22px;
    position:relative;
    overflow:hidden;
  }
  .head-inner{max-width:1040px;margin:0 auto;position:relative;display:flex;align-items:center;gap:16px;flex-wrap:wrap}
  h1{font-family:var(--display);font-size:clamp(1.5rem,4vw,2.1rem);letter-spacing:.02em;line-height:1.05;text-transform:uppercase}
  h1.logo{text-transform:none;letter-spacing:0}
  h1.logo span{color:var(--amber)}
  .tag{font-size:.95rem;color:#CFE3D6;margin-top:2px}
  .demo-pill{
    margin-left:auto;font-family:var(--mono);font-size:.72rem;letter-spacing:.08em;
    border:1.5px dashed var(--amber);color:var(--amber);padding:6px 12px;border-radius:999px;white-space:nowrap;
  }

  main{max-width:1040px;margin:0 auto;padding:28px 20px 60px}
  .step-label{
    font-family:var(--mono);font-size:.72rem;letter-spacing:.14em;text-transform:uppercase;
    color:var(--muted);margin-bottom:10px;
  }
  .step-label b{color:var(--turf-deep)}
  h2{font-family:var(--display);font-size:clamp(1.15rem,3vw,1.5rem);text-transform:uppercase;letter-spacing:.01em;margin-bottom:6px}
  .lede{color:var(--muted);max-width:56ch;margin-bottom:22px}

  /* ---------- clip cards ---------- */
  .clips{display:grid;grid-template-columns:repeat(auto-fit,minmax(250px,1fr));gap:16px}
  .clip{
    text-align:left;background:var(--card);border:1.5px solid #D8D4C6;border-radius:var(--radius);
    padding:0;overflow:hidden;transition:transform .15s ease,border-color .15s ease,box-shadow .15s ease;
  }
  .clip:hover{transform:translateY(-3px);border-color:var(--turf);box-shadow:0 10px 24px rgba(21,37,28,.12)}
  .thumb{height:118px;position:relative;background:linear-gradient(160deg,var(--turf) 0%,var(--turf-deep) 100%)}
  .thumb svg{position:absolute;inset:0;width:100%;height:100%}
  .rec{
    position:absolute;top:10px;left:10px;font-family:var(--mono);font-size:.68rem;color:#fff;
    background:rgba(21,37,28,.55);padding:3px 8px;border-radius:6px;display:flex;align-items:center;gap:6px;
  }
  .rec i{width:7px;height:7px;border-radius:50%;background:#FF5A47;display:inline-block}
  .clip-body{padding:14px 16px 16px}
  .clip-body h3{font-size:1rem;font-weight:700;margin-bottom:3px}
  .clip-meta{font-size:.82rem;color:var(--muted)}
  .clip-cta{
    margin-top:12px;display:inline-block;font-weight:700;font-size:.85rem;color:var(--turf-deep);
  }
  .clip-cta::after{content:" →"}

  .phone-note{
    margin-top:18px;font-size:.85rem;color:var(--muted);background:#E4E0D2;border-radius:10px;
    padding:10px 14px;display:inline-block;
  }

  /* ---------- upload tile ---------- */
  .upload-tile{
    display:flex;align-items:center;gap:16px;background:var(--turf-deep);color:var(--chalk);
    border:1.5px dashed var(--amber);border-radius:var(--radius);padding:18px 20px;cursor:pointer;
    margin-bottom:18px;transition:background .15s ease,transform .15s ease;
  }
  .upload-tile:hover{background:#254E37;transform:translateY(-2px)}
  .upload-tile input[type="file"]{
    position:absolute;width:1px;height:1px;padding:0;margin:-1px;overflow:hidden;clip:rect(0,0,0,0);border:0;
  }
  .upload-icon{
    flex:none;width:46px;height:46px;border-radius:10px;background:rgba(247,245,236,.1);
    display:grid;place-items:center;
  }
  .upload-icon svg{width:24px;height:24px}
  .upload-tile h3{font-family:var(--display);font-size:1rem;text-transform:uppercase;margin-bottom:2px}
  .upload-tile p{font-size:.85rem;color:#CFE3D6}
  .upload-tile.has-file{border-style:solid}
  .upload-tile.has-file .upload-icon{background:var(--amber)}
  .upload-tile.has-file .upload-icon svg path{stroke:var(--turf-deep)}

  .or-divider{
    display:flex;align-items:center;gap:12px;color:var(--muted);font-size:.8rem;
    font-family:var(--mono);letter-spacing:.08em;text-transform:uppercase;margin-bottom:16px;
  }
  .or-divider::before,.or-divider::after{content:"";flex:1;height:1px;background:#D8D4C6}

  /* ---------- self-rating form ---------- */
  .rating-form{background:var(--card);border:1.5px solid #D8D4C6;border-radius:var(--radius);padding:6px 18px;margin-top:6px}
  .rating-row{display:flex;align-items:center;justify-content:space-between;gap:14px;padding:14px 0;border-bottom:1px solid #E4E0D2;flex-wrap:wrap}
  .rating-row:last-child{border-bottom:none}
  .rlabel{font-weight:700;font-size:.92rem}
  .rating-scale{display:flex;gap:6px}
  .rating-scale button{
    width:32px;height:32px;border-radius:50%;border:1.5px solid #D8D4C6;background:var(--paper);
    font-family:var(--mono);font-weight:700;font-size:.8rem;color:var(--muted);transition:all .12s ease;
  }
  .rating-scale button:hover{border-color:var(--turf)}
  .rating-scale button[aria-pressed="true"]{background:var(--turf-deep);border-color:var(--turf-deep);color:var(--chalk)}
  .position-scale{flex-wrap:wrap;gap:8px}
  .position-scale button{width:auto;height:auto;border-radius:999px;padding:7px 14px;font-size:.78rem;letter-spacing:.02em}
  .rating-actions{display:flex;align-items:center;gap:14px;flex-wrap:wrap;margin-top:16px}
  .rating-hint{font-size:.82rem;color:var(--muted)}
  .btn2:disabled{opacity:.45;cursor:not-allowed}
  .drill-cat{
    display:inline-block;font-family:var(--mono);font-size:.62rem;letter-spacing:.08em;text-transform:uppercase;
    color:var(--turf-deep);background:#E1EFE4;border-radius:999px;padding:2px 9px;margin-bottom:6px;
  }

  /* ---------- fixtures & table ---------- */
  .fx-grid{display:grid;grid-template-columns:1fr 1fr;gap:20px}
  @media (max-width:720px){.fx-grid{grid-template-columns:1fr}}
  .fx-col h3{font-family:var(--display);font-size:1rem;text-transform:uppercase;margin-bottom:10px;color:var(--turf-deep)}
  .fx-card{
    display:flex;align-items:center;justify-content:space-between;gap:12px;background:var(--card);
    border:1.5px solid #D8D4C6;border-radius:12px;padding:12px 14px;margin-bottom:10px;
  }
  .fx-date{font-family:var(--mono);font-size:.68rem;letter-spacing:.05em;color:var(--muted);white-space:nowrap}
  .fx-opp{font-weight:700;font-size:.92rem}
  .fx-venue{font-size:.75rem;color:var(--muted)}
  .fx-score{font-family:var(--display);font-size:1.05rem;white-space:nowrap}
  .badge-result{
    font-family:var(--mono);font-weight:700;font-size:.72rem;width:26px;height:26px;border-radius:50%;
    display:grid;place-items:center;flex:none;color:var(--chalk);
  }
  .badge-w{background:var(--turf)}
  .badge-d{background:var(--amber-deep)}
  .badge-l{background:var(--loss)}
  .table-wrap{overflow-x:auto;border:1.5px solid #D8D4C6;border-radius:var(--radius);background:var(--card)}
  .league-table{width:100%;border-collapse:collapse;font-size:.85rem;min-width:420px}
  .league-table th{
    font-family:var(--mono);font-size:.65rem;letter-spacing:.06em;text-transform:uppercase;color:var(--muted);
    text-align:left;padding:10px 12px;border-bottom:1.5px solid #D8D4C6;
  }
  .league-table td{padding:9px 12px;border-bottom:1px solid #E4E0D2}
  .league-table tr:last-child td{border-bottom:none}
  .league-table td:first-child,.league-table th:first-child{width:28px;color:var(--muted);font-family:var(--mono)}
  .league-table tr.us{background:#E1EFE4}
  .league-table tr.us td{font-weight:700;color:var(--turf-deep)}
  .league-table td:nth-child(2){font-weight:600}

  /* ---------- processing ---------- */
  .proc-wrap{background:var(--turf-deep);border-radius:var(--radius);padding:22px;color:var(--chalk)}
  .scanbox{position:relative;border-radius:10px;overflow:hidden;background:var(--turf)}
  .scanbox svg{display:block;width:100%;height:auto}
  .scanline{
    position:absolute;top:0;bottom:0;width:3px;background:var(--amber);
    box-shadow:0 0 18px 4px rgba(242,183,5,.65);left:0;
    animation:scan 2.6s linear infinite alternate;
  }
  @keyframes scan{from{left:2%}to{left:97%}}
  .proc-log{
    font-family:var(--mono);font-size:.8rem;margin-top:16px;min-height:120px;
    color:#CFE3D6;
  }
  .proc-log div{opacity:0;animation:fadein .4s forwards}
  .proc-log .ok{color:var(--amber)}
  @keyframes fadein{to{opacity:1}}
  .bar{height:8px;background:rgba(247,245,236,.18);border-radius:99px;margin-top:14px;overflow:hidden}
  .bar i{display:block;height:100%;width:0;background:var(--amber);border-radius:99px;transition:width .5s ease}

  /* ---------- dashboard ---------- */
  .dash-head{display:flex;gap:20px;flex-wrap:wrap;align-items:stretch;margin-bottom:22px}
  .jersey{
    flex:0 0 210px;background:var(--turf-deep);border-radius:var(--radius);color:var(--chalk);
    padding:18px;position:relative;overflow:hidden;display:flex;flex-direction:column;justify-content:space-between;min-height:220px;
  }
  .jersey .num{font-family:var(--display);font-size:4.4rem;line-height:1;color:var(--amber);position:relative}
  .jersey .pname{font-family:var(--display);font-size:1.15rem;text-transform:uppercase;position:relative}
  .jersey .pos{font-family:var(--mono);font-size:.75rem;letter-spacing:.12em;color:#CFE3D6;position:relative}
  .ovr{position:absolute;top:14px;right:14px;text-align:center}
  .ovr b{font-family:var(--display);font-size:1.7rem;display:block;color:var(--chalk)}
  .ovr span{font-family:var(--mono);font-size:.62rem;letter-spacing:.14em;color:#CFE3D6}
  .touch-badge{position:absolute;top:14px;left:14px;text-align:center}
  .touch-badge b{font-family:var(--display);font-size:1.7rem;display:block;color:var(--amber)}
  .touch-badge span{font-family:var(--mono);font-size:.62rem;letter-spacing:.14em;color:#CFE3D6}

  .keystats{flex:1 1 320px;display:grid;grid-template-columns:repeat(auto-fit,minmax(130px,1fr));gap:12px}
  .stat{background:var(--card);border:1.5px solid #D8D4C6;border-radius:var(--radius);padding:14px 16px}
  .stat b{font-family:var(--display);font-size:1.5rem;display:block;color:var(--turf-deep)}
  .stat span{font-size:.78rem;color:var(--muted)}
  .stat .delta{font-family:var(--mono);font-size:.7rem;color:var(--turf);display:block;margin-top:3px}
  .stat-headline{border-color:var(--amber-deep);box-shadow:inset 0 0 0 1px var(--amber)}
  .stat-headline b{color:var(--amber-deep)}

  /* tabs */
  .tabs{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:16px}
  .tab{
    border:1.5px solid #D8D4C6;background:var(--card);border-radius:999px;padding:8px 18px;
    font-weight:600;font-size:.88rem;color:var(--muted);
  }
  .tab[aria-selected="true"]{background:var(--turf-deep);border-color:var(--turf-deep);color:var(--chalk)}
  .panel{background:var(--card);border:1.5px solid #D8D4C6;border-radius:var(--radius);padding:20px}
  .panel h3{font-family:var(--display);font-size:1rem;text-transform:uppercase;margin-bottom:4px}
  .panel .sub{font-size:.85rem;color:var(--muted);margin-bottom:16px}
  .panel[hidden]{display:none}

  .pitchbox{background:var(--turf);border-radius:10px;overflow:hidden}
  .pitchbox svg{display:block;width:100%;height:auto}

  .half-toggle{display:flex;gap:8px;margin-bottom:12px}
  .half-toggle button{
    border:1.5px solid #D8D4C6;background:var(--paper);border-radius:8px;padding:6px 14px;font-size:.82rem;font-weight:600;color:var(--muted);
  }
  .half-toggle button[aria-pressed="true"]{background:var(--amber);border-color:var(--amber-deep);color:var(--ink)}

  .legend{display:flex;gap:18px;flex-wrap:wrap;margin-top:12px;font-size:.8rem;color:var(--muted)}
  .legend i{display:inline-block;width:22px;height:0;border-top:3px solid var(--chalk);vertical-align:middle;margin-right:6px}
  .legend .miss i{border-top-style:dashed;border-top-color:var(--amber)}

  .radar-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(230px,1fr));gap:20px;align-items:center}
  .attr{margin-bottom:12px}
  .attr .row{display:flex;justify-content:space-between;font-size:.82rem;font-weight:600;margin-bottom:4px}
  .attr .row span:last-child{font-family:var(--mono)}
  .attr .track{height:9px;background:#E4E0D2;border-radius:99px;overflow:hidden}
  .attr .track i{display:block;height:100%;background:var(--turf);border-radius:99px;width:0;transition:width .8s ease}

  .coach-note{
    margin-top:16px;background:#FBF3D6;border:1.5px solid var(--amber);border-radius:10px;padding:12px 16px;font-size:.88rem;
  }
  .coach-note b{color:var(--amber-deep)}

  .dash-foot{display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:12px;margin-top:22px}
  .restart{
    background:var(--turf-deep);color:var(--chalk);border:none;border-radius:10px;padding:11px 20px;font-weight:700;font-size:.9rem;
  }
  .restart:hover{background:var(--turf)}
  .sim-note{font-family:var(--mono);font-size:.72rem;color:var(--muted);letter-spacing:.05em}

  footer{max-width:1040px;margin:0 auto;padding:0 20px 40px;font-size:.8rem;color:var(--muted)}

  /* ---------- app nav ---------- */
  .appnav{background:var(--card);border-bottom:1.5px solid #D8D4C6;position:sticky;top:0;z-index:5}
  .appnav-inner{max-width:1040px;margin:0 auto;padding:0 20px;display:flex;gap:2px;overflow-x:auto}
  .navbtn{border:none;background:none;padding:14px 16px 11px;font-weight:700;font-size:.9rem;color:var(--muted);border-bottom:3px solid transparent;white-space:nowrap}
  .navbtn[aria-current="true"]{color:var(--turf-deep);border-bottom-color:var(--amber)}
  .navbtn:hover{color:var(--turf-deep)}
  /* "Try the real app" button: stays pinned to the right edge so it is still reachable when the tabs scroll on a phone */
  .nav-cta-wrap{margin-left:auto;flex:none;position:sticky;right:0;display:flex;align-items:center;padding-left:12px;background:var(--card);box-shadow:-8px 0 8px -6px rgba(21,37,28,.18)}
  .nav-cta{background:var(--amber);color:var(--ink);font-weight:700;font-size:.82rem;text-decoration:none;white-space:nowrap;padding:7px 14px;border-radius:999px;border:1.5px solid var(--amber-deep)}
  .nav-cta:hover{background:var(--amber-deep)}
  /* Sample-data label: always on screen on the match report and season screens (sticks just under the nav while scrolling) */
  .sample-label{position:sticky;top:47px;z-index:4;display:block;width:fit-content;max-width:100%;margin:0 0 14px;padding:5px 12px;border:1.5px dashed var(--amber-deep);border-radius:10px;background:#FBF3D6;color:var(--ink);font-family:var(--mono);font-size:.7rem;letter-spacing:.04em;line-height:1.4}

  /* ---------- season ---------- */
  .season-chart{background:var(--card);border:1.5px solid #D8D4C6;border-radius:var(--radius);padding:18px}
  .season-tiles{display:grid;grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:12px;margin-top:16px}

  /* ---------- training ---------- */
  .week-head{display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px;margin-bottom:14px}
  .week-prog{font-family:var(--mono);font-size:.78rem;color:var(--muted)}
  .drills{display:grid;gap:14px}
  .drill{display:flex;gap:14px;align-items:flex-start;background:var(--card);border:1.5px solid #D8D4C6;border-radius:var(--radius);padding:16px 18px}
  .drill input{width:22px;height:22px;accent-color:var(--turf);margin-top:2px;flex:none;cursor:pointer}
  .drill h4{font-size:.98rem;margin-bottom:2px}
  .drill .why{font-size:.82rem;color:var(--muted)}
  .drill .why b{color:var(--amber-deep)}
  .drill .dur{font-family:var(--mono);font-size:.72rem;color:var(--turf-deep);margin-top:5px;display:block}
  .drill.done{border-color:var(--turf)}
  .drill.done h4{text-decoration:line-through;color:var(--muted)}

  /* ---------- share card ---------- */
  .share-wrap{display:grid;grid-template-columns:minmax(250px,330px) 1fr;gap:24px;align-items:start}
  @media (max-width:640px){.share-wrap{grid-template-columns:1fr}}
  .pcard{background:linear-gradient(165deg,var(--turf-deep) 0%,#0F2C1C 100%);border:3px solid var(--amber);border-radius:18px;color:var(--chalk);padding:20px;position:relative;overflow:hidden}
  .pcard-top{display:flex;justify-content:space-between;align-items:flex-start;position:relative}
  .pcard .big-ovr{font-family:var(--display);font-size:2.9rem;line-height:1;color:var(--amber)}
  .pcard .ovr-lbl{font-family:var(--mono);font-size:.6rem;letter-spacing:.14em;color:#CFE3D6}
  .pcard .cnum{font-family:var(--display);font-size:2.9rem;line-height:1;opacity:.35}
  .pcard .cname{font-family:var(--display);font-size:1.3rem;text-transform:uppercase;margin-top:26px;position:relative}
  .pcard .cmatch{font-family:var(--mono);font-size:.7rem;color:#CFE3D6;margin-bottom:14px;position:relative}
  .pcard-stats{display:grid;grid-template-columns:1fr 1fr;gap:8px 18px;position:relative}
  .pcard-stats div{display:flex;justify-content:space-between;font-size:.82rem;border-bottom:1px solid rgba(247,245,236,.15);padding-bottom:4px}
  .pcard-stats b{font-family:var(--mono)}
  .pcard-foot{margin-top:16px;font-family:var(--mono);font-size:.62rem;letter-spacing:.12em;color:var(--amber);position:relative}
  .share-side label{font-weight:700;font-size:.88rem;display:block;margin-bottom:6px}
  .share-side select{font-family:inherit;font-size:.9rem;padding:9px 12px;border-radius:10px;border:1.5px solid #D8D4C6;background:var(--card);width:100%;max-width:340px;margin-bottom:16px}
  .btnrow{display:flex;gap:10px;flex-wrap:wrap}
  .btn2{background:var(--card);border:1.5px solid var(--turf-deep);color:var(--turf-deep);border-radius:10px;padding:10px 18px;font-weight:700;font-size:.88rem}
  .btn2:hover{background:var(--turf-deep);color:var(--chalk)}
  .copied{font-family:var(--mono);font-size:.75rem;color:var(--turf);margin-top:8px;min-height:1.2em}

  @media (prefers-reduced-motion:reduce){
    *{animation:none!important;transition:none!important}
    .proc-log div{opacity:1}
  }
  @media (max-width:560px){
    .jersey{flex-basis:100%}
    .demo-pill{margin-left:0}
  }
```

### B.2 Document skeleton (HTML)

```html
<body>

<header>
  <div class="head-inner">
    <div>
      <h1 class="logo">Academy<span>Tracker</span></h1>
      <p class="tag">Match insights from any phone camera. No special kit.</p>
    </div>
    <span class="demo-pill">CONCEPT DEMO · SIMULATED DATA</span>
  </div>
</header>

<nav class="appnav" aria-label="App sections">
  <div class="appnav-inner">
    <button class="navbtn" data-view="view-match" aria-current="true">⚽ Analyse a match</button>
    <button class="navbtn" data-view="view-season" aria-current="false">📈 My season</button>
    <button class="navbtn" data-view="view-train" aria-current="false">🏋️ Training plan</button>
    <button class="navbtn" data-view="view-card" aria-current="false">🃏 Player card</button>
    <button class="navbtn" data-view="view-fixtures" aria-current="false">📅 Fixtures & table</button>
    <div class="nav-cta-wrap">
      <!-- TRY THE REAL APP: replace href="#" on the next line with the real app URL -->
      <a class="nav-cta" href="#">Try the real app</a>
    </div>
  </div>
</nav>

<main>

<div id="view-match">

  <!-- STEP 1 -->
  <section id="step-upload" aria-label="Choose footage">
    <p class="step-label"><b>STEP 1 / 3</b> · FOOTAGE</p>
    <h2>Pick a match recording</h2>
    <p class="lede">Film from the sideline with a normal phone. AcademyTracker finds the players, tracks the ball, and turns 60 shaky minutes into numbers you can act on.</p>

    <label class="upload-tile" for="video-upload">
      <input type="file" id="video-upload" accept="video/*" aria-describedby="upload-hint">
      <div class="upload-icon" aria-hidden="true">
        <svg viewBox="0 0 24 24" fill="none"><path d="M12 16V4M12 4 7 9M12 4l5 5" stroke="#F2B705" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/><path d="M4 16v3a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2v-3" stroke="#F7F5EC" stroke-width="1.8" stroke-linecap="round"/></svg>
      </div>
      <div>
        <h3>Upload your own match footage</h3>
        <p id="upload-hint">Any phone video works — tap to choose a file from your camera roll.</p>
      </div>
    </label>

    <p class="or-divider"><span>or try a sample match</span></p>

    <div class="clips" id="clip-list"></div>
    <p class="phone-note">📱 The upload button above accepts a real video file — but since the CV pipeline isn't built yet, it runs the same simulated analysis as the samples below.</p>
  </section>

  <!-- STEP 2 -->
  <section id="step-processing" hidden aria-label="Analysing footage" aria-live="polite">
    <p class="step-label"><b>STEP 2 / 3</b> · ANALYSIS</p>
    <h2>AcademyTracker is watching the match…</h2>
    <p class="lede">Slow and steady: every frame gets scanned, every player gets an ID, every touch gets logged.</p>
    <div class="proc-wrap">
      <div class="scanbox" id="scanbox"></div>
      <div class="bar" aria-hidden="true"><i id="proc-bar"></i></div>
      <div class="proc-log" id="proc-log"></div>
    </div>
  </section>

  <!-- STEP 3 -->
  <section id="dashboard" hidden aria-label="Match insights">
    <p class="step-label"><b>STEP 3 / 3</b> · INSIGHTS</p>
    <h2 id="dash-title">Your match report</h2>
    <p class="lede" id="dash-sub"></p>
    <p class="sample-label" role="note"><b>Sample data</b> — figures shown are for demonstration only.</p>

    <div class="dash-head">
      <div class="jersey">
        <div class="ovr"><b id="p-ovr"></b><span>MATCH RATING</span></div>
        <div class="touch-badge"><b id="p-touches"></b><span>TOUCHES</span></div>
        <div class="num" id="p-num"></div>
        <div>
          <div class="pname" id="p-name"></div>
          <div class="pos" id="p-pos"></div>
        </div>
      </div>
      <div class="keystats" id="keystats"></div>
    </div>

    <div class="tabs" role="tablist" aria-label="Insight views">
      <button class="tab" role="tab" data-panel="panel-heat" aria-selected="true">Heatmap</button>
      <button class="tab" role="tab" data-panel="panel-sprint" aria-selected="false">Sprints</button>
      <button class="tab" role="tab" data-panel="panel-pass" aria-selected="false">Passing</button>
      <button class="tab" role="tab" data-panel="panel-shot" aria-selected="false">Shot map</button>
      <button class="tab" role="tab" data-panel="panel-attr" aria-selected="false">Attributes</button>
      <button class="tab" role="tab" data-panel="panel-team" aria-selected="false">Team shape</button>
    </div>

    <div class="panel" id="panel-heat" role="tabpanel">
      <h3>Where you played</h3>
      <p class="sub">Every second on the ball and off it, mapped onto the pitch. Attacking left → right.</p>
      <div class="half-toggle" role="group" aria-label="Choose half">
        <button id="h1-btn" aria-pressed="true">1st half</button>
        <button id="h2-btn" aria-pressed="false">2nd half</button>
      </div>
      <div class="pitchbox" id="heat-pitch"></div>
      <div class="coach-note" id="heat-note"></div>
    </div>

    <div class="panel" id="panel-sprint" role="tabpanel" hidden>
      <h3>Sprint map</h3>
      <p class="sub">High-intensity runs per 5-minute block. Hover or tap a bar for detail.</p>
      <div id="sprint-chart"></div>
      <div class="coach-note" id="sprint-note"></div>
    </div>

    <div class="panel" id="panel-pass" role="tabpanel" hidden>
      <h3>Pass map</h3>
      <p class="sub">Completed passes in chalk, missed in amber. Attacking left → right.</p>
      <div class="pitchbox" id="pass-pitch"></div>
      <div class="legend"><span><i></i>Completed</span><span class="miss"><i></i>Missed</span></div>
      <div class="coach-note" id="pass-note"></div>
    </div>

    <div class="panel" id="panel-shot" role="tabpanel" hidden>
      <h3>Shot map</h3>
      <p class="sub">Every shot, where it came from and how it ended. Attacking left → right.</p>
      <div class="pitchbox" id="shot-pitch"></div>
      <div class="legend"><span><i style="border-top-color:var(--amber)"></i>Goal</span><span><i></i>On target</span><span class="miss"><i></i>Off target</span></div>
      <div class="coach-note" id="shot-note"></div>
    </div>

    <div class="panel" id="panel-attr" role="tabpanel" hidden>
      <h3>Match attributes</h3>
      <p class="sub">Your performance this match, scored 0–99 against players in your age group. Not a permanent rating — it moves every game.</p>
      <div class="radar-grid">
        <div id="attr-bars"></div>
        <div class="coach-note" id="attr-note" style="margin-top:0"></div>
      </div>
    </div>

    <div class="panel" id="panel-team" role="tabpanel" hidden>
      <h3>Team shape</h3>
      <p class="sub">Average positions across the match. You are the amber dot.</p>
      <div class="pitchbox" id="team-pitch"></div>
      <div class="coach-note" id="team-note"></div>
    </div>

    <div class="dash-foot">
      <button class="restart" id="restart">← Analyse another match</button>
      <span class="sim-note">All numbers simulated for concept validation — no real CV yet.</span>
    </div>
  </section>

</div><!-- /view-match -->

<!-- SEASON VIEW -->
<section id="view-season" hidden aria-label="Season trends">
  <p class="step-label"><b>MY SEASON</b> · 8 MATCHES ANALYSED</p>
  <h2>Are you actually improving?</h2>
  <p class="lede">One match is noise. A season is a signal. AcademyTracker tracks every analysed match so you can see the trend, not just the highlight.</p>
  <p class="sample-label" role="note"><b>Sample data</b> — figures shown are for demonstration only.</p>
  <div class="half-toggle" role="group" aria-label="Choose metric" id="metric-toggle">
    <button data-metric="rating" aria-pressed="true">Match rating</button>
    <button data-metric="touches" aria-pressed="false">Touches</button>
    <button data-metric="pass" aria-pressed="false">Pass %</button>
  </div>
  <div class="season-chart" id="season-chart"></div>
  <div class="season-tiles" id="season-tiles"></div>
  <div class="coach-note" id="season-note"></div>
</section>

<!-- TRAINING VIEW -->
<section id="view-train" hidden aria-label="Training plan">
  <p class="step-label"><b>TRAINING PLAN</b> · WEEK OF 20 JUL</p>
  <h2>This week's homework</h2>

  <div class="half-toggle" role="group" aria-label="Choose plan source" id="plan-source-toggle">
    <button data-source="match" aria-pressed="true">From my match data</button>
    <button data-source="self" aria-pressed="false">From self-rating</button>
  </div>

  <div id="plan-match">
    <p class="lede">AcademyTracker picks three drills from your weakest match data — not a generic programme. Tick them off as you go; next match shows whether they worked.</p>
    <div class="week-head">
      <span class="week-prog" id="week-prog">0 of 3 drills done this week</span>
    </div>
    <div class="drills" id="drill-list"></div>
  </div>

  <div id="plan-self" hidden>
    <p class="lede">No footage yet? Rate yourself 1–5 in each area and AcademyTracker builds a plan on the spot — more drills where you're weakest, fewer where you're already strong.</p>
    <div class="rating-form" id="rating-form"></div>
    <div class="rating-actions">
      <button class="btn2" id="generate-plan" disabled>Generate my plan</button>
      <span class="rating-hint" id="rating-hint">Rate all 5 areas and pick a position to generate your plan</span>
    </div>
    <div id="self-plan-output" hidden>
      <div class="week-head" style="margin-top:20px">
        <span class="week-prog" id="self-week-prog"></span>
      </div>
      <div class="drills" id="self-drill-list"></div>
    </div>
  </div>

  <div class="coach-note" style="margin-top:16px"><b>Honesty check:</b> drills only count if next match's numbers move. AcademyTracker will compare your take-on success and shot placement before vs after this training block.</div>
</section>

<!-- CARD VIEW -->
<section id="view-card" hidden aria-label="Shareable player card">
  <p class="step-label"><b>PLAYER CARD</b> · SHARE YOUR MATCH</p>
  <h2>Your match, one card</h2>
  <p class="lede">A stats card built from real match data — made for the group chat, not just the coach's laptop.</p>
  <div class="share-wrap">
    <div class="pcard" id="pcard" role="img" aria-label="Player stats card">
      <div class="pcard-top">
        <div><span class="big-ovr" id="c-ovr"></span><br><span class="ovr-lbl">MATCH RATING</span></div>
        <div class="cnum" id="c-num"></div>
      </div>
      <div class="cname" id="c-name"></div>
      <div class="cmatch" id="c-match"></div>
      <div class="pcard-stats" id="c-stats"></div>
      <div class="pcard-foot">ACADEMYTRACKER · VERIFIED FROM FOOTAGE</div>
    </div>
    <div class="share-side">
      <label for="card-match">Build card from</label>
      <select id="card-match"></select>
      <div class="btnrow">
        <button class="btn2" id="copy-caption">Copy caption for socials</button>
        <button class="btn2" id="save-card">Save as image</button>
      </div>
      <p class="copied" id="copy-status" aria-live="polite"></p>
      <div class="coach-note" style="margin-top:14px"><b>Why this matters:</b> "verified from footage" is the difference between this and typing your own stats into a bio. If scouts and academies trust the badge, the card becomes your CV.</div>
    </div>
  </div>
</section>

<!-- FIXTURES & TABLE VIEW -->
<section id="view-fixtures" hidden aria-label="Fixtures and league table">
  <p class="step-label"><b>FIXTURES & TABLE</b> · OAKFIELD COLTS U17</p>
  <h2>The season so far</h2>
  <p class="lede">Where the team stands, what's coming up, and every result that got you here. Demo data — hook this up to your league's real fixture feed later.</p>

  <div class="fx-grid">
    <div class="fx-col">
      <h3>Upcoming fixtures</h3>
      <div id="fx-upcoming"></div>
    </div>
    <div class="fx-col">
      <h3>Past results</h3>
      <div id="fx-results"></div>
    </div>
  </div>

  <h3 style="margin-top:26px">League table</h3>
  <div class="table-wrap">
    <table class="league-table" id="league-table">
      <thead>
        <tr><th>#</th><th>Team</th><th>P</th><th>W</th><th>D</th><th>L</th><th>GD</th><th>Pts</th></tr>
      </thead>
      <tbody id="league-body"></tbody>
    </table>
  </div>
  <div class="coach-note" style="margin-top:16px" id="fx-note"></div>
</section>

</main>

<footer>
  AcademyTracker · concept MVP for the Imperial EDGE accelerator. This demo shows the intended experience; the computer-vision pipeline is not built yet.
</footer>
```

### B.3 Pitch helpers (shared SVG)

```js
function pitchMarkings(){
  const c="#3C935D";
  return `
    <rect x="0" y="0" width="100" height="64" fill="var(--turf)"/>
    <rect x="0" y="0" width="12.5" height="64" fill="rgba(255,255,255,0.03)"/>
    <rect x="25" y="0" width="12.5" height="64" fill="rgba(255,255,255,0.03)"/>
    <rect x="50" y="0" width="12.5" height="64" fill="rgba(255,255,255,0.03)"/>
    <rect x="75" y="0" width="12.5" height="64" fill="rgba(255,255,255,0.03)"/>
    <rect x="1.5" y="1.5" width="97" height="61" fill="none" stroke="${c}" stroke-width="0.7"/>
    <line x1="50" y1="1.5" x2="50" y2="62.5" stroke="${c}" stroke-width="0.7"/>
    <circle cx="50" cy="32" r="7.5" fill="none" stroke="${c}" stroke-width="0.7"/>
    <rect x="1.5" y="18" width="13" height="28" fill="none" stroke="${c}" stroke-width="0.7"/>
    <rect x="85.5" y="18" width="13" height="28" fill="none" stroke="${c}" stroke-width="0.7"/>
    <rect x="1.5" y="25.5" width="5" height="13" fill="none" stroke="${c}" stroke-width="0.7"/>
    <rect x="93.5" y="25.5" width="5" height="13" fill="none" stroke="${c}" stroke-width="0.7"/>`;
}
function pitchSVG(inner,label){
  return `<svg viewBox="0 0 100 64" role="img" aria-label="${label}" preserveAspectRatio="xMidYMid meet">
    ${pitchMarkings()}${inner}</svg>`;
}
```

### B.4 Match data, season data and metric notes

```js
/* Top speed and distance cannot be measured from phone footage, so they are never displayed.
   The values stay in the data below; this filter only controls what is shown. */
const HIDDEN_STATS=/^(Top speed|Distance)/;
const visibleStats=stats=>stats.filter(s=>!HIDDEN_STATS.test(s.l));

const CLIPS = [
  {
    id:"rovers",
    title:"U17 League · vs Rovers",
    meta:"Phone on tripod · 68 min · won 3–1",
    sub:"Sunday 12 Jul · Hackney Marshes, Pitch 4",
    player:{num:7,name:"A. Khan",pos:"RIGHT WING",ovr:82},
    stats:[
      {v:"54",l:"Touches",d:"+9 vs season avg"},
      {v:"28.4",l:"Top speed (km/h)",d:"season best"},
      {v:"81%",l:"Pass accuracy",d:"+4%"},
      {v:"9.8",l:"Distance (km)",d:"+0.6 km"},
      {v:"4",l:"Shots (2 on target)",d:"1 goal"},
      {v:"11",l:"Take-ons (7 won)",d:"64% success"}
    ],
    attrs:[["Pace",88],["Shooting",71],["Passing",76],["Dribbling",84],["Defending",45],["Physical",62]],
    heat:{
      h1:[[72,22,26],[80,30,20],[62,25,18],[85,45,14],[55,38,12]],
      h2:[[68,28,24],[78,48,20],[60,52,16],[88,35,18],[50,30,10]]
    },
    heatNote:"<b>AcademyTracker spotted:</b> you hug the right touchline in the first half, then drift inside after minute 55 — that drift produced your goal. Worth showing your coach.",
    sprints:[1,2,1,3,2,0,2,3,1,2,1,0,2,1],
    sprintNote:"<b>AcademyTracker spotted:</b> 21 sprints total, but a flat spot between minutes 25–30. Hydration break or switched off? Only you know — the data just asks the question.",
    passes:[
      {x1:70,y1:20,x2:82,y2:38,ok:true},{x1:75,y1:30,x2:60,y2:45,ok:true},
      {x1:62,y1:22,x2:74,y2:12,ok:true},{x1:80,y1:40,x2:92,y2:50,ok:false},
      {x1:58,y1:35,x2:70,y2:24,ok:true},{x1:84,y1:30,x2:93,y2:44,ok:false},
      {x1:66,y1:18,x2:78,y2:30,ok:true},{x1:73,y1:42,x2:85,y2:52,ok:true}
    ],
    passNote:"<b>AcademyTracker spotted:</b> both missed passes were low crosses into the box under pressure. Cut-backs to the penalty spot completed 4/4.",
    shots:[
      {x:82,y:20,tx:97,ty:30,result:"goal"},
      {x:78,y:45,tx:96,ty:36,result:"saved"},
      {x:70,y:15,tx:103,ty:8,result:"off"},
      {x:85,y:50,tx:102,ty:54,result:"off"}
    ],
    shotNote:"<b>AcademyTracker spotted:</b> your goal came from a central cutback — same pocket of space as 3 of your 4 shots. That's now your go-to spot, worth drilling.",
    team:{shape:"4-3-3",dots:[[8,50],[25,18],[22,40],[22,60],[25,82],[42,32],[38,50],[42,68],[68,20,true],[62,50],[68,80]]},
    teamNote:"<b>AcademyTracker spotted:</b> the team's average shape is a clean 4-3-3, but the right side (your side) sits 6m higher than the left — you're stretching play well."
  },
  {
    id:"cup",
    title:"County Cup · vs Athletic",
    meta:"Handheld phone · 74 min · lost 1–2",
    sub:"Saturday 4 Jul · Memorial Ground",
    player:{num:7,name:"A. Khan",pos:"RIGHT WING",ovr:69},
    stats:[
      {v:"38",l:"Touches",d:"−7 vs season avg"},
      {v:"26.1",l:"Top speed (km/h)",d:"−2.3 km/h"},
      {v:"72%",l:"Pass accuracy",d:"−5%"},
      {v:"10.4",l:"Distance (km)",d:"+1.2 km"},
      {v:"1",l:"Shots (0 on target)",d:"—"},
      {v:"6",l:"Take-ons (2 won)",d:"33% success"}
    ],
    attrs:[["Pace",80],["Shooting",58],["Passing",70],["Dribbling",66],["Defending",52],["Physical",64]],
    heat:{
      h1:[[55,25,24],[48,35,20],[62,20,16],[40,45,14],[35,30,12]],
      h2:[[45,30,26],[38,45,20],[52,25,14],[30,55,16],[42,60,10]]
    },
    heatNote:"<b>AcademyTracker spotted:</b> you spent 31% of this match in your own half — double your league average. Their left-back pinned you deep; talk to your coach about support behind you.",
    sprints:[2,1,2,1,1,2,1,1,0,1,2,1,1,0,1],
    sprintNote:"<b>AcademyTracker spotted:</b> you sprinted less than usual — lots of jogging recovery runs, few attacking bursts. Effort ≠ threat.",
    passes:[
      {x1:45,y1:25,x2:58,y2:20,ok:true},{x1:50,y1:35,x2:38,y2:45,ok:true},
      {x1:55,y1:22,x2:68,y2:30,ok:false},{x1:40,y1:40,x2:55,y2:48,ok:true},
      {x1:60,y1:28,x2:74,y2:18,ok:false},{x1:35,y1:50,x2:48,y2:38,ok:true},
      {x1:52,y1:30,x2:62,y2:42,ok:false}
    ],
    passNote:"<b>AcademyTracker spotted:</b> all 3 missed passes were forward balls attempted under a two-player press. Consider one-touch layoffs when doubled up.",
    shots:[
      {x:60,y:22,tx:103,ty:15,result:"off"}
    ],
    shotNote:"<b>AcademyTracker spotted:</b> just 1 shot all match, rushed and off target — a symptom of being pinned deep, not a finishing problem. Fix the supply, and shots will come back.",
    team:{shape:"4-4-2",dots:[[8,50],[22,20],[20,40],[20,60],[22,80],[40,22,true],[36,42],[36,58],[40,78],[58,42],[58,58]]},
    teamNote:"<b>AcademyTracker spotted:</b> the midfield four sat very narrow, leaving you isolated 1-v-2 wide. This is a team-structure issue, not just a personal one."
  },
  {
    id:"training",
    title:"Training · small-sided game",
    meta:"Phone leaning on a bag · 26 min",
    sub:"Wednesday 15 Jul · School 3G pitch",
    player:{num:7,name:"A. Khan",pos:"RIGHT WING",ovr:77},
    stats:[
      {v:"61",l:"Touches",d:"small-sided boost"},
      {v:"24.9",l:"Top speed (km/h)",d:"short pitch"},
      {v:"86%",l:"Pass accuracy",d:"+9%"},
      {v:"3.1",l:"Distance (km)",d:"26 min session"},
      {v:"6",l:"Shots (4 on target)",d:"2 goals"},
      {v:"14",l:"Take-ons (9 won)",d:"64% success"}
    ],
    attrs:[["Pace",78],["Shooting",75],["Passing",82],["Dribbling",83],["Defending",50],["Physical",60]],
    heat:{
      h1:[[60,30,26],[70,45,22],[50,55,18],[75,25,16],[55,40,12]],
      h2:[[65,35,24],[72,50,20],[58,28,16],[80,45,14],[48,52,12]]
    },
    heatNote:"<b>AcademyTracker spotted:</b> in tight spaces you naturally move central. Your take-on success in the middle third (78%) is far higher than wide (52%).",
    sprints:[3,2,3,2,3,2],
    sprintNote:"<b>AcademyTracker spotted:</b> sprint frequency in training is nearly double your match rate. The engine is there — matches just aren't seeing it yet.",
    passes:[
      {x1:55,y1:30,x2:66,y2:40,ok:true},{x1:62,y1:45,x2:74,y2:35,ok:true},
      {x1:50,y1:50,x2:62,y2:58,ok:true},{x1:70,y1:30,x2:82,y2:42,ok:true},
      {x1:58,y1:38,x2:72,y2:50,ok:false},{x1:66,y1:52,x2:78,y2:40,ok:true}
    ],
    passNote:"<b>AcademyTracker spotted:</b> 86% completion in traffic. One-touch passing under pressure looks like a genuine strength worth building your game around.",
    shots:[
      {x:66,y:35,tx:97,ty:34,result:"goal"},
      {x:72,y:28,tx:96,ty:28,result:"goal"},
      {x:60,y:45,tx:95,ty:40,result:"saved"},
      {x:75,y:22,tx:97,ty:26,result:"saved"},
      {x:55,y:50,tx:104,ty:52,result:"off"},
      {x:80,y:18,tx:103,ty:8,result:"off"}
    ],
    shotNote:"<b>AcademyTracker spotted:</b> 4 of 6 shots on target in small-sided space — well above your match rate. Tight spaces are clearly bringing out your composure in front of goal.",
    team:{shape:"5-a-side",dots:[[15,50],[38,25],[38,75],[62,35,true],[62,65]]},
    teamNote:"<b>AcademyTracker spotted:</b> your pair rotation with the left forward created 4 of the 6 shots. Chemistry worth flagging to your coach."
  }
];
```

```js
const SEASON=[
  {opp:"United (A)",rating:64,touches:31,pass:69,speed:25.2},
  {opp:"Town (H)",rating:70,touches:40,pass:74,speed:26.0},
  {opp:"Wanderers (H)",rating:66,touches:36,pass:71,speed:25.8},
  {opp:"City U17 (A)",rating:74,touches:45,pass:77,speed:26.9},
  {opp:"Albion (H)",rating:71,touches:43,pass:75,speed:27.1},
  {opp:"Athletic (A)",rating:69,touches:38,pass:72,speed:26.1},
  {opp:"Grove (H)",rating:78,touches:49,pass:79,speed:27.6},
  {opp:"Rovers (H)",rating:82,touches:54,pass:81,speed:28.4}
];
const METRICS={
  rating:{label:"Match rating",unit:"",note:"<b>AcademyTracker spotted:</b> ratings trending up for 4 straight matches — your best run this season. The jump lines up with when you started cutting inside more (see match heatmaps)."},
  touches:{label:"Touches",unit:"",note:"<b>AcademyTracker spotted:</b> touches are climbing steadily. You're getting into the game more — either fitness, positioning, or teammates trusting you. Ask your coach which they think it is."},
  pass:{label:"Pass accuracy",unit:"%",note:"<b>AcademyTracker spotted:</b> pass accuracy +12% since match 1. Biggest gains are in the final third, which is where accuracy is hardest — genuine progress, not padding."},
  speed:{label:"Top speed",unit:" km/h",note:"<b>AcademyTracker spotted:</b> top speed rose 3.2 km/h across the season. If you're not doing sprint work, that's growth + match fitness. If you are — it's working."}
};
```

### B.5 Drills and plan generator

```js
const DRILLS=[
  {name:"Weak-side finishing · 20 shots left foot",
   why:"Your <b>Shooting (71)</b> is your lowest attacking attribute, and 3 of 4 shots vs Rovers came off your right even when the left was open.",
   dur:"15 min · needs: 5 balls, a goal"},
  {name:"1-v-2 escape patterns",
   why:"Vs Athletic every missed pass came when <b>doubled up</b>. Practise the one-touch layoff and spin-out with a teammate or wall.",
   dur:"20 min · needs: a wall or a mate"},
  {name:"Repeat sprint blocks · 6 × 30m",
   why:"Your sprint count <b>flat-lines mid-half</b> in matches but not in training — this builds repeat-sprint stamina.",
   dur:"12 min · needs: 30m of grass"}
];
```

```js
const CATEGORIES=["cardio","shooting","passing","dribbling","defending"];
const CATEGORY_LABEL={cardio:"Cardio & fitness",shooting:"Shooting",passing:"Passing",dribbling:"Dribbling",defending:"Defending"};
const DRILL_POOL={
  cardio:[
    {name:"Repeat sprint blocks · 6 × 30m",why:"Builds repeat-sprint endurance for the parts of the game where fitness drops off fastest — the last 15 minutes of each half.",dur:"12 min · needs: 30m of grass"},
    {name:"Shuttle runs · 10 × 20m",why:"Short, sharp changes of pace — closer to real match running than steady jogging.",dur:"15 min · needs: 20m of space, 2 cones"},
    {name:"Continuous small-sided game · 4v4",why:"Match-realistic conditioning: fitness under pressure, not just fitness in isolation.",dur:"20 min · needs: 7 others, small pitch"},
    {name:"Box-to-box shuttle circuit",why:"Midfielders cover more ground than any other position — this builds the stamina to do it for 90 minutes.",dur:"18 min · needs: 40m of space",positions:["midfielder"]}
  ],
  shooting:[
    {name:"Weak-side finishing · 20 shots left foot",why:"Most players lean on their strong foot under pressure — this forces the weaker side to catch up.",dur:"15 min · needs: 5 balls, a goal"},
    {name:"First-time finishing off crosses",why:"Cutbacks and crosses are converted far less often when there's no time to set the ball — this builds that instinct.",dur:"15 min · needs: a server, 10 balls, a goal"},
    {name:"Long-range strikes · 15 shots outside the box",why:"Widens where you're a threat from, so defences can't just show you inside the box.",dur:"15 min · needs: 10 balls, a goal"},
    {name:"Near-post far-post finishing",why:"Strikers live on instinctive finishes in the box — this trains reading near-post vs far-post service.",dur:"15 min · needs: a server, a goal",positions:["striker"]}
  ],
  passing:[
    {name:"One-touch rondo · 4v2",why:"Forces quick decisions and an open body shape — the habits that show up as pass accuracy in matches.",dur:"15 min · needs: 5 others, a small grid"},
    {name:"Long diagonal switches · 20 reps",why:"Switching play is one of the hardest passes to execute cleanly — direct rep work fixes technique fastest.",dur:"15 min · needs: a mate, 40m of space"},
    {name:"Passing under pressure · 1-touch triangle",why:"Simulates being closed down — the exact moment most misplaced passes happen in matches.",dur:"15 min · needs: 2 others, 3 cones"},
    {name:"Progressive line-breaking passes",why:"Midfielders who can split lines with a pass unlock attacks — direct rep work on the pass defenders hate most.",dur:"15 min · needs: cones, a mate",positions:["midfielder"]}
  ],
  dribbling:[
    {name:"1-v-1 escape patterns",why:"Practise the one-touch layoff and spin-out so you have an answer when doubled up, not just when 1v1 is even.",dur:"20 min · needs: a wall or a mate"},
    {name:"Cone weave · close control",why:"Tight-space ball control at speed — the foundation every dribbling move is built on.",dur:"12 min · needs: 8 cones, a ball"},
    {name:"Change of direction · step-overs & cuts",why:"Adds real moves on top of close control, so defenders can't just read your first touch.",dur:"15 min · needs: 6 cones, a ball"},
    {name:"1v1 out wide · beat your full-back",why:"Wingers win matches in the 1v1 — repetition here compounds fastest for your position.",dur:"15 min · needs: a mate, a cone gate",positions:["winger"]}
  ],
  defending:[
    {name:"Jockeying & delay · 1v1 defending shape",why:"Good defending is usually about body shape and patience, not just tackling — this drills the shape first.",dur:"15 min · needs: a mate, a small grid"},
    {name:"Recovery runs · track-back sprints",why:"Being beaten happens to everyone — what separates good defenders is how fast they recover.",dur:"12 min · needs: 20m of space"},
    {name:"Interception reads · anticipate the pass",why:"Reading the pass before it's played wins the ball without a tackle at all.",dur:"15 min · needs: 2 others, a ball"},
    {name:"Shot-stopping reactions · reflex saves",why:"Reaction saves from close range are the bread-and-butter of goalkeeping — build the reflex, not just the dive.",dur:"15 min · needs: a server, a goal",positions:["goalkeeper"]},
    {name:"Last-ditch tackle timing",why:"Defenders live or die by the timing of the last tackle — mistime it and it's a card or a goal, so timing gets drilled on its own.",dur:"12 min · needs: a mate",positions:["defender"]}
  ]
};
const WEIGHT={1:3,2:2,3:1,4:1,5:0};
const PLAN_CAP=6;
const ratings={};
const POSITIONS=["goalkeeper","defender","midfielder","winger","striker"];
const POSITION_LABEL={goalkeeper:"Goalkeeper",defender:"Defender",midfielder:"Midfielder",winger:"Winger",striker:"Striker"};
const POSITION_PRIMARY={goalkeeper:"defending",defender:"defending",midfielder:"passing",winger:"dribbling",striker:"shooting"};
let selfPosition=null;
```

```js
function orderedPool(cat){
  const pool=DRILL_POOL[cat];
  const specific=pool.filter(d=>d.positions&&d.positions.includes(selfPosition));
  const general=pool.filter(d=>!d.positions||!d.positions.includes(selfPosition));
  return [...specific,...general];
}
function generateSelfPlan(){
  const sorted=[...CATEGORIES].sort((a,b)=>ratings[a]-ratings[b]);
  let plan=[];
  for(const cat of sorted){
    const need=WEIGHT[ratings[cat]];
    const pool=orderedPool(cat);
    for(let i=0;i<need && plan.length<PLAN_CAP;i++) plan.push({...pool[i%pool.length],cat});
    if(plan.length>=PLAN_CAP) break;
  }
  let note="";
  if(plan.length===0){
    plan=[{...DRILL_POOL.cardio[0],cat:"cardio"},{...DRILL_POOL.passing[0],cat:"passing"}];
    note="You rated yourself strong (5/5) across the board — nice. Here are two light maintenance drills to stay sharp rather than nothing at all.";
  }
  const primaryCat=POSITION_PRIMARY[selfPosition];
  if(plan.length<PLAN_CAP && primaryCat && !plan.some(d=>d.cat===primaryCat)){
    const specific=DRILL_POOL[primaryCat].find(d=>d.positions&&d.positions.includes(selfPosition));
    if(specific) plan.push({...specific,cat:primaryCat,bonus:true});
  }
  renderSelfPlan(plan,note);
}
```

### B.6 Fixtures, results and league data

```js
const UPCOMING=[
  {date:"Sat 2 Aug",opp:"Kingsmead",venue:"Away · KO 10:00"},
  {date:"Sat 9 Aug",opp:"Fairfield",venue:"Home · KO 10:00"},
  {date:"Sat 16 Aug",opp:"Meadow Park",venue:"Away · KO 10:00"},
  {date:"Sat 23 Aug",opp:"Oakwood",venue:"Home · KO 10:00"}
];
const RESULTS=[
  {opp:"United",venue:"A",gf:1,ga:3},
  {opp:"Town",venue:"H",gf:2,ga:2},
  {opp:"Wanderers",venue:"H",gf:0,ga:1},
  {opp:"City U17",venue:"A",gf:1,ga:1},
  {opp:"Albion",venue:"H",gf:2,ga:1},
  {opp:"Athletic",venue:"A",gf:1,ga:1},
  {opp:"Grove",venue:"H",gf:3,ga:0},
  {opp:"Rovers",venue:"H",gf:2,ga:1}
];
const LEAGUE=[
  {team:"Rovers",p:8,w:6,d:1,l:1,gd:14,pts:19},
  {team:"Grove",p:8,w:5,d:2,l:1,gd:9,pts:17},
  {team:"Albion",p:8,w:5,d:1,l:2,gd:6,pts:16},
  {team:"City U17",p:8,w:4,d:2,l:2,gd:3,pts:14},
  {team:"Oakfield Colts U17",p:8,w:3,d:3,l:2,gd:2,pts:12,us:true},
  {team:"Athletic",p:8,w:3,d:2,l:3,gd:-1,pts:11},
  {team:"Town",p:8,w:2,d:3,l:3,gd:-4,pts:9},
  {team:"Wanderers",p:8,w:2,d:1,l:5,gd:-6,pts:7},
  {team:"United",p:8,w:1,d:2,l:5,gd:-9,pts:5}
];
```

### B.7 Remaining behaviour code (analysis, rendering, navigation, card)

```js
/* ================= step 1: clips ================= */
const uploadInput=document.getElementById("video-upload");
const uploadTile=document.querySelector(".upload-tile");
uploadInput.addEventListener("change",()=>{
  const file=uploadInput.files[0];
  if(!file) return;
  uploadTile.classList.add("has-file");
  const sizeMB=(file.size/1_000_000).toFixed(1);
  document.getElementById("upload-hint").textContent=`Selected: ${file.name} (${sizeMB} MB) — tap a sample below or wait, analysing now…`;
  const base=CLIPS[0];
  const customClip={...base,
    id:"custom-upload",
    title:"Your upload · "+file.name.replace(/\.[^.]+$/,""),
    meta:"Your phone footage · analysis simulated for this demo",
    sub:"Uploaded "+new Date().toLocaleDateString(undefined,{day:"numeric",month:"short",year:"numeric"})
  };
  startAnalysis(customClip);
});

const clipList=document.getElementById("clip-list");
CLIPS.forEach(c=>{
  const btn=document.createElement("button");
  btn.className="clip";
  btn.innerHTML=`
    <div class="thumb">
      <svg viewBox="0 0 100 64" preserveAspectRatio="xMidYMid slice">${pitchMarkings()}
        <circle cx="${30+Math.random()*40}" cy="${20+Math.random()*24}" r="2.4" fill="#F2B705"/>
        <circle cx="${20+Math.random()*20}" cy="${15+Math.random()*30}" r="2" fill="#F7F5EC"/>
        <circle cx="${55+Math.random()*25}" cy="${15+Math.random()*30}" r="2" fill="#F7F5EC"/>
        <circle cx="${40+Math.random()*30}" cy="${20+Math.random()*20}" r="1.3" fill="#15251C"/>
      </svg>
      <span class="rec"><i></i>PHONE FOOTAGE</span>
    </div>
    <div class="clip-body">
      <h3>${c.title}</h3>
      <p class="clip-meta">${c.meta}</p>
      <span class="clip-cta">Analyse this match</span>
    </div>`;
  btn.addEventListener("click",()=>startAnalysis(c));
  clipList.appendChild(btn);
});

/* ================= step 2: processing ================= */
const stepUpload=document.getElementById("step-upload");
const stepProc=document.getElementById("step-processing");
const dash=document.getElementById("dashboard");
const procLog=document.getElementById("proc-log");
const procBar=document.getElementById("proc-bar");
const scanbox=document.getElementById("scanbox");
const reduceMotion=matchMedia("(prefers-reduced-motion: reduce)").matches;
let timers=[];

function startAnalysis(clip){
  timers.forEach(clearTimeout);timers=[];
  stepUpload.hidden=true;dash.hidden=true;stepProc.hidden=false;
  window.scrollTo({top:0});
  scanbox.innerHTML=pitchSVG(`
    <circle cx="30" cy="24" r="2" fill="#F7F5EC"><animate attributeName="cx" values="30;44;36;30" dur="5s" repeatCount="indefinite"/></circle>
    <circle cx="62" cy="40" r="2" fill="#F7F5EC"><animate attributeName="cy" values="40;26;34;40" dur="6s" repeatCount="indefinite"/></circle>
    <circle cx="70" cy="22" r="2.4" fill="#F2B705"><animate attributeName="cx" values="70;80;74;70" dur="4.5s" repeatCount="indefinite"/></circle>
    <circle cx="48" cy="32" r="1.3" fill="#15251C"><animate attributeName="cx" values="48;64;55;48" dur="4s" repeatCount="indefinite"/><animate attributeName="cy" values="32;25;38;32" dur="4s" repeatCount="indefinite"/></circle>
    <rect x="66" y="17" width="9" height="11" fill="none" stroke="#F2B705" stroke-width="0.6" stroke-dasharray="2 1"/>`,
    "Simulated footage being scanned")+`<div class="scanline"></div>`;
  procLog.innerHTML="";procBar.style.width="0%";

  const lines=[
    ["Loading footage · "+clip.title.toLowerCase(),8],
    ["Stabilising shaky camera… <span class='ok'>done</span>",22],
    ["Detecting players… <span class='ok'>22 found</span>",40],
    ["Locking onto <span class='ok'>#7 A. Khan</span> (that's you)",58],
    ["Tracking ball · logging touches, passes, sprints…",78],
    ["Building your match report… <span class='ok'>ready</span>",100]
  ];
  const stepMs = reduceMotion? 60 : 850;
  lines.forEach((l,i)=>{
    timers.push(setTimeout(()=>{
      const d=document.createElement("div");d.innerHTML="▸ "+l[0];procLog.appendChild(d);
      procBar.style.width=l[1]+"%";
      if(i===lines.length-1) timers.push(setTimeout(()=>renderDash(clip), reduceMotion?100:700));
    }, stepMs*(i+1)));
  });
}

/* ================= step 3: dashboard ================= */
function renderDash(clip){
  stepProc.hidden=true;dash.hidden=false;window.scrollTo({top:0});
  document.getElementById("dash-sub").textContent=clip.title+" · "+clip.sub;
  document.getElementById("p-num").textContent=clip.player.num;
  document.getElementById("p-name").textContent=clip.player.name;
  document.getElementById("p-pos").textContent=clip.player.pos;
  document.getElementById("p-ovr").textContent=clip.player.ovr;
  const touchStat=clip.stats.find(s=>s.l==="Touches");
  document.getElementById("p-touches").textContent=touchStat?touchStat.v:"—";

  document.getElementById("keystats").innerHTML=visibleStats(clip.stats).map(s=>
    `<div class="stat${s.l==="Touches"?" stat-headline":""}"><b>${s.v}</b><span>${s.l}</span><span class="delta">${s.d}</span></div>`).join("");

  currentClip=clip;currentHalf="h1";
  renderHeat();renderSprints();renderPasses();renderShots();renderAttrs();renderTeam();
  selectTab(document.querySelector(".tab"));
}

let currentClip=null,currentHalf="h1";

function renderHeat(){
  const blobs=currentClip.heat[currentHalf].map(([x,y,r],i)=>
    `<circle cx="${x}" cy="${y}" r="${r}" fill="url(#hg)" opacity="${0.9-i*0.08}"/>`).join("");
  document.getElementById("heat-pitch").innerHTML=pitchSVG(
    `<defs><radialGradient id="hg"><stop offset="0%" stop-color="#F2B705" stop-opacity="0.85"/>
      <stop offset="55%" stop-color="#E0641E" stop-opacity="0.45"/>
      <stop offset="100%" stop-color="#E0641E" stop-opacity="0"/></radialGradient></defs>${blobs}`,
    "Heatmap of player positions");
  document.getElementById("heat-note").innerHTML=currentClip.heatNote;
}
document.getElementById("h1-btn").addEventListener("click",e=>{currentHalf="h1";toggleHalf(e.target);renderHeat();});
document.getElementById("h2-btn").addEventListener("click",e=>{currentHalf="h2";toggleHalf(e.target);renderHeat();});
function toggleHalf(btn){
  document.querySelectorAll(".half-toggle button").forEach(b=>b.setAttribute("aria-pressed",b===btn));
}

function renderSprints(){
  const data=currentClip.sprints,max=Math.max(...data,1);
  const w=100/data.length;
  const bars=data.map((v,i)=>{
    const h=(v/max)*40;
    return `<g class="sbar" tabindex="0" role="img" aria-label="Minutes ${i*5} to ${i*5+5}: ${v} sprint${v===1?"":"s"}">
      <rect x="${i*w+w*0.15}" y="${48-h}" width="${w*0.7}" height="${h}" rx="1" fill="var(--turf)"/>
      <text x="${i*w+w/2}" y="${44-h}" text-anchor="middle" font-size="3.2" fill="#15251C" font-family="var(--mono)">${v||""}</text>
      <text x="${i*w+w/2}" y="53.5" text-anchor="middle" font-size="2.6" fill="#5E6E64">${i*5}'</text>
    </g>`;}).join("");
  document.getElementById("sprint-chart").innerHTML=
    `<svg viewBox="0 0 100 56" role="img" aria-label="Sprints per five minute block" style="width:100%;height:auto">
     <line x1="0" y1="48" x2="100" y2="48" stroke="#D8D4C6" stroke-width="0.4"/>${bars}</svg>
     <style>.sbar rect:hover,.sbar:focus rect{fill:var(--amber)}</style>`;
  document.getElementById("sprint-note").innerHTML=currentClip.sprintNote;
}

function renderPasses(){
  const arrows=currentClip.passes.map(p=>{
    const col=p.ok?"#F7F5EC":"#F2B705",dash=p.ok?"":"stroke-dasharray='1.4 1'";
    return `<line x1="${p.x1}" y1="${p.y1}" x2="${p.x2}" y2="${p.y2}" stroke="${col}" stroke-width="0.55" ${dash} marker-end="url(#${p.ok?'ah':'am'})"/>`;
  }).join("");
  document.getElementById("pass-pitch").innerHTML=pitchSVG(
    `<defs>
      <marker id="ah" markerWidth="3.6" markerHeight="3.6" refX="2.6" refY="1.8" orient="auto"><path d="M0 0 L3 1.8 L0 3.6 Z" fill="#F7F5EC"/></marker>
      <marker id="am" markerWidth="3.6" markerHeight="3.6" refX="2.6" refY="1.8" orient="auto"><path d="M0 0 L3 1.8 L0 3.6 Z" fill="#F2B705"/></marker>
     </defs>${arrows}`,"Pass map");
  document.getElementById("pass-note").innerHTML=currentClip.passNote;
}

function renderShots(){
  const COLORS={goal:"var(--amber)",saved:"#F7F5EC",off:"#F2B705"};
  const shots=currentClip.shots.map(s=>{
    const col=COLORS[s.result];
    const dash=s.result==="off"?"stroke-dasharray='1.4 1'":"";
    return `<line x1="${s.x}" y1="${s.y}" x2="${s.tx}" y2="${s.ty}" stroke="${col}" stroke-width="0.6" ${dash} marker-end="url(#sh-${s.result})"/>
      <circle cx="${s.x}" cy="${s.y}" r="1" fill="${col}"/>`;
  }).join("");
  document.getElementById("shot-pitch").innerHTML=pitchSVG(
    `<defs>
      <marker id="sh-goal" markerWidth="3.8" markerHeight="3.8" refX="2.6" refY="1.9" orient="auto"><path d="M0 0 L3.2 1.9 L0 3.8 Z" fill="var(--amber)"/></marker>
      <marker id="sh-saved" markerWidth="3.8" markerHeight="3.8" refX="2.6" refY="1.9" orient="auto"><path d="M0 0 L3.2 1.9 L0 3.8 Z" fill="#F7F5EC"/></marker>
      <marker id="sh-off" markerWidth="3.8" markerHeight="3.8" refX="2.6" refY="1.9" orient="auto"><path d="M0 0 L3.2 1.9 L0 3.8 Z" fill="#F2B705"/></marker>
     </defs>${shots}`,"Shot map");
  document.getElementById("shot-note").innerHTML=currentClip.shotNote;
}

function renderAttrs(){
  document.getElementById("attr-bars").innerHTML=currentClip.attrs.map(([n,v])=>
    `<div class="attr"><div class="row"><span>${n}</span><span>${v}</span></div>
     <div class="track"><i data-w="${v}"></i></div></div>`).join("");
  requestAnimationFrame(()=>requestAnimationFrame(()=>{
    document.querySelectorAll("#attr-bars .track i").forEach(i=>i.style.width=i.dataset.w+"%");
  }));
  document.getElementById("attr-note").innerHTML=
    "<b>How this works:</b> each score compares this match to a benchmark of players in your age group. One bad game won't sink you — one great game won't crown you. Trends over 5+ matches are what matter.";
}

function renderTeam(){
  const dots=currentClip.team.dots.map(d=>{
    const me=d[2];
    return `<circle cx="${d[0]}" cy="${d[1]}" r="${me?2.6:2}" fill="${me?'#F2B705':'#F7F5EC'}" stroke="#15251C" stroke-width="0.4"/>`;
  }).join("");
  document.getElementById("team-pitch").innerHTML=pitchSVG(
    dots+`<text x="96" y="6" text-anchor="end" font-size="4" fill="#F7F5EC" font-family="var(--display)">${currentClip.team.shape}</text>`,
    "Average team positions");
  document.getElementById("team-note").innerHTML=currentClip.teamNote;
}

/* tabs */
document.querySelectorAll(".tab").forEach(t=>t.addEventListener("click",()=>selectTab(t)));
function selectTab(tab){
  document.querySelectorAll(".tab").forEach(t=>t.setAttribute("aria-selected",t===tab));
  document.querySelectorAll(".panel").forEach(p=>p.hidden=(p.id!==tab.dataset.panel));
  if(tab.dataset.panel==="panel-attr") renderAttrs();
}

document.getElementById("restart").addEventListener("click",()=>{
  dash.hidden=true;stepProc.hidden=true;stepUpload.hidden=false;window.scrollTo({top:0});
});

function resultBadge(gf,ga){
  if(gf>ga) return {cls:"badge-w",l:"W"};
  if(gf===ga) return {cls:"badge-d",l:"D"};
  return {cls:"badge-l",l:"L"};
}

document.getElementById("fx-upcoming").innerHTML=UPCOMING.map(f=>`
  <div class="fx-card">
    <div><div class="fx-date">${f.date}</div><div class="fx-opp">vs ${f.opp}</div><div class="fx-venue">${f.venue}</div></div>
  </div>`).join("");

document.getElementById("fx-results").innerHTML=RESULTS.slice().reverse().map(r=>{
  const b=resultBadge(r.gf,r.ga);
  return `
  <div class="fx-card">
    <div><div class="fx-opp">vs ${r.opp}</div><div class="fx-venue">${r.venue==="H"?"Home":"Away"}</div></div>
    <div style="display:flex;align-items:center;gap:10px">
      <span class="fx-score">${r.gf}–${r.ga}</span>
      <span class="badge-result ${b.cls}">${b.l}</span>
    </div>
  </div>`;
}).join("");

document.getElementById("league-body").innerHTML=LEAGUE.map((t,i)=>`
  <tr class="${t.us?"us":""}">
    <td>${i+1}</td><td>${t.team}</td><td>${t.p}</td><td>${t.w}</td><td>${t.d}</td><td>${t.l}</td>
    <td>${t.gd>0?"+":""}${t.gd}</td><td>${t.pts}</td>
  </tr>`).join("");

document.getElementById("fx-note").innerHTML=
  "<b>AcademyTracker spotted:</b> your last 4 results (Albion, Athletic, Grove, Rovers) are unbeaten, and line up with the same 4-match rating climb shown in My Season — the table's finally catching up to the numbers.";
```
