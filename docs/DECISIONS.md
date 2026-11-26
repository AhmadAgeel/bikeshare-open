# Decisions

One line per methodological decision: what you chose, why, and what it changed.
Agents append here via the `/new-feature` skill; you append here by hand.

- Exclude `casual`, `registered`: they sum to `cnt` exactly. Leaky model R^2 1.000 -> honest ridge 0.583.
- Exclude `cnt_2d_bfr`: a shipped 2-day lag we cannot test the alignment of. Build our own lags instead.
- Exclude `instant`, `yr`, `days_since_2011`: time indexes let a tree predict a flat constant past 2011 and a linear model extrapolate a trend we have no grounds for.
- Split 2011 / H1 2012 / H2 2012 by date. Test set touched once, after all feature work.
- Report mean and persistence baselines with every score. Persistence (test R^2 0.510, MAE 840) is within 0.07 R^2 of ridge and beats it on MAE; that caveat goes in the report.
- 2026-11-25: Keep `humidity_interaction` feature calculation and test; exclude it from predictors due to negative impact on validation R² (Ridge 0.6475 → 0.6438, HGB 0.2185 → 0.1868). Formula: humidity_interaction = hum.shift(1) * temp.shift(1). It is not leakage.
- 2026-11-25: Added `/verify-pipeline` for the repeated test-and-status workflow; updated CLAUDE.md to preserve the documented feature-drop decision and verification procedure.
- 2026-11-25: Exercise 4 compared HGB original, log, day-over-day difference, and ratio targets on validation only. Day-over-day difference gave validation R² 0.6079 and MAE 834.3, close to Ridge (0.6475 / 832.9). The previous-day count is available at prediction time, making this a defensible exploratory transformation. Kept the final untransformed pipeline; additional future validation would be needed before deployment.
- 2026-11-25: Exercise 5 provisional deployment choice: persistence rather than Ridge when MAE and simplicity are priorities, because test MAE is 840.1 versus 947.1. Ridge retains higher test R² (0.5833 versus 0.5099).
