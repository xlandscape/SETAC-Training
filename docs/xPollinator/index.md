# Pollinators — xPollinator Training

This section covers landscape-level risk assessment for **pollinator populations** (honeybees and wild bees) using the [xPollinator](https://github.com/xlandscape/xPollinator) landscape model.

## Getting Started

👉 **New to xPollinator?** Start with the [Complete Workflow Tutorial](complete-workflow.md), which walks you through:
- Setting up the software on Windows
- Running your first simulation (single hive, untreated)
- Importing results and visualizing them in BeeView
- Advanced multi-hive and treated scenarios with statistical comparison

For a dedicated explanation of how pesticide applications become patch residues, in-hive doses, and colony effects, see [Pesticide Fate in xPollinator and BEEHAVEecotox](pesticide-fate.md).

Estimated time: **2-3 hours** for the complete workflow

## What You Will Learn

- **End-to-end workflow**: From landscape scenario setup through xPollinator simulation to interactive visualization in BeeView
- **Pesticide exposure pathways**: How bees encounter pesticides through contaminated nectar, pollen, and guttation water
- **Landscape-level forage modeling**: Computing daily nectar/pollen availability from land use data and vegetation phenology
- **Colony-level effects**: How pesticide exposure affects honeybee population dynamics, survival, and reproduction
- **Statistical analysis**: Comparing treated vs. untreated scenarios across multiple hive locations and replicate populations
- **Regulatory context**: Understanding EFSA Bee Guidance requirements for environmental risk assessment

## The xPollinator Pipeline

xPollinator integrates landscape simulation with bee colony modeling through a multi-component pipeline:

```
Landscape Data (LULC shapefile)
    ↓
[LandscapeScenario] → Extract parcels and vegetation
    ↓
[BeeForage] → Compute daily nectar/pollen availability
    ↓
[CropProtection] → Simulate pesticide applications (if treated)
    ↓
[BeeHaveEcotox] → NetLogo model: 100 replicates of colony dynamics
    ↓
Output: HDF5 data store + CSV colony metrics
    ↓
[BeeView-server] → Load, normalize, serve via REST API
    ↓
[BeeView] → Interactive visualization & statistical comparison
```

For detailed explanations of each component and how to work with the output data, see the [Complete Workflow Tutorial](complete-workflow.md) (sections 1-2).

## Model Overview

### xPollinator
xPollinator is a landscape-scale model that combines:
- **Landscape processing**: Reads GIS shapefiles and categorizes vegetation
- **Forage modeling**: Computes daily nectar and pollen availability from vegetation phenology
- **Exposure calculation**: Determines pesticide contamination of foraging resources
- **Result consolidation**: Prepares data for honeybee colony simulation

### BEEHAVEecotox
Based on the BEEHAVE model (NetLogo), extended with toxicological effects:
- Models **colony dynamics**: Workers, brood, drones, and food stores
- Simulates **foraging behavior**: Resource collection and consumption
- Includes **Varroa parasites**: Pest population and impact on colony survival
- Calculates **toxicological effects**: How pesticide exposure reduces bee fitness and survival

**Key feature**: Multiple replicates (typically 100) use different random seeds to model natural population variability, enabling probabilistic assessment of colony survival under different scenarios.

## Resources

### Tutorials
- **[Complete Workflow Tutorial](complete-workflow.md)** — Step-by-step setup and end-to-end example (Windows)
- Prerequisites, installation, running simulations, visualization, statistical comparison
- **[Pesticide Fate in xPollinator and BEEHAVEecotox](pesticide-fate.md)** — Detailed explanation of application timing, residue decline, uptake, and in-hive exposure

### Documentation & Tools
- [xPollinator GitHub Repository](https://github.com/xlandscape/xPollinator) — Model source code and component reference
- [xLandscape Framework](https://github.com/xlandscape) — Core modeling framework
- [BeeView-server](https://github.com/xlandscape/BeeView-server) — REST API backend and database
- [BeeView](https://github.com/xlandscape/BeeView) — Interactive visualization frontend

### Demo Scenarios
- **Tarn-et-Garonne** — Multi-hive French agricultural landscape with real LULC and weather data
- Includes treated and untreated comparison scenarios
