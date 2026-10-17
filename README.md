# Validating explainable surrogates of SUMO

Code, experimental designs, result tables and figures for the article
**“Validating Explainable Surrogates of Activity-Based Microscopic Traffic Simulation:
Independent Tests, Seed Noise, and Simulator-Checked Interventions in SUMO”**
(Mahbub Hassan and Md Kamrul Islam).

Surrogate models and explainable machine learning are widely used to explore, calibrate and optimize
traffic simulators. This repository tests how far such surrogates can be trusted. It runs a pre-specified
experiment of **10,216 Eclipse SUMO simulations** of an activity-based demand model with nine demand and transit inputs.
Every claim a surrogate makes (accuracy, prediction intervals, sensitivity rankings, interventions and
calibrated parameters) is checked against fresh simulations.

## What the study does

| Question | Experiment |
|---|---|
| How accurate are surrogates on new settings? | 512 Sobol development settings; 200 calibration and 200 test settings drawn independently; models frozen and hashed before testing |
| How much of the output is seed noise? | 24 settings × 10 seeds and 200 test settings × 5 seeds; noise share, R² ceiling, RMSE floor |
| Do prediction intervals cover new runs? | Four interval methods (CQR, split conformal, Gaussian process, GP-normalized conformal); train → calibrate → untouched test |
| Do surrogate explanations describe the simulator? | Direct-SUMO Sobol indices with seed-noise correction (5,632 runs), compared with surrogate Sobol, SHAP and ALE over 50 refits |
| What does bus headway actually do? | 8 × 8 × 10-seed factorial of headway and departure variation at two backgrounds, outcomes split by vehicle type |
| Do surrogate-proposed interventions work? | Proposals re-simulated with 10 matched seeds against the baseline and a random same-distance control |
| Can parameters be recovered by calibration? | 10 excluded targets, two objectives, fresh-seed validation, equifinality scan |
| Do surrogates transfer to another network? | Frozen transfer to an 8 × 8 grid and four adaptation methods |

## Key results (10,216 SUMO runs, 0 failures)

| Question | Answer from fresh simulations |
|---|---|
| Do surrogates generalize? | Frozen ensemble single-run test R² = 0.937 (all-vehicle delay), 0.941 (general-traffic delay), 0.991 (VKT); Ridge 0.478 / 0.417 / 0.945 |
| How close to the simulator's limit? | Seed-noise SD 1.6 s (delay); ensemble test RMSE 2.3 s; R² ceiling 0.969 |
| Do 90 % intervals cover? | 0.80–0.90 overall for delay; only 0.70–0.75 in the most congested tertile |
| What drives delay in SUMO itself? | Direct-SUMO total-order Sobol: departure variation 0.74, bus headway 0.21 (all vehicles); headway 0.014 for general traffic |
| What does bus headway do? | Longer headways remove buses (−84 vehicles), lowering the all-vehicle mean by 17 s; general-traffic delay unchanged (−0.3 s [−1.3, 0.7]) |
| Do surrogate interventions work? | 20/20 reduce delay in SUMO (mean 15.8 % vs 17.0 % predicted) but 10/20 reach the 15 % target; random changes of equal size: 6.8 %, 4/20 |
| Can calibration recover parameters? | Two aggregate outputs: error 0.29 of range (random guess 0.33); eight outputs incl. hourly arrivals: 0.20 |
| Do models transfer to an 8 × 8 grid? | Frozen: R² < −12, coverage 0 %; residual adaptation with 128 runs: R² 0.93 |

<p align="center"><img src="results/full/figures/png/Fig08_headway_mechanism.png" width="85%"></p>

## Repository layout

```
src/sumo_study/        analysis package
  config.py            parameter domain, outputs, experiment sizes, seed registry
  simulator.py         activitygen -> duarouter -> sumo, parsing, run cache
  design.py            Sobol / uniform designs, disjointness checks
  surrogates.py        Ridge, GP, RF, LightGBM, CatBoost, XGBoost, ensemble
  validation.py        independent test, noise components, interval methods
  policy.py            intervention search and matched-seed evaluation
  calibration.py       inverse calibration and equifinality scan
  sensitivity.py       direct-SUMO / surrogate Sobol, SHAP, ALE, DML
  adaptation.py        8×8 transfer adaptation
  workflow.py          ordered, resumable stages (Study class, CLI)
  figures.py           article figures
  export.py            key numbers, LaTeX tables, SHA-256 manifest
notebooks/             SUMO_Surrogate_Study.ipynb – the documented, ordered study
docs/                  ANALYSIS_PROTOCOL.md (fixed before the full run)
results/full/          tables/, figures/{png,pdf,svg}/, designs/, latex_tables/, key_numbers.json
scripts/               run_study.py, build_notebook.py
```

## Reproduce

Requirements: Eclipse SUMO 1.28.0 (`sumo`, `netgenerate`, `activitygen`, `duarouter` on `PATH`) and Python ≥ 3.11.

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# quick end-to-end check (~3 min, small sizes, not for interpretation)
python scripts/run_study.py smoke 8
# full study (10,216 SUMO runs; about 90 min wall time on 8 Apple-M5 cores)
python scripts/run_study.py full 8
```

Or open `notebooks/SUMO_Surrogate_Study.ipynb`; it also runs on Google Colab (`pip install eclipse-sumo==1.28.0`).
Stages are resumable. SUMO runs are cached under `cache/` by a hash of the simulator fingerprint, the
parameters and the seed. Changing the simulator, physics or experiment sizes requires a new output folder.

## Scope

All results are conditional on a synthetic signalized grid (6 × 6, with an 8 × 8 transfer test), SUMO's
activitygen demand model (vehicle trips only; bus outcomes refer to bus vehicles, not passengers) and the stated
parameter ranges. Effects estimated in the simulator are not real-world causal policy effects.

## License and citation

Code: MIT License (see `LICENSE`). Please cite the article and this repository (`CITATION.cff`, release v1.0.0).
