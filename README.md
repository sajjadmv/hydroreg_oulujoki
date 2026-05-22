# HydroReg-Oulujoki: Short-Term Hydropower Regulation in the Oulujoki Cascade

**Repository type:** Research code, data-processing workflow, statistical validation, and figure-generation package  
**Study area:** Oulujoki hydropower cascade, Finland  
**Main theme:** Hydrology-to-operations coupling for short-term hydropower regulation  
**Keywords:** hydropower regulation, Oulujoki, rolling-horizon dispatch, FlexTool, HBV-light, daily outflow validation, hydropeaking, cascade travel time, ramp-rate constraints, reproducible research

---

## 1. Project information

### Project name

**HydroReg-Oulujoki**  
Short-term hydropower regulation realism tests in the Oulujoki cascade.

### Author and maintainer

**Sajjad M. Vatanchi**  
**Affiliation:** University of Oulu / WE3  
**Country:** Finland  
**Email:** Sajjadmv72@gmail.com  
**ORCID:** 0000-0001-5170-4620

### Professional links

- University profile: https://www.oulu.fi/en/researchers/sajjad-mohammadzadeh-vatanchi
- Google Scholar: https://scholar.google.com/citations?user=zNNwXkUAAAAJ&hl=en
- GitHub: https://github.com/sajjadmv
- LinkedIn: https://www.linkedin.com/in/sajjad-m-vatanchi-26889a130/

---

## 2. Project overview

Short-term hydropower regulation is increasingly important in power systems with high shares of variable renewable energy. Hydropower can provide flexibility, but many energy-system studies simplify reservoir dynamics, cascade routing, travel time, and operational constraints. These simplifications can overestimate hydropower flexibility and produce operating schedules that are economically attractive but hydrologically unrealistic.

This project develops a reproducible workflow for evaluating short-term hydropower regulation in the Oulujoki cascade. The wider research framework links daily hydrological forcing to hourly hydropower operation:

1. **Hydrological forcing:** HBV-light produces daily tributary inflows.
2. **Lake inflow reconstruction:** Lake Oulujärvi inflow is reconstructed using a mass-balance approach supported by Kalman filtering and Rauch-Tung-Striebel smoothing.
3. **Operational scheduling:** In the broader research framework, IRENA FlexTool is the intended tool for hourly rolling-horizon hydropower scheduling. In the current repository, the FlexTool/Spine Toolbox simulation is not fully automated because the model is normally executed through a GUI-based workflow. Therefore, the repository includes the already simulated hourly outflow result as a fixed model-output input.
4. **Validation and interpretation:** The provided simulated hourly outflows are aggregated to daily scale and compared with observed daily plant outflows using plant-specific statistical models.

The current notebooks focus especially on reproducible data access, data processing, daily-scale statistical validation, and figure generation. The FlexTool simulation step itself is treated as an external/generated model-output input in this repository because the Spine Toolbox GUI workflow could not be reliably reproduced in a fully script-based way. The workflow is designed so that the same code base can later be extended to routing-delay, ramping, rolling-horizon, and uncertainty experiments.

---

## 3. Research problem

### Problem statement

Short-term hydropower operation must satisfy both market and physical constraints. However, simplified models often ignore the details that determine whether a schedule is feasible in a real river cascade, including storage continuity, turbine and spill limits, routing delay, travel time, and ramp-rate restrictions.

### Challenge statement

The main challenge is the mismatch between hydrological and operational time scales. Hydrology is often represented at daily resolution, while electricity-market operation is hourly or sub-hourly. A fully coupled two-way hydrology-power model can be more realistic, but it is also more complex, computationally demanding, and difficult to reproduce.

### Solution statement

This project uses a transparent one-way coupling concept: daily hydrological inflows are passed to an hourly operation model, while short-term water-energy feasibility is handled inside the operational layer. The validation notebooks then test whether simulated operational outflows remain consistent with observed plant-level daily outflows.

---

## 4. Research questions

The broader research framework is organized around four testable questions.

### RQ1: Rolling-horizon operational foresight

**Question:** How do look-ahead horizon and roll-forward step affect daily outflow realism and sub-daily regulation intensity across the Oulujoki cascade?  
**Hypothesis:** Longer foresight and/or more frequent re-optimization will reduce unrealistic high-frequency regulation while preserving daily water volumes.

### RQ2: Cascade travel time and routing delay

**Question:** Does adding physically motivated travel time improve downstream realism and reduce spurious regulation signatures?  
**Hypothesis:** Routing-delay scenarios should improve downstream daily fit and reduce intra-day volatility compared with a no-delay baseline.

### RQ3: Ramp-rate constraint tightness

**Question:** What ramp-rate constraint is sufficient to reduce unrealistic hydropeaking without removing market responsiveness?  
**Hypothesis:** Tighter ramp constraints will reduce hydropeaking metrics, but excessive tightening will reduce price responsiveness and revenue.

### RQ4: Hydrological uncertainty propagation

**Question:** How do tributary inflow uncertainty and lake inflow uncertainty propagate to regulation outcomes and revenue?  
**Hypothesis:** Joint tributary and lake inflow uncertainty will produce non-additive effects, especially for mid- and downstream plants.

---

## 5. Data sources

The project combines spatial data, hydro-meteorological forcing, hydrological observations, electricity prices, and hydropower plant characteristics.

| Dataset | Provider | Use in the project | Resolution / format |
|---|---|---|---|
| Digital elevation model | National Land Survey of Finland | Catchment and terrain analysis | Raster |
| Precipitation and temperature | Finnish Meteorological Institute Open Data | HBV-light forcing | Daily or hourly, depending on query |
| Catchments and land cover | Finnish Environment Institute / Syke | Sub-catchments and HRU fractions | Vector / raster |
| Discharge, water level, evaporation | Syke / Hertta | HBV calibration, daily plant outflow evaluation, lake mass balance | Daily observations |
| Electricity prices | ENTSO-E Transparency Platform | Hourly day-ahead price driver for FlexTool | Hourly |
| Plant head and installed power | Fortum and Oulun Energia public information | Hydropower conversion and plant characterization | Static metadata |

### Operational assumptions

Because detailed plant-level operational data are not fully available, the workflow uses transparent assumptions:

- Minimum total outflow is represented by the 5th percentile of observed daily total outflow.
- Maximum total outflow is represented by the maximum observed daily total outflow.
- Turbine-generator efficiency is assumed constant at 0.90.
- Simulated hourly outflow is aggregated to daily mean outflow before comparison with daily observations.

---

## 6. Notebook-based workflow

The repository is organized around four main notebook stages.

### SE1: Data access

**Notebook:** `notebooks/SE1_data_access.ipynb`

This notebook documents reproducible raw-data access. It handles public download links and API-based data sources separately. Public raw files are stored outside the Git repository, for example in:

```text
../HydroReg-Oulujoki_rawdata/
```

The notebook also preserves an example FMI Open Data API request and saves the raw XML response. Token-based sources, such as ENTSO-E, are documented but should not store private API keys in the repository.

### SE2: Data processing

**Notebook:** `notebooks/SE2_data_processing.ipynb`

This notebook prepares the raw data for modelling and validation. The processing stage harmonizes spatial and time-series inputs, prepares analysis-ready forcing data, and organizes observed and simulated outflow information for later statistical comparison.

Typical processing tasks include:

- organizing raw files and metadata,
- checking time stamps and temporal coverage,
- preparing daily and hourly time-series objects,
- harmonizing plant names across observed and simulated datasets,
- preparing outputs for statistical modelling and visualization.

### FlexTool / Spine Toolbox simulation status

The intended operational model is FlexTool, which is commonly run through the Spine Toolbox graphical user interface. In this project stage, the GUI-based FlexTool execution could not be made fully reproducible from the notebooks or a command-line wrapper. To keep the repository transparent and runnable for evaluation, the simulated hourly outflow file is included directly in the repository and is used as the starting point for SE3 and SE4.

This means that the current repository reproduces the **post-simulation workflow**: data harmonization, daily aggregation, statistical validation, model interpretation, metric calculation, and figure generation. It does not yet reproduce the full FlexTool optimization run from raw FlexTool input files. The included simulated result should therefore be interpreted as a fixed model-output dataset exported from the operational modelling stage.

### SE3: Model training, validation, and interpretation

**Notebook:** `notebooks/SE3_Model train, validate, and interpret code.ipynb`

This notebook does **not** train HBV-light or FlexTool. Instead, it trains a plant-specific statistical validation model to evaluate whether simulated daily outflows explain observed daily outflows.

The input files are:

```text
Estimated hourly.xlsx   # included simulated hourly result from the FlexTool/Spine Toolbox modelling stage
Observed daily.xlsx     # observed daily outflow data
```

The main steps are:

1. Read hourly simulated outflow and daily observed outflow.
2. Harmonize plant names.
3. Aggregate simulated hourly outflow to daily mean outflow.
4. Remove incomplete edge days from the hourly record.
5. Merge simulated and observed daily values by date and plant.
6. Fit one OLS model for each plant.
7. Use a chronological train-validation split: first 80% for training and final 20% for validation.
8. Export coefficients, predictions, confidence intervals, and model performance metrics.

The fitted model is:

```text
observed_daily_outflow ~ simulated_daily_outflow + sin_doy + cos_doy
```

where:

- `observed_daily_outflow` is the measured daily outflow,
- `simulated_daily_outflow` is the daily mean obtained from hourly simulated outflow,
- `sin_doy` and `cos_doy` are seasonal day-of-year terms.

The notebook calculates the following metrics for both train and validation periods:

- R²,
- adjusted R²,
- RMSE,
- MAE,
- Bias,
- NSE,
- KGE.

Typical outputs include:

```text
model_data/evaluation/daily_comparison_long_realdata.csv
model_data/evaluation/daily_comparison_wide_realdata.csv
model_data/evaluation/fit_metrics_realdata.csv
model_data/statistical_model/coefficients_realdata.csv
```

### SE4: Results visualization

**Notebook:** `notebooks/SE4_Results Visualization.ipynb`

This notebook converts the SE3 outputs into publication-style figures. It uses Python only; no GIS basemap, GUI software, Illustrator, PowerPoint, Photoshop, or post-hoc graphical editing is required.

The main inputs are:

```text
daily_comparison_long.csv
fit_metrics.csv
coefficients.csv
```

The main outputs are:

```text
images/figure1_validation_scatter_by_plant.png
images/figure2_skill_summary_by_plant.png
images/figure3_coefficients_by_plant.png
captions_SE4.md
docs/SE4_methods_results_discussion_update.md
```

---

## 7. Current statistical validation results

The current results evaluate daily-scale agreement between simulated operational outflows and observed daily outflows. They should be interpreted as a validation layer for the operational simulation, not as a complete hydropeaking analysis.

### Important clarification

The trained model in SE3 is a statistical OLS validation model. The project does **not** train the physical hydrological model or the FlexTool operation model in this stage. The OLS model is used to answer this question:

> How well can simulated daily outflow, supported by simple seasonal terms, explain observed daily plant outflow?

### Figure 1: Validation observed versus predicted daily outflow

![Figure 1: Validation observed versus predicted daily outflow by plant](images/figure1_validation_scatter_by_plant.png)

Figure 1 compares observed and predicted daily outflow for each plant during the validation period. The dashed line is the 1:1 reference line. Most points follow the 1:1 line reasonably well, showing that the statistical validation model captures the main daily outflow structure across the cascade. However, scatter increases at higher outflows, indicating that the model has more difficulty during high-flow or strongly regulated periods.

### Figure 2: Daily RMSE and KGE by plant and split

![Figure 2: Daily model skill summary across plants](images/figure2_skill_summary_by_plant.png)

Figure 2 compares training and validation skill for each plant. Training RMSE is lower than validation RMSE, and training KGE is higher than validation KGE for all plants. This pattern is expected because the OLS coefficients are estimated from the training period. The important result is that validation performance does not collapse, which suggests that the fitted relationship generalizes to later data.

Across plants, validation RMSE is approximately **0.393-0.413**, and validation KGE is approximately **0.588-0.605**. These values indicate moderate but consistent daily-scale predictive skill.

### Figure 3: OLS coefficient estimates with robust confidence intervals

![Figure 3: OLS coefficient estimates by plant](images/figure3_coefficients_by_plant.png)

Figure 3 shows the estimated OLS coefficients for each plant with 95% HC3 robust confidence intervals. The coefficient for `simulated_daily_outflow` is positive for every plant and is larger than the seasonal terms. This means that simulated daily outflow is the dominant predictor of observed daily outflow.

The seasonal terms, `sin_doy` and `cos_doy`, have smaller coefficients. They help correct remaining seasonal structure, but they do not dominate the model. This supports the interpretation that the operational simulation contains meaningful information about observed plant-level outflow dynamics.

---

## 8. Interpretation of results

The three figures support the following interpretation:

1. The simulated operational outflow is not random; it has a clear positive relationship with observed daily outflow.
2. The validation model captures the main daily-scale behaviour across all seven plants.
3. The train-validation gap shows some loss of generalization, so validation results should be emphasized more than training results.
4. Remaining errors are larger at higher flows, suggesting that extreme or strongly regulated periods require further model development.
5. The current figures evaluate daily agreement only. They do not yet quantify sub-daily hydropeaking behaviour.

Therefore, the present workflow is a useful validation foundation, but the next research step should extend it from daily-scale statistical evaluation to hourly operational realism testing.

---

## 9. How this code can support the next chapter

The current code can be adapted into a scenario-experiment engine for the next chapter. The main adaptation is to move from one simulated-output file to many controlled FlexTool scenarios.

Recommended extensions:

1. **Scenario metadata table**
   - Add columns for rolling-horizon length, roll-forward step, routing-delay setting, ramp-rate setting, inflow ensemble member, price year, and plant.

2. **Sub-daily hydropeaking metrics**
   - Add hourly ramp-up and ramp-down magnitudes.
   - Add daily peak-to-trough range.
   - Add exceedance frequencies for rapid flow changes.
   - Add event-based hydropeaking indicators.

3. **Routing-delay comparison**
   - Compare RD0, RD1, and RD2 scenarios plant by plant.
   - Evaluate both daily skill and hourly regulation indicators.

4. **Ramp-rate trade-off analysis**
   - Compare RC0-RC5 ramping settings.
   - Quantify trade-offs between hydropeaking reduction, price-response correlation, and revenue proxy.

5. **Uncertainty propagation**
   - Run deterministic and ensemble inflow forcings.
   - Compare HBV-only, lake-only, and joint uncertainty scenarios.
   - Evaluate uncertainty impacts on storage, spill, outflow, generation, and objective value.

6. **Generalized evaluation model**
   - Extend the current OLS diagnostic model to include scenario-level predictors, for example:

```text
metric ~ routing_delay + ramp_constraint + horizon + inflow_uncertainty + plant + season
```

This would allow the next chapter to identify which operational assumptions improve realism and which plants are most sensitive to routing, ramping, and inflow uncertainty.

---

## 10. Repository structure

A recommended structure for the final repository is:

```text
HydroReg-Oulujoki/
├── README.md
├── environment.yml
├── requirements.txt
├── Dockerfile
├── LICENSE
├── CITATION.cff
├── inputs/
│   └── datalinks.txt
├── notebooks/
│   ├── SE1_data_access.ipynb
│   ├── SE2_data_processing.ipynb
│   ├── SE3_Model train, validate, and interpret code.ipynb
│   └── SE4_Results Visualization.ipynb
├── data_raw/
│   └── README.md
├── processed_data/
│   └── README.md
├── model_data/
│   ├── simulated_results/        # included FlexTool/Spine Toolbox output used as SE3 input
│   ├── evaluation/
│   └── statistical_model/
├── images/
│   ├── figure1_validation_scatter_by_plant.png
│   ├── figure2_skill_summary_by_plant.png
│   └── figure3_coefficients_by_plant.png
├── docs/
│   ├── SE4_methods_results_discussion_update.md
│   └── captions_SE4.md
└── src/
    └── README.md
```

Large raw datasets should not be committed to GitHub. Store them outside the repository and document their locations, download links, checksums, or API requests.

---

## 11. Reproducibility instructions

### Current reproducibility scope

Because the FlexTool model is normally executed through the Spine Toolbox GUI, the full optimization run is not currently reproduced automatically from the notebooks. Instead, this repository includes the simulated hourly outflow result and reproduces all steps after that point: aggregation, merging with observed data, OLS validation, metric calculation, and figure generation.

For grading or reuse, the key reproducible path is therefore:

```text
included simulated hourly result + observed daily data
→ SE3 statistical validation
→ SE4 figures and result interpretation
```

### Option A: Run the notebooks manually

Open the notebooks in order and run each notebook from top to bottom:

```text
notebooks/SE1_data_access.ipynb
notebooks/SE2_data_processing.ipynb
notebooks/SE3_Model train, validate, and interpret code.ipynb
notebooks/SE4_Results Visualization.ipynb
```

### Option B: Create a Python environment

Using Conda:

```bash
conda env create -f environment.yml
conda activate hydroreg
```

Using pip:

```bash
python -m venv .venv
source .venv/bin/activate      # macOS/Linux
# .venv\Scripts\activate       # Windows
pip install -r requirements.txt
```

### Option C: Use a wrapper script, if included

If the repository includes `run_reproducibility.py`, the full workflow can be wrapped as:

```bash
python run_reproducibility.py --all
```

or by stage:

```bash
python run_reproducibility.py --stage data_access
python run_reproducibility.py --stage data_processing
python run_reproducibility.py --stage validation
python run_reproducibility.py --stage figures
```

---

## 12. Expected inputs and outputs

The current reproducible workflow requires the simulated hourly result file and the observed daily outflow file. The simulated file is included because the GUI-based FlexTool/Spine Toolbox run is not fully automated in this repository.

Required inputs for SE3:

```text
Estimated hourly.xlsx      # included simulated hourly outflow result
Observed daily.xlsx        # observed daily plant outflow data
```

Running the current notebook workflow should produce:

```text
model_data/evaluation/daily_comparison_long_realdata.csv
model_data/evaluation/daily_comparison_wide_realdata.csv
model_data/evaluation/fit_metrics_realdata.csv
model_data/statistical_model/coefficients_realdata.csv
images/figure1_validation_scatter_by_plant.png
images/figure2_skill_summary_by_plant.png
images/figure3_coefficients_by_plant.png
docs/SE4_methods_results_discussion_update.md
captions_SE4.md
```

---

## 13. Limitations

The current workflow has several important limitations:

- The FlexTool optimization run is not currently reproduced automatically because the operational model is run through the Spine Toolbox GUI framework.
- The repository therefore includes the simulated hourly outflow result and reproduces the post-simulation validation and visualization workflow.
- The SE3 statistical model evaluates daily-scale agreement, not sub-daily hydropeaking.
- The OLS model is diagnostic and interpretive; it is not a replacement for a physical hydropower operation model.
- Hourly simulated outflows are aggregated to daily mean values, so hourly ramping information is lost in the present validation plots.
- Some operational assumptions, including efficiency and discharge bounds, are simplified because detailed plant-level operational data are unavailable.
- Future work should evaluate routing delays, ramp-rate constraints, rolling-horizon foresight, and hydrological uncertainty in a structured scenario framework.

---

## 14. License

Select a license that matches institutional and data restrictions. Recommended options:

- **Code:** MIT or BSD-3-Clause
- **Documentation and figures:** CC BY 4.0

---

## 15. Citation

If you use this repository, please cite the associated manuscript and the archived repository DOI when available.

Example BibTeX:

```bibtex
@software{hydroreg_oulujoki_2026,
  author       = {Vatanchi, Sajjad M.},
  title        = {HydroReg-Oulujoki: Hydrology to Rolling-Horizon Cascade Scheduling},
  year         = {2026},
  version      = {0.1.0},
  doi          = {DOI_PENDING}
}
```

---

## 16. Acknowledgements

This project acknowledges the data providers and modelling tools that support the workflow:

- National Land Survey of Finland,
- Finnish Meteorological Institute,
- Finnish Environment Institute / Syke and Hertta,
- ENTSO-E Transparency Platform,
- Fortum,
- Oulun Energia,
- HBV-light,
- IRENA FlexTool.
