
## Fault 1 - Forbidden casual column

Injected change:
```python
return [c for c in numeric if c not in FORBIDDEN and c != "humidity_interaction"] + ["casual"]
```

Manual hook invocation exit code: 2
Hook stderr (actual):
```text
Leakage tests failed after editing src/features.py:
feats =         dteday  instant  season  ...  cnt_lag_1 cnt_lag_7   cnt_roll_7
0   2011-01-10     10.0  WINTER  ...      822.0...    1787.0  1530.142857
713 2012-12-31    731.0  WINTER  ...     1796.0     920.0  1531.428571

[714 rows x 29 columns]

    def test_no_forbidden_features(feats):
        """Guards the gate, not the data: feature_columns() excludes FORBIDDEN by
        construction, so this cannot discover a *new* leaky column. What it
        catches is someone editing feature_columns() or FORBIDDEN -- which is
        exactly the fault the Step 7 hook demonstrates."""
        cols = set(feature_columns(feats))
>       assert not (cols & FORBIDDEN), f"forbidden columns in feature set: {cols & FORBIDDEN}"
E       AssertionError: forbidden columns in feature set: {'casual'}
E       assert not ({'atemp', 'casual', 'cnt_lag_1', 'cnt_lag_7', 'cnt_roll_7', 'dayofweek', ...} & {'casual', 'cnt', 'cnt_2d_bfr', 'days_since_2011', 'instant', 'registered', ...})

tests/test_leakage.py:32: AssertionError
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_no_forbidden_features - AssertionError: fo...
1 failed, 8 passed in 5.38s

```
Hook stdout (actual):
```text

```
Test exit code: 2
Failing test output (actual):
```text
python -m pytest tests/ -q
.....F..........                                                         [100%]
=================================== FAILURES ===================================
__________________________ test_no_forbidden_features __________________________

feats =         dteday  instant  season  ...  cnt_lag_1 cnt_lag_7   cnt_roll_7
0   2011-01-10     10.0  WINTER  ...      822.0...    1787.0  1530.142857
713 2012-12-31    731.0  WINTER  ...     1796.0     920.0  1531.428571

[714 rows x 29 columns]

    def test_no_forbidden_features(feats):
        """Guards the gate, not the data: feature_columns() excludes FORBIDDEN by
        construction, so this cannot discover a *new* leaky column. What it
        catches is someone editing feature_columns() or FORBIDDEN -- which is
        exactly the fault the Step 7 hook demonstrates."""
        cols = set(feature_columns(feats))
>       assert not (cols & FORBIDDEN), f"forbidden columns in feature set: {cols & FORBIDDEN}"
E       AssertionError: forbidden columns in feature set: {'casual'}
E       assert not ({'atemp', 'casual', 'cnt_lag_1', 'cnt_lag_7', 'cnt_roll_7', 'dayofweek', ...} & {'casual', 'cnt', 'cnt_2d_bfr', 'days_since_2011', 'instant', 'registered', ...})

tests/test_leakage.py:32: AssertionError
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_no_forbidden_features - AssertionError: fo...
1 failed, 15 passed in 13.49s
make: *** [Makefile:16: test] Error 1

```
Repair: original source bytes restored by script.
Agent repair turns: N/A (manual restoration).
PostToolUse hook was invoked manually, not by Claude's Edit tool.
Recovery output:
```text
python -m pytest tests/ -q
................                                                         [100%]
16 passed in 7.98s

```

## Fault 2 - Rolling mean includes today

Injected change:
```python
out[TARGET].rolling(LAG_WARMUP).mean()
```

Manual hook invocation exit code: 2
Hook stderr (actual):
```text
Leakage tests failed after editing src/features.py:
[714 rows x 29 columns]

    def test_rolling_mean_excludes_today(feats):
        """cnt_roll_7 must be the mean of the 7 calendar days *before* today."""
        by_date = feats.set_index("dteday")[TARGET]
        date = feats.dteday.iloc[200]
        window = [date - pd.Timedelta(days=k) for k in range(1, 8)]
        if all(d in by_date.index for d in window):
            expected = sum(by_date[d] for d in window) / 7
>           assert feats.set_index("dteday").cnt_roll_7[date] == pytest.approx(expected)
E           assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
E             
E             comparison failed
E             Obtained: 4389.0
E             Expected: 4414.857142857143 ± 0.00441486

tests/test_leakage.py:58: AssertionError
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today - assert np.fl...
1 failed, 8 passed in 5.31s

```
Hook stdout (actual):
```text

```
Test exit code: 2
Failing test output (actual):
```text
python -m pytest tests/ -q
........F.......                                                         [100%]
=================================== FAILURES ===================================
_______________________ test_rolling_mean_excludes_today _______________________

feats =         dteday  instant  season  ...  cnt_lag_1 cnt_lag_7   cnt_roll_7
0   2011-01-10     10.0  WINTER  ...      822.0...    1787.0  1531.428571
713 2012-12-31    731.0  WINTER  ...     1796.0     920.0  1789.857143

[714 rows x 29 columns]

    def test_rolling_mean_excludes_today(feats):
        """cnt_roll_7 must be the mean of the 7 calendar days *before* today."""
        by_date = feats.set_index("dteday")[TARGET]
        date = feats.dteday.iloc[200]
        window = [date - pd.Timedelta(days=k) for k in range(1, 8)]
        if all(d in by_date.index for d in window):
            expected = sum(by_date[d] for d in window) / 7
>           assert feats.set_index("dteday").cnt_roll_7[date] == pytest.approx(expected)
E           assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
E             
E             comparison failed
E             Obtained: 4389.0
E             Expected: 4414.857142857143 ± 0.00441486

tests/test_leakage.py:58: AssertionError
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today - assert np.fl...
1 failed, 15 passed in 12.94s
make: *** [Makefile:16: test] Error 1

```
Repair: original source bytes restored by script.
Agent repair turns: N/A (manual restoration).
PostToolUse hook was invoked manually, not by Claude's Edit tool.
Recovery output:
```text
python -m pytest tests/ -q
................                                                         [100%]
16 passed in 7.62s

```

## Fault 3 - Weather encoder stops raising

Injected change:
```python
pass
```

Manual hook invocation exit code: 2
Hook stderr (actual):
```text
Leakage tests failed after editing src/features.py:
........F                                                                [100%]
=================================== FAILURES ===================================
______________________ test_unknown_weather_label_raises _______________________

    def test_unknown_weather_label_raises():
        from src.features import encode_weather
>       with pytest.raises(ValueError, match="unmapped weather"):
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
E       Failed: DID NOT RAISE ValueError

tests/test_leakage.py:80: Failed
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_unknown_weather_label_raises - Failed: DID...
1 failed, 8 passed in 5.23s

```
Hook stdout (actual):
```text

```
Test exit code: 2
Failing test output (actual):
```text
python -m pytest tests/ -q
............F...                                                         [100%]
=================================== FAILURES ===================================
______________________ test_unknown_weather_label_raises _______________________

    def test_unknown_weather_label_raises():
        from src.features import encode_weather
>       with pytest.raises(ValueError, match="unmapped weather"):
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
E       Failed: DID NOT RAISE ValueError

tests/test_leakage.py:80: Failed
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_unknown_weather_label_raises - Failed: DID...
1 failed, 15 passed in 12.89s
make: *** [Makefile:16: test] Error 1

```
Repair: original source bytes restored by script.
Agent repair turns: N/A (manual restoration).
PostToolUse hook was invoked manually, not by Claude's Edit tool.
Recovery output:
```text
python -m pytest tests/ -q
................                                                         [100%]
16 passed in 7.58s

```

## Fault 4 - Bash cp bypass

Command: `cp data/raw/bike_daily.csv data/raw/ma6_hook_probe.csv`

The copy was made through Bash, not Claude's Edit or Write tools. The PostToolUse Edit|Write hook therefore did not run automatically.

Original SHA256 before: a6bed1577f6e4de56dcfae6d8315985073cc0ad3446c8c99ed60c86a0b5ad9dc
Original SHA256 after: a6bed1577f6e4de56dcfae6d8315985073cc0ad3446c8c99ed60c86a0b5ad9dc

The temporary copy was deleted. A PreToolUse Bash hook, filesystem permissions, or a raw-data hash check could detect/prevent this bypass.
