# Popup + Editorials + Branding Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add a first-time user orientation popup, 8 clickable editorial briefings, header/footer branding, and replace the sound effect with a clean beep.

**Architecture:** All changes in `index.html` — CSS additions, HTML additions, JS additions. No new files. The popup and editorial slide-out are both modal overlays rendered in the same CRT style. Editorial content is stored as a JS object and rendered on panel click.

**Tech Stack:** Vanilla HTML/CSS/JS (single-file app, no build step)

---

### Task 1: Header/Footer Branding + Sound Change

**Files:**
- Modify: `index.html`

**Step 1: Update header bar HTML (~line 861-872)**

Replace the header bar with a 3-column layout:

```html
<header class="header-bar">
    <div class="header-left">PUBLISHED BY OPTION PIT</div>
    <div class="header-title">OPTION PIT'S GEOPOLITICAL RISK DASHBOARD</div>
    <div class="header-controls">
        <span class="live-indicator">
            <span class="live-dot"></span>
            LIVE
        </span>
        <span id="utc-clock" class="utc-clock">--:--:-- UTC</span>
        <span id="last-update" class="last-update">LAST UPDATE: --:--:--</span>
        <button id="sound-toggle" class="sound-toggle" onclick="toggleSound()">[SND:OFF]</button>
        <button class="admin-toggle" onclick="openAdmin()">[ADMIN]</button>
        <button class="intel-toggle" onclick="openIntel()">[?] INTEL</button>
    </div>
</header>
```

**Step 2: Add header CSS**

Add after existing `.header-title` styles (~line 119-126):

```css
.header-left {
    font-size: 11px;
    letter-spacing: 2px;
    color: var(--text-dim);
    text-transform: uppercase;
    white-space: nowrap;
}

.header-bar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    /* existing styles kept */
}

.header-title {
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    /* existing font styles kept */
}

.intel-toggle {
    background: none;
    border: 1px solid var(--border-green);
    color: var(--text-dim);
    font-family: var(--font-mono);
    font-size: 14px;
    padding: 2px 8px;
    cursor: pointer;
    letter-spacing: 1px;
    transition: color 0.3s, border-color 0.3s, text-shadow 0.3s;
}

.intel-toggle:hover {
    color: var(--text-green);
    border-color: var(--text-green);
    text-shadow: var(--glow-green);
}
```

**Step 3: Add copyright to ticker bar (~line 1018-1038)**

Add a copyright span inside the `.ticker-bar` div, after `.ticker-track`:

```html
<span class="ticker-copyright">&copy; 2026 Option Pit. All rights reserved.</span>
```

CSS:
```css
.ticker-copyright {
    font-size: 10px;
    color: var(--text-dim);
    white-space: nowrap;
    padding: 0 12px;
    letter-spacing: 1px;
    flex-shrink: 0;
}
```

**Step 4: Replace playTypewriter with a clean beep (~line 1968-1977)**

Replace the `playTypewriter` function:

```javascript
function playTypewriter() {
    if (!soundEnabled || !audioCtx) return;
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    osc.connect(gain); gain.connect(audioCtx.destination);
    osc.frequency.value = 880; osc.type = 'sine';
    gain.gain.setValueAtTime(0.06, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.15);
    osc.start(audioCtx.currentTime); osc.stop(audioCtx.currentTime + 0.15);
}
```

**Step 5: Verify**

Open dashboard locally. Confirm:
- "PUBLISHED BY OPTION PIT" appears top-left
- Title is centered
- [?] INTEL button appears in header controls
- Copyright appears at right end of ticker bar
- Toggle sound on, trigger a signal change — hear a clean beep, not a click

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add branding (header, copyright) and clean beep sound"
```

---

### Task 2: First-Time Popup (HTML + CSS + JS)

**Files:**
- Modify: `index.html`

**Step 1: Add popup CSS**

Add after existing modal/admin styles:

```css
/* ============================================
   INTEL POPUP OVERLAY
   ============================================ */
.intel-overlay {
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: rgba(0, 0, 0, 0.85);
    z-index: 10000;
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s;
    pointer-events: none;
}

.intel-overlay.active {
    opacity: 1;
    pointer-events: all;
}

.intel-modal {
    background: var(--bg-panel);
    border: 1px solid var(--text-green);
    box-shadow: 0 0 30px rgba(51, 255, 51, 0.15), inset 0 0 60px rgba(0, 0, 0, 0.5);
    max-width: 640px;
    width: 90%;
    max-height: 80vh;
    overflow-y: auto;
    padding: 24px 28px;
    position: relative;
    font-family: var(--font-mono);
}

.intel-modal h2 {
    color: var(--text-amber);
    text-shadow: var(--glow-amber);
    font-size: 16px;
    letter-spacing: 3px;
    margin-bottom: 16px;
    text-transform: uppercase;
}

.intel-modal p, .intel-modal li {
    color: var(--text-green);
    font-size: 14px;
    line-height: 1.6;
    margin-bottom: 10px;
}

.intel-modal strong {
    color: var(--text-amber);
    text-shadow: none;
}

.intel-modal .intel-label {
    color: var(--text-amber);
    font-weight: bold;
}

.intel-modal ul {
    list-style: none;
    padding-left: 0;
}

.intel-modal ul li::before {
    content: '> ';
    color: var(--text-dim);
}

.intel-close {
    position: absolute;
    top: 10px; right: 14px;
    background: none;
    border: 1px solid var(--text-dim);
    color: var(--text-dim);
    font-family: var(--font-mono);
    font-size: 14px;
    padding: 2px 8px;
    cursor: pointer;
    transition: color 0.2s, border-color 0.2s;
}

.intel-close:hover {
    color: var(--text-red);
    border-color: var(--text-red);
}

.intel-buttons {
    display: flex;
    gap: 10px;
    margin-top: 18px;
    flex-wrap: wrap;
}

.intel-btn {
    background: none;
    border: 1px solid var(--text-green);
    color: var(--text-green);
    font-family: var(--font-mono);
    font-size: 13px;
    padding: 6px 14px;
    cursor: pointer;
    letter-spacing: 1px;
    transition: background 0.2s, color 0.2s;
}

.intel-btn:hover {
    background: var(--text-green);
    color: var(--bg-primary);
}

.intel-btn.primary {
    border-color: var(--text-amber);
    color: var(--text-amber);
}

.intel-btn.primary:hover {
    background: var(--text-amber);
    color: var(--bg-primary);
}

.intel-page-indicator {
    text-align: center;
    margin-top: 14px;
    color: var(--text-dim);
    font-size: 12px;
    letter-spacing: 2px;
}
```

**Step 2: Add popup HTML**

Add right before the closing `</div>` of `#crt-screen` (before the ticker bar, ~line 1017):

```html
<!-- ========== INTEL POPUP ========== -->
<div class="intel-overlay" id="intel-overlay">
    <div class="intel-modal">
        <button class="intel-close" onclick="closeIntel()">[X]</button>
        <div id="intel-content"></div>
        <div class="intel-buttons" id="intel-buttons"></div>
        <div class="intel-page-indicator" id="intel-page-indicator"></div>
    </div>
</div>
```

**Step 3: Add popup JS**

Add after the sound functions (~after line 1977):

```javascript
// ============================================
// Intel Popup System
// ============================================

const INTEL_PAGES_QUICK = [
    {
        title: 'WELCOME TO THE WAR ROOM',
        body: '<p>This dashboard is your early warning system. It watches 5 market instruments &mdash; like trip wires around a perimeter &mdash; and tells you when something moves that shouldn\'t.</p>' +
              '<p><strong>GREEN</strong> = all quiet. <strong style="color:var(--text-amber)">AMBER</strong> = something\'s stirring. <strong style="color:var(--text-red)">RED</strong> = battle stations.</p>',
        buttons: [
            { label: 'QUICK TOUR', action: 'next', primary: true },
            { label: 'FULL BRIEFING', action: 'fullBriefing' },
            { label: 'SKIP \u2014 I\'LL FIGURE IT OUT', action: 'close' }
        ]
    },
    {
        title: 'THE INSTRUMENT PANELS',
        body: '<p>Each of the 6 panels tracks a different threat. Think of them as radar screens, each watching a different part of the sky:</p>' +
              '<ul>' +
              '<li><strong>OIL</strong> \u2014 The supply-shock tripwire. When oil spikes, someone is fighting over it.</li>' +
              '<li><strong>DEFENSE</strong> \u2014 The conflict thermometer. Defense stocks rise when smart money smells gunpowder.</li>' +
              '<li><strong>VIX</strong> \u2014 The fear gauge. It measures how scared Wall Street is right now.</li>' +
              '<li><strong>GOLD</strong> \u2014 The bunker indicator. When gold surges, people are hiding.</li>' +
              '<li><strong>TREASURY</strong> \u2014 The recession radar. When short-term rates pass long-term rates, the economy is driving by looking in the rearview mirror.</li>' +
              '<li><strong>COMPOSITE</strong> \u2014 All five trip wires rolled into one threat level.</li>' +
              '</ul>' +
              '<p>Each panel shows <strong>live news headlines</strong> that update every 5 minutes. Click any headline to read the full story.</p>' +
              '<p><strong>Click any panel</strong> for the full intel briefing on why it matters and how we set the alarm levels.</p>'
    },
    {
        title: 'THE CONTROLS & DATA',
        body: '<p><strong>Scenario Dials</strong> (left sidebar): These are your "what if" sliders. Crank CONFLICT to 10 to see what a war does to markets. Crank CREDIT STRESS to simulate a 2008-style meltdown. Watch the panels react in real time.</p>' +
              '<p><strong>FAVORED / AVOID</strong> (bottom): Based on current threat levels, these are the sectors you want to own &mdash; or run from.</p>' +
              '<p><strong>"What the Traders Think"</strong> (bottom): Articles written by Option Pit\'s traders about the current conflict and market environment.</p>' +
              '<p><strong>Market data refreshes every 15 minutes. News updates every 5 minutes.</strong></p>' +
              '<p>Questions? Suggestions? Email <strong>support@optionpit.com</strong></p>',
        buttons: [
            { label: 'GOT IT \u2014 TAKE ME IN', action: 'close', primary: true }
        ]
    }
];

const INTEL_PAGES_FULL = [
    {
        title: 'THE COMPOSITE SCORE',
        body: '<p>The composite score takes all 5 instruments and rolls them into one number from 0 to 100.</p>' +
              '<p>Each instrument gets a weight based on how good it is at predicting trouble:</p>' +
              '<ul>' +
              '<li><strong>VIX</strong> gets the biggest share (30%) &mdash; it reacts to everything</li>' +
              '<li><strong>Oil</strong> and <strong>Treasury</strong> each get 20% &mdash; one watches supply shocks, the other watches recessions</li>' +
              '<li><strong>Defense</strong> and <strong>Gold</strong> each get 15% &mdash; they confirm what the others are saying</li>' +
              '</ul>' +
              '<p><strong>0-30 = GREEN.</strong> All quiet. <strong style="color:var(--text-amber)">30-60 = AMBER.</strong> Hedges up. <strong style="color:var(--text-red)">60+ = RED.</strong> Defensive posture.</p>'
    },
    {
        title: 'THE FAVORED/AVOID MATRIX',
        body: '<p>When threat levels change, some investments benefit and others get hurt. The matrix at the bottom scores 12 sectors based on how they historically react to each threat.</p>' +
              '<p>For example: when oil spikes, energy stocks win and airlines lose. When VIX spikes, cash and gold miners win, growth tech loses.</p>' +
              '<p>The arrows tell you how strong the signal is. One arrow = mild. Three arrows = strong conviction.</p>'
    },
    {
        title: 'THE SCENARIO DIALS',
        body: '<p>The three dials on the left let you stress-test the dashboard:</p>' +
              '<ul>' +
              '<li><strong>CONFLICT</strong> (1-10): Simulates military escalation. At max, oil jumps 25%, defense stocks surge 20%, and VIX climbs 15 points. Based on what actually happened during the Ukraine invasion.</li>' +
              '<li><strong>CREDIT STRESS</strong> (1-10): Simulates a financial crisis. At max, VIX spikes 35 points and the yield curve flattens hard. Based on the 2008 crash and COVID panic.</li>' +
              '<li><strong>SUPPLY DISRUPTION</strong> (1-10): Simulates a trade or energy blockade. At max, oil jumps 30%. Based on scenarios like a Strait of Hormuz closure.</li>' +
              '</ul>' +
              '<p>The dials stack &mdash; you can combine all three to model a worst-case scenario.</p>',
        buttons: [
            { label: 'GOT IT \u2014 TAKE ME IN', action: 'close', primary: true }
        ]
    }
];

let intelPages = [];
let intelPage = 0;
let intelFullMode = false;

function openIntel() {
    intelFullMode = false;
    intelPages = INTEL_PAGES_QUICK.slice();
    intelPage = 0;
    renderIntelPage();
    document.getElementById('intel-overlay').classList.add('active');
}

function closeIntel() {
    document.getElementById('intel-overlay').classList.remove('active');
    localStorage.setItem('riskTerminal_seen', '1');
}

function renderIntelPage() {
    const page = intelPages[intelPage];
    const contentEl = document.getElementById('intel-content');
    const buttonsEl = document.getElementById('intel-buttons');
    const indicatorEl = document.getElementById('intel-page-indicator');

    contentEl.innerHTML = '<h2>' + page.title + '</h2>' + page.body;

    // Buttons
    const btns = page.buttons || [
        { label: 'NEXT', action: 'next', primary: true },
        { label: 'BACK', action: 'back' }
    ];

    buttonsEl.innerHTML = btns.map(function(b) {
        const cls = b.primary ? 'intel-btn primary' : 'intel-btn';
        return '<button class="' + cls + '" onclick="intelAction(\'' + b.action + '\')">' + b.label + '</button>';
    }).join('');

    // Hide BACK on first page
    if (intelPage === 0) {
        buttonsEl.querySelectorAll('.intel-btn').forEach(function(btn) {
            if (btn.textContent === 'BACK') btn.style.display = 'none';
        });
    }

    indicatorEl.textContent = (intelPage + 1) + ' / ' + intelPages.length;
}

function intelAction(action) {
    if (action === 'close') {
        closeIntel();
    } else if (action === 'next') {
        if (intelPage < intelPages.length - 1) {
            intelPage++;
            renderIntelPage();
        }
    } else if (action === 'back') {
        if (intelPage > 0) {
            intelPage--;
            renderIntelPage();
        }
    } else if (action === 'fullBriefing') {
        intelFullMode = true;
        intelPages = INTEL_PAGES_QUICK.concat(INTEL_PAGES_FULL);
        intelPage = 1;
        renderIntelPage();
    }
}

// Show popup on first visit
if (!localStorage.getItem('riskTerminal_seen')) {
    window.addEventListener('DOMContentLoaded', function() {
        setTimeout(openIntel, 1500);
    });
}
```

**Step 4: Verify**

Clear localStorage and reload:
```javascript
localStorage.removeItem('riskTerminal_seen'); location.reload();
```

Confirm:
- Popup appears after 1.5s
- QUICK TOUR advances through 3 screens
- FULL BRIEFING shows all 6 screens
- SKIP closes immediately
- [?] INTEL button re-opens popup
- [X] closes popup
- Popup doesn't show on next visit

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add first-time intel popup with quick tour and full briefing"
```

---

### Task 3: Editorial Slide-Out Modal (CSS + JS framework)

**Files:**
- Modify: `index.html`

**Step 1: Add slide-out CSS**

Add after the intel popup CSS:

```css
/* ============================================
   EDITORIAL SLIDE-OUT
   ============================================ */
.editorial-overlay {
    position: fixed;
    top: 0; left: 0;
    width: 100%; height: 100%;
    background: rgba(0, 0, 0, 0.6);
    z-index: 10001;
    opacity: 0;
    transition: opacity 0.3s;
    pointer-events: none;
}

.editorial-overlay.active {
    opacity: 1;
    pointer-events: all;
}

.editorial-panel {
    position: fixed;
    top: 0; right: 0;
    width: 42%;
    min-width: 340px;
    max-width: 600px;
    height: 100%;
    background: var(--bg-panel);
    border-left: 1px solid var(--text-green);
    box-shadow: -4px 0 30px rgba(51, 255, 51, 0.1), inset 0 0 60px rgba(0, 0, 0, 0.5);
    overflow-y: auto;
    padding: 24px 28px;
    font-family: var(--font-mono);
    transform: translateX(100%);
    transition: transform 0.35s ease;
}

.editorial-overlay.active .editorial-panel {
    transform: translateX(0);
}

.editorial-panel h2 {
    color: var(--text-amber);
    text-shadow: var(--glow-amber);
    font-size: 15px;
    letter-spacing: 2px;
    margin-bottom: 18px;
    text-transform: uppercase;
    border-bottom: 1px solid var(--border-green);
    padding-bottom: 10px;
}

.editorial-panel h3 {
    color: var(--text-amber);
    font-size: 13px;
    letter-spacing: 2px;
    margin: 18px 0 8px 0;
    text-transform: uppercase;
}

.editorial-panel p {
    color: var(--text-green);
    font-size: 13px;
    line-height: 1.7;
    margin-bottom: 10px;
}

.editorial-panel .war-story {
    border-left: 2px solid var(--text-amber);
    padding-left: 10px;
    margin: 8px 0;
}

.editorial-panel .war-story strong {
    color: var(--text-amber);
}

.editorial-panel .zone-action {
    padding: 4px 0;
}

.editorial-panel .zone-green { color: var(--text-green); }
.editorial-panel .zone-amber { color: var(--text-amber); }
.editorial-panel .zone-red { color: var(--text-red); }

.editorial-close {
    position: sticky;
    top: 0;
    float: right;
    background: none;
    border: 1px solid var(--text-dim);
    color: var(--text-dim);
    font-family: var(--font-mono);
    font-size: 14px;
    padding: 2px 8px;
    cursor: pointer;
    z-index: 1;
    transition: color 0.2s, border-color 0.2s;
}

.editorial-close:hover {
    color: var(--text-red);
    border-color: var(--text-red);
}

@media (max-width: 768px) {
    .editorial-panel {
        width: 100%;
        max-width: 100%;
    }
}
```

**Step 2: Add slide-out HTML**

Add after the intel overlay HTML:

```html
<!-- ========== EDITORIAL SLIDE-OUT ========== -->
<div class="editorial-overlay" id="editorial-overlay" onclick="closeEditorial(event)">
    <div class="editorial-panel" id="editorial-panel">
        <button class="editorial-close" onclick="closeEditorial()">[X]</button>
        <div id="editorial-content"></div>
    </div>
</div>
```

**Step 3: Add slide-out JS**

```javascript
// ============================================
// Editorial Slide-Out System
// ============================================

function openEditorial(key) {
    const editorial = EDITORIALS[key];
    if (!editorial) return;
    document.getElementById('editorial-content').innerHTML = editorial;
    document.getElementById('editorial-overlay').classList.add('active');
}

function closeEditorial(e) {
    if (e && e.target && e.target.closest('.editorial-panel')) return;
    document.getElementById('editorial-overlay').classList.remove('active');
}
```

**Step 4: Wire up panel clicks**

Modify the existing panel click handlers. Each instrument panel already has `cursor: pointer` in CSS. Add `onclick` attributes to each panel div:

```html
<div class="instrument-panel" id="panel-oil" onclick="openEditorial('oil')">
<div class="instrument-panel" id="panel-defense" onclick="openEditorial('defense')">
<div class="instrument-panel" id="panel-vix" onclick="openEditorial('vix')">
<div class="instrument-panel" id="panel-gold" onclick="openEditorial('gold')">
<div class="instrument-panel" id="panel-treasury" onclick="openEditorial('treasury')">
<div class="instrument-panel" id="panel-composite" onclick="openEditorial('composite')">
```

Also wire the impact matrix and dials:

Add `onclick="openEditorial('matrix')"` to the `.impact-matrix` section.

Add `onclick="openEditorial('dials')"` to the dials container label or add a clickable label.

**Step 5: Verify**

Click any panel — slide-out should appear from right with placeholder content. Click outside or [X] to close. Verify it doesn't interfere with news headline clicks (those should still open in new tabs).

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add editorial slide-out modal framework with panel click triggers"
```

---

### Task 4: Write All 8 Editorial Briefings

**Files:**
- Modify: `index.html`

**Step 1: Add the EDITORIALS object**

Add the full editorial content as a JS object. Each value is an HTML string following the 5-section structure (WHAT IT IS, WHY IT'S ON THIS DASHBOARD, WHERE WE DREW THE LINES, WHAT THE COLORS TELL YOU TO DO, THIS HAS HAPPENED BEFORE).

```javascript
const EDITORIALS = {
    oil: '<h2>INTEL BRIEFING: CRUDE OIL &mdash; THE SUPPLY-SHOCK TRIPWIRE</h2>' +

         '<h3>WHAT IT IS</h3>' +
         '<p>Oil runs everything. Your car, your Amazon delivery, the plane you flew last week. When oil gets expensive, everything gets expensive. And oil gets expensive fast when people start fighting over it.</p>' +

         '<h3>WHY IT\'S ON THIS DASHBOARD</h3>' +
         '<p>Oil moves before the headlines do. Traders smell conflict and start buying oil futures before a single tank crosses a border. That makes oil a great alarm bell.</p>' +
         '<p>Think of it like a smoke detector. The house isn\'t on fire yet, but something is burning.</p>' +
         '<p>The Strait of Hormuz &mdash; a narrow strip of water near Iran &mdash; carries one out of every five barrels of oil on the planet. One missile, one blocked ship, and oil doesn\'t creep up. It jumps.</p>' +

         '<h3>WHERE WE DREW THE LINES</h3>' +
         '<p>We pulled 20 years of oil prices (2005-2025) and looked at what happened at each level:</p>' +
         '<p><strong>Below $80</strong> &mdash; Oil sat below this price 70 out of every 100 years we checked. Saudi Arabia needs about $81 a barrel just to pay its bills. Below $80, nobody is sweating.</p>' +
         '<p><strong>$80 to $95</strong> &mdash; Now we\'re in the top 30% of the 20-year range. Libya\'s civil war in 2011 kept oil above $90 all year. The months before Russia invaded Ukraine looked like this too. Something is cooking.</p>' +
         '<p><strong>Above $95</strong> &mdash; Only one year out of twenty stayed above $95 &mdash; and that was 2008, right before the financial crisis hit. JPMorgan calls this the "demand destruction" zone. That means regular people stop driving, stop flying, and start cutting back. It hurts.</p>' +

         '<h3>WHAT THE COLORS TELL YOU TO DO</h3>' +
         '<p class="zone-action zone-green"><strong>GREEN</strong> (under $80): Cruise control. No action needed.</p>' +
         '<p class="zone-action zone-amber"><strong>AMBER</strong> ($80-$95): Pay attention. Airlines and shipping companies start to feel the squeeze.</p>' +
         '<p class="zone-action zone-red"><strong>RED</strong> (above $95): Energy stocks win. Airlines and anything that burns fuel loses. Consumer spending takes a hit.</p>' +

         '<h3>THIS HAS HAPPENED BEFORE</h3>' +
         '<div class="war-story"><strong>Ukraine, 2022:</strong> Russia invaded. Oil shot from $75 to $133 in two weeks flat. Airlines dropped 35%. Energy stocks gained 65% that year while everything else bled.</div>' +
         '<div class="war-story"><strong>Saudi Arabia, 2019:</strong> A drone hit one oil facility. One. Oil jumped 14% in a single day. It calmed down in a few weeks, but it proved how fast this alarm can go off.</div>' +
         '<div class="war-story"><strong>The 2008 Spike:</strong> Oil hit $147 in July. By December it crashed to $33. The spike warned us. The crash confirmed the recession was real.</div>',

    defense: '<h2>INTEL BRIEFING: DEFENSE STOCKS &mdash; THE CONFLICT THERMOMETER</h2>' +

         '<h3>WHAT IT IS</h3>' +
         '<p>We track four of the biggest defense companies: Lockheed Martin, Raytheon, Northrop Grumman, and General Dynamics. When their stocks all jump at once, it means big money is betting on more conflict.</p>' +

         '<h3>WHY IT\'S ON THIS DASHBOARD</h3>' +
         '<p>Defense stocks are the market\'s way of placing bets on war. These companies build missiles, fighter jets, and missile defense systems. When tensions rise, governments order more hardware. Wall Street knows this and moves first.</p>' +
         '<p>Think of defense stocks as a thermometer for geopolitical heat. The higher they climb, the hotter things are getting.</p>' +

         '<h3>WHERE WE DREW THE LINES</h3>' +
         '<p>We measure the average daily percentage change across all four stocks. Here\'s what the numbers mean:</p>' +
         '<p><strong>Under 3%</strong> &mdash; Normal trading. Defense stocks move around like everything else. No signal here.</p>' +
         '<p><strong>3% to 8%</strong> &mdash; Something caught Wall Street\'s attention. A new conflict, a big weapons deal, or troop movements. Worth watching.</p>' +
         '<p><strong>Over 8%</strong> &mdash; This is what happened when Russia invaded Ukraine. Lockheed Martin jumped 37% in weeks. Northrop Grumman gained 41%. When defense stocks surge this hard, the smart money is screaming that a real conflict is underway.</p>' +

         '<h3>WHAT THE COLORS TELL YOU TO DO</h3>' +
         '<p class="zone-action zone-green"><strong>GREEN</strong> (under 3%): Standard allocation. Business as usual.</p>' +
         '<p class="zone-action zone-amber"><strong>AMBER</strong> (3%-8%): Consider adding aerospace and defense names to your portfolio.</p>' +
         '<p class="zone-action zone-red"><strong>RED</strong> (over 8%): Full overweight on defense. These stocks tend to keep climbing once a real conflict starts.</p>' +

         '<h3>THIS HAS HAPPENED BEFORE</h3>' +
         '<div class="war-story"><strong>Ukraine, 2022:</strong> Russia invaded on February 24. The S&P fell 19% that year. The defense sector gained 8.6%. Individual names did even better &mdash; Northrop Grumman returned 41%.</div>' +
         '<div class="war-story"><strong>Israel-Hamas, October 2023:</strong> Defense stocks popped 5-8% in the weeks after October 7. The market treated it as a localized conflict, not a global one &mdash; so the spike was real but contained.</div>' +
         '<div class="war-story"><strong>Iraq, 2003:</strong> Defense stocks rallied 12-18% during the lead-up to the invasion. Once the "mission accomplished" banner went up, they gave some back. Timing matters.</div>',

    vix: '<h2>INTEL BRIEFING: VIX &mdash; THE FEAR GAUGE</h2>' +

         '<h3>WHAT IT IS</h3>' +
         '<p>The VIX is Wall Street\'s fear meter. It measures how much traders expect the stock market to bounce around over the next 30 days. When the number is low, people are calm. When it spikes, people are panicking.</p>' +

         '<h3>WHY IT\'S ON THIS DASHBOARD</h3>' +
         '<p>The VIX catches everything. Wars, financial crises, pandemics, surprise elections &mdash; they all show up here. It\'s the broadest alarm we have.</p>' +
         '<p>Think of it like a seismograph. It doesn\'t tell you what caused the earthquake. It tells you how big it is.</p>' +

         '<h3>WHERE WE DREW THE LINES</h3>' +
         '<p>We looked at every VIX reading since 1990. The long-run average sits around 19.4. Here\'s what the levels mean:</p>' +
         '<p><strong>Below 20</strong> &mdash; Calm seas. The VIX spends about two-thirds of its time below 20. Markets are functioning normally. Every major financial source from Bankrate to the CBOE itself uses 20 as the "normal" line.</p>' +
         '<p><strong>20 to 30</strong> &mdash; Choppy water. Something is making traders nervous. The Ukraine invasion pushed VIX into the mid-30s. Year-end 2022 closed at 21.67. You\'re in the top 30% of all readings.</p>' +
         '<p><strong>Above 30</strong> &mdash; Storm. Every time the VIX closed above 30, something serious was happening. The 2008 financial crisis pushed it to 80. COVID hit 82. The 2025 tariff shock broke 60. This is the top 10% of all readings in history.</p>' +

         '<h3>WHAT THE COLORS TELL YOU TO DO</h3>' +
         '<p class="zone-action zone-green"><strong>GREEN</strong> (under 20): Risk-on. Growth stocks and speculative positions are comfortable here.</p>' +
         '<p class="zone-action zone-amber"><strong>AMBER</strong> (20-30): Hedge your tail risk. This is where you buy protection, not sell it.</p>' +
         '<p class="zone-action zone-red"><strong>RED</strong> (above 30): Full portfolio protection. Cash, gold, and defensive names only. Growth and speculative assets get crushed at these levels.</p>' +

         '<h3>THIS HAS HAPPENED BEFORE</h3>' +
         '<div class="war-story"><strong>COVID, March 2020:</strong> The VIX hit 82.69 on March 16 &mdash; the highest closing level in history. The S&P had just fallen 34% in 23 trading days. Anyone holding protection made a fortune.</div>' +
         '<div class="war-story"><strong>2008 Financial Crisis:</strong> The VIX closed at 80.86 on November 20. Banks were failing. The world felt like it was ending. It took over a year for VIX to settle below 30.</div>' +
         '<div class="war-story"><strong>Volmageddon, February 2018:</strong> The VIX doubled in a single day. Products that bet against volatility blew up overnight. XIV, a popular short-VIX product, lost 96% of its value in hours and was shut down.</div>',

    gold: '<h2>INTEL BRIEFING: GOLD &mdash; THE BUNKER INDICATOR</h2>' +

         '<h3>WHAT IT IS</h3>' +
         '<p>Gold is the oldest safe haven on the planet. When people get scared &mdash; really scared &mdash; they buy gold. They\'ve been doing it for 5,000 years. That hasn\'t changed.</p>' +

         '<h3>WHY IT\'S ON THIS DASHBOARD</h3>' +
         '<p>Gold tells you when the big players are running for cover. Central banks, sovereign wealth funds, billionaire family offices &mdash; when these players start stacking gold, it means they don\'t trust paper currencies or stock markets to hold up.</p>' +
         '<p>Think of gold as a bunker. Nobody builds one when the weather is nice. When you see construction crews pouring concrete, something bad is coming.</p>' +

         '<h3>WHERE WE DREW THE LINES</h3>' +
         '<p>Gold is tricky because it\'s in a historic bull run. Central banks bought over 1,000 tonnes a year for three straight years (2022-2024). That changed the game.</p>' +
         '<p><strong>Below $5,100</strong> &mdash; This is where gold sat before the Iran military escalation in February 2026. It\'s the "new normal" floor created by central bank buying. Elevated compared to history, but stable for the current environment.</p>' +
         '<p><strong>$5,100 to $5,700</strong> &mdash; Gold is running hot. There\'s a fear premium baked in on top of the structural demand. People are nervous but not panicking yet.</p>' +
         '<p><strong>Above $5,700</strong> &mdash; This is above the all-time high of $5,589 set on January 28, 2026. If gold breaks $5,700, something genuinely new and scary is happening that the market hasn\'t priced in before.</p>' +

         '<h3>WHAT THE COLORS TELL YOU TO DO</h3>' +
         '<p class="zone-action zone-green"><strong>GREEN</strong> (under $5,100): Standard precious metals allocation. No panic.</p>' +
         '<p class="zone-action zone-amber"><strong>AMBER</strong> ($5,100-$5,700): Increase your gold and precious metals exposure. The market is telling you to hedge.</p>' +
         '<p class="zone-action zone-red"><strong>RED</strong> (above $5,700): Flight to safety is active. This is a full-on risk-off environment. Gold miners, physical gold, and cash are your friends.</p>' +

         '<h3>THIS HAS HAPPENED BEFORE</h3>' +
         '<div class="war-story"><strong>2025 &mdash; The Year Gold Went Vertical:</strong> Gold started the year at $2,600 and ended at $4,310 &mdash; up 65%. It set 53 new all-time highs. Central banks couldn\'t buy it fast enough.</div>' +
         '<div class="war-story"><strong>Ukraine, 2022:</strong> Gold spiked to $2,074 on March 8. But then the Fed raised rates aggressively, and gold fell to $1,656 by October. Lesson: gold\'s signal is strongest when the Fed isn\'t fighting it.</div>' +
         '<div class="war-story"><strong>COVID, 2020:</strong> Gold briefly dipped in March as investors sold everything for cash. Then it rocketed to $2,067 by August as the Fed printed trillions. The dip was a fake-out. The surge was real.</div>',

    treasury: '<h2>INTEL BRIEFING: TREASURY YIELD CURVE &mdash; THE RECESSION RADAR</h2>' +

         '<h3>WHAT IT IS</h3>' +
         '<p>We track the difference between 10-year and 2-year Treasury yields. Normally, you earn more for lending money for 10 years than for 2 years. That makes sense &mdash; more time, more risk, more reward.</p>' +
         '<p>When that flips &mdash; when 2-year yields pay more than 10-year yields &mdash; it\'s called an "inversion." It means the bond market thinks the near future is more dangerous than the distant future. That\'s a recession warning.</p>' +

         '<h3>WHY IT\'S ON THIS DASHBOARD</h3>' +
         '<p>The yield curve has predicted 6 out of the last 7 recessions. That\'s an 87% hit rate going back decades. The Federal Reserve Bank of New York, the San Francisco Fed, and academic researchers all agree: when this curve inverts, pay attention.</p>' +
         '<p>Think of it like a canary in a coal mine. The bird doesn\'t cause the gas leak, but when it stops singing, you get out.</p>' +

         '<h3>WHERE WE DREW THE LINES</h3>' +
         '<p>We studied every inversion since 1976 and cross-checked with Federal Reserve research:</p>' +
         '<p><strong>Above +0.50%</strong> &mdash; The long-run median spread is about +0.79%. Above +0.50%, the curve is healthy. The economy is borrowing, lending, and growing normally.</p>' +
         '<p><strong>0% to +0.50%</strong> &mdash; The curve is flattening. Both the 2006 pre-financial-crisis period and the 2019 pre-COVID period spent months in this zone before inverting. It\'s the "something\'s not right" zone.</p>' +
         '<p><strong>Below 0% (inverted)</strong> &mdash; Red alert. Any inversion &mdash; even just barely crossing zero &mdash; has historically been a recession signal. The 2006 inversion only reached -0.19% and it still preceded the worst financial crisis since the Depression. The 2022-2024 inversion hit -1.08%, the deepest since 1981.</p>' +

         '<h3>WHAT THE COLORS TELL YOU TO DO</h3>' +
         '<p class="zone-action zone-green"><strong>GREEN</strong> (above +0.50%): Growth outlook is positive. Normal allocations work.</p>' +
         '<p class="zone-action zone-amber"><strong>AMBER</strong> (0% to +0.50%): Caution on duration. Start thinking about what a slowdown means for your portfolio.</p>' +
         '<p class="zone-action zone-red"><strong>RED</strong> (below 0%): Recession signal active. Reduce cyclical exposure. Banks, REITs, and growth stocks historically suffer most. Cash and short-duration bonds are your shelter.</p>' +

         '<h3>THIS HAS HAPPENED BEFORE</h3>' +
         '<div class="war-story"><strong>2006-2008:</strong> The curve inverted in late 2005 &mdash; barely, just -0.19%. Most people shrugged it off. Two years later, Lehman Brothers collapsed and the global financial system nearly ended. The canary was right.</div>' +
         '<div class="war-story"><strong>2019:</strong> A brief inversion in August 2019 at just -0.05%. Six months later, COVID hit. The inversion didn\'t cause the pandemic, but the economy was already fragile enough to crack.</div>' +
         '<div class="war-story"><strong>2022-2024:</strong> The longest and deepest inversion in modern history. It lasted 25 months and hit -1.08%. As of early 2026, no official recession has been called &mdash; making it potentially the first false positive. But the debate isn\'t over.</div>',

    composite: '<h2>INTEL BRIEFING: COMPOSITE RISK &mdash; THE WAR ROOM SCORE</h2>' +

         '<h3>WHAT IT IS</h3>' +
         '<p>The composite score takes all five instruments and boils them down to a single number from 0 to 100. Zero means every trip wire is quiet. A hundred means every alarm is screaming.</p>' +

         '<h3>WHY IT EXISTS</h3>' +
         '<p>Individual instruments can give false signals. Oil might spike because of an OPEC meeting, not a war. VIX might jump because of an earnings season, not a crisis. The composite smooths out the noise by requiring multiple trip wires to trigger before it moves.</p>' +
         '<p>Think of it like a vote. One alarm going off is worth investigating. Three alarms going off at once means the building is on fire.</p>' +

         '<h3>HOW THE MATH WORKS</h3>' +
         '<p>Each instrument gets a weight based on how reliably it predicts broad market stress:</p>' +
         '<p><strong>VIX: 30%</strong> &mdash; It gets the biggest share because it reacts to everything &mdash; wars, pandemics, financial crises, political shocks.</p>' +
         '<p><strong>Oil: 20%</strong> &mdash; The primary channel for geopolitical supply shocks hitting the real economy.</p>' +
         '<p><strong>Treasury: 20%</strong> &mdash; The best long-term leading indicator. It moves slowly but it\'s almost always right.</p>' +
         '<p><strong>Defense: 15%</strong> &mdash; A specialized military/geopolitical signal. It gets less weight because it doesn\'t react to financial crises or recessions.</p>' +
         '<p><strong>Gold: 15%</strong> &mdash; Confirms flight-to-safety but is noisier now due to central bank buying. It\'s a confirming signal, not a leading one.</p>' +
         '<p>Each instrument scores 0 (GREEN), 50 (AMBER), or 100 (RED). The weighted average becomes the composite.</p>' +

         '<h3>WHAT THE SCORE TELLS YOU</h3>' +
         '<p class="zone-action zone-green"><strong>GREEN</strong> (0-30): All systems nominal. Standard risk allocation.</p>' +
         '<p class="zone-action zone-amber"><strong>AMBER</strong> (30-60): Increase hedging. Review allocations. Something is moving.</p>' +
         '<p class="zone-action zone-red"><strong>RED</strong> (60-100): Full defensive posture. This is the "sell what lets you sleep at night" zone.</p>',

    matrix: '<h2>INTEL BRIEFING: THE FAVORED/AVOID MATRIX</h2>' +

         '<h3>WHAT IT IS</h3>' +
         '<p>The matrix at the bottom of the dashboard scores 12 investment sectors and tells you which ones historically do well when risk rises &mdash; and which ones get crushed.</p>' +

         '<h3>HOW IT WORKS</h3>' +
         '<p>Each sector has a sensitivity score to each of the five instruments. We built these scores by studying how sector ETFs actually performed during real crises &mdash; the 2008 crash, COVID, Ukraine, the 2025 tariff shock.</p>' +
         '<p>Think of it like a weather report for sectors. Some crops thrive in a storm (defense, energy, gold). Others wilt (airlines, growth tech, real estate).</p>' +

         '<h3>THE KEY RELATIONSHIPS</h3>' +
         '<p><strong>When oil spikes:</strong> Energy stocks win. Airlines lose. It\'s simple &mdash; fuel is their biggest cost.</p>' +
         '<p><strong>When VIX spikes:</strong> Cash and consumer staples win. Growth tech and crypto lose. Scared money runs to safety.</p>' +
         '<p><strong>When gold surges:</strong> Gold miners win obviously. Growth tech tends to lose because the same fear that drives gold kills risk appetite.</p>' +
         '<p><strong>When the yield curve inverts:</strong> REITs and long-duration bonds get hit hardest because they\'re the most sensitive to interest rates. Banks lose their profit margin.</p>' +
         '<p><strong>When defense stocks surge:</strong> Defense keeps winning. Airlines and emerging markets tend to lose because conflict disrupts trade routes and airspace.</p>' +

         '<h3>THE ARROWS</h3>' +
         '<p>One arrow = mild signal. Two arrows = moderate. Three arrows = strong conviction. A diamond means neutral &mdash; this sector doesn\'t care much about what\'s happening.</p>' +

         '<h3>WHERE THE NUMBERS CAME FROM</h3>' +
         '<p>We sourced these from IMF research, CFA Institute studies, Federal Reserve publications, and actual ETF performance data during crisis periods. Every weight has a real-world basis. See each instrument\'s briefing for the specific crisis data.</p>',

    dials: '<h2>INTEL BRIEFING: THE SCENARIO DIALS</h2>' +

         '<h3>WHAT THEY ARE</h3>' +
         '<p>The three dials on the left sidebar let you play "what if" with the dashboard. They simulate different types of crises by adjusting the live data, so you can see how the dashboard would react before the crisis actually hits.</p>' +

         '<h3>THE THREE SCENARIOS</h3>' +
         '<p><strong>CONFLICT (1-10):</strong> Simulates military escalation. Turning this up is like asking: "What if a war breaks out?" At maximum:</p>' +
         '<ul style="list-style:none;padding-left:0;">' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Oil jumps 25% (Ukraine saw +77%)</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Defense stocks surge 20%</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Gold climbs 12%</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>VIX adds 15 points</li>' +
         '</ul>' +

         '<p><strong>CREDIT STRESS (1-10):</strong> Simulates a financial crisis. Think 2008 or the early days of COVID. At maximum:</p>' +
         '<ul style="list-style:none;padding-left:0;">' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>VIX spikes 35 points (2008 saw +60)</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Gold rises 10%</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Yield curve flattens by 0.8%</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Defense dips 5% (war isn\'t the problem here)</li>' +
         '</ul>' +

         '<p><strong>SUPPLY DISRUPTION (1-10):</strong> Simulates a blockade or trade war. Think Strait of Hormuz closure or severe sanctions. At maximum:</p>' +
         '<ul style="list-style:none;padding-left:0;">' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Oil jumps 30%</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Gold rises 8%</li>' +
         '<li style="color:var(--text-green);font-size:13px;margin:4px 0;"><span style="color:var(--text-dim);">&gt; </span>Yield curve flattens by 0.4%</li>' +
         '</ul>' +

         '<h3>HOW TO USE THEM</h3>' +
         '<p>Start with one dial at a time. Crank CONFLICT to 7 and watch what happens to the panels. Then reset and try CREDIT STRESS. Once you understand each one individually, combine them to model a worst-case scenario.</p>' +
         '<p>The dials stack. CONFLICT at 5 plus CREDIT STRESS at 5 is worse than either one alone. That\'s by design &mdash; real crises rarely come in just one flavor.</p>' +

         '<h3>WHERE THE NUMBERS CAME FROM</h3>' +
         '<p>Every multiplier is based on what actually happened during real crises. The oil +25% for conflict comes from Ukraine (which actually saw +77% &mdash; we\'re being conservative). The VIX +35 for credit stress comes from the 2008 crash (which saw +60). These are stress tests, not worst-case fantasies.</p>'
};
```

**Step 2: Verify**

Click each of the 6 instrument panels and verify each editorial loads correctly with proper formatting. Click the FAVORED/AVOID area and the dial labels to test those two editorials.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add 8 editorial briefings for all dashboard components

Oil, Defense, VIX, Gold, Treasury, Composite, Matrix, Dials.
Written in OP editorial style with crisis war stories and
empirical methodology explained at sub-7th-grade reading level."
```

---

### Task 5: Final Smoke Test

**Step 1: Full reload with cleared state**

```javascript
localStorage.removeItem('riskTerminal_seen');
localStorage.removeItem('riskTerminal_thresholds');
location.reload();
```

**Step 2: Verify popup flow**
- Popup appears on load
- Quick tour: 3 screens, navigation works
- Full briefing: 6 screens total
- Skip closes immediately
- [?] INTEL re-opens popup
- Popup doesn't auto-show on next visit

**Step 3: Verify editorials**
- Click each of 6 panels: editorial slides in from right
- Click outside: closes
- Click [X]: closes
- News headline clicks still open in new tab (don't trigger editorial)
- FAVORED/AVOID click opens matrix editorial
- Dial label click opens dials editorial

**Step 4: Verify branding**
- "PUBLISHED BY OPTION PIT" top-left
- Title centered
- Copyright in ticker bar
- Sound is a clean beep

**Step 5: Check for JS errors**
Open browser console. Confirm no errors on load or during interactions.
