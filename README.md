# PYROSHIELD — Drone Swarm Wildfire Coordination

Interactive visualization of autonomous drone swarm coordination strategies for wildfire suppression, integrating multiple real-world geospatial datasets.

## Live Demo

**[View Live →](https://shenal-maker.github.io/drone-wildfire-coordination/)**

## Overview

PYROSHIELD simulates a fleet of 10 autonomous drones coordinating to suppress wildfire hotspots across Northern California's fire-prone regions. The system compares three coordination strategies in real-time:

| Strategy | Approach | Best For |
|---|---|---|
| **Greedy Nearest** | Each drone targets the closest active fire | Fast initial response |
| **Auction-Based** | Drones bid on fires weighted by distance × priority | Optimal resource allocation |
| **Coverage Path** | Systematic zone sweep with area partitioning | Complete area coverage |

## Integrated Datasets

| Source | Data | Resolution |
|---|---|---|
| **NASA FIRMS** | VIIRS Active Fire detections | 375m, Near Real-Time |
| **NOAA GFS** | 10m wind vectors | 0.25° global grid |
| **USGS 3DEP** | Digital Elevation Model | 1/3 arc-second (~10m) |
| **CAL FIRE** | Historical fire perimeters | Vector/GIS |

Fire hotspot coordinates are based on real locations from the Camp Fire (2018), Dixie Fire (2021), and North Complex Fire (2020) regions in Butte and Plumas counties, California.

## Features

- Real-time animated drone agents with flight path trails
- Flickering fire hotspot visualization with FRP-based sizing
- Animated wind particle field overlay
- Terrain elevation heatmap layer
- Live metrics dashboard (suppression count, coverage %, response time, fleet utilization)
- Strategy comparison chart
- Simulation controls (play/pause/reset, speed 0.5×–4×)
- Toggleable map layers

## Tech Stack

- **deck.gl** — GPU-accelerated geospatial layers
- **MapLibre GL** — Vector map rendering (CARTO Dark basemap)
- **D3.js** — Dashboard charts
- **Canvas API** — Wind particle animation

Zero build step. Single `index.html` deployable anywhere.

## Local Development

```bash
# Just open the file
open index.html

# Or serve locally
npx serve .
```

## License

MIT
