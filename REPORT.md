# Results

I evaluated four approaches to forecasting daily Capital Bikeshare demand: a training-mean baseline, yesterday's count (persistence), Ridge regression, and histogram gradient boosting (HGB). The chronological split contains 348 training days, 182 validation days, and 184 test days. I used validation performance to make feature decisions; test results are included for final comparison, not feature selection.

The feature experiment added `humidity_interaction`, defined as the product of the previous day's humidity and temperature. "Before" uses the original 14 predictors; "With feature" uses 15. R² is higher-is-better, and MAE is lower-is-better.

| Model | Feature state | Valid R² | Valid MAE | Test R² | Test MAE |
|---|---|---:|---:|---:|---:|
| Mean baseline | Before | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| Mean baseline | With feature | -0.9318 | 2002.2 | -2.1441 | 2847.7 |
| Persistence | Before | 0.4507 | 900.6 | 0.5099 | 840.1 |
| Persistence | With feature | 0.4507 | 900.6 | 0.5099 | 840.1 |
| Ridge | Before | 0.6475 | 832.9 | 0.5833 | 947.1 |
| Ridge | With feature | 0.6438 | 841.6 | 0.5818 | 951.8 |
| HGB | Before | 0.2185 | 1296.6 | -0.4975 | 1972.6 |
| HGB | With feature | 0.1868 | 1326.4 | -0.6263 | 2064.5 |

The baselines do not depend on this additional predictor. With the feature included, Ridge's validation R² decreased from 0.6475 to 0.6438, and its validation MAE increased from 832.9 to 841.6. HGB also deteriorated on both validation metrics. I therefore **dropped the feature from the predictors**. Its implementation and regression test remain in the repository so the experiment is reproducible. The final `reports/metrics.json` was generated with the original 14 predictors, not with the unsuccessful candidate.

# Headline finding

My main finding is that a plausible, leakage-safe feature does not necessarily improve forecasting. The humidity-temperature interaction is available before the predicted day because it uses `shift(1)` on both inputs, but availability alone does not make it useful. The validation results favored keeping the simpler feature set. This matters more than whether the formula sounds sensible: the objective is to forecast unseen future days, not to accumulate engineered columns.

The seeded-fault experiments also demonstrated why tests are essential in an agent-assisted workflow. Adding `casual` to the predictor list violated the forbidden-column contract; removing the shift from the rolling count introduced target leakage; and removing the weather encoder's exception allowed unexpected labels to pass silently. Each fault caused a real test failure and a manually invoked hook to exit with code 2. I restored the code and reran the tests successfully. These were **manual hook invocations**, not observed automatic Claude Edit-tool events; I document that limitation in `docs/SEEDED_FAULTS.md`. A separate Bash `cp` demonstration showed that an Edit/Write hook does not cover arbitrary shell operations.

# Exercise 4: target transformation

The original HGB model performed poorly on validation (R² 0.2185, MAE 1296.6). I compared several target representations using only the training and validation partitions, leaving the test set out of transformation selection.

| HGB target | Validation R² | Validation MAE |
|---|---:|---:|
| Original daily count | 0.2185 | 1296.6 |
| Log of daily count | 0.1352 | 1363.6 |
| Daily change from yesterday | **0.6079** | **834.3** |
| Ratio to yesterday | 0.5455 | 924.5 |

The most effective transformation was to model the difference between today's count and yesterday's: `delta_t = cnt_t - cnt_lag_1`. At inference time, the predicted difference is added back to the known previous-day count. This produces a validation R² of 0.6079 and MAE of 834.3, approaching Ridge's R² of 0.6475 and MAE of 832.9.

I consider this scientifically defensible as an experiment rather than an arbitrary numerical trick. A previous-day count is available at forecasting time, and predicting changes may be more stable than predicting absolute demand as the overall demand level changes. However, this validation improvement alone does not prove generalization. Unusual events, abrupt level changes, and the dependence on a reliable previous-day observation remain risks. Because I compared multiple transformations on the same validation period, an additional future holdout would provide stronger evidence. I kept the final tracked pipeline and metrics on the **untransformed target** as required.

# Exercise 5: ridge vs persistence

Ridge substantially outperformed persistence on validation: R² was 0.6475 versus 0.4507, while MAE was 832.9 versus 900.6. Its validation performance suggests that weather, calendar, and lagged demand provide information beyond simply repeating yesterday.

On the held-out test period, however, the decision depends on the objective. Ridge had higher R² (0.5833 versus 0.5099), but persistence had **lower MAE** (840.1 versus Ridge's 947.1). In other words, Ridge captured more variation according to R² yet made larger absolute errors on average. These metrics evaluate different properties and should not be collapsed into one statement that a model simply "wins."

For a deployment where reducing average daily count error and maintaining reliability are the priorities, **I would deploy persistence first**. It requires no model fitting, is easy to explain and monitor, and had the better test MAE. Ridge remains a reasonable challenger if explained variance is the primary objective or future evaluation shows improved absolute error. A production decision should also examine error across seasons and exceptional days, not just these aggregate scores.

# Attribution

The project began from the provided DSC 180A bikeshare starter repository. The underlying bikeshare dataset is attributed in `src/config.py` to Fanaee-T and Gama and the processed source used by Molnar's *Interpretable Machine Learning* materials.

I used ChatGPT (GPT-6) to help plan the workflow, draft shell/Python commands for reproducible experiments, interpret test output, and prepare this report draft. I used Claude Code configured with `gpt-oss:20b-cloud` through Ollama Cloud; attempts to make it carry out the seeded-fault editing workflow were refused. The seeded faults were consequently executed and restored through terminal scripts, not Claude Edit calls. I have explicitly distinguished those observations in the documentation rather than presenting them as autonomous agent repair. The numerical metrics reported here came from actual `make` and validation-only experiment outputs, not AI-generated predictions.
