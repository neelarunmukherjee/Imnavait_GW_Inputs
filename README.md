# imnavait-gw-inputs

**Data and code for:**

> Mukherjee, N., Chen, J., Neilson, B.T., Kling, G.W., and Cardenas, M.B. (2024). Water and carbon fluxes from a supra-permafrost aquifer to a stream across hydrologic states. *Journal of Hydrology*, 645, 132285. https://doi.org/10.1016/j.jhydrol.2024.132285

[![DOI](https://img.shields.io/badge/DOI-10.1016%2Fj.jhydrol.2024.132285-blue)](https://doi.org/10.1016/j.jhydrol.2024.132285)

A statistical ensemble groundwater flow model (8,466,850 realizations) using vertically-integrated Darcy's Law to quantify groundwater and dissolved organic carbon (DOC) fluxes from a supra-permafrost aquifer into Imnavait Creek, North Slope of Alaska.

---

## Repository Structure

```
.
├── codes/
│   ├── simulation.m                 # MATLAB: full ensemble model (see below)
│   ├── main_weir_results.ipynb      # Python: figures and analysis for main weir
│   └── upstream_weir_results.ipynb  # Python: figures and analysis for upstream weir
│
├── input_data/
│   ├── thickness_riparian_master.xlsx            # Soil stratigraphy: zac, dtC, dti, dtw
│   ├── ksat.xlsx                                 # Hydraulic conductivity: Kac, Kct, Kmn
│   ├── Mukherjee_etal_Supplementary_Chemistry.xlsx  # Groundwater DOC concentrations
│   ├── C_C_Hydrozoid.xlsx                        # Stream DOC at main weir
│   ├── KaneWeir_Discharge.xlsx                   # Main weir discharge hydrograph
│   ├── IC Flume Discharge 2016.xlsx              # Main weir discharge, 2016
│   └── IC-FlumeUp-15_2013-2014-2015_Q_C_S_SpC_T (1).csv  # Upstream weir discharge
│
└── MukherjeeEtAl_JOH_final.pdf     # Published manuscript
```

---

## simulation.m — What it does

The script runs the full ensemble model in MATLAB. Steps in order:

**1. Load and fit soil property distributions** from `thickness_riparian_master.xlsx` and `ksat.xlsx`:

| Parameter | Distribution | μ (or a) | σ (or b) |
|-----------|-------------|----------|----------|
| dti — depth to ice (m) | Lognormal | −0.67 | 0.21 |
| dtw — depth to water (m) | Lognormal | −1.70 | 1.00 |
| zac — acrotelm thickness (m) | Lognormal | −2.15 | 0.45 |
| dtC — depth to catotelm–mineral boundary (m) | Lognormal | −1.20 | 0.26 |
| Kac — acrotelm K (m/s) | Weibull | 0.00275 | 3.2 |
| Kct — catotelm K (m/s) | Lognormal | −10.15 | 1.25 |
| Kmn — mineral soil K (m/s) | Lognormal | −14.55 | 0.89 |

**2. Generate 10,000,000 random samples** from each fitted PDF into a matrix `D` (columns: zac, dtC, dti, dtw, Kac, Kct, Kmn).

**3. Remove physically impossible realizations** (dti − dtw < 0, i.e., no saturated layer). Leaves **8,466,850** valid ensemble members.

**4. Compute transmissivity T = Keff × b** for each realization across six ice/water table position scenarios (Eqs. 3–13 in paper):

| Case | Ice table in | Water table in |
|------|-------------|----------------|
| A1 | Mineral soil | Mineral soil |
| A2 | Mineral soil | Catotelm |
| A3 | Mineral soil | Acrotelm |
| B1 | Catotelm | Catotelm |
| B2 | Catotelm | Acrotelm |
| C1 | Acrotelm | Acrotelm |

**5. Compute groundwater discharge** at the **main weir (Kane weir)** and **upstream weir (Pool 2)**:

```
Q_gw [L/s] = 1000 × (ARC-integrated head gradient buffer) × T × (cell_size × 2)
```

- Cell size: 0.2052 m (DEM pixel width)
- Head gradient buffer (Kane weir, 2-cell): 261.447 (from ArcGIS Spatial Analyst)
- Head gradient buffer (upstream weir, 2-cell): 573.69

**6. Compute DOC flux** at the main weir:

- Fit a lognormal PDF to 294 groundwater DOC concentrations (`C_C_Hydrozoid.xlsx`)
- Draw 8,466,850 random DOC values
- `DOC_flux [mol/s] = Q_gw × DOC_MC`

**Outputs** (commented-out `writematrix` lines — uncomment to save):
- `Q_gw_Kane_model_revisionB.csv` — ensemble groundwater discharge at main weir [L/s]
- `Q_gw_pool2_model_revisionB.csv` — ensemble groundwater discharge at upstream weir [L/s]
- `DOC_flux_Kane_Model.csv` — ensemble DOC flux at main weir [mol/s]

---

## Running the Code

### MATLAB — ensemble simulation
```matlab
% From the codes/ directory:
simulation.m
% To save outputs, uncomment the writematrix lines near the end of the script
```

**Requires:** MATLAB Statistics Toolbox (`lognfit`, `wblrnd`, `lognrnd`)

### Python — figures
```bash
cd codes
jupyter notebook main_weir_results.ipynb
jupyter notebook upstream_weir_results.ipynb
```

**Python dependencies:** `numpy`, `scipy`, `pandas`, `matplotlib` (Python ≥ 3.9)

---

## Input Data Summary

| File | Contents | N obs. |
|------|----------|--------|
| `thickness_riparian_master.xlsx` | dtw, dti, zac, dtC (riparian zone) | 239 dti, 181 dtw, 45 zac, 40 dtC |
| `ksat.xlsx` | Kac (Weibull), Kct, Kmn (lognormal) | from O'Connor et al. (2020) |
| `Mukherjee_etal_Supplementary_Chemistry.xlsx` | Groundwater DOC | 294 values |
| `C_C_Hydrozoid.xlsx` | Stream DOC, main weir, 2002–2009 | — |
| `KaneWeir_Discharge.xlsx` | Main weir discharge (2006, 2014, 2016, 2017) | — |
| `IC-FlumeUp-15...csv` | Upstream weir discharge (2013–2016) | — |

---

## Study Site

**Imnavait Creek**, North Slope of Alaska
- Main weir (Kane weir): 68°37′0.60″N, 149°19′4.30″W
- Upstream weir: 68°36′38.08″N, 149°18′57.93″W
- Watershed area: 2.2 km²; DEM resolution: 20 cm

---

## Citation

```bibtex
@article{mukherjee2024,
  author  = {Mukherjee, Neelarun and Chen, Jingyi and Neilson, Bethany T.
             and Kling, George W. and Cardenas, M. Bayani},
  title   = {Water and carbon fluxes from a supra-permafrost aquifer to a
             stream across hydrologic states},
  journal = {Journal of Hydrology},
  volume  = {645},
  pages   = {132285},
  year    = {2024},
  doi     = {10.1016/j.jhydrol.2024.132285}
}
```

---

## Funding

- U.S. DOE Office of Science, BER Environmental System Science Program (DE-SC0024091)
- NASA Terrestrial Hydrology Program (80NSSC18K0983) and FINESST (80NSSC20K1622)
- U.S. NSF Arctic grants: ARC-1204220, DEB-1026843, 1637459, 0639805, 2224743, PLR-1504006, OPP-1107593, 1936759

---

## Contact

**Neelarun Mukherjee** — [neelarun@utexas.edu](mailto:neelarun@utexas.edu)  
Department of Earth and Planetary Sciences, The University of Texas at Austin
