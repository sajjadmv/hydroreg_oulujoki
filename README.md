# Short-term Hydropower Regulation: Hydrology → Rolling-Horizon Cascade Scheduling (Oulujoki, Finland)

**Repository type:** Research code + data-processing workflow  
**Keywords:** hydropower regulation, rolling-horizon dispatch, hydrology–operations coupling, hydropeaking metrics, cascade travel-time, ramp-rate constraints

---

## Name

**Project name:** *HydroReg-Oulujoki* (short-term hydropower regulation realism tests in the Oulujoki cascade)

> Replace the project name above with your final repository name if different.

---

## Contact information

**Author / Maintainer:** Sajjad M. Vatanchi*  
**Email:** *Sajjadmv72@gmail.com*  
**ORCID:** * 0000-0001-5170-4620*

---

## Organizational affiliation

**Affiliation:** *Universty of Oulu/WE3*  
**Country:** Finland

---

## Links to professional websites

- Personal/Group website: *https://www.oulu.fi/en/researchers/sajjad-mohammadzadeh-vatanchi*  
- Google Scholar: *https://scholar.google.com/citations?user=zNNwXkUAAAAJ&hl=en*  
- GitHub: *https://github.com/sajjadmv*  
- LinkedIn: *https://www.linkedin.com/in/sajjad-m-vatanchi-26889a130/*

---

## Project overview

### Problem statement
Short-term hydropower regulation is increasingly analyzed with high-temporal-resolution operation models, yet many power-system studies still **overestimate hydropower flexibility** because they simplify reservoir dynamics, cascade routing, and operational limits. This creates schedules that can be economically appealing but **hydrologically infeasible** and environmentally unrealistic, especially when market time resolution tightens and penalties for imbalance increase.

### Challenge statement (why this has not been solved before)
The main barrier is **model coupling at mismatched time scales and physics**:
- Hydrology is often modeled daily (or coarsely) while dispatch is hourly/sub-hourly.
- Capturing realistic cascade behavior requires continuity, routing/travel time, turbine/spill feasibility, and ramping limits.
- Tight (two-way/iterative) hydrology–operations coupling improves realism but is computationally expensive and hard to reproduce across studies.

### Solution statement (what this research resolves)
This repository implements a **reproducible, transparent one-way coupling pipeline**:
- **HBV-light** produces *daily* inflows (tributaries) using calibrated, transferable parameters.
- A **lake inflow reconstruction** for Lake Oulujärvi uses mass balance + Kalman filtering/RTS smoothing to suppress non-physical residual noise.
- **IRENA FlexTool** performs *hourly* rolling-horizon scheduling while enforcing **water–energy feasibility internally** (continuity, cascade constraints) and testing **structural realism** via routing delays and ramp-rate constraints.

### Objective (how this will be accomplished)
1. Build analysis-ready spatial and time-series inputs for the Oulujoki basin and cascade.
2. Generate daily inflow forcings (tributaries + reconstructed lake inflow), plus uncertainty ensembles.
3. Run a suite of rolling-horizon dispatch scenarios in FlexTool:
   - foresight horizons / re-optimization frequency,
   - routing delay representations,
   - ramp-rate constraint families,
   - hydrological uncertainty propagation.
4. Validate realism against observed *daily* plant outflows and quantify sub-daily regulation intensity (hydropeaking metrics).

---

## Literature review (workflow-foundational / methods papers)

This project’s analytical workflow is primarily grounded in:
- **HBV-light** conceptual rainfall–runoff modeling and teaching-oriented reproducible workflows (Seibert & Vis, 2012).  
- **FlexTool** dispatch formulation and scenario structure (Taibi et al., 2018) — used as the operational “scaffolding” for adding hydrological inflows, cascade routing, ramping, and rolling-horizon experiments.  
- Hydropower representation and coupling strategies in power-system studies (Ibanez et al., 2014; Koh et al., 2022; Baratgin et al., 2025).  
- Hydropower flexibility valuation and operational envelope derivation approaches (Roni et al., 2023) and typical conversion efficiency assumptions (Quaranta et al., 2022).

> Full references are listed in the manuscript

---

## Research questions (testable hypotheses)

Each question maps to explicit scenario levers and metrics.

### RQ1 — Rolling-horizon operational foresight
**Question:** How do look-ahead horizon and roll-forward step affect (i) daily outflow realism and (ii) sub-daily regulation intensity across the Oulujoki cascade?  
**Hypothesis (H1):** Longer foresight and/or more frequent re-optimization reduces unrealistic high-frequency regulation while preserving daily water volumes, improving agreement with observed daily outflows.

### RQ2 — Cascade travel time / routing delay
**Question:** Does adding physically motivated travel time (constant or distributed delay) improve realism and suppress spurious regulation signatures?  
**Hypothesis (H2):** Routing delays improve downstream daily fit and reduce intra-day volatility relative to no-delay; distributed delay outperforms constant delay where dispersion matters.

### RQ3 — Ramp-rate constraint tightness
**Question:** What ramp-rate constraint tightness is necessary and sufficient to prevent unrealistically abrupt regulation without destroying market responsiveness?  
**Hypothesis (H3):** Tightening ramps reduces hydropeaking; beyond a threshold it significantly reduces price-response correlation and revenue, yielding a Pareto frontier.

### RQ4 — Hydrological uncertainty propagation
**Question:** How do tributary inflow uncertainty and lake inflow uncertainty propagate to regulation outcomes and revenue?  
**Hypothesis (H4):** Joint uncertainty produces non-additive impacts (storage/spill/constraint activation) relative to individual uncertainties, especially mid-to-downstream.

---

## Data sources

**Study area:** Oulujoki basin and hydropower cascade, Finland  
**Period (typical):** 2014–2024 (with some datasets extending to 2025 for forcing/metadata)
Direct-download links and API requests used for reproducible raw-data access are documented in `inputs/datalinks.txt` and `notebooks/SE1_data_access.ipynb`.

### Published / institutional data sources

| Dataset | Provider | What it is used for | Temporal / spatial resolution | Access |
|---|---|---|---|---|
| Digital Elevation Model (DEM) | National Land Survey of Finland (NLS) | catchment delineation, terrain analysis | raster | public download |
| Meteorological forcing (P, T) | Finnish Meteorological Institute (FMI) Open Data | HBV-light forcing | daily; station/gridded products | public API/download |
| Catchments + land cover | Finnish Environment Institute (Syke) spatial datasets | sub-catchments, HRU fractions (urban/agri/forest) | vector/raster | public download |
| Discharge / water level / evaporation | Hertta (via Syke open interfaces) | HBV calibration/verification; plant daily outflow evaluation; lake mass balance | daily | public interface |
| Electricity prices | ENTSO-E Transparency Platform | hourly day-ahead price driver in FlexTool | hourly | registration/API |
| Plant characteristics (head, MW) | Fortum (Oulujoki plants), Oulun Energia (Merikoski) | head + installed output for generation mapping | static | public webpages/docs |

**Operational envelope assumptions (when detailed plant ops data are unavailable):**
- Minimum total outflow = 5th percentile (P5) of observed daily total outflow (2014–2025).
- Maximum total outflow = max observed daily total outflow (2014–2025).
- Turbine–generator efficiency assumed constant at 0.90.


---

## Methods

### End-to-end workflow (conceptual)
1. **Spatial preprocessing**
   - Delineate Oulujoki basin + sub-catchments
   - Compute HRU fractions (urban/agriculture/forest)
   - Map sub-catchments to FlexTool inflow nodes

2. **Daily hydrology: HBV-light**
   - Drive HBV-light with daily P, T, and PET (PET interpolated from monthly means)
   - Calibrate at a near-natural reference gauge using GAP optimization and KGE
   - Transfer calibrated parameters to ungauged sub-catchments (regionalization)
   - Output: daily inflows at FlexTool tributary nodes (2014–2024)

3. **Daily lake inflow reconstruction: Oulujärvi**
   - Lake mass-balance: stage → storage/area + regulated flows + lake P/E
   - Use a **state-space model** with Kalman filter + Rauch–Tung–Striebel smoothing
   - Monte Carlo perturbations produce an inflow **ensemble** for uncertainty experiments

4. **Hourly operations: FlexTool rolling-horizon dispatch**
   - Disaggregate daily inflows to hourly by piecewise-constant repetition (volume-preserving)
   - Align hourly day-ahead prices
   - Run rolling-horizon optimization; pass end-of-window storages forward for continuity

5. **Structural realism mechanisms**
   - Routing delay: RD0 (none), RD1 (constant integer lag), RD2 (distributed delay map)
   - Ramping: RC0–RC5 (scaled to discharge capacity; symmetric/asymmetric families)

6. **Evaluation**
   - Daily aggregation of simulated hourly releases → daily outflows
   - Daily skill: KGE / RMSE / Bias vs observed daily outflows
   - Sub-daily: hydropeaking metrics (ramp distributions, peak–trough, exceedance frequencies)
   - Market response: price-response correlation; objective value/revenue proxy

---

## Repository contents

| Path | Contents |
|---|---|
| `notebooks/` | exploratory + figure-generation notebooks (e.g., SE1–SE4) |
| `src/` | reusable Python modules (data ingestion, processing, metrics) |
| `configs/` | YAML/JSON configs for scenarios (rolling horizon, delays, ramps, uncertainty) |
| `inputs/` | *minimal* helper inputs (e.g., simplified polygons, variable maps); no large datasets |
| `data_raw/` | optional raw-data manifests, checksums, and metadata only; large raw datasets are stored outside the repository (e.g. `../HydroReg-Oulujoki_rawdata/`) |
| `processed_data/` | analysis-ready datasets (catchments, HRUs, daily/hours time series) |
| `model_data/` | saved model outputs, FlexTool runs, scenario logs |
| `figures/` | manuscript figures + summary tables |
| `docs/` | data source documentation, assumptions, references, changelog |
| `run_reproducibility.py` | end-to-end reproducibility wrapper (recommended) |
| `environment.yml` | pinned conda environment (recommended) |
| `requirements.txt` | pip fallback environment |
| `Dockerfile` | containerized reproduction |
| `CITATION.cff` | citation metadata (Zenodo-compatible) |
| `LICENSE` | license text |

---

## How to reproduce results

### Computational requirements
- OS: Linux, macOS, or Windows (WSL recommended for Windows)
- CPU: multi-core recommended (rolling-horizon scenario sweeps can be compute-heavy)
- RAM: ≥ 16 GB recommended
- Optional: Docker for fully reproducible execution

### 1) Create environment
**Conda (recommended)**
```bash
conda env create -f environment.yml
conda activate hydroreg
```

**Pip (fallback)**
```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### 2) Configure data access
1. Populate `inputs/datalinks.txt` with direct download/API endpoints (when stable).
2. For ENTSO-E, set your API token (example):
```bash
export ENTSOE_API_TOKEN="YOUR_TOKEN"
```
3. If any Syke/Hertta endpoints require credentials, store tokens securely (never commit).

#### Reproducible raw-data access

Raw data access is documented in `notebooks/SE1_data_access.ipynb`.  
The notebook downloads public raw files from stable URLs, stores them outside the Git repository in `../HydroReg-Oulujoki_rawdata/`, and preserves one successful FMI Open Data API request as a raw XML response.  
Direct-download files and API-based resources are handled separately because `inputs/datalinks.txt` contains mixed resource types, including downloadable files, service endpoints, and reference webpages.  
The FMI example in the notebook retrieves hourly weather observations for Oulu and saves the exact successful query for reproducibility.  
Authentication-free public resources are included in this notebook, while token-based sources such as ENTSO-E are documented separately.

### 3) Run the pipeline
```bash
python run_reproducibility.py --all
```

Typical stages (can be run separately):
```bash
python run_reproducibility.py --stage spatial
python run_reproducibility.py --stage hydrology
python run_reproducibility.py --stage lake_inflow
python run_reproducibility.py --stage dispatch
python run_reproducibility.py --stage evaluation
```

### 4) Expected outputs
- `processed_data/`  
  - catchment/HRU layers  
  - daily inflow forcing time series  
  - hourly forcing bundles for FlexTool
- `model_data/`  
  - scenario outputs: hourly releases, storage, spill, generation
- `figures/`  
  - daily skill plots (KGE/RMSE/Bias)  
  - hydropeaking metrics summaries  
  - trade-off curves (ramp tightness vs revenue/response)

---

## Summary of results (to be updated after running)

This repository is designed to produce:
- Quantified effects of **foresight** (rolling-horizon settings) on daily realism and sub-daily regulation intensity.
- Quantified benefits of including **routing delay** (constant/distributed) in cascade linking.
- A **Pareto trade-off** between ramp tightness and market responsiveness (revenue proxy / price correlation).
- Sensitivity of regulation and revenue outcomes to **tributary + lake inflow uncertainty**, including potential non-additive effects.


## License

Choose a license that matches your institution and data constraints:
- **Code:** MIT or BSD-3-Clause (permissive)
- **Docs/Figures:** CC BY 4.0 (attribution)

---

## Citation

If you use this repository, cite:
1) the associated manuscript (once published), and  
2) the archived repository DOI (Zenodo recommended).

**Repository DOI:** `DOI_PENDING`  
**Manuscript citation:** `TO_BE_ADDED`

Example BibTeX (edit):
```bibtex
@software{hydroreg_oulujoki_2026,
  author       = {Sajjad, M. Vatanchi},
  title        = {HydroReg-Oulujoki: Hydrology to Rolling-Horizon Cascade Scheduling},
  year         = {2026},
  version      = {0.1.0},
  doi          = {DOI_PENDING},
}
```

---

## Contribution guidelines

Contributions that improve the quality, clarity, and reproducibility of this project are welcome.

- Open an issue before making major or result-affecting changes.
- Keep pull requests focused; describe what changed and why.
- Follow existing code style; update documentation and tests.
- Do not modify code/data used to reproduce published results without discussion.
- Ensure workflows remain reproducible (environment, dependencies, random seeds).
- Do not commit large or restricted datasets; respect data licenses.

By contributing, you agree that your work will be released under the project’s license.

---

## Acknowledgements

- Data providers: NLS, FMI, Syke/Hertta, ENTSO-E, Fortum, Oulun Energia.
- Tools: HBV-light, IRENA FlexTool.


 
