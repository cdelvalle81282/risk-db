# Empirical Calibration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace all hardcoded/arbitrary values in the Risk Dashboard with empirically derived numbers from historical data.

**Architecture:** Four edits to constants/logic in a single file (`index.html`). Each task updates one constant block, is independently verifiable, and gets its own commit.

**Tech Stack:** Vanilla JavaScript (single-file app, no build step, no tests)

---

### Task 1: Update Zone Thresholds

**Files:**
- Modify: `index.html:1088-1095`

**Step 1: Edit DEFAULT_THRESHOLDS**

Replace lines 1088-1095 with:

```javascript
const DEFAULT_THRESHOLDS = {
    oil: { green: 80, amber: 95 },
    defense: { green: 3, amber: 8 },
    vix: { green: 20, amber: 30 },
    gold: { green: 5100, amber: 5700 },
    treasury: { green: 0.50, amber: 0 },
    weights: { oil: 0.20, defense: 0.20, vix: 0.25, gold: 0.15, treasury: 0.20 }
};
```

Note: weights stay unchanged here — they are updated in Task 3.

**Step 2: Clear localStorage overrides**

Users may have old thresholds cached. Open browser console on the dashboard and run:
```javascript
localStorage.removeItem('riskTerminal_thresholds');
location.reload();
```

**Step 3: Verify**

Open the dashboard. Confirm:
- Oil panel shows GREEN if current WTI is below $80 (likely ~$72-76 in March 2026)
- Treasury panel reflects the new +0.50 GREEN boundary (current spread ~+59-66bps should be GREEN)
- Gold panel reflects $5,100/$5,700 thresholds
- Defense panel uses 3%/8% thresholds

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: calibrate zone thresholds from 20yr percentile + crisis data

Oil GREEN $75->$80 (70th pctile, Saudi breakeven)
Defense GREEN 2%->3%, RED 5%->8% (Ukraine invasion benchmarks)
Gold GREEN $5000->$5100, RED $5500->$5700 (structural regime shift)
Treasury GREEN 0->+0.50, RED -0.5->0 (any inversion = RED per Estrella-Mishkin)"
```

---

### Task 2: Update Sector Sensitivity Matrix

**Files:**
- Modify: `index.html:1528-1541`

**Step 1: Edit SECTOR_SENSITIVITY**

Replace lines 1528-1541 with:

```javascript
const SECTOR_SENSITIVITY = {
    'Defense/Aerospace':    { oil: 0.4,  vix: 0.5,  gold: 0.1,  treasury: -0.2,  defense: 1.0  },
    'Energy/Oil Majors':    { oil: 1.0,  vix: 0.2,  gold: 0.3,  treasury: -0.1,  defense: 0.4  },
    'Gold/Precious Metals': { oil: 0.3,  vix: 0.7,  gold: 1.0,  treasury: -0.3,  defense: 0.3  },
    'Cash/Money Market':    { oil: 0,    vix: 0.8,  gold: 0,    treasury: 0.8,   defense: 0    },
    'Growth Tech':          { oil: -0.6, vix: -0.9, gold: -0.2, treasury: -0.7,  defense: -0.1 },
    'Airlines/Transport':   { oil: -1.0, vix: -0.6, gold: 0,    treasury: -0.2,  defense: -0.4 },
    'EM Equities':          { oil: -0.5, vix: -0.8, gold: 0.2,  treasury: -0.6,  defense: -0.4 },
    'Banks/Financials':     { oil: -0.2, vix: -0.6, gold: -0.2, treasury: 0.6,   defense: 0    },
    'Real Estate/REITs':    { oil: -0.3, vix: -0.5, gold: 0.1,  treasury: -0.9,  defense: -0.1 },
    'Consumer Staples':     { oil: -0.1, vix: 0.4,  gold: 0.1,  treasury: -0.1,  defense: 0    },
    'Long-duration Bonds':  { oil: -0.1, vix: 0.3,  gold: 0.4,  treasury: -1.0,  defense: -0.2 },
    'Crypto/Speculative':   { oil: -0.5, vix: -0.8, gold: 0,    treasury: -0.5,  defense: 0    }
};
```

**Step 2: Verify**

Open the dashboard. Check the FAVORED/NEUTRAL/AVOID columns at the bottom:
- In current calm conditions (most zones GREEN), most sectors should be NEUTRAL
- Toggle conflict dial to 7-8: Defense/Aerospace and Energy should appear in FAVORED; Airlines and Growth Tech in AVOID
- Toggle credit stress dial to 7-8: Cash/Money Market and Gold should appear FAVORED; Growth Tech, Crypto, EM Equities in AVOID

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: calibrate sector sensitivities from crisis-period correlations

Sources: IMF GFSR 2025, CFA Institute, NBER, Chicago Fed, Nareit, FactSet.
Key changes: Consumer Staples VIX +0.2->+0.4 (defensive rotation),
Long Bonds VIX +0.5->+0.3 (2022 stock-bond correlation breakdown),
EM oil -0.8->-0.5 (index includes oil exporters),
treasury sensitivities added across board."
```

---

### Task 3: Update Composite Weights

**Files:**
- Modify: `index.html:1088-1095` (the weights line within DEFAULT_THRESHOLDS, already modified in Task 1)

**Step 1: Edit weights**

Find the `weights:` line inside `DEFAULT_THRESHOLDS` (should be near line 1094 after Task 1 edits) and change:

```javascript
weights: { oil: 0.20, defense: 0.15, vix: 0.30, gold: 0.15, treasury: 0.20 }
```

**Step 2: Verify**

Open the dashboard. Check the COMPOSITE RISK panel (bottom-right of the 2x3 grid):
- With all zones GREEN, composite should read 0
- Toggle VIX to AMBER (set conflict dial ~3): composite should move more than before (VIX now weighted 30% vs 25%)
- Toggle defense to AMBER: composite should move less than before (defense now 15% vs 20%)

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: reweight composite score — VIX 25%->30%, defense 20%->15%

VIX is the strongest real-time predictor of equity drawdowns.
Defense stocks are a narrow geopolitical-only signal."
```

---

### Task 4: Update Scenario Dial Multipliers

**Files:**
- Modify: `index.html:1223-1248` (the applyScenario function body)

**Step 1: Edit conflict multipliers**

Replace the conflict block (lines ~1223-1231):

```javascript
// Conflict (1-10): scales defense +20%, oil +25%, gold +12%, VIX +15pts at max
if (s.conflict > 1) {
    const factor = (s.conflict - 1) / 9;
    effective.defense.price *= (1 + 0.20 * factor);
    effective.defense.changePercent += factor * 20;
    effective.oil.price *= (1 + 0.25 * factor);
    effective.gold.price *= (1 + 0.12 * factor);
    effective.vix.price += 15 * factor;
}
```

**Step 2: Edit credit stress multipliers**

Replace the credit stress block (lines ~1233-1240):

```javascript
// Credit Stress (1-10): VIX +35pts, gold +10%, treasury spread flattens 0.8, defense -5% at max
if (s.credit > 1) {
    const factor = (s.credit - 1) / 9;
    effective.vix.price += 35 * factor;
    effective.gold.price *= (1 + 0.10 * factor);
    effective.treasury.spread -= 0.8 * factor;
    effective.defense.price *= (1 - 0.05 * factor);
}
```

**Step 3: Edit supply multipliers**

Replace the supply block (lines ~1242-1248):

```javascript
// Supply (1-10): oil +30%, gold +8%, spread flattens 0.4 at max
if (s.supply > 1) {
    const factor = (s.supply - 1) / 9;
    effective.oil.price *= (1 + 0.30 * factor);
    effective.gold.price *= (1 + 0.08 * factor);
    effective.treasury.spread -= 0.4 * factor;
}
```

**Step 4: Verify**

Open the dashboard. Test scenario dials:
- Set conflict to 10: oil should spike ~25% from live price, VIX should add ~15pts (not 20 as before)
- Set credit stress to 10: VIX should add ~35pts (was 25), spread should flatten by 0.8 (was 0.5)
- Set supply to 10: oil should spike ~30% (was 20%)
- Combine conflict 7 + credit 5 + supply 6: dashboard should show multiple RED zones and FAVORED should show Defense, Energy, Gold, Cash

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: calibrate scenario dial multipliers from historical crisis magnitudes

Conflict: oil +15%->+25% (Ukraine +77%), VIX +20->+15pts (Ukraine only +14-16pts)
Credit: VIX +25->+35pts (GFC pushed +60pts), spread -0.5->-0.8 (2022 hit -108bps)
Supply: oil +20%->+30% (Hormuz scenario), gold +5%->+8% (inflation pass-through)"
```

---

### Task 5: Final Smoke Test

**Step 1: Full page reload with cleared cache**

```javascript
localStorage.removeItem('riskTerminal_thresholds');
location.reload();
```

**Step 2: Verify all panels render without JS errors**

Open browser DevTools console. Confirm no errors.

**Step 3: Verify live data still loads**

Wait for data fetch (up to 15 minutes on first load). Confirm:
- Oil, VIX, Gold prices appear with correct zone colors
- Treasury spread shows and zone color matches new thresholds
- Defense % change displays
- Composite score calculates
- Signal ticker updates
- FAVORED/NEUTRAL/AVOID matrix populates

**Step 4: Test admin panel**

Click ADMIN button. Verify threshold inputs show the new default values ($80, $95 for oil, etc.). Adjust one threshold, save, confirm it takes effect, then reset to defaults.
