# Geopolitical Risk Dashboard — Design Document

**Date:** 2026-03-02
**Status:** Approved

## Overview

A public-facing, live-updated Geopolitical Risk Dashboard with a retro sci-fi CRT aesthetic. Tracks oil prices, defense stocks, VIX, gold, and Treasury yields. Features interactive 3D scenario simulation dials and an Investment Impact Matrix that translates raw data into sector-level trading guidance.

## Tech Stack

- Single HTML file with embedded CSS and JS
- Three.js (CDN) for 3D radar, rotary dials, CRT monitor bezel
- HTML Canvas for CRT noise/scanline effects
- Free APIs: Yahoo Finance v8, FRED
- localStorage for threshold persistence and data caching
- No build step, no backend, deploy to any static host

## Layout

```
+====================================================================+
|  [GEOPOLITICAL RISK TERMINAL v2.1]           [LIVE] [UTC CLOCK]    |
+====================================================================+
|                    |                                                |
|   RADAR DISPLAY    |     INSTRUMENT PANELS (2x3 grid)              |
|   (3D rotating     |     Each panel: price, direction, sparkline,  |
|    sweep, color =  |     signal zone (GREEN/AMBER/RED), active     |
|    composite risk) |     trading signal if triggered               |
|                    |     +----------+-----------+----------+       |
|   SCENARIO DIALS   |     |   OIL    |  DEFENSE  |   VIX    |      |
|   4x 3D rotary     |     +----------+-----------+----------+       |
|   knobs:           |     |   GOLD   | TREASURY  | COMPOSITE|      |
|   - Oil Price      |     |          |           |  SCORE   |      |
|   - Conflict Level |     +----------+-----------+----------+       |
|   - Sanctions      |                                               |
|   - Supply Disrupt |                                               |
+--------------------+-----------------------------------------------+
|  INVESTMENT IMPACT MATRIX                                          |
|  Favored / Neutral / Avoid — real-time sector guidance             |
+====================================================================+
|  SIGNAL TICKER — scrolling tape of active trading signals          |
+====================================================================+
```

## Instrument Thresholds & Trading Signals

### OIL (WTI Crude)
- GREEN (< $75): Stable. Favor consumer discretionary, airlines
- AMBER ($75–$95): Monitor. Energy sector neutral-positive
- RED (> $95): BUY energy/defense. SELL airlines, logistics. Inflation hedge on

### DEFENSE STOCKS (avg of LMT, RTX, NOC, GD)
- GREEN (< +2% daily avg): No geopolitical escalation priced in
- AMBER (+2% to +5%): Tensions rising. ACCUMULATE defense positions
- RED (> +5%): Active crisis. ROTATE into defense + commodities

### VIX (Fear Index)
- GREEN (< 20): Low fear. Risk-on. Favor growth/equities
- AMBER (20–30): Elevated fear. HEDGE positions. Tighten stops
- RED (> 30): Panic. BUY protective puts. SELL risk assets. Accumulate gold

### GOLD (XAU/USD)
- GREEN (< $2,000): Stable. No safe-haven rush
- AMBER ($2,000–$2,200): Flight to safety starting. ACCUMULATE gold
- RED (> $2,200): Crisis pricing. HOLD gold. Risk-off across equities

### TREASURY YIELDS (2Y, 10Y, 30Y)
- GREEN (2Y-10Y spread > 0): Normal curve. Favor banks, financials
- AMBER (Spread 0 to -0.5%): Flattening/inverting. Recession signal. Reduce equity exposure
- RED (Spread < -0.5%): Deep inversion. SELL cyclicals. MOVE to short-duration bonds + cash

### Composite Risk Score
- Oil zone (20%) + Defense move (20%) + VIX (25%) + Gold (15%) + Yield curve (20%)
- GREEN=0pts, AMBER=50pts, RED=100pts per instrument
- Score 0–100, drives radar color and Investment Impact Matrix

## Scenario Simulation Engine

### Dials
1. OIL PRICE — range $40–$150, default: live price
2. CONFLICT LEVEL — range 1–10, multiplier on defense + gold
3. SANCTIONS SEVERITY — range 1–10, amplifies oil + supply chain
4. SUPPLY DISRUPTION — range 1–10, affects oil, gold, yields

### Correlation Cascade
- Oil +20% -> Defense +8%, Gold +5%, VIX +12pts, curve flattens 0.3%
- Conflict 5/10 -> Defense +15%, Oil +10%, Gold +8%, VIX +18pts
- Sanctions 7/10 -> Oil +25%, Gold +4%, VIX +8pts
- Effects stack multiplicatively across dials
- Double-click any dial to snap back to live data

## Investment Impact Matrix

12 sectors scored from -100 (strong avoid) to +100 (strong favor):

| Sector | Oil | VIX | Gold | Yield Curve | Defense |
|---|---|---|---|---|---|
| Defense/Aerospace | +med | +med | neutral | neutral | +direct |
| Energy/Oil Majors | +direct | +low | neutral | neutral | +med |
| Gold/Precious Metals | +med | +high | +direct | -med | +med |
| Cash/Money Market | neutral | +high | neutral | +high | neutral |
| Growth Tech | -high | -high | neutral | -high | neutral |
| Airlines/Transport | -direct | -med | neutral | neutral | -med |
| EM Equities | -high | -high | neutral | -med | -med |
| Banks/Financials | neutral | -med | neutral | +high | neutral |
| Real Estate/REITs | -med | -med | neutral | -high | neutral |
| Consumer Staples | neutral | +low | neutral | neutral | neutral |
| Long-duration Bonds | neutral | +med | +low | -direct | neutral |
| Crypto/Speculative | -med | -high | neutral | -med | neutral |

Display: Favored (>+30, green, up arrows), Neutral (-30 to +30, amber), Avoid (<-30, red, down arrows). Arrow count reflects conviction.

## Visual Design

### Retro Sci-Fi CRT Aesthetic
- Background: #0a0a0a (near-black)
- Primary text: #33ff33 (phosphor green)
- Amber warnings: #ffaa00
- Red alerts: #ff3333 with pulsing glow
- Panel borders: #1a3a1a
- Inactive: #0d330d

### Effects
- CRT scanlines (CSS repeating gradient)
- Phosphor glow (double text-shadow)
- Screen flicker (CSS keyframe opacity 0.97–1.0)
- VHS noise (Canvas random pixels)
- CRT monitor bezel (Three.js)
- Screen curvature (CSS border-radius + transform)

### 3D Elements (Three.js)
- Radar dish with rotating sweep beam, color = composite risk
- Rotary dial knobs with drag interaction and momentum
- CRT monitor housing frame

### Animations
- Radar sweep: 4s rotation
- Sparklines: oscilloscope trace draw-in
- Signal ticker: horizontal scroll teletype
- Threshold breach: screen-shake + flash
- Dial turn: ripple/pulse on connected panels

### Sound (optional, off by default)
- Radar ping on sweep
- Geiger click on rising composite score
- Typewriter clack on new signals

## Data Sources

| Instrument | Source | Update |
|---|---|---|
| Oil (WTI/Brent) | Yahoo Finance v8 | 60s |
| Defense (LMT,RTX,NOC,GD) | Yahoo Finance v8 | 60s |
| VIX | Yahoo Finance v8 | 60s |
| Gold (XAU/USD) | Yahoo Finance v8 | 60s |
| Treasury yields | FRED API | Daily |

### Fallback Strategy
- CORS proxy fallback (allorigins.win)
- "STALE DATA" warning on API failure
- localStorage cache of last successful response
- FRED requires free API key (default included, admin-overridable)

## Admin Panel

- Access: `?admin=true` URL param or Ctrl+Shift+A
- Edit all thresholds, API keys, weights
- Stored in localStorage
- Reset-to-defaults button
- Same CRT aesthetic as main dashboard
