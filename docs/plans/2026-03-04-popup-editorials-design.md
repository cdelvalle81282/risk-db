# How-To Popup + Editorial Briefings Design

**Date:** 2026-03-04

**Goal:** Add a first-time user orientation popup, 8 in-panel editorial briefings, header/footer branding, and replace the sound effect with a clean beep.

---

## 1. Dashboard Chrome Changes

### Header Bar
- Left: `PUBLISHED BY OPTION PIT`
- Center: `OPTION PIT'S GEOPOLITICAL RISK DASHBOARD` (existing title, centered)
- Right: existing controls + new **[?] INTEL** button

### Footer / Ticker Bar
- Far right of ticker bar: `© 2026 Option Pit. All rights reserved.`

### Sound
- Replace `playTypewriter()` square wave with a clean beep: sine tone, ~880Hz, 150ms duration
- Plays on new signal appearances and zone breaches

---

## 2. First-Time Popup

### Behavior
- Shows automatically on first visit (tracked via `localStorage` key `riskTerminal_seen`)
- After dismissal, accessible via **[?] INTEL** button in header
- CRT-styled modal, dark panel, green/amber text, scanline overlay

### Quick Tour (3 screens, default)

**Screen 1 — "WELCOME TO THE WAR ROOM"**

> This dashboard is your early warning system. It watches 5 market instruments — like trip wires around a perimeter — and tells you when something moves that shouldn't.
>
> GREEN = all quiet. AMBER = something's stirring. RED = battle stations.

Buttons: [QUICK TOUR] [FULL BRIEFING] [SKIP — I'LL FIGURE IT OUT]

**Screen 2 — "THE INSTRUMENT PANELS"**

> Each of the 6 panels tracks a different threat. Think of them as radar screens, each watching a different part of the sky:
>
> - **OIL** — The supply-shock tripwire. When oil spikes, someone is fighting over it.
> - **DEFENSE** — The conflict thermometer. Defense stocks rise when smart money smells gunpowder.
> - **VIX** — The fear gauge. It measures how scared Wall Street is right now.
> - **GOLD** — The bunker indicator. When gold surges, people are hiding.
> - **TREASURY** — The recession radar. When short-term rates pass long-term rates, the economy is driving by looking in the rearview mirror.
> - **COMPOSITE** — All five trip wires rolled into one threat level.
>
> Each panel shows **live news headlines** that update every 5 minutes. Click any headline to read the full story.
>
> **Click any panel** for the full intel briefing on why it matters and how we set the alarm levels.

**Screen 3 — "THE CONTROLS & DATA"**

> **Scenario Dials** (left sidebar): These are your "what if" sliders. Crank CONFLICT to 10 to see what a war does to markets. Crank CREDIT STRESS to simulate a 2008-style meltdown. Watch the panels react in real time.
>
> **FAVORED / AVOID** (bottom): Based on current threat levels, these are the sectors you want to own — or run from.
>
> **"What the Traders Think"** (bottom): Articles written by Option Pit's traders about the current conflict and market environment.
>
> **Market data refreshes every 15 minutes. News updates every 5 minutes.**
>
> Questions? Suggestions? Email **support@optionpit.com**

Button: [GOT IT — TAKE ME IN]

### Full Briefing (3 additional screens)

**Screen 4 — "THE COMPOSITE SCORE"**

> The composite score takes all 5 instruments and rolls them into one number from 0 to 100.
>
> Each instrument gets a weight based on how good it is at predicting trouble:
> - VIX gets the biggest share (30%) — it reacts to everything
> - Oil and Treasury each get 20% — one watches supply shocks, the other watches recessions
> - Defense and Gold each get 15% — they confirm what the others are saying
>
> **0-30 = GREEN.** All quiet. **30-60 = AMBER.** Hedges up. **60+ = RED.** Defensive posture.

**Screen 5 — "THE FAVORED/AVOID MATRIX"**

> When threat levels change, some investments benefit and others get hurt. The matrix at the bottom of the dashboard scores 12 sectors based on how they historically react to each threat.
>
> For example: when oil spikes, energy stocks win and airlines lose. When VIX spikes, cash and gold miners win, growth tech loses.
>
> The arrows tell you how strong the signal is. One arrow = mild. Three arrows = strong conviction.

**Screen 6 — "THE SCENARIO DIALS"**

> The three dials on the left let you stress-test the dashboard:
>
> **CONFLICT** (1-10): Simulates military escalation. At max, oil jumps 25%, defense stocks surge 20%, and VIX climbs 15 points. Based on what actually happened during the Ukraine invasion.
>
> **CREDIT STRESS** (1-10): Simulates a financial crisis. At max, VIX spikes 35 points and the yield curve flattens hard. Based on the 2008 crash and COVID panic.
>
> **SUPPLY DISRUPTION** (1-10): Simulates a trade or energy blockade. At max, oil jumps 30%. Based on scenarios like a Strait of Hormuz closure.
>
> The dials stack — you can combine all three to model a worst-case scenario.

Button: [GOT IT — TAKE ME IN]

---

## 3. Editorial Briefings (8 total)

### Trigger
Clicking an instrument panel opens a slide-out modal from the right (~40% screen width on desktop, full-width on mobile).

### Design
- Dark CRT panel, green text, amber section headers
- Scanline overlay continues over the modal
- [X] close button top-right
- Clicking outside the modal also closes it

### Tone
- Option Pit editorial style: engaging, entertaining, informative
- Active verbs, short sentences, punchy
- Under 7th grade reading level
- Jargon explained immediately when used
- Metaphors to anchor abstract concepts
- Real crisis "war stories" as proof

### Structure (each editorial)
1. **WHAT IT IS** — One plain-English sentence with a metaphor
2. **WHY IT'S ON THIS DASHBOARD** — What this instrument tells you about risk
3. **WHERE WE DREW THE LINES** — The empirical basis, explained simply
4. **WHAT THE COLORS TELL YOU TO DO** — GREEN/AMBER/RED actions
5. **THIS HAS HAPPENED BEFORE** — 2-3 brief real crisis stories

### Editorial List

1. **Oil: The Supply-Shock Tripwire** — click oil panel
2. **Defense: The Conflict Thermometer** — click defense panel
3. **VIX: The Fear Gauge** — click VIX panel
4. **Gold: The Bunker Indicator** — click gold panel
5. **Treasury: The Recession Radar** — click treasury panel
6. **Composite: The War Room Score** — click composite panel
7. **The FAVORED/AVOID Matrix** — click FAVORED/AVOID section
8. **The Scenario Dials** — click any dial label in the sidebar

---

## 4. Summary of All Changes

1. Header bar: add "PUBLISHED BY OPTION PIT" left, center title, add [?] INTEL button
2. Footer: add copyright to ticker bar
3. Sound: replace typewriter with clean beep (880Hz sine, 150ms)
4. First-time popup: 3-screen quick tour + 3-screen full briefing, localStorage tracked
5. [?] INTEL button: re-opens the popup anytime
6. 8 editorial briefings: slide-out modals on panel click
7. All editorial content written in OP style (active, engaging, <7th grade reading level)
