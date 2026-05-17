# Complete xPollinator Workflow Tutorial

This tutorial walks you through the entire xPollinator landscape simulation and visualization pipeline on Windows: from setting up the software, running a simple simulation, importing results into the visualization server, and exploring outputs in the interactive BeeView interface.

**Estimated time**: 2-3 hours for the complete workflow (including initial setup and downloads)

---

## 1. The xLandscape Components: An Overview

The xPollinator workflow integrates four key software components:

### 1.1 xLandscape Framework (Core Engine)

The **[xLandscape framework](https://github.com/xlandscape)** is a microkernel-based landscape modeling platform that provides:
- **Component-based architecture**: Modular software components that transform spatial and temporal data
- **Semantic data handling**: Automatic tracking of units, coordinate systems, and data scales
- **Multidimensional storage**: HDF5-based data store for efficient access to large spatiotemporal arrays
- **Monte Carlo simulation**: Support for propagating natural variability across replicate runs

### 1.2 xPollinator Landscape Model

**[xPollinator](https://github.com/xlandscape/xPollinator)** is a landscape-scale model that simulates:
- **Land use/land cover** processing: Reads GIS shapefiles and categorizes vegetation types
- **Bee forage availability**: Computes daily nectar and pollen availability from vegetation phenology
- **Pesticide exposure**: Calculates how pesticide applications contaminate foraging resources
- **Population dynamics**: Feeds forage and exposure data to the BEEHAVEecotox model

For a detailed explanation of the pesticide lifecycle from field application to residue decline and colony exposure, see [Pesticide Fate in xPollinator and BEEHAVEecotox](pesticide-fate.md).

**Data pipeline**:
```
Landscape Scenario (shapefile + vegetation tables)
    ↓
Vegetation classification per land parcel
    ↓
Daily nectar/pollen timeseries (365 days)
    ↓
Pesticide applications → exposure in forage
    ↓
Input preparation for bee colony model
    ↓
HDF5 output store (arr.dat)
```

### 1.3 BEEHAVEecotox (Bee Colony Model)

**BEEHAVEecotox** is a NetLogo-based honeybee colony simulation that models:
- **Colony dynamics**: Worker bees, brood, drones, food stores over time
- **Foraging behavior**: Resource collection and consumption
- **Varroa parasites**: Pest mortality and effects
- **Toxicological effects**: How pesticide exposure reduces bee survival and reproduction

**Key feature**: Multiple replicates (typically 100) of the same scenario use different random seeds for population variability, allowing probabilistic assessment of colony survival.

**Output**: CSV file with daily colony metrics (workers, brood, foragers, food stores) for each replicate.

### 1.4 BeeView-server (Data Processing Backend)

**[BeeView-server](https://github.com/xlandscape/BeeView-server)** is a FastAPI web service that:
- Loads xPollinator HDF5 outputs and BEEHAVEecotox CSV results
- Stores data in a DuckDB spatial database with normalized tables
- Serves REST API endpoints for spatial (GeoJSON features) and temporal (timeseries) queries
- Calculates derived metrics (averages, percentiles, survival probabilities)

**In your workflow**: Bridge between simulation outputs and interactive visualization.

### 1.5 BeeView (Interactive Frontend)

**[BeeView](https://github.com/xlandscape/BeeView)** is a React + Vite web application providing:
- **Interactive map**: Visualize nectar, pollen, vegetation, or pesticide exposure across the landscape (see [Map Layers](https://github.com/xlandscape/BeeView/tree/develop/docs/map-layers.md))
- **Time-series charts**: Daily nectar/pollen availability and bee population metrics (see [Nectar & Pollen Charts](https://github.com/xlandscape/BeeView/tree/develop/docs/charts-nectar-pollen.md), [Bee Population Charts](https://github.com/xlandscape/BeeView/tree/develop/docs/charts-bee-population.md))
- **Comparison view**: Analyze survival differences between treated and untreated scenarios (see [Compare Runs](https://github.com/xlandscape/BeeView/tree/develop/docs/charts-compare.md))
- **Feature selection**: Click map polygons to drill down into specific parcels

For the full BeeView user guide, see the [BeeView Documentation](https://github.com/xlandscape/BeeView/tree/develop/docs/index.md).

---

## 2. Prerequisites

Before starting, ensure you have the following on your Windows machine:

### 2.1 Software Requirements

| Software | Version | Purpose | Download |
|----------|---------|---------|----------|
| **Git** | ≥2.30 | Version control & cloning repos | [git-scm.com](https://git-scm.com/download/win) |
| **Python** | 3.9.7+ | Required for xLandscape | Bundled in xPollinator (see §3.1) |
| **Java runtime (JDK 24)** | xLandscape bundle | Required by BEEHAVEecotox | [jdk-24-v1.0.zip](https://xlandscape.org/releases/jdk-24-v1.0.zip) |
| **Node.js** | ≥18 | JavaScript runtime for BeeView | [nodejs.org](https://nodejs.org/en/) (LTS recommended) |
| **Python virtualenv** | Built-in | Virtual environment | `pip install virtualenv` |
| **Text editor/IDE** | VS Code recommended | For viewing configs & logs | [code.visualstudio.com](https://code.visualstudio.com/) |

### 2.2 Hardware Requirements

- **Disk space**: ~5 GB (xPollinator framework + demo scenarios + node_modules)
- **RAM**: 8 GB minimum; 16 GB recommended
- **Network**: During setup to download dependencies and clone repositories

### 2.3 System Configuration

- **Windows 10/11** with command prompt or PowerShell
- **Firewall**: May need to allow localhost traffic on ports 32000 (server) and 8081 (frontend)
- **SSL/Certificate issues**: If behind corporate VPN, see [Git network troubleshooting](../../reference/troubleshooting.md)

### 2.4 One-Time Java Runtime Setup for BeeHaveEcotox

Before running any xPollinator simulation with BEEHAVEecotox, download and extract the Java runtime bundle:

1. Download: [jdk-24-v1.0.zip](https://xlandscape.org/releases/jdk-24-v1.0.zip)
2. Extract the folder `jdk-24` to:
  `C:\xLandscape\xPollinator\model\variant\BeeHaveEcotox\`
3. Verify the file exists:
  `C:\xLandscape\xPollinator\model\variant\BeeHaveEcotox\jdk-24\bin\java.exe`

If this runtime is missing, simulation startup fails with: `Java runtime not found`.

---

## 3. Setting Up Your Local xLandscape Environment

### 3.1 Clone the xPollinator Repository

Open PowerShell and navigate to where you want to install xPollinator:

```powershell
# Create a workspace folder
mkdir C:\xLandscape
cd C:\xLandscape

# Clone xPollinator (includes bundled Python and xLandscape core)
git clone https://github.com/xlandscape/xPollinator.git
cd xPollinator

# Initialize required submodules (model/core and BeeHaveEcotox)
git submodule update --init --recursive

# Clone BeeView-server (data processing backend)
cd ..
git clone https://github.com/xlandscape/BeeView-server.git

# Clone BeeView (frontend visualization)
git clone https://github.com/xlandscape/BeeView.git

# BeeView source code is on develop branch
cd BeeView
git fetch --all
git checkout develop
cd ..
```

**Directory structure after cloning**:
```
C:\xLandscape\
├── xPollinator/
│   ├── model/core/          (xLandscape framework + bundled Python)
│   ├── model/variant/       (xPollinator-specific components)
│   ├── scenario/            (landscape scenarios)
│   └── run/                 (simulation outputs)
├── BeeView-server/
│   ├── model/               
│   ├── data/                (HDF5 and CSV files go here)
│   └── main.py              (FastAPI server)
└── BeeView/
    ├── src/
    └── package.json
```

### 3.2 Verify xPollinator Installation

The bundled Python interpreter is located at `model/core/bin/python-3.9.7-amd64/python.exe`. Verify it works:

```powershell
cd C:\xLandscape\xPollinator
model\core\bin\python-3.9.7-amd64\python.exe --version
# Output: Python 3.9.7
```

### 3.3 Set Up BeeView-server

Navigate to the BeeView-server directory and install Python dependencies:

```powershell
cd C:\xLandscape\BeeView-server

# Create a virtual environment
python -m venv venv

# Activate it
venv\Scripts\Activate.ps1

# If you encounter execution policy issues, run:
# Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Install dependencies
pip install --upgrade pip
pip install -r requirements.txt

# If pip fails with SSL certificate errors (common on corporate VPNs), run:
# pip install --verbose --trusted-host pypi.org --trusted-host pypi.python.org --trusted-host files.pythonhosted.org pip-system-certs
# pip install --trusted-host pypi.python.org --trusted-host files.pythonhosted.org --trusted-host pypi.org -r requirements.txt
```

**Expected output**: Installation of FastAPI, DuckDB, h5py, numpy, geopandas, shapely, SQLAlchemy, and other dependencies (~100 MB).

### 3.4 Set Up BeeView Frontend

Install Node.js dependencies for the React + Vite application:

```powershell
cd C:\xLandscape\BeeView

# Install npm packages
npm install

# Verify Vite is installed
npm list vite
# Should show: vite@6.x.x
```

---

## 4. Running a Simple xPollinator Simulation (Single Hive, Untreated)

This section walks through a single-hive, untreated scenario using the Tarn-et-Garonne demo data.

### 4.1 Understand the Parameter File (.xrun)

xPollinator is configured via XML parameter files (.xrun). A template is provided:

**File**: `C:\xLandscape\xPollinator\template.xrun`

**Key parameters** (edit as needed):

```xml
<Parameters>
  <SimID>my_first_run</SimID>
  <Project>scenario/Tarn-et-Garonne-small</Project>
  <NumberMC>1</NumberMC>
  <SimulationStart>2021-01-01</SimulationStart>
  <SimulationEnd>2021-12-31</SimulationEnd>
  <NumberBeeHaveTimesteps>365</NumberBeeHaveTimesteps>
  <BeeHaveMapCenterPointX>127406.9408</BeeHaveMapCenterPointX>
  <BeeHaveMapCenterPointY>5482559.8501</BeeHaveMapCenterPointY>
  <NumberBeeHaveReplicates>1</NumberBeeHaveReplicates>
  <BeeHaveRandomSeed>1</BeeHaveRandomSeed>
  <BeeHaveWeather>Rothamsted (2009)</BeeHaveWeather>
  <MinNumberApplications>0</MinNumberApplications>
  <MaxNumberApplications>0</MaxNumberApplications>
  <RunLabel>my_first_run</RunLabel>
  <HiveGroupId>default</HiveGroupId>
</Parameters>
```

| Parameter | Meaning | Example |
|-----------|---------|---------|
| `SimID` | Unique name for this run (creates `run/<SimID>/`) | `my_first_run` |
| `Project` | Path to landscape scenario (relative to repository root) | `scenario/Tarn-et-Garonne-small` |
| `NumberMC` | Number of Monte Carlo runs | `1` |
| `NumberBeeHaveTimesteps` | Simulation duration in days (typically 365) | `365` |
| `BeeHaveMapCenterPointX` | Hive X coordinate (UTM Zone 31N for France) | `127406.9408` |
| `BeeHaveMapCenterPointY` | Hive Y coordinate (UTM Zone 31N for France) | `5482559.8501` |

⚠️ **Important**: `SimID` must be unique. If you re-run with the same `SimID`, you will overwrite the previous results.

### 4.2 Create Your First Parameter File

Create a new file in the xPollinator root:

**File**: `C:\xLandscape\xPollinator\tutorial_single_untreated.xrun`

```xml
<?xml version="1.0" encoding="utf-8"?>
<Parameters>
  <SimID>tutorial_single_untreated</SimID>
  <Project>scenario/Tarn-et-Garonne-small</Project>
  <NumberMC>1</NumberMC>
  <SimulationStart>2021-01-01</SimulationStart>
  <SimulationEnd>2021-12-31</SimulationEnd>
  <NumberBeeHaveTimesteps>365</NumberBeeHaveTimesteps>
  <BeeHaveMapCenterPointX>127406.9408</BeeHaveMapCenterPointX>
  <BeeHaveMapCenterPointY>5482559.8501</BeeHaveMapCenterPointY>
  <NumberBeeHaveReplicates>1</NumberBeeHaveReplicates>
  <BeeHaveRandomSeed>1</BeeHaveRandomSeed>
  <BeeHaveWeather>Rothamsted (2009)</BeeHaveWeather>
  <MinNumberApplications>0</MinNumberApplications>
  <MaxNumberApplications>0</MaxNumberApplications>
  <RunLabel>tutorial_single_untreated</RunLabel>
  <HiveGroupId>default</HiveGroupId>
</Parameters>
```

**Note**: This scenario is **untreated** because the xPollinator component pipeline is configured to skip pesticide applications (see `model/variant/experiment.xml`).

### 4.3 Run the Simulation

Navigate to the xPollinator root and run the simulation:

```powershell
cd C:\xLandscape\xPollinator

# Method 1: Drag & drop
# Drag tutorial_single_untreated.xrun onto __start__.bat in Windows Explorer

# Method 2: Command line (PowerShell)
.\__start__.bat tutorial_single_untreated.xrun
```

**Expected runtime**: ~10-15 minutes

**Console output** (watch for progress):
```
[INFO] Loading LandscapeScenario...
[INFO] Loading vegetation classes...
[INFO] Computing BeeForage (nectar/pollen timeseries)...
[INFO] Preparing BeeHaveEcotox inputs...
[INFO] Running BEEHAVEecotox (1 replicate)...
[INFO] Simulation complete!
```

### 4.4 Verify Output

After the simulation completes, check the output directory:

```powershell
# Navigate to outputs
cd C:\xLandscape\xPollinator\run\tutorial_single_untreated\mcs\0

# List files
ls

# Expected structure:
# store/
#   └── arr.dat          (HDF5 multidimensional data store)
# processing/
#   └── BeeHave/
#       ├── output.csv   (1 replicate x 365 days of colony metrics in this tutorial)
#       └── segments.shp (radial grid geometry)
```

**Key output files**:
- `arr.dat` — HDF5 store with nectar, pollen, vegetation, feature IDs (~50 MB)
- `output.csv` — BEEHAVE results with colony metrics (workers, brood, foragers)

---

## 5. Setting Up BeeView-server and Importing the Run

### 5.1 Copy Output Files to BeeView-server Data Directory

Copy the simulation outputs to where BeeView-server expects them:

```powershell
# Copy HDF5 store
Copy-Item "C:\xLandscape\xPollinator\run\tutorial_single_untreated\mcs\0\store\arr.dat" `
          "C:\xLandscape\BeeView-server\data\arr.dat"

# Copy BEEHAVE population metrics
Copy-Item "C:\xLandscape\xPollinator\run\tutorial_single_untreated\mcs\0\processing\BeeHave\output.csv" `
          "C:\xLandscape\BeeView-server\data\output.csv"

# Copy land use/land cover shapefile (already in scenario, but ensure it's in data/)
Copy-Item "C:\xLandscape\xPollinator\scenario\Tarn-et-Garonne-small\geo\lulc.*" `
          "C:\xLandscape\BeeView-server\data\"

# Verify files are present
ls C:\xLandscape\BeeView-server\data\
```

**Expected directory contents**:
```
data/
├── arr.dat                                  (HDF5 store ~50 MB)
├── beeview.duckdb                          (Database, may be recreated)
├── lulc.shp, lulc.dbf, lulc.prj, lulc.shx (Shapefile components)
├── output.csv                              (BEEHAVE results)
├── vegetation classes.json                 (Vegetation definitions)
├── land cover to vegetation default mapping.csv
└── applications.txt                        (Optional: pesticide applications)
```

### 5.2 Start BeeView-server

In a **new PowerShell window** (keep xPollinator window open if still running), navigate to BeeView-server and start the server:

```powershell
cd C:\xLandscape\BeeView-server

# Activate virtual environment
venv\Scripts\Activate.ps1

# Start the server
python main.py
```

**Expected output**:
```
[INFO] Loading LandscapeScenario data...
[INFO] Loading vegetation classes...
[INFO] Loading vegetation timeseries...
[INFO] Loading nectar timeseries...
[INFO] Loading pollen timeseries...
[INFO] Loading bee population metrics...
[INFO] Server ready at http://0.0.0.0:32000
```

⚠️ **Database initialization**: On first run, the server creates `data/beeview.duckdb` and loads all data. This may take 2-5 minutes. You should see progress messages.

### 5.3 Verify Server is Running

Open a browser and test the API:

```
http://localhost:32000/docs
```

You should see an interactive **Swagger UI** listing all available endpoints. This confirms the server is running correctly.

**Key endpoints** (visible in Swagger):
- `/geojson` — Returns all land cover features as GeoJSON
- `/geojson/viewport` — Returns features within map bounds
- `/api/nectar/max` — Maximum nectar values per feature
- `/api/pollen/max` — Maximum pollen values per feature
- `/api/timeseries/averages` — Daily nectar/pollen averages
- `/api/bee-population/timeseries` — Bee colony metrics over time

---

## 6. Running the BeeView Frontend

### 6.1 Start the Frontend Development Server

In a **third PowerShell window**, navigate to the BeeView directory and start the frontend:

```powershell
cd C:\xLandscape\BeeView

# Start Vite development server (will auto-open http://localhost:8081)
npm run dev
```

**Expected output**:
```
VITE v6.4.0  ready in XXX ms

➜  Local:   http://localhost:8081/
➜  press h to show help
```

The browser will open automatically to `http://localhost:8081/`. If not, open it manually.

### 6.2 Verify Connection to Server

Once BeeView loads, you should see:
1. An **interactive map** showing the Tarn-et-Garonne landscape with land parcels
2. A **left sidebar** with layer controls, time slider, and statistics
3. **No red error messages** (if red messages appear, see [Troubleshooting](#troubleshooting))

### 6.3 Interface Overview

#### Map View

**[PLACEHOLDER: Add screenshot of main map interface with labeled components]**

The central map displays land cover classes using color-coded polygons (see [Map Layers](https://github.com/xlandscape/BeeView/tree/develop/docs/map-layers.md) for full details):

- **Layer Controls** (top left): Toggle between visualizations:
  - **Land Use (LULC)**: Shows land cover classification (arable, grassland, forest, water, etc.)
  - **Nectar**: Daily nectar availability [mg/m²/day], color intensity = amount
  - **Pollen**: Daily pollen availability [g/m²/day]
  - **Vegetation**: Vegetation classes per parcel
  - **Beehive**: Center point of the simulated hive

- **Time Slider** (bottom left): Scrub through the 365-day simulation
  - Day 1 = January 1
  - Day 365 = December 31

#### Left Sidebar

**[PLACEHOLDER: Add screenshot of sidebar with annotations]**

The left panel provides data summaries and controls (see [Sidebar](https://github.com/xlandscape/BeeView/tree/develop/docs/sidebar.md) for full details):

- **Run Selector**: Choose which simulation run to view (if multiple runs imported) — see [Run Selection](https://github.com/xlandscape/BeeView/tree/develop/docs/run-selection.md)
- **Layer Controls**: Toggle map layers on/off
- **Statistics Panel**: Shows aggregated statistics for selected layer/time
- **Feature List** (if features selected on map): Drill-down data for clicked parcels

#### Time Series Chart

**[PLACEHOLDER: Add screenshot of time series chart]**

Below the map, the **Time Series Chart** displays (see [Nectar & Pollen Charts](https://github.com/xlandscape/BeeView/tree/develop/docs/charts-nectar-pollen.md)):

- **X-axis**: Day of year (1-365)
- **Y-axis**: Nectar or pollen availability [units/m²/day]
- **Line color**: Green for selected features, gray for unselected
- **Interaction**: Click points to see exact values; hover for tooltips

#### Bee Population Chart

**[PLACEHOLDER: Add screenshot of bee population chart]**

A secondary chart shows **colony metrics over time** (see [Bee Population Charts](https://github.com/xlandscape/BeeView/tree/develop/docs/charts-bee-population.md)):

- **Worker bees**: Daily active foragers
- **Brood**: Developing bee cohorts
- **Food stores**: Nectar and honey reserves
- **Varroa load**: Parasite population (if applicable)

### 6.4 Interacting with the Map

Try these interactions to explore the data:

1. **Select a parcel**: Click any polygon on the map
   - The parcel should highlight (outline)
   - The Time Series Chart updates to show data for that parcel
   - Sidebar statistics update

2. **Select multiple parcels**: Hold Ctrl and click additional parcels
   - The Time Series Chart will show averages across selected parcels

3. **Change the layer**: Use Layer Controls to switch from Nectar to Pollen
   - Map colors update to show pollen availability instead
   - Time Series Chart switches to pollen values

4. **Advance time**: Drag the Time Slider or use keyboard arrows
   - Map colors reflect nectar/pollen for that specific day
   - All charts update to show current time point

5. **Pan and zoom**: Use mouse wheel to zoom; drag to pan
   - Time Series Chart remains synchronized
   - Viewport information is sent to the server for optimization

### 6.5 Understanding Untreated Scenario Results

For your untreated simulation, you should observe:

- **Nectar/Pollen**: Strong seasonal patterns with peaks in spring/summer
- **Bee Population**: Colony grows in spring as resources increase, shrinks in fall
- **No pesticide exposure**: Pesticide widgets and exposures not visible (scenario is untreated). For treated runs, the Exposure tab shows daily residue timeseries — see [Exposure Charts](https://github.com/xlandscape/BeeView/tree/develop/docs/charts-exposure.md)

**Sample observations**:
- Day 60-150 (March-May): High nectar/pollen availability, growing bee population
- Day 1-59, 289-365 (Winter): Low forage, stable/declining colony size

---

## 7. Exploring More Complex Setups: Multi-Hive and Treated Scenarios

### 7.1 Understanding the Manifest Configuration

For more sophisticated analyses, xPollinator supports **batch runs** using a **manifest** configuration file. The manifest defines:

- Multiple hive locations and treatment scenarios
- Number of replicates per scenario
- Pesticide application schedules

The manifest controls how often applications are sampled for treated runs, but the full residue lifecycle is controlled by additional xPollinator and BEEHAVEecotox parameters described in [Pesticide Fate in xPollinator and BEEHAVEecotox](pesticide-fate.md).

**Example manifest**: `xPollinator/scripts/Scenario-TarnEtGaronne_manifest.json`

```json
{
  "description": "Tarn-et-Garonne: 5 hive locations, treated vs untreated",
  "scenario": "scenario/Tarn-et-Garonne",
  "replicates": 100,
  "weather": "Weather File",
  "hives": [
    {
      "group_id": 3614,
      "location": "Hive A",
      "seed": 1001,
      "coordinates": [127406.9408, 5482559.8501]
    },
    {
      "group_id": 2114,
      "location": "Hive B",
      "seed": 1002,
      "coordinates": [125000.0000, 5480000.0000]
    }
  ],
  "treatments": [
    {
      "name": "untreated",
      "min_applications": 0,
      "max_applications": 0
    },
    {
      "name": "treated",
      "min_applications": 1,
      "max_applications": 5
    }
  ]
}
```

**Key concepts**:

- **Replicates**: Typically 100. Each replicate runs BEEHAVEecotox with a different random seed (same applications, different population dynamics)
- **Hives**: List of hive locations with group IDs and center coordinates
- **Treatments**: Defines pesticide application scenarios (untreated = 0 apps, treated = 1-5 apps)

### 7.2 Running a Multi-Hive Batch

Create a new parameter file for a manifest-based batch run:

**File**: `C:\xLandscape\xPollinator\tutorial_multi_hives.xrun`

```xml
<Parameters>
  <SimID>tutorial_multi_hives</SimID>
  <Project>scenario/Tarn-et-Garonne</Project>
  <Manifest>scripts/Scenario-TarnEtGaronne_manifest.json</Manifest>
  <NumberBeeHaveTimesteps>365</NumberBeeHaveTimesteps>
</Parameters>
```

Run the manifest:

```powershell
cd C:\xLandscape\xPollinator
.\__start__.bat tutorial_multi_hives.xrun
```

**What happens**:
1. xPollinator processes each combination of hive + treatment
2. For each combination, 100 BEEHAVE replicates are executed
3. Output is stored separately for each hive and treatment (e.g., `run/tutorial_multi_hives/<hive_id>/<treatment>/`)

**Expected duration**: 30-60 minutes (depending on hardware and number of hives/treatments)

### 7.3 Importing Multi-Run Data into BeeView-server

For multi-run analyses, use the `import_run.py` script to batch-import multiple xPollinator outputs:

```powershell
cd C:\xLandscape\BeeView-server

# Activate environment
venv\Scripts\Activate.ps1

# Import a single run
python import_run.py --sim-id tutorial_multi_hives --run-path ../xPollinator/run/tutorial_multi_hives

# Or import interactively (script will prompt for paths)
python import_run.py
```

**Database structure** after import:

The DuckDB database now contains multiple "runs" with separate nectar/pollen/population tables:
- `runs` table: Stores run metadata (name, scenario, hive location, etc.)
- Timeseries tables reference the `run_id` for cross-run comparison

### 7.4 Using BeeView's Compare Runs Feature

Once multiple runs are imported, BeeView can compare them:

1. **Open BeeView** (refresh if already open): `http://localhost:8081/`

2. **Run Selector** (top left sidebar):
   - You should see multiple runs listed
   - Select multiple runs (Ctrl+click)

3. **View Modes** (top right):
   - Switch from "Single Run" to "Compare Runs"

4. **Percentile Reduction Matrix** displays (see [Compare Runs](https://github.com/xlandscape/BeeView/tree/develop/docs/charts-compare.md) for full computation details):

   **[PLACEHOLDER: Add screenshot of percentile reduction matrix]**

   The matrix is a **9×9 grid** where rows represent **spatial percentiles** (p10–p90 across hive locations) and columns represent **temporal percentiles** (p10–p90 across days). Each cell shows the percent reduction of colony size in the treated run relative to the untreated baseline.

   The computation follows four steps:

   a. **Daily reduction per replicate pair**: For each day and matched replicate, compute the fractional colony-size reduction $r_i(d) = (B_i(d) - S_i(d)) / B_i(d)$, where $B$ is baseline (untreated) and $S$ is scenario (treated).

   b. **Average across replicates**: Average the daily reductions across all replicate pairs within each spatial unit (hive location × landscape realisation).

   c. **Temporal percentiles**: For each spatial unit, compute percentiles (p10–p90) across all simulation days. Low percentiles capture "good" days; high percentiles capture "worst" days.

   d. **Spatial percentiles**: Across all spatial units, compute percentiles of each temporal percentile. Low percentiles represent mildly affected locations; high percentiles represent the most severely affected locations.

   **Color coding**: White/green = negligible effect, yellow = 5–10%, orange = 10–20%, red = >20% reduction. Blue indicates the treated colony is larger (stochastic variability).

### 7.5 Interpreting Results

For the Tarn-et-Garonne multi-hive scenario:

**Expected patterns**:

- **Low spatial + low temporal percentiles** (top-left corner): Represent the mildest effects — most locations on most days see little colony-size reduction
- **High spatial + high temporal percentiles** (bottom-right corner): Represent worst-case combinations — the most affected locations on the worst days
- **Diagonal trend**: Values typically increase from top-left to bottom-right
- **Hive location variation**: Hives in richer forage landscapes show smaller reductions (lower spatial percentiles)
- **Replicates**: Averaging across 100 BEEHAVE replicates per run smooths stochastic noise

**Ecosystem risk perspective**:

- The spatial dimension of the matrix allows estimating **landscape-level risk** (what fraction of hive locations experience meaningful effects)
- The temporal dimension shows whether effects are transient (high only at extreme percentiles) or persistent (high across all percentiles)

---

## 8. Troubleshooting

### Server Connection Issues

**Problem**: BeeView shows red error "Cannot connect to server"

**Solution**:
1. Verify BeeView-server is running: Open `http://localhost:32000/docs` in browser
2. Check port is not blocked: Some firewalls block port 32000
3. Check data was loaded: Server console should show progress messages (may take 5 minutes on first run)

### Missing Data Files

**Problem**: Server starts but no map shows; console shows "No features found"

**Solution**:
1. Verify files are in `BeeView-server/data/`:
   ```powershell
   ls C:\xLandscape\BeeView-server\data\
   ```
2. Check file permissions (should be readable by your user)
3. Ensure `arr.dat` is not corrupted: HDF5 file should be >40 MB for Tarn-et-Garonne-small
4. Delete `data/beeview.duckdb` and restart server to force re-import

### Slow Performance

**Problem**: Map is sluggish; charts lag behind time slider

**Solution**:
- This is expected on first-time viewport loads; subsequent pans/zooms are faster (cached)
- Reduce number of selected features (click fewer parcels)
- Close other applications to free RAM
- For large datasets, consider using production build: `npm run build`, then serve via `npm run preview`

### xPollinator Simulation Fails

**Problem**: `__start__.bat` closes or shows error

**Solution**:
1. Check .xrun file has valid XML (no typos in tags)
2. Verify coordinates are in correct CRS (UTM for Tarn-et-Garonne, EPSG:32631)
3. Ensure scenario path exists: `C:\xLandscape\xPollinator\scenario\Tarn-et-Garonne-small\`
4. Check disk space (needs ~500 MB per run)
5. Re-run with unique `<SimID>` (don't overwrite previous runs)

---

## 9. Next Steps

### For Risk Assessment Work

1. **Adapt scenarios**: Modify landscape data or pesticide schedules for your region of interest
2. **Sensitivity analysis**: Run multiple scenarios (e.g., different application timings) and compare survival curves
3. **Population-level risk**: Use multi-hive results to estimate landscape-scale impact

### For Advanced Customization

1. **BeeHave parameters**: Modify colony initial conditions, food consumption rates in `model/variant/BeeHaveEcotox/BEEHAVEecotox.txt`
2. **Vegetation phenology**: Update nectar/pollen availability tables in the scenario folder
3. **Exposure routes**: Extend pesticide modules to include soil contact or water exposure
4. **Custom visualizations**: Extend BeeView with new D3.js charts or map layers

### Resources

- **xPollinator Documentation**: [xPollinator docs](https://github.com/xlandscape/xPollinator/blob/main/docs/index.md)
- **xLandscape Framework**: [xLandscape GitHub](https://github.com/xlandscape/xLandscape)
- **BeeView GitHub**: [BeeView repository](https://github.com/xlandscape/BeeView)
- **BeeView User Guide**: [BeeView Documentation](https://github.com/xlandscape/BeeView/tree/develop/docs/index.md)
- **Pesticide Fate Reference**: [Pesticide Fate in xPollinator and BEEHAVEecotox](pesticide-fate.md)

---

## Appendix: Parameter Reference

### .xrun File Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `SimID` | string | Yes | Unique simulation identifier (alphanumeric, no spaces) |
| `Project` | string | Yes | Path to scenario folder (relative to repository root) |
| `NumberMC` | integer | Yes | Number of Monte Carlo runs (single-run tutorial uses `1`) |
| `SimulationStart` | date (YYYY-MM-DD) | Yes | Simulation start date |
| `SimulationEnd` | date (YYYY-MM-DD) | Yes | Simulation end date |
| `Manifest` | string | No | Path to manifest JSON for batch runs |
| `NumberBeeHaveTimesteps` | integer | Yes | Simulation duration in days (typically 365) |
| `BeeHaveMapCenterPointX` | float | Yes* | Hive X coordinate (UTM or project CRS) |
| `BeeHaveMapCenterPointY` | float | Yes* | Hive Y coordinate (UTM or project CRS) |
| `NumberBeeHaveReplicates` | integer | Yes | Number of BEEHAVE replicates |
| `BeeHaveRandomSeed` | integer | Yes | Base random seed |
| `BeeHaveWeather` | string | Yes | Weather profile used by BEEHAVE |
| `MinNumberApplications` | integer | Yes | Minimum pesticide applications |
| `MaxNumberApplications` | integer | Yes | Maximum pesticide applications |
| `RunLabel` | string | Yes | Run label used for tracking outputs |
| `HiveGroupId` | string or integer | Yes | Hive group identifier |

*Only required if `Manifest` is not provided (single-run mode)

### Manifest File Structure

| Key | Type | Description |
|-----|------|-------------|
| `description` | string | Human-readable description |
| `scenario` | string | Path to scenario folder |
| `replicates` | integer | Number of BEEHAVEecotox replicates per combination |
| `weather` | string | Weather data source |
| `hives` | array | List of hive locations with coordinates and IDs |
| `treatments` | array | Pesticide application scenarios |

---

**Document Version**: 1.0  
**xPollinator Version**: 0.1.x  
**Last Updated**: May 2026
