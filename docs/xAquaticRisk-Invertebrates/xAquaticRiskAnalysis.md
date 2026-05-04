# xAquaticRiskAnalysis — Standalone Analysis Tool

**xAquaticRiskAnalysis (xARA)** is a standalone web application for exploring and analysing the outputs of [xAquaticRisk](https://github.com/xlandscape/xAquaticRisk) simulation runs.

- GitHub: [xlandscape/xAquaticRiskAnalysis](https://github.com/xlandscape/xAquaticRiskAnalysis)
- Runs on **port 8091** (xAquaticRisk's own control panel uses port 8090 — both can run simultaneously)

---

## What it does

After xAquaticRisk has completed one or more Monte Carlo simulation runs, xARA provides:

- **Analysis workflows** — launch `run_basic_analysis.py` for a selected MC run to compute PEC metrics and GUTS risk indicators; monitor progress and download output tables and figures directly from the browser
- **Map explorer** — visualise reach geometries and spray-drift timeseries for any experiment and MC run on an interactive map
- **Run browser** — list all experiments and MC folders in the shared `run/` folder without opening xAquaticRisk itself

## Relationship to xAquaticRisk

xARA is intentionally separate from the xAquaticRisk prep-and-run control panel. The two applications share only a single folder on disk — the `run/` folder that xAquaticRisk writes simulation output into — and communicate that location through the `XAQ_RUN_DIR` environment variable.

```
xAquaticRisk (port 8090)          xAquaticRiskAnalysis (port 8091)
  ├── controlpanel/                  ├── server.py
  ├── run/  ◄────────────────────────────── XAQ_RUN_DIR
  └── scenario/                      ├── analysis/
                                     └── start.bat
```

This separation means:

- xARA can be deployed on a different machine from the one running simulations, as long as the `run/` folder is accessible (e.g., via a network share)
- Analysts can use xARA without access to the xAquaticRisk parameterisation or model binaries
- Updates to the analysis tooling do not require touching the simulation server

---

## Getting Started

### Prerequisites

- Windows 64-bit (required for the bundled analysis Python runtime)
- No system Python required for normal use — a bundled `analysis/python/` runtime is included
- A completed xAquaticRisk simulation in a `run/` folder you have read access to

### Step 1 — Get xAquaticRiskAnalysis

**Option A — Clone from GitHub (developers)**

```powershell
git clone https://github.com/xlandscape/xAquaticRiskAnalysis.git
cd xAquaticRiskAnalysis
```

**Option B — Download ZIP (end users)**

1. Open [https://github.com/xlandscape/xAquaticRiskAnalysis](https://github.com/xlandscape/xAquaticRiskAnalysis)
2. Click the green **Code** button → **Download ZIP**
3. Extract to a folder of your choice

The extracted folder is xcopy-ready — no installer needed.

### Step 2 — (Optional) Set `XAQ_RUN_DIR`

Tell xARA where xAquaticRisk writes its simulation output by setting the `XAQ_RUN_DIR` environment variable:

```bat
set XAQ_RUN_DIR=C:\LocalWork\xAquaticRisk\run
```

!!! tip "Making the variable permanent"
    Windows → Start → "Edit environment variables for your account" → New → Name: `XAQ_RUN_DIR`, Value: your path.

If `XAQ_RUN_DIR` is not set, `start.bat` defaults to `C:\`. Setting it explicitly is recommended so the UI opens directly on the correct run folder.

### Step 3 — Launch

From the `xAquaticRiskAnalysis` folder:

```bat
start.bat
```

`start.bat` automatically detects the bundled `analysis\python\python.exe` runtime if present; otherwise it falls back to whichever `python` is on `PATH`.

Open your browser and navigate to:

```
http://localhost:8091
```

**Custom port:**

```bat
set XAQ_PORT=9000
start.bat
```

**Direct Python launch:**

```powershell
python server.py --run-dir C:\LocalWork\xAquaticRisk\run --port 9000
```

### Step 4 — Select a run and start analysis

1. Open the **Analysis** tab in the browser UI
2. Choose an experiment from the **Experiment** dropdown — populated from the `run/` folder set via `XAQ_RUN_DIR`
3. Select a MC run within that experiment
4. Click **Start Analysis** — xARA launches `analysis/run_basic_analysis.py` as a subprocess and streams progress in the log panel
5. When the job completes, output figures and Excel tables are available for download directly from the browser

---

## Running alongside xAquaticRisk

Both servers can run simultaneously on the same machine. They use separate ports (8090 for the xAquaticRisk control panel, 8091 for xARA by default) and communicate only through the shared `run/` folder.

```bat
REM Terminal 1 — xAquaticRisk prep server (port 8090)
cd C:\LocalWork\xAquaticRisk\controlpanel
python server.py

REM Terminal 2 — xAquaticRiskAnalysis server (port 8091)
cd C:\LocalWork\xAquaticRiskAnalysis
start.bat
```

---

## Configuration Reference

| Variable / Argument | Default | Description |
|---|---|---|
| `XAQ_RUN_DIR` / `--run-dir` | `C:\` (start.bat); none (direct) | Path to the xAquaticRisk `run/` folder |
| `XAQ_PORT` / `--port` | `8091` | HTTP server port |
| `XAQ_ANALYSIS_MODE` | `local` | Analysis execution mode (`local` only at present) |

---

## Troubleshooting

**Server won't start**

- Verify `XAQ_RUN_DIR` is set and points to an existing folder
- Check port 8091 is not in use: `netstat -ano | findstr :8091`
- Try a custom port: `start.bat 9000`

**Python / package errors**

- Delete `analysis\python\` and re-run `setup_analysis_python.bat`
- Or use system Python: `python server.py --run-dir C:\...`

**Analysis jobs fail**

- Check `analysis_output/` folder for error logs
- Ensure the run folder contains completed xAquaticRisk outputs (e.g., `hydro.h5`)

---

## Resources

- [xAquaticRiskAnalysis GitHub Repository](https://github.com/xlandscape/xAquaticRiskAnalysis)
- [xAquaticRisk GitHub Repository](https://github.com/xlandscape/xAquaticRisk)
