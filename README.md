# Explainable SUMO surrogates — a validation-first study

Code, simulation designs, result tables and figures for the manuscript
**“Validating explainable surrogates of activity-based microscopic traffic simulation:
Independent tests, seed noise, and simulator-checked interventions in SUMO”**
(M. Hassan, M. K. Islam).

The study replaces a 128-scenario, internally cross-validated surrogate analysis with a
pre-specified, validation-first experiment of 10,216 Eclipse SUMO runs. Every claim made by a surrogate
(accuracy, intervals, sensitivity rankings, interventions, calibrations) is checked against fresh SUMO simulations.

| Limitation of the earlier analysis | What this repository does |
|---|---|
| Internal cross-validation only | 512 Sobol development settings; 200 calibration and 200 test settings drawn independently; models frozen and hashed before testing |
| One simulation per setting | 24 settings × 10 seeds and 200 test settings × 5 seeds; noise share, R² ceiling, RMSE floor |
| Interval coverage computed on calibration data | Four interval methods; train → calibrate → untouched test; Wilson / cluster-bootstrap CIs |
| Recourse reductions only predicted | Proposals re-simulated with 10 matched seeds, plus a random same-distance control arm |
| Calibration target inside the training data | 10 excluded targets, two objectives, fresh-seed validation, equifinality scan |
| Sensitivity from one fitted surrogate | Direct-SUMO Sobol indices (noise-corrected) + 50 refits of the surrogate |
| Headway coefficient misread | 8 × 8 × 10-seed factorial at two backgrounds, outcomes split by vehicle type |

## Key results (full profile, 10,216 SUMO runs, 0 failures)

| Question | Answer from fresh simulations |
|---|---|
| Do surrogates generalize? | Frozen ensemble single-run test R² = 0.937 (all-vehicle delay), 0.941 (general-traffic delay), 0.991 (VKT); Ridge 0.478 / 0.417 / 0.945 |
| How close to the simulator's limit? | Seed noise SD 1.6 s (delay); ensemble test RMSE 2.3 s; R² ceiling 0.969 |
| Do 90 % intervals cover? | 0.80–0.90 marginally for delay; only 0.70–0.75 in the most congested tertile |
| What drives delay in SUMO itself? | Direct-SUMO Sobol S_T: departure variation 0.74, bus headway 0.21 (all vehicles); headway 0.014 for general traffic |
| What does bus headway do? | Longer headway removes buses (−84 vehicles), cutting the all-vehicle mean by 17 s; general-traffic delay unchanged (−0.3 s [−1.3, 0.7]) |
| Do surrogate interventions work? | 20/20 reduce delay in SUMO (mean 15.8 % vs 17.0 % predicted) but only 10/20 reach the 15 % target; random same-size changes: 6.8 %, 4/20 |
| Can calibration recover parameters? | Two aggregate outputs: error 0.29 of range (random guess 0.33); eight outputs incl. hourly arrivals: 0.20 |
| Do models transfer to an 8×8 grid? | Frozen: R² < −13, coverage 0 %; residual adaptation with 128 runs: R² 0.93 |

## Repository layout

```
src/sumo_study/        analysis package
  config.py            parameter domain, outputs, experiment sizes, seed registry
  simulator.py         activitygen -> duarouter -> sumo, parsing, run cache
  design.py            Sobol / uniform designs, disjointness checks
  surrogates.py        Ridge, GP, RF, LightGBM, CatBoost, XGBoost, ensemble
  validation.py        independent test, noise components, interval methods
  policy.py            recourse search and matched-seed evaluation
  calibration.py       inverse calibration and equifinality scan
  sensitivity.py       direct-SUMO / surrogate Sobol, SHAP, ALE, DML
  adaptation.py        8×8 transfer adaptation
  workflow.py          ordered, resumable stages (Study class, CLI)
  figures.py           manuscript figures (figure-studio classic theme)
  export.py            key numbers, LaTeX tables, SHA-256 manifest
notebooks/             SUMO_Surrogate_Study.ipynb – the documented, ordered study
docs/                  ANALYSIS_PROTOCOL.md (fixed before the full run)
results/full/          tables/, figures/{png,pdf,svg,tiff}/, designs/, latex_tables/, key_numbers.json
```

## Reproduce

Requirements: Eclipse SUMO 1.28.0 (`sumo`, `netgenerate`, `activitygen`, `duarouter` on `PATH`) and Python ≥ 3.11.

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
# quick end-to-end check (~3 min)
python scripts/run_study.py smoke 8
# full study (~10,000 SUMO runs; about 90 min wall time on 8 Apple-M5 cores)
python scripts/run_study.py full 8      # or: PYTHONPATH=src python -m sumo_study.workflow --profile full --workers 8
```

Or open `notebooks/SUMO_Surrogate_Study.ipynb` (also runs on Google Colab: `pip install eclipse-sumo==1.28.0`).
Stages are resumable; SUMO runs are cached under `cache/` by a hash of the simulator fingerprint,
parameters and seed. Changing the simulator, physics or experiment sizes requires a new output folder.

## Scope

All results are conditional on a synthetic signalised grid (6 × 6, with an 8 × 8 stress test), SUMO's
activitygen demand model (vehicle trips only; bus outcomes are bus vehicles, not passengers) and the stated
parameter ranges. Simulator-internal effects are not real-world causal policy effects.

## License and citation

Code: MIT License (see `LICENSE`). Please cite the article and this repository (`CITATION.cff`).
