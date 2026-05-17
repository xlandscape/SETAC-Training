# Pesticide Fate in xPollinator and BEEHAVEecotox

This page explains how pesticide applications are translated into nectar, pollen, contact, and in-hive exposure in the current xPollinator workflow used by the SETAC training material.

It focuses on the modelling chain implemented in the working copy of the repositories (all hosted under the [xLandscape GitHub organisation](https://github.com/xlandscape)):
- [xPollinator](https://github.com/xlandscape/xPollinator) prepares patch-level pesticide applications and forage inputs
- BEEHAVEecotox applies decline, uptake, and toxic effects
- [BeeView-server](https://github.com/xlandscape/BeeView-server) and [BeeView](https://github.com/xlandscape/BeeView) visualize the resulting colony and landscape outputs

## Why This Matters

For training and risk assessment, it is important to distinguish between three different layers of parameterization:

1. **Landscape application setup**
   This defines where, when, and how often pesticide applications occur.

2. **Patch-level exposure concentrations**
   This defines the pesticide concentrations assigned to nectar, pollen, contact exposure, and optionally water.

3. **Toxicokinetic and toxicodynamic response in BEEHAVEecotox**
   This defines how long residues persist, how they move into the hive, and how strongly they affect larvae, in-hive bees, and foragers.

## End-to-End Pesticide Fate Workflow

The pesticide lifecycle in the current xPollinator setup is:

```text
Manifest or .xrun parameters
    -> xPollinator assigns treated patches and application dates
    -> xPollinator writes patch-level pesticide fields to Sources.txt
    -> BEEHAVEecotox reads Sources.txt as INPUT_FILE
    -> patch concentrations are activated on application day
    -> first-order decline is applied on patch
    -> foragers receive oral and contact doses during foraging
    -> contaminated nectar, pollen, and water enter the hive
    -> in-hive concentrations may decline further in honey stores
    -> cohort-specific dose-response functions determine mortality
```

## Stage 1. Application Setup in xPollinator

### Manifest-driven treatment setup

For the Tarn-et-Garonne batch workflow, the manifest controls the number of applications per treated run.

Current manifest file:
- [scripts/Scenario-TarnEtGaronne_manifest.json](https://github.com/xlandscape/xPollinator/blob/main/scripts/Scenario-TarnEtGaronne_manifest.json)

Relevant fields:
- `treated_min_applications`
- `treated_max_applications`
- `random_seed` for each hive

In the current manifest, treated runs allow between `0` and `3` applications per treated patch. Because the minimum is `0`, a treated run can still contain some treated patches with no sampled application.

### How the manifest becomes .xrun files

The batch runner writes these values into generated `.xrun` files:
- [scripts/run_batch.py](https://github.com/xlandscape/xPollinator/blob/main/scripts/run_batch.py)

For each hive:
- untreated runs set `MinNumberApplications = 0` and `MaxNumberApplications = 0`
- treated runs set `MinNumberApplications` and `MaxNumberApplications` from the manifest

The same hive random seed is reused for treated and untreated run generation, so treatment sampling is reproducible for that hive.

## Stage 2. Patch-Level Exposure Generation in xPollinator

The BeeHaveEcotox component in xPollinator creates the patch-level exposure file used by NetLogo:
- [model/variant/BeeHaveEcotox/BeeHaveEcotox.py](https://github.com/xlandscape/xPollinator/blob/main/model/variant/BeeHaveEcotox/BeeHaveEcotox.py)

### What happens during source generation

Inside `create_forage_input_file`, xPollinator:

1. Consolidates the landscape into BEEHAVE-compatible forage patches.
2. Identifies which consolidated patches belong to the treated vegetation type.
3. Samples the number of applications for each treated patch.
4. Samples the application day for each application.
5. Draws one nectar concentration, one pollen concentration, and one contact value for that patch.
6. Writes those values into `Sources.txt` and `applications.txt`.

### Current patch-level inputs from xPollinator

These values are passed from the xLandscape composition in:
- [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml)

Relevant parameters are:

| Parameter | Current role | Current default/location |
|---|---|---|
| `AppliedVegetationId` | Which vegetation type can receive applications | `2102` in [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml) |
| `MinNumberApplications` | Minimum sampled applications per treated patch | from `.xrun` / manifest |
| `MaxNumberApplications` | Maximum sampled applications per treated patch | from `.xrun` / manifest |
| `FirstDayOfYearApplications` | Earliest sampled day of year | `100` |
| `LastDayOfYearApplications` | Latest sampled day of year | `250` |
| `ConcNectarMean` | Mean nectar residue concentration | `1320 µg/kg` |
| `ConcNectarStd` | Standard deviation for nectar concentration | `132 µg/kg` |
| `ConcPollenMean` | Mean pollen residue concentration | `36200 µg/kg` |
| `ConcPollenStd` | Standard deviation for pollen concentration | `3620 µg/kg` |
| `ContactToxicityMean` | Mean contact application term | `0.4 µg/bee` in xPollinator naming, written into patch contact field |
| `ContactToxicityStd` | Standard deviation for contact term | `0.04` |
| `ExposurePeriod` | Duration written for each patch application | `10000 d` |
| `RandomSeed` | Controls the patch-level random draws | from `.xrun` / manifest |

### Files produced for BEEHAVEecotox

The main file is `Sources.txt`, written into each run folder under:
- `run/<SimID>/mcs/<MC>/processing/BeeHave/Sources.txt`

Each patch line contains these ecotox columns:
- `ETOX_ApplicationList_patch`
- `ETOX_ExposurePeriodsList_patch`
- `ETOX_PPPConcNectar_patch`
- `ETOX_PPPConcPollen_patch`
- `ETOX_PPPContact_patch`
- `ETOX_WaterVolume_patch`
- `ETOX_WaterConc_patch`
- `ETOX_RUD_patch`

The accompanying `applications.txt` file is also written for downstream inspection and BeeView-server exposure summaries.

## Stage 3. How BEEHAVEecotox Reads the Landscape Exposure

The NetLogo experiment used by xPollinator is templated here:
- [model/variant/BeeHaveEcotox/template/experiment.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/BeeHaveEcotox/template/experiment.xml)

Important settings in that experiment template:
- `ReadInfile = true`
- `INPUT_FILE = "Sources.txt"`

That means BEEHAVEecotox uses the patch-specific exposure information written by xPollinator rather than the two-patch GUI-only setup.

The updated landscape extension is documented in the bundled reports (located in `model/variant/BeeHaveEcotox/BEEHAVEecotox/` inside the xPollinator installation):
- `R2260099-1_ODD_Amendment_BEEHAVEecotox_lanscape_extension.pdf`
- `R2260099-2_Manual_BEEHAVEecotox_landscape_extension.pdf`

These reports confirm that the new input-file format stores one flowering period per patch plus the ecotoxicological application fields listed above.

## Stage 4. Patch-Level Fate in BEEHAVEecotox

The main patch-level exposure procedure is in `model/variant/BeeHaveEcotox/BEEHAVEEcotox/A7-ModelCode_BEEHAVE-ECOTOX.nlogo` (bundled with the BEEHAVEecotox installation).

### What happens on application day

When the current simulation day matches `ETOX_NextAppDay_patch`, the model:

1. Adds nectar residue concentration to the active nectar contamination pool.
2. Adds pollen residue concentration to the active pollen contamination pool.
3. Converts the patch contact term into a per-bee contact dose using `ETOX_RUD_patch`.
4. Optionally activates water contamination if water foraging is enabled.

### How patch decline is modelled

After activation, the patch-level residue declines by first-order kinetics using the selected substance DT50:

$$
C_{t+1} = C_t \cdot e^{-\ln(2) / DT50}
$$

This same form is applied to:
- nectar concentration at the patch
- pollen concentration at the patch
- contact dose term at the patch
- water concentration at the patch, if water foraging is enabled

### When patch exposure is reset

If the configured exposure period ends and no overlapping application starts on the same day, the patch-level contamination terms are reset to zero. Otherwise, they continue to decline or accumulate depending on the application sequence.

### Key patch-level parameters affecting fate

| Parameter | Meaning | Where to change |
|---|---|---|
| `ETOX_ApplicationList_patch` | Start day(s) of application | generated by xPollinator in `Sources.txt` |
| `ETOX_ExposurePeriodsList_patch` | Duration of each application | generated by xPollinator in `Sources.txt` |
| `ETOX_PPPConcNectar_patch` | Initial nectar concentration at patch | generated by xPollinator in `Sources.txt` |
| `ETOX_PPPConcPollen_patch` | Initial pollen concentration at patch | generated by xPollinator in `Sources.txt` |
| `ETOX_PPPContact_patch` | Initial contact application rate term | generated by xPollinator in `Sources.txt` |
| `ETOX_RUD_patch` | Contact conversion factor | currently written as `21` by xPollinator |
| `etox_DT50_Substance` | Residue dissipation half-life on patch | substance definition in NetLogo |
| `ETOX_contactexp_oneday` | If true, contact exposure only occurs on application day | experiment/default settings in NetLogo |
| `Etox_ContactSum` | If true, contact doses sum over visits instead of averaging | NetLogo switch/default |

## Stage 5. Uptake by Foragers

Forager uptake occurs during nectar, pollen, and optional water collection in `model/variant/BeeHaveEcotox/BEEHAVEEcotox/A7-ModelCode_BEEHAVE-ECOTOX.nlogo`.

### Nectar route

When a forager returns from a nectar patch:
- patch nectar concentration is converted into an oral dose in the honey stomach
- part of that dose remains with the forager
- part is transferred into hive honey stores

### Pollen route

When a pollen forager returns:
- pollen contamination is carried in the pollen basket
- the resulting pollen dose enters later in-hive feeding processes

### Contact route

Contact dose is updated on flower visits using the patch-specific contact term. The model can either:
- sum contact exposure over visits, or
- average it,

depending on the `Etox_ContactSum` switch.

### Water route

If `ETOX_Water_foraging` is enabled:
- contaminated water can be collected from patches
- part of the dose enters the forager
- part is mixed into stored hive resources

## Stage 6. In-Hive Fate

In-hive exposure update is handled in `model/variant/BeeHaveEcotox/BEEHAVEEcotox/A7-ModelCode_BEEHAVE-ECOTOX.nlogo`.

### Honey compartments

BEEHAVEecotox tracks pesticide concentrations in several honey compartments:
- uncapped honey of today (`D0`)
- uncapped honey of previous days (`D1` to `D4`)
- capped honey

This allows daily inflow, ageing, and mixing of contaminated nectar-derived stores.

### Nurse-bee filter effect

Larval exposure is not identical to the raw hive concentration. Two filter factors modify the effective dose passed on by nurse bees:
- `etox_FF_Nursebees_Nectar_Substance`
- `etox_FF_Nursebees_Pollen_Substance`

These scale how much of the nectar- and pollen-derived dose reaches larvae.

### Optional degradation in honey

If honey degradation is enabled for the selected substance, in-hive concentrations also decline by first-order kinetics using `etox_DT50_honey_Substance`.

The current code applies this to all honey compartments before the daily compartment shift and recapping logic.

## Stage 7. Toxic Effects

Mortality is then computed from daily oral and contact doses using dose-response functions based on LD50 and slope parameters.

These are applied separately for:
- foragers
- in-hive bees
- larvae
- drones

The key substance parameters are:
- `etox_Forager_Oral_LD50_Substance`
- `etox_Forager_Oral_slope_Substance`
- `etox_Forager_contact_LD50_Substance`
- `etox_Forager_contact_slope_Substance`
- `etox_Larvae_Oral_LD50_Substance`
- `etox_Larvae_Oral_slope_Substance`
- `etox_Forager_ImmediateMortality_Substance`

## Parameter Reference: Where to Change What

### A. xPollinator landscape and application sampling

| Change goal | Parameter(s) | File |
|---|---|---|
| Change treated vs untreated application counts | `treated_min_applications`, `treated_max_applications` | [scripts/Scenario-TarnEtGaronne_manifest.json](https://github.com/xlandscape/xPollinator/blob/main/scripts/Scenario-TarnEtGaronne_manifest.json) |
| Change hive-specific reproducibility | `random_seed` per hive | [scripts/Scenario-TarnEtGaronne_manifest.json](https://github.com/xlandscape/xPollinator/blob/main/scripts/Scenario-TarnEtGaronne_manifest.json) |
| Change the treated vegetation type | `AppliedVegetationId` | [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml) |
| Change allowed application window | `FirstDayOfYearApplications`, `LastDayOfYearApplications` | [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml) |
| Change mean and variability of nectar residues | `ConcNectarMean`, `ConcNectarStd` | [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml) |
| Change mean and variability of pollen residues | `ConcPollenMean`, `ConcPollenStd` | [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml) |
| Change mean and variability of contact term | `ContactToxicityMean`, `ContactToxicityStd` | [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml) |
| Change duration written for each exposure event | `ExposurePeriod` | [model/variant/mc.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/mc.xml) |

### B. Patch-level fate inside BEEHAVEecotox

| Change goal | Parameter(s) | File |
|---|---|---|
| Change residue dissipation speed on patch | `etox_DT50_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |
| Change contact conversion factor | `ETOX_RUD_patch` or GUI `ETOX_RUD` | `Sources.txt` generated by xPollinator or NetLogo GUI mode |
| Make contact only occur on application day | `ETOX_contactexp_oneday` | [model/variant/BeeHaveEcotox/template/experiment.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/BeeHaveEcotox/template/experiment.xml) or NetLogo default |
| Sum rather than average repeated contact visits | `Etox_ContactSum` | NetLogo default/interface |
| Enable water route | `ETOX_Water_foraging` | [model/variant/BeeHaveEcotox/template/experiment.xml](https://github.com/xlandscape/xPollinator/blob/main/model/variant/BeeHaveEcotox/template/experiment.xml) |

### C. In-hive fate and toxicity

| Change goal | Parameter(s) | File |
|---|---|---|
| Change in-hive honey degradation | `etox_degradation_honey_Substance`, `etox_DT50_honey_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |
| Change nectar filtering to larvae | `etox_FF_Nursebees_Nectar_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |
| Change pollen filtering to larvae | `etox_FF_Nursebees_Pollen_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |
| Change forager oral sensitivity | `etox_Forager_Oral_LD50_Substance`, `etox_Forager_Oral_slope_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |
| Change forager contact sensitivity | `etox_Forager_contact_LD50_Substance`, `etox_Forager_contact_slope_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |
| Change larval oral sensitivity | `etox_Larvae_Oral_LD50_Substance`, `etox_Larvae_Oral_slope_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |
| Enable acute immediate forager mortality | `etox_Forager_ImmediateMortality_Substance` | NetLogo substance definition or `SUBSTANCE_FILE` |

## Important Notes for the SETAC Tarn-et-Garonne Training Setup

1. **Patch concentrations are currently generated in xPollinator, not entered manually in NetLogo.**
   For SETAC training runs, the operational source of patch exposure is the `Sources.txt` file written by xPollinator.

2. **The treated scenario currently allows zero applications on a patch.**
   This follows directly from the current manifest setting `treated_min_applications = 0`.

3. **The same application pattern is reused across replicate colony runs for a given hive and treatment.**
   Replicates differ through BEEHAVE random seeds, not through independently resampled application patterns.

4. **Patch residue decline and in-hive honey decline are separate processes.**
   `etox_DT50_Substance` governs decline on the patch. `etox_DT50_honey_Substance` governs optional decline inside hive honey stores.

## Recommended Reading Path

If you are following the SETAC training material, read the pages in this order:

1. [xPollinator overview](index.md)
2. [Complete workflow tutorial](complete-workflow.md)
3. [This pesticide fate page](pesticide-fate.md)

The workflow tutorial explains how to run the model. This page explains how to interpret the pesticide-related parameters in those runs.
