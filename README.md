# Experiment 1 — Optimal Fuel Treatment Placement for Robotic Prescribed Burn Fleets

**An agent-based simulation comparing spatial treatment strategies for mechanized wildfire prevention, calibrated to Burnbot RX2 operational constraints.**

## Live Demo

**[View Live →](https://shenal-maker.github.io/drone-wildfire-coordination/)**

---

## Research Question

> Given a fleet of N treatment robots operating under real terrain, weather, and operational constraints, which spatial allocation strategy for prescribed burns maximizes wildfire risk reduction — and what are the tradeoffs between protecting communities and preserving wildland?

## Background

The US treats under 2% of the acres requiring fuel management each year. Manual prescribed burns are slow (~1 acre/day), weather-constrained, and expensive ($1,000–2,500/acre). [Burnbot's RX2](https://burnbot.com/technology/) treats at **1 acre/hour**, operates in **winds up to 32 km/h**, handles **slopes up to 30°**, and runs **24/7** — changing the optimization problem fundamentally.

Published research on fuel treatment placement (Thompson et al. 2022; Pais et al. 2021) assumes manual crew constraints. **No published work optimizes treatment placement under robotic operational constraints**, where access feasibility, travel time between parcels, and machine slope limits create different tradeoff frontiers.

### Key References

- Rothermel, R.C. (1972). *A mathematical model for predicting fire spread in wildland fuels.* USDA Forest Service Research Paper INT-115.
- Pais, C. et al. (2021). *Downstream Protection Value: Detecting critical zones for effective fuel-treatment under wildfire risk.* Computers & Operations Research.
- Thompson, M. et al. (2022). *Comparing risk-based fuel treatment prioritization with alternative strategies.* Fire Ecology.
- Andrews, P.L. (2018). *The Rothermel surface fire spread model and associated developments.* USDA Forest Service RMRS-GTR-371.
- NWCG (2024). *Surface Fire Behavior Lookup Tables.* PMS 437.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    index.html                           │
│                                                         │
│  ┌───────────────┐  ┌────────────────────────────────┐  │
│  │   SIDEBAR      │  │          MAP (MapLibre)        │  │
│  │                │  │                                │  │
│  │  Landscape     │  │   deck.gl ScatterplotLayer     │  │
│  │  Stats         │  │   ┌─────────────────────┐     │  │
│  │                │  │   │  ~5,400 grid cells   │     │  │
│  │  Rothermel     │  │   │  color = f(state)    │     │  │
│  │  Parameters    │  │   │  • operable (green)  │     │  │
│  │                │  │   │  • marginal (yellow) │     │  │
│  │  Fire Spread   │  │   │  • inoperable (red)  │     │  │
│  │  Test          │  │   │  • WUI (purple)      │     │  │
│  │                │  │   │  • burning (orange)   │     │  │
│  │  Experiment    │  │   │  • burned (gray)     │     │  │
│  │  Controls      │  │   │  • treated (blue)    │     │  │
│  │  (Step 2)      │  │   └─────────────────────┘     │  │
│  │                │  │                                │  │
│  │  Results       │  │   FIRMS hotspot overlay (cyan) │  │
│  │  Charts        │  │                                │  │
│  │  (Step 2)      │  │                                │  │
│  │                │  │                                │  │
│  │  Export        │  │                                │  │
│  │  (Step 3)      │  │                                │  │
│  └───────────────┘  └────────────────────────────────┘  │
│                                                         │
│  ┌─────────────────────────────────────────────────────┐│
│  │              SIMULATION ENGINE (JS)                  ││
│  │                                                     ││
│  │  buildGrid() → cells[] with:                        ││
│  │    elevation, fuel, slope, ignProb, wuiWeight,      ││
│  │    valueAtRisk, operable                            ││
│  │                                                     ││
│  │  spreadProb(cell, neighbor, params) → probability   ││
│  │    = baseROS × moisture × slope × wind × fuel       ││
│  │    × treatmentReduction (if treated)                ││
│  │                                                     ││
│  │  simulateFire(ignition, params, treatedSet)         ││
│  │    → { burnedCells, totalDamage, wuiDamage,         ││
│  │        wildlandBurned, ticks }                      ││
│  │                                                     ││
│  │  generateIgnitionSet(100) → stratified sample       ││
│  │  sampleWeather() → {deadFM, liveFM, wind, event}   ││
│  └─────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────┘
```

---

## Experimental Design

### Landscape

| Parameter | Value | Source |
|---|---|---|
| Region | Butte & Plumas County, CA | — |
| Grid | ~90 × 60 cells (~5,400 total) | — |
| Cell size | ~1.1 km (~250 acres) | Performance-constrained |
| Elevation | 20–2200m, modeled from USGS NED profiles | Feather River Canyon, Paradise bench, Sierra ridgelines |
| Fuel model | NWCG SH5 (high-load dry-climate shrub) | NWCG PMS 437 |
| Fuel load | 0.05–1.0, peaks at mid-elevation (400–1200m) | Chaparral/mixed conifer distribution |
| Ignition probability | Derived from NASA FIRMS VIIRS historical hotspot density | 48 hotspots from Camp Fire, Dixie Fire, North Complex |
| WUI zones | 8 communities, 2.4 km buffer | CAL FIRE standard |
| Value-at-risk | WUI: up to 10, Timber: 3, Wildland: 1 | Thompson et al. 2022 framework |
| Slope operability | ≤20° operable, 20–30° marginal, >30° infeasible | Burnbot RX2 spec (≤30°) |

### Fire Spread Model (Simplified Rothermel)

```
ROS_effective = baseROS × moistureDamping(deadFM) × slopeFactor(Δelev) × windFactor(dir, speed) × fuelLoad

spreadProbability = ROS_effective × tickDuration / cellSize

treatmentEffect = conditional on wind speed and fuel moisture:
  - Calm conditions (wind <20 km/h): ~85% spread reduction
  - Moderate (20-35 km/h): ~60-72% reduction
  - Diablo event (>35 km/h, dry): ~40% reduction
  - Range: 35-90%, meta-analysis benchmark: 62-72%
```

| Parameter | Value | Reference |
|---|---|---|
| Base ROS | 1.2 km/h | NWCG SH5 moderate conditions |
| Slope factor | 2× per 20° uphill | Rothermel 1972 |
| Downhill penalty | 0.3× minimum | Rothermel 1972 |
| Moisture extinction | 15% dead FM | SH5 fuel model |
| Tick duration | 30 minutes real time | — |
| Max simulation | 200 ticks (100 hours) | — |

### Weather Sampling

Each Monte Carlo run samples weather independently:

| Condition | Probability | Dead FM | Wind Speed | Wind Direction |
|---|---|---|---|---|
| Normal summer | 70% | 4–8% | 10–25 km/h | 240–360° (W to N) |
| Diablo wind event | 30% | 3–5% | 35–60 km/h | 30–70° (NE) |

### Ignition Sampling (Stratified)

100 ignition points sampled from FIRMS-weighted spatial density, stratified by quartile:

| Stratum | FIRMS density | Sample weight | Purpose |
|---|---|---|---|
| Q1 (low) | Bottom 25% | 10% | Tests robustness to unexpected ignitions |
| Q2 (moderate) | 25–50% | 20% | — |
| Q3 (high) | 50–75% | 30% | — |
| Q4 (very high) | Top 25% | 40% | Historical fire-start zones |

Same 100 ignition points used across all strategies (paired comparison).

### Treatment Strategies

| # | Strategy | Logic | Novel? |
|---|---|---|---|
| 1 | **Housing Protection** | Treat operable cells closest to WUI, ranked by wuiWeight | Baseline (Thompson et al. 2022) |
| 2 | **Fire Transmission** | Treat cells on highest-probability fire pathways toward WUI | Literature standard |
| 3 | **Highest Fuel Load** | Treat most flammable operable cells first | Naive baseline |
| 4 | **DPV-Optimal** | Score cells by Downstream Protection Value, treat highest-scored | Pais et al. 2021 |
| 5 | **Operability-Constrained DPV** | DPV scores filtered by Burnbot travel time between parcels — scattered optimal cells cost more travel time, reducing total treated area | **Novel contribution** |

### Treatment Budget

- Campaign: 2 weeks (140 working hours per Burnbot)
- Treatment rate: 1 acre/hour per Burnbot
- Budget per unit: 140 acres
- Travel between parcels: ~20 km/h (reduces effective treatment time for scattered strategies)
- **Key tradeoff**: DPV-Optimal may pick scattered cells → more travel → fewer acres treated vs. Housing Protection picks tight cluster → less travel → more acres treated

### Metrics

| Metric | Description |
|---|---|
| **Damage-weighted burn area** | Σ(burned cell × value-at-risk). Primary outcome variable. |
| **WUI damage** | Σ(burned WUI cell × wuiWeight). Community impact. |
| **Wildland burned** | Count of non-WUI cells burned. Ecological cost. |
| **Acres treated** | Total acres each strategy managed to treat in 2-week budget (varies by strategy due to travel). |
| **WUI saved per acre treated** | (baseline WUI damage − strategy WUI damage) / acres treated. Efficiency. |
| **Wildland sacrificed per WUI protected** | Increase in wildland burn / decrease in WUI damage. Cost ratio. |
| **Pareto front** | WUI damage vs. total wildland burned, per strategy. Shows fundamental tradeoff. |

### Reporting

- Results broken down by ignition stratum (Q1–Q4) to test strategy robustness
- Mean ± standard deviation across 100 Monte Carlo runs per strategy
- Spatial treatment maps showing which cells each strategy selected
- Summary tables of all raw data

---

## File Structure

```
index.html              — Full application (zero build step)
data/
  firms_sample.json     — NASA FIRMS VIIRS snapshot (48 hotspots, real coordinates)
  wind_grid.json        — Open-Meteo GFS wind snapshot (20 grid points)
README.md               — This file
```

## Tech Stack

- **deck.gl** — GPU-accelerated geospatial cell rendering
- **MapLibre GL** — Map tiles (CARTO Dark basemap)
- **Vanilla JS** — Simulation engine, Monte Carlo runner
- Zero dependencies. Zero build step. Single HTML file.

## Local Development

```bash
# Serve locally (needed for data file loading)
npx serve .
# or
python3 -m http.server 8000
```

## Implementation Status

- [x] Step 1: Landscape grid + Rothermel fire spread (interactive)
- [ ] Step 2: Treatment strategies + Monte Carlo + results/charts
- [ ] Step 3: Export CSV + PDF

## License

MIT
