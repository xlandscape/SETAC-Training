# Pollinators — xPollinator Training

This section covers landscape-level risk assessment for **pollinator populations** (honeybees and wild bees) using the [xPollinator](https://github.com/xlandscape/xPollinator) landscape model.

!!! note "Coming Soon"
    Training materials for this section are being prepared and will be published here.

## What You Will Learn

- How to set up and run the **xPollinator** landscape model
- Understanding **pesticide exposure** pathways for bees in agricultural landscapes (oral, contact, residues in pollen/nectar)
- Modelling **landscape-level foraging** and exposure of honeybee colonies
- Applying **population-level effect models** for colony dynamics
- Relevant regulatory context (e.g., EFSA Bee Guidance)

## Key Model Components

| Component | Description |
|-----------|-------------|
| Landscape & crop data | Spatial representation of foraging resources |
| Exposure modules | Pesticide residues in pollen, nectar, and guttation water |
| BeeHaveEcotox | Population dynamics of honeybee colonies |

## Introduction

- xPollinator is using the BEEHAVEecotox model (which is based on the BEEHAVE model) to simulate Honeybee populations in agricultural landscapes
- While the current version simulates Honeybees only, the underlying concept can be applied to other pollinators as well
- BEEHAVE is a population model implemented in Netlogo that supports
    - colony and population dynamics
    - foraging
    - broodcare
    - Varroa and beekeeping
- BEEHAVEecotox has an additional toxicological module that allows to simulate the effects of pesticides
- xPollinator uses the following (simplified) pipeline to compute effects of pesticide applications, as well as hive location, to the colony dynamics:
    - as BEEHAVE does not scale well with a large number of 'patches', the available land use layer is sampled to generate an approximation of the real landscape for processing with BEEHAVE
    - a BeeForage module computes Nectar and Pollen availability for the selected scenario, based on the vegetation as inferred from the LULC layer
    - pesticide concentration in Nectar and Pollen is computed based on applications in the field (simulated by the CropProtection component) and the exposure via ...
    - the results are consolidated into an input file that BEEHAVEecotox processes
    - the BEEHAVEecotox simulation computes population dynamics for the given scenario, foraging, and exposure profiles
    - all simulation results are stored in the xLandscape HDF5 store
- 
- requirements
    - a scenario is required defining
        - the landscape around the hive (minimum 5km radius) as LULC layers
    - a mapping table provides Nectar and Pollen availability for selected vegetations

## Resources

- [xPollinator GitHub Repository](https://github.com/xlandscape/xPollinator)
- [xLandscape Framework](https://github.com/xlandscape)
- [BeeView-server](https://github.com/xlandscape/BeeView-server)
- [BeeView](https://github.com/xlandscape/BeeView)
