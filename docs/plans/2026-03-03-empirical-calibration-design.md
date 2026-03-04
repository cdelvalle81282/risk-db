# Empirical Calibration Design — Geopolitical Risk Dashboard

**Date:** 2026-03-03
**Methodology:** Hybrid — 20-year statistical percentiles cross-validated against known geopolitical/financial crisis events.
**Approach:** Research and hardcode empirically justified values. No runtime computation needed.

---

## 1. Revised Zone Thresholds

### Oil (WTI Crude)
- **GREEN < $80** (was $75) — 70th percentile of 20yr annual avgs; Saudi fiscal breakeven $80.90 (IMF 2023)
- **AMBER $80–$95** — Spans 70th–90th percentile; Libya 2011 ($94.88 avg), Ukraine 2022 pre-invasion trajectory
- **RED > $95** (unchanged) — 90th percentile; demand destruction confirmed (JPMorgan); only 2008 exceeded $95 as annual avg

Sources: EIA, Statista, JPMorgan demand destruction analysis, IMF Saudi breakeven

### VIX
- **No change: GREEN < 20, AMBER 20–30, RED > 30**
- 20 = practitioner consensus + long-run mean (19.39); 30 = 90th percentile; every close above 30 was a confirmed systemic event
- Validated by Bankrate, S&P Indexology, Bansal & Stivers (2025, *Financial Management*)

Sources: FRED, Macroption, CBOE, volatilitytradingstrategies.com

### Gold
- **GREEN < $5,100** (was $5,000) — Pre-Iran-escalation baseline (Feb 2026); structural floor from 3yr central bank buying (1,000+ tonnes/yr)
- **AMBER $5,100–$5,700** — Elevated above structural baseline by 1–12%
- **RED > $5,700** (was $5,500) — ~2% above ATH ($5,589 Jan 28 2026); requires genuine new crisis to trigger

Sources: World Gold Council, Goldman Sachs forecast, BullionVault, CNN, CNBC

### Treasury (10Y-2Y Spread)
- **GREEN > +0.50%** (was > 0) — Lower bound of "normal"; long-run median +79bps (Estrella-Mishkin/NY Fed)
- **AMBER 0 to +0.50%** — Caution zone; both 2006 and 2019 cycles spent time here pre-inversion
- **RED < 0** (was < -0.5) — Any inversion preceded 6/7 modern recessions (87.5% accuracy). Old -0.5 threshold missed the entire pre-GFC inversion (peaked at only -19bps)

Sources: FRED T10Y2Y, SF Fed Economic Letters (2018, 2022), NY Fed Estrella-Mishkin, Chicago Fed

### Defense (Daily % Change of LMT/RTX/NOC/GD average)
- **GREEN < 3%** (was 2%) — 2% daily moves are common in normal vol
- **AMBER 3%–8%** — Captures elevated geopolitical tension
- **RED > 8%** (was 5%) — Ukraine invasion drove defense stocks +8.6% in weeks; individual contractors +37–41%

Sources: Defense News, PMC geopolitical risk studies

---

## 2. Revised Sector Sensitivity Matrix

All weights backed by crisis-period correlations and academic sources (IMF GFSR 2025, CFA Institute, NBER, Chicago Fed, Nareit, FactSet).

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

### Key directional changes:
- Consumer Staples VIX doubled (+0.2 → +0.4): defensive rotation is one of the most robust tactical signals (FactSet)
- Long-duration Bonds VIX reduced (+0.5 → +0.3): 2022 showed TLT doesn't reliably hedge supply-shock VIX spikes (CFA Institute)
- EM oil reduced (-0.8 → -0.5): EEM includes oil exporters (Saudi, Brazil), diluting negative sensitivity
- Treasury sensitivities added across board: many sectors previously had 0 despite real rate exposure
- Banks treasury reduced (+0.8 → +0.6): regime-dependent — demand-driven steepening positive, abrupt spikes painful

---

## 3. Revised Composite Weights

```javascript
weights: { oil: 0.20, defense: 0.15, vix: 0.30, gold: 0.15, treasury: 0.20 }
```

| Instrument | Old | New | Rationale |
|---|---|---|---|
| VIX | 25% | 30% | Strongest real-time predictor of equity drawdowns; captures widest range of risk events |
| Defense | 20% | 15% | Narrow signal — only responds to military/geopolitical conflict, not financial crises or recessions |
| Oil, Gold, Treasury | unchanged | unchanged | Appropriate as-is |

---

## 4. Revised Scenario Dial Multipliers

### Conflict Dial (1–10, at max):
- Defense: +20% (was +15%) — Ukraine drove individual contractors +37–41%
- Oil: +25% (was +15%) — Ukraine drove oil from ~$75 to $133 (+77%)
- Gold: +12% (was +10%) — Gold rose ~15% in weeks post-Ukraine
- VIX: +15pts (was +20pts) — Ukraine only pushed VIX to ~34–36 from ~20 baseline

### Credit Stress Dial (1–10, at max):
- VIX: +35pts (was +25pts) — 2008 GFC pushed VIX +60pts; +25 understated financial crisis impact
- Gold: +10% (was +8%) — Flight-to-safety during 2008 recovery and COVID drove +15–20%
- Spread: -0.8 (was -0.5) — 2022–2024 inversion reached -108bps; -0.5 too mild
- Defense: -5% (unchanged)

### Supply Disruption Dial (1–10, at max):
- Oil: +30% (was +20%) — Abqaiq drove +14% on single facility; Hormuz closure scenario would far exceed +20%
- Gold: +8% (was +5%) — Supply disruptions drive inflation fears → gold demand
- Spread: -0.4 (was -0.3) — Stagflation fears from supply shocks flatten curve

---

## Code Changes Summary

In `index.html`, update these constants:

1. `DEFAULT_THRESHOLDS` — new zone breakpoints
2. `SECTOR_SENSITIVITY` — revised weights
3. `DEFAULT_THRESHOLDS.weights` — composite weights
4. Scenario multipliers in `applyScenario()` function
