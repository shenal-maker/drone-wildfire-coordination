# Burnbot Treatment Optimizer

**Where should wildfire-prevention robots do prescribed burns? A Monte Carlo simulation comparing five placement strategies.**

**[View Live Demo →](https://shenal-maker.github.io/drone-wildfire-coordination/)**

---

## The Problem

The US treats <2% of acres needing fuel management. Manual burns: ~1 acre/day, $1,000–2,500/acre. [Burnbot's RX2](https://burnbot.com/technology/) treats **1 acre/hour, 24/7, on slopes up to 30°**. That speed creates a new question: given limited time before fire season, **which land should robots treat first?**

No published work answers this under robotic constraints — where travel time between parcels, slope limits, and machine access change the tradeoffs.

## Two Experiments

| | Exp 1: Regional | Exp 2: Paradise Zoom |
|---|---|---|
| **Area** | Butte & Plumas County, CA | ~20×16 km around Paradise |
| **Grid** | ~5,400 cells at ~1.1 km | ~35,000 cells at ~100m |
| **Cell size** | ~250 acres | ~2.5 acres |
| **Treatment coverage** | ~0.05% of landscape | ~1% of landscape |
| **MC runs** | 100 per strategy | 30 per strategy |
| **Purpose** | Mirrors real-world scale constraint | Higher resolution — strategy differences become visible |

Toggle between experiments in the sidebar.

## Five Strategies Compared

| Strategy | Logic | Source |
|---|---|---|
| **Housing Protection** | Treat closest to communities | Thompson et al. 2022 |
| **Fire Transmission** | Block fire corridors toward WUI | Literature standard |
| **Highest Fuel** | Treat most flammable land | Naive baseline |
| **DPV-Optimal** | Maximize downstream protection value | Pais et al. 2021 |
| **Constrained DPV** | DPV adjusted for robot travel time | **Novel** |

The key insight: mathematically optimal cells may be scattered — robots waste hours driving between them. Constrained DPV balances placement quality vs. operational efficiency.

## How It Works

1. **Landscape** — Grid cells with elevation (modeled from USGS NED), fuel load (NWCG SH5 chaparral), slope, WUI zones (8 communities, CAL FIRE 2.4 km buffer), and ignition probability (NASA FIRMS hotspots).

2. **Fire spread** — Simplified Rothermel model (1972). Spread probability = f(fuel moisture, slope, wind, fuel load). Treatment reduces spread 35–90% depending on conditions (benchmark: 62–72%, Fernandes & Botelho 2003).

3. **Monte Carlo** — Each strategy faces the same ignition points (stratified by FIRMS density) under random weather (70% normal summer, 30% Diablo wind events). Only treatment placement varies.

4. **Results** — Damage-weighted burn area, Pareto front (WUI vs. wildland tradeoff), efficiency metrics, robustness by ignition stratum. Treatment maps shown as white firebreak lines on the map.

## References

- Rothermel (1972). *A mathematical model for predicting fire spread in wildland fuels.* USDA Forest Service INT-115.
- Pais et al. (2021). *Downstream Protection Value.* Computers & Operations Research.
- Thompson et al. (2022). *Comparing risk-based fuel treatment prioritization.* Fire Ecology.
- Andrews (2018). *The Rothermel surface fire spread model.* USDA Forest Service RMRS-GTR-371.
- NWCG (2024). *Surface Fire Behavior Lookup Tables.* PMS 437.

## Run Locally

```bash
npx serve .
# or
python3 -m http.server 8000
```

Single HTML file. Zero build step. deck.gl + MapLibre + D3 from CDN.

## License

MIT
