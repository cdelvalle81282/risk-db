# Geopolitical Risk Dashboard — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a public-facing Geopolitical Risk Dashboard with retro sci-fi CRT aesthetic, live market data, interactive 3D scenario simulation dials, and an Investment Impact Matrix.

**Architecture:** Single `index.html` file with embedded CSS and JS. Three.js (from CDN) handles 3D radar, rotary dials, and CRT bezel. All data fetched client-side from free APIs (Yahoo Finance v8, FRED). State management via a central `DashboardState` object that drives all rendering. Scenario dials override state values and trigger cascading recalculations.

**Tech Stack:** HTML5, CSS3 (animations, custom properties), JavaScript (ES6+), Three.js (CDN), Canvas API, Yahoo Finance v8 API, FRED API, localStorage.

**Note:** This is a single-file frontend project with no test framework. Each task builds incrementally and is verified by opening `index.html` in a browser. "Verify" steps mean visual/console inspection.

---

### Task 1: HTML Skeleton + CRT Base Styling

**Files:**
- Create: `index.html`

**Step 1: Create the foundational HTML structure**

Create `index.html` with the full layout skeleton. This includes:
- Document head with meta tags, title, Three.js CDN script tag
- CSS custom properties for the color palette
- Base CRT styling: dark background, phosphor green text, scanline overlay
- Grid layout: left sidebar (radar + dials) | right main (2x3 instrument grid) | bottom (impact matrix) | footer (ticker)
- Placeholder content in each section

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GEOPOLITICAL RISK TERMINAL v2.1</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        /* --- CSS Custom Properties (Color Palette) --- */
        :root {
            --bg-primary: #0a0a0a;
            --bg-panel: #0d1a0d;
            --border-green: #1a3a1a;
            --text-green: #33ff33;
            --text-amber: #ffaa00;
            --text-red: #ff3333;
            --text-dim: #0d330d;
            --glow-green: 0 0 10px #33ff33, 0 0 20px #33ff3366;
            --glow-amber: 0 0 10px #ffaa00, 0 0 20px #ffaa0066;
            --glow-red: 0 0 10px #ff3333, 0 0 20px #ff333366;
            --font-mono: 'Courier New', 'Lucida Console', monospace;
        }

        /* --- Reset & Base --- */
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            background: var(--bg-primary);
            color: var(--text-green);
            font-family: var(--font-mono);
            overflow-x: hidden;
            min-height: 100vh;
        }

        /* --- CRT Scanline Overlay --- */
        .crt-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: repeating-linear-gradient(
                0deg,
                transparent,
                transparent 2px,
                rgba(0, 0, 0, 0.15) 2px,
                rgba(0, 0, 0, 0.15) 4px
            );
            pointer-events: none;
            z-index: 9999;
        }

        /* --- Screen Flicker --- */
        @keyframes flicker {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.97; }
        }
        .screen-container {
            animation: flicker 0.1s infinite alternate;
        }

        /* --- CRT Screen Curvature --- */
        .crt-frame {
            max-width: 1600px;
            margin: 0 auto;
            padding: 20px;
            border-radius: 20px;
            box-shadow: inset 0 0 100px rgba(0,0,0,0.8), 0 0 40px rgba(51,255,51,0.1);
        }

        /* --- Header Bar --- */
        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 20px;
            border-bottom: 1px solid var(--border-green);
            margin-bottom: 15px;
        }
        .header h1 {
            font-size: 18px;
            letter-spacing: 3px;
            text-shadow: var(--glow-green);
        }
        .header .status {
            display: flex; gap: 15px; font-size: 12px;
        }
        .live-indicator {
            color: var(--text-green);
            text-shadow: var(--glow-green);
        }
        @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0.3} }
        .live-dot {
            display: inline-block; width: 8px; height: 8px;
            background: var(--text-green); border-radius: 50%;
            animation: blink 1s infinite; margin-right: 5px;
            vertical-align: middle;
        }

        /* --- Main Layout Grid --- */
        .main-grid {
            display: grid;
            grid-template-columns: 280px 1fr;
            gap: 15px;
            margin-bottom: 15px;
        }

        /* --- Left Sidebar --- */
        .sidebar {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        .radar-container {
            height: 280px;
            border: 1px solid var(--border-green);
            border-radius: 8px;
            overflow: hidden;
            background: var(--bg-panel);
        }
        .dials-container {
            border: 1px solid var(--border-green);
            border-radius: 8px;
            padding: 15px;
            background: var(--bg-panel);
        }
        .dials-title {
            font-size: 11px;
            letter-spacing: 2px;
            color: var(--text-dim);
            margin-bottom: 10px;
            text-transform: uppercase;
        }

        /* --- Instrument Panels Grid --- */
        .instruments-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            grid-template-rows: repeat(2, 1fr);
            gap: 12px;
        }
        .instrument-panel {
            border: 1px solid var(--border-green);
            border-radius: 8px;
            padding: 15px;
            background: var(--bg-panel);
            position: relative;
            overflow: hidden;
            min-height: 180px;
        }
        .panel-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        .panel-label {
            font-size: 10px;
            letter-spacing: 2px;
            text-transform: uppercase;
            color: var(--text-dim);
        }
        .panel-zone {
            font-size: 9px;
            padding: 2px 8px;
            border-radius: 3px;
            letter-spacing: 1px;
            font-weight: bold;
        }
        .zone-green { background: #33ff3322; color: var(--text-green); border: 1px solid var(--text-green); }
        .zone-amber { background: #ffaa0022; color: var(--text-amber); border: 1px solid var(--text-amber); }
        .zone-red { background: #ff333322; color: var(--text-red); border: 1px solid var(--text-red); }
        .panel-price {
            font-size: 28px;
            font-weight: bold;
            text-shadow: var(--glow-green);
            margin-bottom: 5px;
        }
        .panel-change {
            font-size: 12px;
            margin-bottom: 10px;
        }
        .panel-signal {
            font-size: 10px;
            padding: 5px 8px;
            border-radius: 3px;
            margin-top: 8px;
            letter-spacing: 1px;
        }
        .sparkline-canvas {
            width: 100%;
            height: 40px;
        }

        /* --- Impact Matrix --- */
        .impact-matrix {
            border: 1px solid var(--border-green);
            border-radius: 8px;
            padding: 15px;
            background: var(--bg-panel);
            margin-bottom: 15px;
        }
        .matrix-title {
            font-size: 11px;
            letter-spacing: 2px;
            text-transform: uppercase;
            color: var(--text-dim);
            margin-bottom: 12px;
        }
        .matrix-columns {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 15px;
        }
        .matrix-column h3 {
            font-size: 11px;
            letter-spacing: 1px;
            margin-bottom: 8px;
            padding-bottom: 5px;
            border-bottom: 1px solid var(--border-green);
        }
        .matrix-column.favored h3 { color: var(--text-green); text-shadow: var(--glow-green); }
        .matrix-column.neutral h3 { color: var(--text-amber); }
        .matrix-column.avoid h3 { color: var(--text-red); }
        .matrix-item {
            font-size: 11px;
            padding: 3px 0;
            display: flex;
            justify-content: space-between;
        }
        .matrix-item.favored { color: var(--text-green); }
        .matrix-item.neutral { color: var(--text-amber); }
        .matrix-item.avoid { color: var(--text-red); }

        /* --- Signal Ticker --- */
        .ticker-bar {
            border: 1px solid var(--border-green);
            border-radius: 8px;
            padding: 10px 15px;
            background: var(--bg-panel);
            overflow: hidden;
            white-space: nowrap;
        }
        .ticker-label {
            font-size: 9px;
            color: var(--text-dim);
            letter-spacing: 2px;
            display: inline-block;
            margin-right: 15px;
        }
        @keyframes ticker-scroll {
            0% { transform: translateX(100%); }
            100% { transform: translateX(-100%); }
        }
        .ticker-content {
            display: inline-block;
            animation: ticker-scroll 30s linear infinite;
            font-size: 12px;
        }

        /* --- Phosphor glow on all text --- */
        h1, h2, h3, .panel-price { text-shadow: var(--glow-green); }

        /* --- VHS Noise Canvas --- */
        #noise-canvas {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none; z-index: 9998; opacity: 0.03;
        }
    </style>
</head>
<body>
    <canvas id="noise-canvas"></canvas>
    <div class="crt-overlay"></div>
    <div class="screen-container">
        <div class="crt-frame">
            <!-- HEADER -->
            <div class="header">
                <h1>GEOPOLITICAL RISK TERMINAL v2.1</h1>
                <div class="status">
                    <span class="live-indicator"><span class="live-dot"></span>LIVE</span>
                    <span id="utc-clock">00:00:00 UTC</span>
                </div>
            </div>

            <!-- MAIN GRID: Sidebar + Instruments -->
            <div class="main-grid">
                <!-- LEFT SIDEBAR -->
                <div class="sidebar">
                    <div class="radar-container" id="radar-mount"></div>
                    <div class="dials-container">
                        <div class="dials-title">// SCENARIO SIMULATION</div>
                        <div id="dials-mount"></div>
                    </div>
                </div>

                <!-- INSTRUMENT PANELS -->
                <div class="instruments-grid">
                    <div class="instrument-panel" id="panel-oil">
                        <div class="panel-header">
                            <span class="panel-label">OIL / WTI CRUDE</span>
                            <span class="panel-zone zone-green" id="oil-zone">GREEN</span>
                        </div>
                        <div class="panel-price" id="oil-price">--</div>
                        <div class="panel-change" id="oil-change">--</div>
                        <canvas class="sparkline-canvas" id="oil-sparkline"></canvas>
                        <div class="panel-signal" id="oil-signal"></div>
                    </div>

                    <div class="instrument-panel" id="panel-defense">
                        <div class="panel-header">
                            <span class="panel-label">DEFENSE / AVG</span>
                            <span class="panel-zone zone-green" id="defense-zone">GREEN</span>
                        </div>
                        <div class="panel-price" id="defense-price">--</div>
                        <div class="panel-change" id="defense-change">--</div>
                        <canvas class="sparkline-canvas" id="defense-sparkline"></canvas>
                        <div class="panel-signal" id="defense-signal"></div>
                    </div>

                    <div class="instrument-panel" id="panel-vix">
                        <div class="panel-header">
                            <span class="panel-label">VIX / FEAR INDEX</span>
                            <span class="panel-zone zone-green" id="vix-zone">GREEN</span>
                        </div>
                        <div class="panel-price" id="vix-price">--</div>
                        <div class="panel-change" id="vix-change">--</div>
                        <canvas class="sparkline-canvas" id="vix-sparkline"></canvas>
                        <div class="panel-signal" id="vix-signal"></div>
                    </div>

                    <div class="instrument-panel" id="panel-gold">
                        <div class="panel-header">
                            <span class="panel-label">GOLD / XAU-USD</span>
                            <span class="panel-zone zone-green" id="gold-zone">GREEN</span>
                        </div>
                        <div class="panel-price" id="gold-price">--</div>
                        <div class="panel-change" id="gold-change">--</div>
                        <canvas class="sparkline-canvas" id="gold-sparkline"></canvas>
                        <div class="panel-signal" id="gold-signal"></div>
                    </div>

                    <div class="instrument-panel" id="panel-treasury">
                        <div class="panel-header">
                            <span class="panel-label">TREASURY YIELDS</span>
                            <span class="panel-zone zone-green" id="treasury-zone">GREEN</span>
                        </div>
                        <div class="panel-price" id="treasury-price">--</div>
                        <div class="panel-change" id="treasury-change">--</div>
                        <canvas class="sparkline-canvas" id="treasury-sparkline"></canvas>
                        <div class="panel-signal" id="treasury-signal"></div>
                    </div>

                    <div class="instrument-panel" id="panel-composite">
                        <div class="panel-header">
                            <span class="panel-label">COMPOSITE RISK</span>
                            <span class="panel-zone zone-green" id="composite-zone">GREEN</span>
                        </div>
                        <div class="panel-price" id="composite-score">0</div>
                        <div class="panel-change" id="composite-label">RISK SCORE / 100</div>
                        <canvas class="sparkline-canvas" id="composite-sparkline"></canvas>
                        <div class="panel-signal" id="composite-signal"></div>
                    </div>
                </div>
            </div>

            <!-- INVESTMENT IMPACT MATRIX -->
            <div class="impact-matrix">
                <div class="matrix-title">// INVESTMENT IMPACT MATRIX</div>
                <div class="matrix-columns" id="impact-matrix-content">
                    <div class="matrix-column favored">
                        <h3>FAVORED</h3>
                        <div id="matrix-favored"></div>
                    </div>
                    <div class="matrix-column neutral">
                        <h3>NEUTRAL</h3>
                        <div id="matrix-neutral"></div>
                    </div>
                    <div class="matrix-column avoid">
                        <h3>AVOID</h3>
                        <div id="matrix-avoid"></div>
                    </div>
                </div>
            </div>

            <!-- SIGNAL TICKER -->
            <div class="ticker-bar">
                <span class="ticker-label">SIGNALS</span>
                <span class="ticker-content" id="ticker-content">
                    INITIALIZING FEED...
                </span>
            </div>
        </div>
    </div>

    <script>
    // === ALL JAVASCRIPT GOES HERE ===
    // (Built up incrementally in subsequent tasks)
    </script>
</body>
</html>
```

**Step 2: Verify in browser**

Open `index.html` in Chrome. Verify:
- Dark background with green text
- Scanline overlay visible
- Subtle screen flicker
- Grid layout: sidebar left, 2x3 panels right, matrix below, ticker at bottom
- All placeholder text shows "--"
- Green phosphor glow on headings
- LIVE indicator blinks

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: scaffold HTML skeleton with CRT retro styling and grid layout"
```

---

### Task 2: VHS Noise Generator + UTC Clock

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Add the VHS noise Canvas renderer and UTC clock**

Inside the `<script>` tag, add:

```javascript
// === VHS NOISE GENERATOR ===
(function initNoise() {
    const canvas = document.getElementById('noise-canvas');
    const ctx = canvas.getContext('2d');
    function resize() {
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;
    }
    resize();
    window.addEventListener('resize', resize);

    function drawNoise() {
        const imageData = ctx.createImageData(canvas.width, canvas.height);
        const data = imageData.data;
        for (let i = 0; i < data.length; i += 4) {
            const v = Math.random() * 255;
            data[i] = v; data[i+1] = v; data[i+2] = v;
            data[i+3] = 255;
        }
        ctx.putImageData(imageData, 0, 0);
        requestAnimationFrame(drawNoise);
    }
    drawNoise();
})();

// === UTC CLOCK ===
(function initClock() {
    const el = document.getElementById('utc-clock');
    function tick() {
        const now = new Date();
        el.textContent = now.toUTCString().split(' ')[4] + ' UTC';
    }
    tick();
    setInterval(tick, 1000);
})();
```

**Step 2: Verify in browser**

- Subtle noise grain visible over entire screen
- UTC clock updates every second in header

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add VHS noise canvas and live UTC clock"
```

---

### Task 3: Dashboard State Engine + Threshold Logic

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Implement the central state engine**

Add before the noise generator code:

```javascript
// === DASHBOARD STATE ENGINE ===
const DashboardState = {
    // Current live values
    live: {
        oil: { price: null, change: null, changePercent: null, history: [] },
        defense: { price: null, change: null, changePercent: null, history: [] },
        vix: { price: null, change: null, changePercent: null, history: [] },
        gold: { price: null, change: null, changePercent: null, history: [] },
        treasury: { y2: null, y10: null, y30: null, spread: null, history: [] }
    },
    // Scenario overrides (null = use live)
    scenario: { oil: null, conflict: 1, sanctions: 1, supply: 1 },
    // Computed zones and scores
    zones: { oil: 'GREEN', defense: 'GREEN', vix: 'GREEN', gold: 'GREEN', treasury: 'GREEN' },
    compositeScore: 0,
    signals: [],
    lastUpdate: null,
    staleData: false
};

// === DEFAULT THRESHOLDS (admin-adjustable) ===
const DEFAULT_THRESHOLDS = {
    oil: { green: 75, amber: 95 },
    defense: { green: 2, amber: 5 },
    vix: { green: 20, amber: 30 },
    gold: { green: 2000, amber: 2200 },
    treasury: { green: 0, amber: -0.5 },
    weights: { oil: 0.20, defense: 0.20, vix: 0.25, gold: 0.15, treasury: 0.20 }
};

function loadThresholds() {
    const saved = localStorage.getItem('dashboard-thresholds');
    return saved ? { ...DEFAULT_THRESHOLDS, ...JSON.parse(saved) } : { ...DEFAULT_THRESHOLDS };
}

let thresholds = loadThresholds();

// === ZONE CALCULATOR ===
function calculateZone(instrument, value) {
    if (value == null) return 'GREEN';
    const t = thresholds[instrument];
    if (instrument === 'treasury') {
        // Treasury: spread-based, inverted logic
        if (value > t.green) return 'GREEN';
        if (value > t.amber) return 'AMBER';
        return 'RED';
    }
    if (value < t.green) return 'GREEN';
    if (value < t.amber) return 'AMBER';
    return 'RED';
}

const ZONE_POINTS = { GREEN: 0, AMBER: 50, RED: 100 };

function calculateComposite() {
    const w = thresholds.weights;
    const z = DashboardState.zones;
    return Math.round(
        ZONE_POINTS[z.oil] * w.oil +
        ZONE_POINTS[z.defense] * w.defense +
        ZONE_POINTS[z.vix] * w.vix +
        ZONE_POINTS[z.gold] * w.gold +
        ZONE_POINTS[z.treasury] * w.treasury
    );
}

// === SIGNAL GENERATOR ===
const SIGNAL_MESSAGES = {
    oil: {
        GREEN: 'OIL STABLE — Favor consumer discretionary, airlines',
        AMBER: 'OIL ELEVATED — Energy sector neutral-positive',
        RED: 'OIL CRITICAL — BUY energy/defense, SELL airlines & logistics'
    },
    defense: {
        GREEN: 'DEFENSE CALM — No escalation priced in',
        AMBER: 'DEFENSE RISING — ACCUMULATE defense positions',
        RED: 'DEFENSE SURGE — ROTATE into defense + commodities'
    },
    vix: {
        GREEN: 'VIX LOW — Risk-on, favor growth & equities',
        AMBER: 'VIX ELEVATED — HEDGE positions, tighten stops',
        RED: 'VIX PANIC — BUY puts, SELL risk assets, accumulate gold'
    },
    gold: {
        GREEN: 'GOLD STABLE — No safe-haven rush',
        AMBER: 'GOLD RISING — Flight to safety, ACCUMULATE gold',
        RED: 'GOLD CRISIS — HOLD gold, risk-off across equities'
    },
    treasury: {
        GREEN: 'YIELD CURVE NORMAL — Favor banks & financials',
        AMBER: 'CURVE FLATTENING — Recession signal, reduce equity exposure',
        RED: 'CURVE INVERTED — SELL cyclicals, MOVE to short bonds & cash'
    }
};

function generateSignals() {
    const signals = [];
    for (const [inst, zone] of Object.entries(DashboardState.zones)) {
        if (zone !== 'GREEN') {
            signals.push({ instrument: inst, zone, message: SIGNAL_MESSAGES[inst][zone] });
        }
    }
    DashboardState.signals = signals;
    return signals;
}

// === SCENARIO CORRELATION CASCADE ===
function applyScenario() {
    const s = DashboardState.scenario;
    const live = DashboardState.live;

    // Start from live values
    let oilPrice = live.oil.price || 70;
    let defenseChange = live.defense.changePercent || 0;
    let vixPrice = live.vix.price || 15;
    let goldPrice = live.gold.price || 1900;
    let spread = live.treasury.spread || 0.5;

    // Oil dial override
    if (s.oil !== null) oilPrice = s.oil;

    // Conflict multiplier (1-10)
    if (s.conflict > 1) {
        const f = (s.conflict - 1) / 9; // 0 to 1
        defenseChange += f * 15;
        oilPrice *= (1 + f * 0.15);
        goldPrice *= (1 + f * 0.10);
        vixPrice += f * 20;
    }

    // Sanctions multiplier (1-10)
    if (s.sanctions > 1) {
        const f = (s.sanctions - 1) / 9;
        oilPrice *= (1 + f * 0.30);
        goldPrice *= (1 + f * 0.06);
        vixPrice += f * 10;
    }

    // Supply disruption (1-10)
    if (s.supply > 1) {
        const f = (s.supply - 1) / 9;
        oilPrice *= (1 + f * 0.20);
        goldPrice *= (1 + f * 0.05);
        spread -= f * 0.3;
    }

    // Calculate zones from effective values
    DashboardState.zones.oil = calculateZone('oil', oilPrice);
    DashboardState.zones.defense = calculateZone('defense', defenseChange);
    DashboardState.zones.vix = calculateZone('vix', vixPrice);
    DashboardState.zones.gold = calculateZone('gold', goldPrice);
    DashboardState.zones.treasury = calculateZone('treasury', spread);
    DashboardState.compositeScore = calculateComposite();

    generateSignals();

    return { oilPrice, defenseChange, vixPrice, goldPrice, spread };
}

// === FULL RECALCULATION ===
function recalculate() {
    const effective = applyScenario();
    renderPanels(effective);
    renderImpactMatrix();
    renderTicker();
    updateRadarColor();
}
```

**Step 2: Verify in browser console**

Open console, type `DashboardState` and verify the object exists with correct structure. Type `calculateZone('oil', 80)` and verify it returns `'AMBER'`. Type `calculateZone('vix', 35)` and verify `'RED'`.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add state engine with threshold logic, zone calculator, and scenario cascade"
```

---

### Task 4: Panel Rendering + Zone Styling

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Implement panel rendering functions**

Add after the `recalculate` function:

```javascript
// === PANEL RENDERING ===
function renderPanels(effective) {
    // Oil
    renderSinglePanel('oil', {
        price: effective.oilPrice,
        priceFormat: '$' + effective.oilPrice.toFixed(2),
        change: DashboardState.live.oil.change,
        changePercent: DashboardState.live.oil.changePercent,
        zone: DashboardState.zones.oil,
        history: DashboardState.live.oil.history
    });
    // Defense
    renderSinglePanel('defense', {
        price: effective.defenseChange,
        priceFormat: (effective.defenseChange >= 0 ? '+' : '') + effective.defenseChange.toFixed(2) + '%',
        change: DashboardState.live.defense.change,
        changePercent: effective.defenseChange,
        zone: DashboardState.zones.defense,
        history: DashboardState.live.defense.history
    });
    // VIX
    renderSinglePanel('vix', {
        price: effective.vixPrice,
        priceFormat: effective.vixPrice.toFixed(2),
        change: DashboardState.live.vix.change,
        changePercent: DashboardState.live.vix.changePercent,
        zone: DashboardState.zones.vix,
        history: DashboardState.live.vix.history
    });
    // Gold
    renderSinglePanel('gold', {
        price: effective.goldPrice,
        priceFormat: '$' + effective.goldPrice.toFixed(0),
        change: DashboardState.live.gold.change,
        changePercent: DashboardState.live.gold.changePercent,
        zone: DashboardState.zones.gold,
        history: DashboardState.live.gold.history
    });
    // Treasury
    const spread = effective.spread;
    renderSinglePanel('treasury', {
        price: spread,
        priceFormat: '2Y-10Y: ' + (spread >= 0 ? '+' : '') + spread.toFixed(2) + '%',
        change: null,
        changePercent: null,
        zone: DashboardState.zones.treasury,
        history: DashboardState.live.treasury.history,
        extraLine: DashboardState.live.treasury.y2 != null ?
            '2Y:' + DashboardState.live.treasury.y2.toFixed(2) + '% | 10Y:' + DashboardState.live.treasury.y10.toFixed(2) + '% | 30Y:' + DashboardState.live.treasury.y30.toFixed(2) + '%' : null
    });
    // Composite
    renderCompositePanel();
}

function renderSinglePanel(id, data) {
    const priceEl = document.getElementById(id + '-price');
    const changeEl = document.getElementById(id + '-change');
    const zoneEl = document.getElementById(id + '-zone');
    const signalEl = document.getElementById(id + '-signal');

    priceEl.textContent = data.priceFormat;

    // Color the price by zone
    const zoneColors = { GREEN: 'var(--text-green)', AMBER: 'var(--text-amber)', RED: 'var(--text-red)' };
    const zoneGlows = { GREEN: 'var(--glow-green)', AMBER: 'var(--glow-amber)', RED: 'var(--glow-red)' };
    priceEl.style.color = zoneColors[data.zone];
    priceEl.style.textShadow = zoneGlows[data.zone];

    // Change display
    if (data.changePercent != null) {
        const arrow = data.changePercent > 0 ? '\u25B2' : data.changePercent < 0 ? '\u25BC' : '\u25C6';
        changeEl.textContent = arrow + ' ' + (data.changePercent >= 0 ? '+' : '') + data.changePercent.toFixed(2) + '%';
        changeEl.style.color = data.changePercent > 0 ? 'var(--text-green)' : data.changePercent < 0 ? 'var(--text-red)' : 'var(--text-amber)';
    } else if (data.extraLine) {
        changeEl.textContent = data.extraLine;
        changeEl.style.color = 'var(--text-dim)';
    }

    // Zone badge
    zoneEl.textContent = data.zone;
    zoneEl.className = 'panel-zone zone-' + data.zone.toLowerCase();

    // Signal message
    const signal = SIGNAL_MESSAGES[id]?.[data.zone];
    if (signal && data.zone !== 'GREEN') {
        signalEl.textContent = '\u26A0 ' + signal;
        signalEl.style.color = zoneColors[data.zone];
        signalEl.style.background = data.zone === 'RED' ? '#ff333311' : '#ffaa0011';
    } else {
        signalEl.textContent = '';
        signalEl.style.background = 'none';
    }

    // Sparkline
    if (data.history.length > 1) {
        drawSparkline(id + '-sparkline', data.history, data.zone);
    }
}

function renderCompositePanel() {
    const score = DashboardState.compositeScore;
    const scoreEl = document.getElementById('composite-score');
    const zoneEl = document.getElementById('composite-zone');
    const signalEl = document.getElementById('composite-signal');

    scoreEl.textContent = score + ' / 100';

    let zone = 'GREEN';
    if (score > 60) zone = 'RED';
    else if (score > 30) zone = 'AMBER';

    const zoneColors = { GREEN: 'var(--text-green)', AMBER: 'var(--text-amber)', RED: 'var(--text-red)' };
    const zoneGlows = { GREEN: 'var(--glow-green)', AMBER: 'var(--glow-amber)', RED: 'var(--glow-red)' };
    scoreEl.style.color = zoneColors[zone];
    scoreEl.style.textShadow = zoneGlows[zone];

    zoneEl.textContent = zone;
    zoneEl.className = 'panel-zone zone-' + zone.toLowerCase();

    const riskLevel = score > 60 ? 'HIGH RISK' : score > 30 ? 'ELEVATED' : 'LOW RISK';
    signalEl.textContent = riskLevel + ' — Composite geopolitical threat assessment';
    signalEl.style.color = zoneColors[zone];
}
```

**Step 2: Verify by calling recalculate with mock data**

In console, set mock values and call recalculate:
```javascript
DashboardState.live.oil.price = 82;
DashboardState.live.vix.price = 24;
DashboardState.live.gold.price = 2100;
DashboardState.live.defense.changePercent = 1.5;
DashboardState.live.treasury.spread = 0.2;
recalculate();
```

Verify panels update with correct prices, zone colors, and signals.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add panel rendering with zone coloring and signal display"
```

---

### Task 5: Sparkline Renderer (Oscilloscope Style)

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Implement Canvas sparkline drawing**

Add after `renderCompositePanel`:

```javascript
// === SPARKLINE RENDERER (Oscilloscope Style) ===
function drawSparkline(canvasId, data, zone) {
    const canvas = document.getElementById(canvasId);
    if (!canvas || data.length < 2) return;
    const ctx = canvas.getContext('2d');
    const dpr = window.devicePixelRatio || 1;
    canvas.width = canvas.offsetWidth * dpr;
    canvas.height = canvas.offsetHeight * dpr;
    ctx.scale(dpr, dpr);
    const w = canvas.offsetWidth;
    const h = canvas.offsetHeight;

    ctx.clearRect(0, 0, w, h);

    const min = Math.min(...data);
    const max = Math.max(...data);
    const range = max - min || 1;
    const step = w / (data.length - 1);

    const colors = { GREEN: '#33ff33', AMBER: '#ffaa00', RED: '#ff3333' };
    const color = colors[zone] || colors.GREEN;

    // Glow effect
    ctx.shadowColor = color;
    ctx.shadowBlur = 6;
    ctx.strokeStyle = color;
    ctx.lineWidth = 1.5;
    ctx.beginPath();
    data.forEach((val, i) => {
        const x = i * step;
        const y = h - ((val - min) / range) * (h - 4) - 2;
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
    });
    ctx.stroke();

    // Faint fill under line
    ctx.shadowBlur = 0;
    ctx.lineTo(w, h);
    ctx.lineTo(0, h);
    ctx.closePath();
    ctx.fillStyle = color.replace(')', ', 0.05)').replace('rgb', 'rgba');
    // Use hex conversion for fill
    ctx.globalAlpha = 0.08;
    ctx.fill();
    ctx.globalAlpha = 1;
}
```

**Step 2: Verify by setting history data**

```javascript
DashboardState.live.oil.history = [68,70,72,71,74,76,78,80,82,81,79,82];
recalculate();
```

Verify oscilloscope-style green glowing line appears in oil panel.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add oscilloscope-style sparkline renderer with glow effect"
```

---

### Task 6: Investment Impact Matrix Engine

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Implement the sector scoring and rendering**

Add after the sparkline function:

```javascript
// === INVESTMENT IMPACT MATRIX ===
const SECTOR_SENSITIVITY = {
    'Defense/Aerospace':    { oil: 0.5,  vix: 0.5,  gold: 0,    treasury: 0,     defense: 1.0  },
    'Energy/Oil Majors':    { oil: 1.0,  vix: 0.2,  gold: 0,    treasury: 0,     defense: 0.5  },
    'Gold/Precious Metals': { oil: 0.5,  vix: 0.8,  gold: 1.0,  treasury: -0.5,  defense: 0.5  },
    'Cash/Money Market':    { oil: 0,    vix: 0.8,  gold: 0,    treasury: 0.8,   defense: 0    },
    'Growth Tech':          { oil: -0.8, vix: -0.8, gold: 0,    treasury: -0.8,  defense: 0    },
    'Airlines/Transport':   { oil: -1.0, vix: -0.5, gold: 0,    treasury: 0,     defense: -0.5 },
    'EM Equities':          { oil: -0.8, vix: -0.8, gold: 0,    treasury: -0.5,  defense: -0.5 },
    'Banks/Financials':     { oil: 0,    vix: -0.5, gold: 0,    treasury: 0.8,   defense: 0    },
    'Real Estate/REITs':    { oil: -0.5, vix: -0.5, gold: 0,    treasury: -0.8,  defense: 0    },
    'Consumer Staples':     { oil: 0,    vix: 0.2,  gold: 0,    treasury: 0,     defense: 0    },
    'Long-duration Bonds':  { oil: 0,    vix: 0.5,  gold: 0.2,  treasury: -1.0,  defense: 0    },
    'Crypto/Speculative':   { oil: -0.5, vix: -0.8, gold: 0,    treasury: -0.5,  defense: 0    }
};

function scoreSectors() {
    const z = DashboardState.zones;
    const zoneScores = {
        oil: ZONE_POINTS[z.oil],
        vix: ZONE_POINTS[z.vix],
        gold: ZONE_POINTS[z.gold],
        treasury: ZONE_POINTS[z.treasury],
        defense: ZONE_POINTS[z.defense]
    };

    const results = [];
    for (const [sector, sens] of Object.entries(SECTOR_SENSITIVITY)) {
        let score = 0;
        for (const [inst, weight] of Object.entries(sens)) {
            score += zoneScores[inst] * weight;
        }
        // Normalize: max possible ~250 (all RED, weight 1.0), scale to -100..+100
        // Positive sensitivity + high zone score = favored
        // Negative sensitivity + high zone score = avoid
        const normalized = Math.max(-100, Math.min(100, score / 2.5));
        results.push({ sector, score: normalized });
    }
    return results.sort((a, b) => b.score - a.score);
}

function getArrows(score) {
    const abs = Math.abs(score);
    if (abs > 70) return score > 0 ? '\u25B2\u25B2\u25B2' : '\u25BC\u25BC\u25BC';
    if (abs > 40) return score > 0 ? '\u25B2\u25B2' : '\u25BC\u25BC';
    if (abs > 15) return score > 0 ? '\u25B2' : '\u25BC';
    return '\u25C6';
}

function renderImpactMatrix() {
    const sectors = scoreSectors();
    const favored = sectors.filter(s => s.score > 30);
    const neutral = sectors.filter(s => s.score >= -30 && s.score <= 30);
    const avoid = sectors.filter(s => s.score < -30);

    function renderColumn(containerId, items, cssClass) {
        const el = document.getElementById(containerId);
        el.innerHTML = items.map(s =>
            '<div class="matrix-item ' + cssClass + '">' +
                '<span>' + s.sector + '</span>' +
                '<span>' + getArrows(s.score) + '</span>' +
            '</div>'
        ).join('') || '<div class="matrix-item" style="color:var(--text-dim)">None</div>';
    }

    renderColumn('matrix-favored', favored, 'favored');
    renderColumn('matrix-neutral', neutral, 'neutral');
    renderColumn('matrix-avoid', avoid, 'avoid');
}
```

**Step 2: Verify**

In console, set some zones to AMBER/RED and call `renderImpactMatrix()`. Verify sectors sort into correct columns.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add investment impact matrix with sector scoring engine"
```

---

### Task 7: Signal Ticker Tape

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Implement ticker rendering**

```javascript
// === SIGNAL TICKER ===
function renderTicker() {
    const el = document.getElementById('ticker-content');
    const signals = DashboardState.signals;
    if (signals.length === 0) {
        el.textContent = 'ALL CLEAR — No active signals — Risk level: LOW';
        el.style.color = 'var(--text-green)';
        return;
    }
    const text = signals.map(s => {
        const icon = s.zone === 'RED' ? '\u26A0\u26A0' : '\u26A0';
        return icon + ' ' + s.message;
    }).join('  \u2502  ');
    el.textContent = text + '  \u2502  ' + text; // Double for seamless scroll
    el.style.color = signals.some(s => s.zone === 'RED') ? 'var(--text-red)' : 'var(--text-amber)';
}
```

**Step 2: Verify**

Set VIX to RED zone and call `recalculate()`. Verify ticker scrolls with warning message.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add scrolling signal ticker tape"
```

---

### Task 8: API Data Fetching (Yahoo Finance + FRED)

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Implement data fetching with CORS proxy fallback**

```javascript
// === API DATA FETCHING ===
const CORS_PROXY = 'https://api.allorigins.win/raw?url=';
const FRED_API_KEY = localStorage.getItem('fred-api-key') || 'DEMO_KEY';

async function fetchWithFallback(url) {
    try {
        const resp = await fetch(url);
        if (resp.ok) return await resp.json();
        throw new Error('HTTP ' + resp.status);
    } catch (e) {
        // Try CORS proxy
        try {
            const resp = await fetch(CORS_PROXY + encodeURIComponent(url));
            if (resp.ok) return await resp.json();
        } catch (e2) { /* fall through */ }
        return null;
    }
}

async function fetchYahoo(symbol) {
    const url = 'https://query1.finance.yahoo.com/v8/finance/chart/' + symbol + '?range=1d&interval=5m';
    return fetchWithFallback(url);
}

function parseYahooChart(data) {
    if (!data?.chart?.result?.[0]) return null;
    const result = data.chart.result[0];
    const meta = result.meta;
    const closes = result.indicators?.quote?.[0]?.close || [];
    const validCloses = closes.filter(c => c != null);
    return {
        price: meta.regularMarketPrice,
        previousClose: meta.chartPreviousClose,
        change: meta.regularMarketPrice - meta.chartPreviousClose,
        changePercent: ((meta.regularMarketPrice - meta.chartPreviousClose) / meta.chartPreviousClose) * 100,
        history: validCloses.slice(-50)
    };
}

async function fetchFRED(series) {
    const url = 'https://api.stlouisfed.org/fred/series/observations?series_id=' + series +
        '&api_key=' + FRED_API_KEY + '&file_type=json&sort_order=desc&limit=5';
    const data = await fetchWithFallback(url);
    if (!data?.observations?.length) return null;
    const val = parseFloat(data.observations[0].value);
    return isNaN(val) ? null : val;
}

async function fetchAllData() {
    const staleIndicator = document.querySelector('.live-indicator');
    try {
        const [oilData, vixData, goldData, lmtData, rtxData, nocData, gdData, y2, y10, y30] =
            await Promise.all([
                fetchYahoo('CL=F'),
                fetchYahoo('%5EVIX'),
                fetchYahoo('GC=F'),
                fetchYahoo('LMT'),
                fetchYahoo('RTX'),
                fetchYahoo('NOC'),
                fetchYahoo('GD'),
                fetchFRED('DGS2'),
                fetchFRED('DGS10'),
                fetchFRED('DGS30')
            ]);

        // Oil
        const oil = parseYahooChart(oilData);
        if (oil) Object.assign(DashboardState.live.oil, oil);

        // VIX
        const vix = parseYahooChart(vixData);
        if (vix) Object.assign(DashboardState.live.vix, vix);

        // Gold
        const gold = parseYahooChart(goldData);
        if (gold) Object.assign(DashboardState.live.gold, gold);

        // Defense average
        const defenseStocks = [lmtData, rtxData, nocData, gdData].map(parseYahooChart).filter(Boolean);
        if (defenseStocks.length > 0) {
            const avgChange = defenseStocks.reduce((sum, s) => sum + s.changePercent, 0) / defenseStocks.length;
            const avgPrice = defenseStocks.reduce((sum, s) => sum + s.price, 0) / defenseStocks.length;
            DashboardState.live.defense.price = avgPrice;
            DashboardState.live.defense.changePercent = avgChange;
            DashboardState.live.defense.change = avgChange;
            DashboardState.live.defense.history = defenseStocks[0].history;
        }

        // Treasury
        if (y2 != null && y10 != null) {
            DashboardState.live.treasury.y2 = y2;
            DashboardState.live.treasury.y10 = y10;
            DashboardState.live.treasury.y30 = y30;
            DashboardState.live.treasury.spread = y10 - y2;
            DashboardState.live.treasury.history.push(y10 - y2);
            if (DashboardState.live.treasury.history.length > 30) DashboardState.live.treasury.history.shift();
        }

        DashboardState.lastUpdate = new Date();
        DashboardState.staleData = false;
        staleIndicator.innerHTML = '<span class="live-dot"></span>LIVE';
        staleIndicator.style.color = 'var(--text-green)';

        // Cache to localStorage
        localStorage.setItem('dashboard-cache', JSON.stringify(DashboardState.live));

    } catch (e) {
        console.error('Fetch error:', e);
        DashboardState.staleData = true;
        staleIndicator.innerHTML = '\u26A0 STALE DATA';
        staleIndicator.style.color = 'var(--text-red)';

        // Load from cache
        const cached = localStorage.getItem('dashboard-cache');
        if (cached) {
            const parsed = JSON.parse(cached);
            Object.assign(DashboardState.live, parsed);
        }
    }

    recalculate();
}

// Initial fetch + interval
fetchAllData();
setInterval(fetchAllData, 60000);
```

**Step 2: Verify**

Open in browser with DevTools Network tab open. Verify API calls fire, data populates panels. If CORS blocks Yahoo, verify proxy fallback attempts. Check localStorage for cached data.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Yahoo Finance and FRED API fetching with CORS proxy fallback"
```

---

### Task 9: Three.js Radar Display

**Files:**
- Modify: `index.html` (inside the `<script>` tag)

**Step 1: Implement 3D radar with rotating sweep beam**

```javascript
// === THREE.JS RADAR DISPLAY ===
let radarScene, radarCamera, radarRenderer, radarSweep, radarBlips = [];
let radarColor = new THREE.Color(0x33ff33);

function initRadar() {
    const container = document.getElementById('radar-mount');
    const w = container.offsetWidth;
    const h = container.offsetHeight;

    radarScene = new THREE.Scene();
    radarCamera = new THREE.PerspectiveCamera(50, w / h, 0.1, 100);
    radarCamera.position.set(0, 3, 3);
    radarCamera.lookAt(0, 0, 0);

    radarRenderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    radarRenderer.setSize(w, h);
    radarRenderer.setPixelRatio(window.devicePixelRatio);
    container.appendChild(radarRenderer.domElement);

    // Radar base disc
    const baseGeo = new THREE.CylinderGeometry(1.8, 1.8, 0.05, 64);
    const baseMat = new THREE.MeshBasicMaterial({ color: 0x0d1a0d });
    const base = new THREE.Mesh(baseGeo, baseMat);
    radarScene.add(base);

    // Concentric rings
    for (let r = 0.5; r <= 1.5; r += 0.5) {
        const ringGeo = new THREE.RingGeometry(r - 0.01, r + 0.01, 64);
        const ringMat = new THREE.MeshBasicMaterial({ color: 0x1a3a1a, side: THREE.DoubleSide });
        const ring = new THREE.Mesh(ringGeo, ringMat);
        ring.rotation.x = -Math.PI / 2;
        ring.position.y = 0.03;
        radarScene.add(ring);
    }

    // Cross hairs
    for (let angle = 0; angle < Math.PI; angle += Math.PI / 4) {
        const lineGeo = new THREE.BufferGeometry().setFromPoints([
            new THREE.Vector3(-1.8 * Math.cos(angle), 0.03, -1.8 * Math.sin(angle)),
            new THREE.Vector3(1.8 * Math.cos(angle), 0.03, 1.8 * Math.sin(angle))
        ]);
        const lineMat = new THREE.LineBasicMaterial({ color: 0x1a3a1a });
        radarScene.add(new THREE.Line(lineGeo, lineMat));
    }

    // Sweep beam (triangle)
    const sweepGeo = new THREE.CircleGeometry(1.8, 1, 0, 0.3);
    const sweepMat = new THREE.MeshBasicMaterial({
        color: 0x33ff33, transparent: true, opacity: 0.3, side: THREE.DoubleSide
    });
    radarSweep = new THREE.Mesh(sweepGeo, sweepMat);
    radarSweep.rotation.x = -Math.PI / 2;
    radarSweep.position.y = 0.04;
    radarScene.add(radarSweep);

    // Blips for each instrument
    const instruments = ['oil', 'defense', 'vix', 'gold', 'treasury'];
    const angles = [0, 1.26, 2.51, 3.77, 5.03]; // evenly spaced around circle
    instruments.forEach((inst, i) => {
        const blipGeo = new THREE.SphereGeometry(0.06, 16, 16);
        const blipMat = new THREE.MeshBasicMaterial({ color: 0x33ff33 });
        const blip = new THREE.Mesh(blipGeo, blipMat);
        blip.position.y = 0.06;
        blip.userData = { instrument: inst, baseAngle: angles[i] };
        radarScene.add(blip);
        radarBlips.push(blip);
    });

    animateRadar();
}

function animateRadar() {
    requestAnimationFrame(animateRadar);
    radarSweep.rotation.z -= 0.025; // ~4s full rotation
    radarSweep.material.color.copy(radarColor);

    // Update blip positions and colors based on zones
    radarBlips.forEach(blip => {
        const inst = blip.userData.instrument;
        const zone = DashboardState.zones[inst];
        const distance = zone === 'RED' ? 1.3 : zone === 'AMBER' ? 0.8 : 0.4;
        const angle = blip.userData.baseAngle;
        blip.position.x = Math.cos(angle) * distance;
        blip.position.z = Math.sin(angle) * distance;

        const colors = { GREEN: 0x33ff33, AMBER: 0xffaa00, RED: 0xff3333 };
        blip.material.color.setHex(colors[zone]);
    });

    radarRenderer.render(radarScene, radarCamera);
}

function updateRadarColor() {
    const score = DashboardState.compositeScore;
    if (score > 60) radarColor.setHex(0xff3333);
    else if (score > 30) radarColor.setHex(0xffaa00);
    else radarColor.setHex(0x33ff33);
}

// Initialize after DOM ready
initRadar();
```

**Step 2: Verify**

- 3D radar appears in left sidebar
- Sweep beam rotates continuously
- 5 blips visible (one per instrument)
- Change a zone to RED in console, call `recalculate()`, verify blip moves outward and turns red

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Three.js 3D radar with rotating sweep and instrument blips"
```

---

### Task 10: Three.js Scenario Dials (3D Rotary Knobs)

**Files:**
- Modify: `index.html` (inside the `<script>` tag, and add CSS for dial labels)

**Step 1: Add CSS for dial UI**

Add to the `<style>` section:

```css
.dial-row {
    display: flex;
    align-items: center;
    margin-bottom: 12px;
    gap: 10px;
}
.dial-canvas {
    width: 60px;
    height: 60px;
    cursor: grab;
}
.dial-canvas:active { cursor: grabbing; }
.dial-info {
    flex: 1;
}
.dial-label {
    font-size: 10px;
    letter-spacing: 1px;
    color: var(--text-green);
    text-transform: uppercase;
}
.dial-value {
    font-size: 16px;
    font-weight: bold;
    text-shadow: var(--glow-green);
}
.dial-reset {
    font-size: 8px;
    color: var(--text-dim);
    cursor: pointer;
}
.dial-reset:hover {
    color: var(--text-green);
}
```

**Step 2: Implement 3D dial rendering and interaction**

```javascript
// === THREE.JS SCENARIO DIALS ===
const DIAL_CONFIG = [
    { id: 'oil', label: 'OIL PRICE', min: 40, max: 150, unit: '$', key: 'oil' },
    { id: 'conflict', label: 'CONFLICT LEVEL', min: 1, max: 10, unit: '', key: 'conflict' },
    { id: 'sanctions', label: 'SANCTIONS', min: 1, max: 10, unit: '', key: 'sanctions' },
    { id: 'supply', label: 'SUPPLY DISRUPT', min: 1, max: 10, unit: '', key: 'supply' }
];

function initDials() {
    const mount = document.getElementById('dials-mount');
    mount.innerHTML = '';

    DIAL_CONFIG.forEach(cfg => {
        const row = document.createElement('div');
        row.className = 'dial-row';

        const canvas = document.createElement('canvas');
        canvas.className = 'dial-canvas';
        canvas.width = 120;
        canvas.height = 120;
        canvas.id = 'dial-canvas-' + cfg.id;

        const info = document.createElement('div');
        info.className = 'dial-info';

        const label = document.createElement('div');
        label.className = 'dial-label';
        label.textContent = cfg.label;

        const value = document.createElement('div');
        value.className = 'dial-value';
        value.id = 'dial-value-' + cfg.id;

        const defaultVal = cfg.key === 'oil' ?
            (DashboardState.live.oil.price || 70) :
            DashboardState.scenario[cfg.key];
        value.textContent = cfg.unit + (typeof defaultVal === 'number' ? defaultVal.toFixed(cfg.key === 'oil' ? 0 : 0) : defaultVal);

        const reset = document.createElement('div');
        reset.className = 'dial-reset';
        reset.textContent = '[DOUBLE-CLICK TO RESET]';

        info.append(label, value, reset);
        row.append(canvas, info);
        mount.appendChild(row);

        // Draw initial dial
        drawDial(canvas, 0);

        // Interaction
        let dragging = false;
        let startY = 0;
        let startAngle = 0;
        let currentAngle = 0; // -150 to 150 degrees

        function valueToAngle(val) {
            return ((val - cfg.min) / (cfg.max - cfg.min)) * 300 - 150;
        }
        function angleToValue(angle) {
            return cfg.min + ((angle + 150) / 300) * (cfg.max - cfg.min);
        }

        // Set initial angle
        currentAngle = valueToAngle(defaultVal);
        drawDial(canvas, currentAngle);

        canvas.addEventListener('mousedown', e => {
            dragging = true;
            startY = e.clientY;
            startAngle = currentAngle;
            e.preventDefault();
        });
        window.addEventListener('mousemove', e => {
            if (!dragging) return;
            const dy = startY - e.clientY;
            currentAngle = Math.max(-150, Math.min(150, startAngle + dy * 1.5));
            drawDial(canvas, currentAngle);
            const val = angleToValue(currentAngle);
            value.textContent = cfg.unit + Math.round(val);

            // Update scenario state
            if (cfg.key === 'oil') {
                DashboardState.scenario.oil = val;
            } else {
                DashboardState.scenario[cfg.key] = val;
            }
            recalculate();
        });
        window.addEventListener('mouseup', () => { dragging = false; });

        // Double-click to reset
        canvas.addEventListener('dblclick', () => {
            if (cfg.key === 'oil') {
                DashboardState.scenario.oil = null;
                const livePrice = DashboardState.live.oil.price || 70;
                currentAngle = valueToAngle(livePrice);
                value.textContent = cfg.unit + Math.round(livePrice);
            } else {
                DashboardState.scenario[cfg.key] = 1;
                currentAngle = -150;
                value.textContent = cfg.unit + '1';
            }
            drawDial(canvas, currentAngle);
            recalculate();
        });
    });
}

function drawDial(canvas, angleDeg) {
    const ctx = canvas.getContext('2d');
    const cx = 60, cy = 60, r = 45;
    ctx.clearRect(0, 0, 120, 120);

    // Outer ring
    ctx.beginPath();
    ctx.arc(cx, cy, r, 0, Math.PI * 2);
    ctx.strokeStyle = '#1a3a1a';
    ctx.lineWidth = 2;
    ctx.stroke();

    // Tick marks
    for (let deg = -150; deg <= 150; deg += 30) {
        const rad = (deg - 90) * Math.PI / 180;
        const inner = r - 8;
        const outer = r - 2;
        ctx.beginPath();
        ctx.moveTo(cx + Math.cos(rad) * inner, cy + Math.sin(rad) * inner);
        ctx.lineTo(cx + Math.cos(rad) * outer, cy + Math.sin(rad) * outer);
        ctx.strokeStyle = '#33ff3366';
        ctx.lineWidth = 1;
        ctx.stroke();
    }

    // Knob body
    const gradient = ctx.createRadialGradient(cx - 5, cy - 5, 5, cx, cy, r - 10);
    gradient.addColorStop(0, '#2a2a2a');
    gradient.addColorStop(1, '#111');
    ctx.beginPath();
    ctx.arc(cx, cy, r - 10, 0, Math.PI * 2);
    ctx.fillStyle = gradient;
    ctx.fill();
    ctx.strokeStyle = '#333';
    ctx.lineWidth = 1;
    ctx.stroke();

    // Indicator line
    const indicatorRad = (angleDeg - 90) * Math.PI / 180;
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.lineTo(cx + Math.cos(indicatorRad) * (r - 14), cy + Math.sin(indicatorRad) * (r - 14));
    ctx.strokeStyle = '#33ff33';
    ctx.lineWidth = 2;
    ctx.shadowColor = '#33ff33';
    ctx.shadowBlur = 8;
    ctx.stroke();
    ctx.shadowBlur = 0;

    // Center dot
    ctx.beginPath();
    ctx.arc(cx, cy, 4, 0, Math.PI * 2);
    ctx.fillStyle = '#33ff33';
    ctx.fill();
}

initDials();
```

**Step 2: Verify**

- 4 dials appear in left sidebar below radar
- Click-drag up/down rotates each dial
- Oil dial changes oil price, panels and matrix update in real time
- Conflict/Sanctions/Supply dials cascade effects across all instruments
- Double-click resets to live/default values

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add interactive scenario simulation dials with cascade engine"
```

---

### Task 11: Admin Settings Panel

**Files:**
- Modify: `index.html` (CSS + JS sections)

**Step 1: Add admin panel CSS**

```css
/* --- Admin Panel --- */
.admin-overlay {
    display: none;
    position: fixed; top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(0,0,0,0.85); z-index: 10000;
    justify-content: center; align-items: center;
}
.admin-overlay.active { display: flex; }
.admin-panel {
    background: var(--bg-primary);
    border: 1px solid var(--text-green);
    border-radius: 8px;
    padding: 30px;
    max-width: 600px;
    width: 90%;
    max-height: 80vh;
    overflow-y: auto;
    box-shadow: 0 0 40px rgba(51,255,51,0.2);
}
.admin-panel h2 {
    font-size: 14px;
    letter-spacing: 3px;
    margin-bottom: 20px;
    text-shadow: var(--glow-green);
}
.admin-group {
    margin-bottom: 15px;
    padding-bottom: 15px;
    border-bottom: 1px solid var(--border-green);
}
.admin-group label {
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-size: 11px;
    margin-bottom: 5px;
}
.admin-group input {
    background: var(--bg-panel);
    border: 1px solid var(--border-green);
    color: var(--text-green);
    font-family: var(--font-mono);
    padding: 5px 10px;
    width: 100px;
    text-align: right;
}
.admin-btn {
    background: none;
    border: 1px solid var(--text-green);
    color: var(--text-green);
    font-family: var(--font-mono);
    padding: 8px 20px;
    cursor: pointer;
    margin-right: 10px;
    font-size: 11px;
    letter-spacing: 1px;
}
.admin-btn:hover {
    background: var(--text-green);
    color: var(--bg-primary);
}
.admin-btn.danger {
    border-color: var(--text-red);
    color: var(--text-red);
}
.admin-btn.danger:hover {
    background: var(--text-red);
    color: var(--bg-primary);
}
```

**Step 2: Add admin panel HTML**

Add before closing `</div>` of `crt-frame`:

```html
<!-- ADMIN PANEL -->
<div class="admin-overlay" id="admin-overlay">
    <div class="admin-panel">
        <h2>// ADMIN: THRESHOLD SETTINGS</h2>
        <div id="admin-fields"></div>
        <div style="margin-top:20px">
            <button class="admin-btn" onclick="saveAdmin()">SAVE</button>
            <button class="admin-btn danger" onclick="resetAdmin()">RESET DEFAULTS</button>
            <button class="admin-btn" onclick="closeAdmin()">CLOSE</button>
        </div>
    </div>
</div>
```

**Step 3: Add admin panel JavaScript**

```javascript
// === ADMIN PANEL ===
function openAdmin() {
    const overlay = document.getElementById('admin-overlay');
    const fields = document.getElementById('admin-fields');
    fields.innerHTML = '';

    const sections = [
        { title: 'OIL THRESHOLDS', key: 'oil', fields: [
            { label: 'Green below ($)', field: 'green' },
            { label: 'Amber below ($)', field: 'amber' }
        ]},
        { title: 'DEFENSE THRESHOLDS', key: 'defense', fields: [
            { label: 'Green below (%)', field: 'green' },
            { label: 'Amber below (%)', field: 'amber' }
        ]},
        { title: 'VIX THRESHOLDS', key: 'vix', fields: [
            { label: 'Green below', field: 'green' },
            { label: 'Amber below', field: 'amber' }
        ]},
        { title: 'GOLD THRESHOLDS', key: 'gold', fields: [
            { label: 'Green below ($)', field: 'green' },
            { label: 'Amber below ($)', field: 'amber' }
        ]},
        { title: 'TREASURY SPREAD', key: 'treasury', fields: [
            { label: 'Green above (%)', field: 'green' },
            { label: 'Amber above (%)', field: 'amber' }
        ]},
        { title: 'COMPOSITE WEIGHTS', key: 'weights', fields: [
            { label: 'Oil weight', field: 'oil' },
            { label: 'Defense weight', field: 'defense' },
            { label: 'VIX weight', field: 'vix' },
            { label: 'Gold weight', field: 'gold' },
            { label: 'Treasury weight', field: 'treasury' }
        ]},
        { title: 'API KEYS', key: '_api', fields: [
            { label: 'FRED API Key', field: 'fred', value: localStorage.getItem('fred-api-key') || '' }
        ]}
    ];

    sections.forEach(section => {
        const group = document.createElement('div');
        group.className = 'admin-group';
        group.innerHTML = '<div style="font-size:10px;color:var(--text-dim);letter-spacing:2px;margin-bottom:8px">' + section.title + '</div>';

        section.fields.forEach(f => {
            const label = document.createElement('label');
            const val = section.key === '_api' ? (f.value || '') : thresholds[section.key][f.field];
            label.innerHTML = '<span>' + f.label + '</span><input type="text" id="admin-' + section.key + '-' + f.field + '" value="' + val + '">';
            group.appendChild(label);
        });

        fields.appendChild(group);
    });

    overlay.classList.add('active');
}

function closeAdmin() {
    document.getElementById('admin-overlay').classList.remove('active');
}

function saveAdmin() {
    ['oil','defense','vix','gold','treasury'].forEach(key => {
        thresholds[key].green = parseFloat(document.getElementById('admin-' + key + '-green').value);
        thresholds[key].amber = parseFloat(document.getElementById('admin-' + key + '-amber').value);
    });
    ['oil','defense','vix','gold','treasury'].forEach(key => {
        thresholds.weights[key] = parseFloat(document.getElementById('admin-weights-' + key).value);
    });
    localStorage.setItem('dashboard-thresholds', JSON.stringify(thresholds));

    const fredKey = document.getElementById('admin-_api-fred').value;
    if (fredKey) localStorage.setItem('fred-api-key', fredKey);

    closeAdmin();
    recalculate();
}

function resetAdmin() {
    thresholds = { ...DEFAULT_THRESHOLDS };
    localStorage.removeItem('dashboard-thresholds');
    closeAdmin();
    recalculate();
    openAdmin(); // Re-open with defaults
}

// Admin access: Ctrl+Shift+A or ?admin=true
document.addEventListener('keydown', e => {
    if (e.ctrlKey && e.shiftKey && e.key === 'A') {
        e.preventDefault();
        openAdmin();
    }
});
if (new URLSearchParams(window.location.search).has('admin')) {
    setTimeout(openAdmin, 1000);
}
```

**Step 4: Verify**

- Press Ctrl+Shift+A, admin panel opens with current thresholds
- Change oil green threshold to 60, save — verify oil panel recalculates
- Click Reset Defaults — verify original values restored
- Navigate to `?admin=true` — verify panel auto-opens

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add admin settings panel for threshold and API key configuration"
```

---

### Task 12: Screen Shake + Threshold Breach Animations

**Files:**
- Modify: `index.html` (CSS + JS)

**Step 1: Add CSS animations**

```css
@keyframes screen-shake {
    0%, 100% { transform: translate(0); }
    25% { transform: translate(-3px, 2px); }
    50% { transform: translate(3px, -2px); }
    75% { transform: translate(-2px, -3px); }
}
.shake { animation: screen-shake 0.3s ease; }
@keyframes panel-pulse {
    0% { box-shadow: inset 0 0 0 rgba(255,51,51,0); }
    50% { box-shadow: inset 0 0 30px rgba(255,51,51,0.3); }
    100% { box-shadow: inset 0 0 0 rgba(255,51,51,0); }
}
.panel-breach { animation: panel-pulse 0.6s ease; }
@keyframes red-pulse {
    0%, 100% { text-shadow: 0 0 10px #ff3333, 0 0 20px #ff333366; }
    50% { text-shadow: 0 0 20px #ff3333, 0 0 40px #ff333399; }
}
.zone-red-pulse { animation: red-pulse 1s infinite; }
```

**Step 2: Add zone change detection**

```javascript
// === THRESHOLD BREACH ANIMATION ===
let previousZones = { oil: 'GREEN', defense: 'GREEN', vix: 'GREEN', gold: 'GREEN', treasury: 'GREEN' };

function checkBreaches() {
    let breached = false;
    for (const inst of ['oil', 'defense', 'vix', 'gold', 'treasury']) {
        const prev = previousZones[inst];
        const curr = DashboardState.zones[inst];
        if (curr !== prev && (curr === 'RED' || (curr === 'AMBER' && prev === 'GREEN'))) {
            breached = true;
            const panel = document.getElementById('panel-' + inst);
            panel.classList.add('panel-breach');
            setTimeout(() => panel.classList.remove('panel-breach'), 600);
        }
        previousZones[inst] = curr;
    }
    if (breached) {
        document.querySelector('.crt-frame').classList.add('shake');
        setTimeout(() => document.querySelector('.crt-frame').classList.remove('shake'), 300);
    }

    // Pulsing red glow on RED zone prices
    for (const inst of ['oil', 'defense', 'vix', 'gold', 'treasury']) {
        const priceEl = document.getElementById(inst + '-price');
        if (DashboardState.zones[inst] === 'RED') {
            priceEl.classList.add('zone-red-pulse');
        } else {
            priceEl.classList.remove('zone-red-pulse');
        }
    }
}
```

**Step 3: Wire checkBreaches into recalculate**

Find the `recalculate` function and add `checkBreaches()` at the end:

```javascript
function recalculate() {
    const effective = applyScenario();
    renderPanels(effective);
    renderImpactMatrix();
    renderTicker();
    updateRadarColor();
    checkBreaches();
}
```

**Step 4: Verify**

Turn the oil dial past $95. Verify:
- Oil panel flashes with inner glow
- Screen shakes briefly
- Oil price text pulses with red glow

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add screen shake and pulse animations on threshold breach"
```

---

### Task 13: Sound System (Optional, Off by Default)

**Files:**
- Modify: `index.html` (JS + add sound toggle to header)

**Step 1: Add sound toggle button to header HTML**

Add to the `.header .status` div:

```html
<span id="sound-toggle" style="cursor:pointer;color:var(--text-dim)" onclick="toggleSound()">[SND:OFF]</span>
```

**Step 2: Implement Web Audio API sound effects**

```javascript
// === SOUND SYSTEM ===
let audioCtx = null;
let soundEnabled = false;

function toggleSound() {
    soundEnabled = !soundEnabled;
    if (soundEnabled && !audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    document.getElementById('sound-toggle').textContent = soundEnabled ? '[SND:ON]' : '[SND:OFF]';
    document.getElementById('sound-toggle').style.color = soundEnabled ? 'var(--text-green)' : 'var(--text-dim)';
}

function playPing() {
    if (!soundEnabled || !audioCtx) return;
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.frequency.value = 1200;
    osc.type = 'sine';
    gain.gain.setValueAtTime(0.05, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.3);
    osc.start(audioCtx.currentTime);
    osc.stop(audioCtx.currentTime + 0.3);
}

function playGeiger() {
    if (!soundEnabled || !audioCtx) return;
    const bufferSize = audioCtx.sampleRate * 0.02;
    const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
    const data = buffer.getChannelData(0);
    for (let i = 0; i < bufferSize; i++) data[i] = (Math.random() * 2 - 1) * 0.1;
    const source = audioCtx.createBufferSource();
    source.buffer = buffer;
    source.connect(audioCtx.destination);
    source.start();
}

function playTypewriter() {
    if (!soundEnabled || !audioCtx) return;
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    osc.frequency.value = 800;
    osc.type = 'square';
    gain.gain.setValueAtTime(0.03, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.05);
    osc.start(audioCtx.currentTime);
    osc.stop(audioCtx.currentTime + 0.05);
}
```

**Step 3: Wire sounds into existing functions**

- Add `playPing()` call in the radar animation loop (every full rotation)
- Add `playGeiger()` in `checkBreaches` when composite score increases
- Add `playTypewriter()` in `renderTicker` when new signals appear

**Step 4: Verify**

Click [SND:OFF] to toggle on. Turn dials to trigger threshold breaches. Verify sounds play.

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add optional sound effects (radar ping, geiger click, typewriter)"
```

---

### Task 14: Responsive Design + Polish

**Files:**
- Modify: `index.html` (CSS)

**Step 1: Add responsive breakpoints**

```css
@media (max-width: 1024px) {
    .main-grid {
        grid-template-columns: 1fr;
    }
    .sidebar {
        flex-direction: row;
    }
    .radar-container {
        width: 250px;
        height: 250px;
    }
    .instruments-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}
@media (max-width: 640px) {
    .sidebar {
        flex-direction: column;
    }
    .instruments-grid {
        grid-template-columns: 1fr;
    }
    .matrix-columns {
        grid-template-columns: 1fr;
    }
    .header h1 { font-size: 12px; letter-spacing: 1px; }
}
```

**Step 2: Add loading state**

Replace the initial `recalculate()` call path to show "INITIALIZING..." in all panels while data loads, then clear once first fetch completes.

**Step 3: Handle Three.js container resize**

```javascript
window.addEventListener('resize', () => {
    const container = document.getElementById('radar-mount');
    if (radarRenderer && container) {
        radarCamera.aspect = container.offsetWidth / container.offsetHeight;
        radarCamera.updateProjectionMatrix();
        radarRenderer.setSize(container.offsetWidth, container.offsetHeight);
    }
});
```

**Step 4: Verify**

Resize browser to mobile width. Verify layout stacks vertically without overflow.

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add responsive design and window resize handling"
```

---

### Task 15: Final Integration Test + Deploy Readiness

**Files:**
- Review: `index.html`

**Step 1: Full integration check**

Open `index.html` in Chrome. Verify end-to-end:
- [ ] Header shows title, LIVE indicator blinks, UTC clock ticks
- [ ] CRT effects: scanlines, noise, flicker all active
- [ ] Radar: 3D, rotates, blips visible, color matches composite
- [ ] All 6 instrument panels show data (or "--" if APIs blocked)
- [ ] Sparklines render in panels with data
- [ ] Zone badges show correct GREEN/AMBER/RED
- [ ] Impact Matrix shows sectors in Favored/Neutral/Avoid columns
- [ ] Signal ticker scrolls at bottom
- [ ] Dials: drag to turn, values update, panels recalculate live
- [ ] Double-click dial resets to live
- [ ] Ctrl+Shift+A opens admin panel
- [ ] Admin: change threshold, save, verify recalculation
- [ ] Admin: reset defaults works
- [ ] Sound toggle works (click [SND:OFF])
- [ ] Threshold breach triggers screen shake + panel pulse
- [ ] STALE DATA warning appears if APIs fail
- [ ] Responsive: resize to mobile, layout stacks

**Step 2: Fix any issues found**

**Step 3: Final commit**

```bash
git add index.html
git commit -m "feat: complete geopolitical risk dashboard v2.1"
```
