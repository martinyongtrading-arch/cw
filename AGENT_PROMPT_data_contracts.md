# Agent brief: real-data contracts and open policy items for crowding_matrix

## 0. Values the user fills before the agent starts

| Placeholder | Meaning | Value |
|---|---|---|
| `<REPO_DIR>` | The absolute path of the local `crowding_matrix` repository. | |
| `<DB_ACCESS>` | The client tool or Python library for the database, and the name of the environment variable that holds a read-only connection string. Do not paste the secret here. | |
| `<REGISTRY>` | The location of the mapping registry: schema and table names, or file paths, and any field-dictionary documents. | |
| `<REPORT_DIR>` | The absolute path of a folder outside the repository. The agent writes all queries, exports, scripts and reports there. | |
| `<SCOPE>` | The universe and date range for evidence queries. Example: "ASX 300 constituents, 2019-01-01 to 2026-10-30". | |
| `<AUTHORIZED_ITEMS>` | The decision items from Section 8 that the user approves now, each with the chosen option. Write "none" to approve nothing yet. | |

If a cell in the Value column is empty, stop. Ask the user for the value.

## 1. Goal

The repository `crowding_matrix` scores short-interest crowding for ASX stocks and stock indices. Its inputs are securities-lending data from S&P Global. Its test suite uses synthetic data only. Your goal is to connect the code to the user's real data contract.

Do these phases in this order:

1. Phase 1: map each canonical column to its registry field, with evidence.
2. Phase 2: measure the data properties that the open questions depend on.
3. Phase 3: write the report and the decisions that the user must make. Then stop.
4. Phase 4: change code only for items in `<AUTHORIZED_ITEMS>`, or for items that the user approves in chat.

## 2. Rules

### 2.1 Data safety

1. Use read-only access to the database. Do not run any statement that writes data or schema, for example `INSERT`, `UPDATE`, `DELETE`, `MERGE`, `CREATE`, `ALTER` or `DROP`.
2. Read the connection secret from the environment variable in `<DB_ACCESS>`. Do not print, log or save the secret.
3. Keep all queries, exports, scripts and reports in `<REPORT_DIR>`. Never write them into the repository.
4. Report counts, shares and quantiles. Do not copy raw vendor rows into a report.
5. Do not write real data, real export paths, real vendor field names or statistics about real data into the repository. Exception: the benchmark and execution scripts record the repository's own absolute path in tracked JSON files. That is allowed.
6. Do not compute returns, profit and loss, portfolio legs or comparison reports on real data. A later, pre-registered evaluation needs that data untouched.
7. For that reason, do not call `run_local_exports`, `run_analysis`, `make_labels`, `evaluate_legs`, `comparison_report` or `chronological_predictions` on real data. Do not export prices.
8. Keep each query inside `<SCOPE>`. Use a sample when a full scan is not necessary.

### 2.2 Repository safety

1. Do not commit, push, create branches or change git configuration. Leave every change uncommitted. The user reviews the changes.
2. Do not edit `work/code_quality_baseline.py`. It is the archived runtime that the benchmark compares against.
3. Never put real values into the `STOCK_EXPORTS` and `INDEX_EXPORTS` templates in `work/build_notebook.py`. An approved B1 or B2 item may add one key whose value is `None`.
4. Do not edit `ASX_Crowding_Framework.ipynb` by hand. The build and the execution script write it.
5. Do not redo the fixes in Section 4.7. Do not reopen an existing test without a new reproducer.

### 2.3 Work method

1. Before you change runtime code, write a test that fails. Run it. Check that it fails for the expected reason.
2. Make the smallest change that makes the test pass. Match the style of the surrounding code.
3. Do not add features, settings or abstractions that an approved item does not need. A new private helper function is allowed when Section 4.8 rule 8 requires it.
4. If the registry and the data do not establish a fact, write "unknown". Do not infer vendor meaning from a column name.
5. Use the terms in Section 3. Use one term for one thing.

## 3. Glossary

| Term | Meaning |
|---|---|
| canonical column | A column name that the code reads, for example `on_loan`. |
| registry | The user's mapping registry at `<REGISTRY>`. |
| vintage | One delivered version of a row. Its `available_at` timestamp identifies it. |
| decision | One session at its `decision_at` time. The code builds features from data available at that time. |
| source session | The session whose lending data a decision uses. The sessions table gives it as `expected_event_date`. The output column `data_cutoff` shows it. |
| scored stock | A stock whose `raw_scarcity_level`, `raw_positioning_level` and `raw_participation_level` are all finite at a decision. |
| Q, S, U, L | The canonical columns `on_loan`, `shares_outstanding`, `utilisation` and `lendable`. |

## 4. Repository facts

This section describes the code whose `work/notebook_source.py` has SHA-256 `c31711405181ec9d75de25d1f1cf502196911fd16cd4bd6d6d622d0174e9b3c7`.

### 4.1 Baseline checks

Run these checks before Phase 1:

1. Check that `<REPO_DIR>/.gitattributes` contains the line `* -text`. This line stops line-ending conversion.
2. Check two SHA-256 values. `work/notebook_source.py` must be `c31711405181ec9d75de25d1f1cf502196911fd16cd4bd6d6d622d0174e9b3c7`. `ASX_Crowding_Framework.ipynb` must be `ec0978237e5a456eb869fd8cb5d0e4e81a99b01bca743b75fe4c18b7635d8ba7`.
3. Set up Python. Install the verified versions if you can:
   - Python 3.13.6
   - numpy 1.26.4, pandas 2.2.3, scipy 1.17.1, scikit-learn 1.8.0
   - matplotlib 3.10.3, threadpoolctl 3.6.0, pyarrow 24.0.0
   - nbformat 5.10.4, nbclient 0.10.4, ipykernel 6.29.5, pytest 9.1.1
   - Ruff 0.16.4, psutil 7.0.0

   numpy 1.26.4 may need a source build on Python 3.13.
4. If you cannot install those versions, run `python -m pip install -r requirements-build.txt`. It pins Ruff and psutil exactly, but the packages from `requirements.txt` have version ranges only.
5. Run the test command in Section 4.6. It must report 211 passed and 0 failed.

If a check fails, stop. Report the actual values, the output of `git config core.autocrlf` and your package versions. Newer packages can emit warnings, and the test command treats warnings as failures.

### 4.2 Layout

| Path | Role |
|---|---|
| `work/notebook_source.py` | The runtime. Every function named in this brief is in this file. |
| `ASX_Crowding_Framework.ipynb` | The delivered notebook. The build embeds the runtime, the tests and six notes files into it. |
| `work/*.md` | The six embedded notes: `packaging_notes.md`, `code_quality_notes.md`, `requirement_coverage.md`, `framework_review.md`, `archetype_literature.md` and `research.md`. |
| `tests/` | Six test files. The build embeds all six into the notebook. |
| `work/build_notebook.py` | Builds the notebook. |
| `work/execute_notebook.py` | Runs the notebook in a fresh kernel. Writes `work/verification.json`. |
| `work/benchmark_code_quality.py` | Times the archived runtime against the current runtime. The build requires its records. |
| `work/packaging_notes.md` | Implementation notes. Its last paragraphs list the 2026-10-30 fixes and the open items. |

Load the runtime in Python with `api = runpy.run_path(r"<REPO_DIR>/work/notebook_source.py")`. Use the absolute path, because your scripts run from `<REPORT_DIR>`. Then call a function as `api["build_feature_panel"](...)`.

### 4.3 Canonical input tables

| Table | Required columns | Optional columns that the code reads |
|---|---|---|
| lending | `security_id`, `event_date`, `available_at`, `utilisation`, `on_loan`, `lendable`, `shares_outstanding`, `broker_count` | `annual_fee`, `fee_definition`, `utilisation_definition`, `quantity_definition`, `count_definition`, `active_lenders`, `total_lenders` |
| sessions | `session`, `decision_at` | `open_at`, `close_at`, `expected_event_date` |
| universe | `session`, `security_id`, `eligible`, `gics_sector`, `available_at` | `borrowable`, `borrow_available`, `round_trip_cost_bps`, `adv_shares`, `baseline_score`, `index_name`, `index_asx200`, `index_asx300` |
| prices | `security_id`, `session`, `open`, `close` | none |
| events | `event_id`, `security_id`, `event_date`, `available_at`, `event_type`, `status` | none |
| index metrics | `index_id`, `event_date`, `available_at`, `positioning_metric` | none |
| memberships | `session`, `security_id`, `index_id`, `available_at`, `adv_eligible` | none |

The memberships table feeds `aggregate_initial_stock_vintages`. The sessions table needs `expected_event_date` for real stock data, and `close_at` for both index paths. `capacity_report` reads `adv_shares`, `borrow_available` and `entry_price` from the frame that it receives.

### 4.4 Value contracts in the code

1. Timestamps `available_at`, `decision_at`, `open_at` and `close_at` need a timezone or a UTC offset. A timestamp without one raises `InputError`.
2. `session`, `event_date` and `expected_event_date` are local ASX dates without a timezone. A timezone on `expected_event_date` raises `TypeError`. Every lending `event_date` must be a session in the sessions table.
3. Put each `decision_at` on the same Australia/Sydney date as its `session`. The code takes the decision date from `decision_at`. If that date is not a listed session, the code raises `InputError`. If it is another listed session, the code does not raise. It then labels the rows with that other date, which creates duplicate rows. Check this rule in your own script.
4. Without `expected_event_date`, the source session is the decision session. A non-missing `expected_event_date` must be a listed session no later than the decision session, or the code raises `InputError`. A missing `expected_event_date` gives that decision no output rows.
5. `eligible`, `borrowable` and `adv_eligible` accept True, False, 0, 1 or a missing value. A missing `eligible` value excludes the stock. A missing `adv_eligible` value makes that index metric missing for that session.
6. Columns that end in `_id` must not be missing or blank. The CSV reader keeps them as text, with leading zeros.
7. U is a percentage from 0 to 100. A value below 0 or above 100 becomes missing and counts in `invalid_utilisation_count`. An invalid current value sets `data_artifact` and leaves the stock unscored. A fraction scale from 0 to 1 passes these checks without a warning.
8. Q must be finite and not negative. S must be finite and positive. If Q/S overflows, the code raises `InputError`. The code reports Q/S above 1 in `positioning_above_one_count` and does not reject it.
9. `invalid_utilisation_count`, `positioning_above_one_count` and `blank_definition_label_count` count the last 250 source sessions (`Config.window`).
10. Q and S must use the same share unit and the same corporate-action basis in every vintage. Positioning flow is the EMA contrast of Q, divided by the current S.
11. `broker_count`, `active_lenders` and `total_lenders` are entity counts. Known values must be finite integers that are not negative. On rows with both lender counts, `active_lenders` must not exceed `total_lenders`. Do not map an average or a weighted count to these columns.
12. `annual_fee` is an annual borrower fee rate. `canonical_fee` converts it with a unit: `decimal`, `percent` or `bps`. Known values must be finite and not negative. An ordinal score, for example DCBS, is not a rate.
13. Definition labels: a null label on a delivered row does not start a new definition. That row counts in `blank_definition_label_count`. An empty string is a different label. It starts a new definition and does not count, so export empty labels as null. History starts after the last labelled row whose label differs from the latest label. An absent row is not a definition change. If a mapped definition column has no label on any delivered row up to the source session, the code discards that theme's history. The theme's flow, curvature and unusualness are then missing, and its valid count is 0. The level stays. For `fee_definition`, `fee_valid_count` becomes 0. For `quantity_definition`, `quantity_growth` is also missing. Do not map a definition column that has no labels.
14. Events: the code skips a `cancelled` row. On an `active` row, `event_type` must be exactly `dividend` or `meeting`. Any other status or type raises `InputError`. Each active event that is available at a decision must have its `event_date` in the sessions table.
15. A `dividend` event masks the sessions from 10 before to 2 after its `event_date`. A `meeting` event masks the sessions from 20 before to 2 after. The mask excludes those sessions from the three theme histories: flows, curvatures, unusualness, the `*_valid_count` columns and `scarcity_instability`. It does not change the current levels, the fee statistics, `quantity_growth`, `supply_withdrawal`, `optional_demand_quadrant` or the counts in item 9. `current_session_masked` means that a mask covers the source session.
16. `positioning_metric` must be numeric, finite and not negative. The code assumes that it is the equal-weighted mean of stock Q/S after the ADV mask. Its `event_date` must be a session.
17. The stock path selects the latest vintage of each (`security_id`, `event_date`) that is available at the decision. That vintage replaces the whole row.
18. The two index paths select the first vintage of each row, and read the row whose `event_date` equals the session's `expected_event_date` (or `session` when that column is absent). If that first vintage arrives after the cutoff, the value is missing. The cutoff is the earlier of `decision_at` and `close_at`.
19. `aggregate_initial_stock_vintages` keeps the latest membership vintage of each (`session`, `security_id`, `index_id`). A correction that omits a stock leaves that stock in the index.

### 4.5 Entry points and settings

1. `load_table(path, mapping)` reads `.csv`, `.parquet` or `.pq`. `mapping` maps each canonical column to one export column. The mapping must be one to one.
2. `canonical_fee(values, unit)` converts a fee series to an annual decimal rate. `unit` is `decimal`, `percent` or `bps`.
3. `build_feature_panel(lending, sessions, universe, config, events)` returns one row per eligible stock and decision. The code scores a row only when `raw_scarcity_level`, `raw_positioning_level` and `raw_participation_level` are all finite. Other rows have missing `baseline` and `overlay`. It computes no returns. Use it for real-data features.
4. `run_index_exports(settings)` reads index metrics and sessions, then computes scores and the absorption ratio (AR). It computes no returns. Its keys are `index_metrics_path`, `index_metrics_columns`, `sessions_path`, `sessions_columns` and the optional `normalize_252`. It does not raise on bad input. It returns `status` `input_error` with a `message`. On success, `status` is `research_complete`, and the AR rows are in `result["series"]`. If a `_path` or `_columns` key is missing or empty, `status` is `setup_required`, with a `missing` list.
5. `aggregate_initial_stock_vintages(lending, memberships, sessions)` builds index metrics from stock vintages. Pass its result to `index_crowding_scores(metrics)`. Pass those scores to `absorption_ratio(scores)`. Its `"series"` value holds the AR rows. AR needs at least 20 indices with usable changes.
6. `run_local_exports(settings, config)` also computes returns. Do not call it on real data in this task. Its settings are still the contract for later work. Its keys are `<table>_path` and `<table>_columns` for `lending`, `sessions`, `universe` and `prices`, plus the optional `events_path` and `events_columns`.
7. `run_local_exports` requires five declarations set to True: `vintages_are_as_received`, `row_availability_is_max_input`, `point_in_time_universe`, `consistent_corporate_actions` and `prices_include_distributions`.
8. If lending maps `annual_fee`, `run_local_exports` also requires `fee_unit` and `fee_kind`. `fee_kind` is `actual_borrower`, `indicative` or `benchmark`. For `benchmark`, `borrow_fee_multiplier` must be a finite number of at least 1.

### 4.6 Commands

Run every command from `<REPO_DIR>`.

| Task | Command (PowerShell) |
|---|---|
| Tests | `$env:PYTEST_DISABLE_PLUGIN_AUTOLOAD='1'; $env:LOKY_MAX_CPU_COUNT='1'; python -W error -m pytest tests -q --tb=short -p no:cacheprovider --basetemp='<REPORT_DIR>\pytest_tmp'` |
| Benchmark, 10 decisions | `python work/benchmark_code_quality.py` |
| Benchmark, full | `python work/benchmark_code_quality.py --full --sizes 8 200 --output work/code_quality_benchmark_full` |
| Build | `python work/build_notebook.py` |
| Execute | `python -u work/execute_notebook.py` |
| Format | `python -m ruff format work/notebook_source.py` |

In Bash, set the two variables before `python` in the same line: `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 LOKY_MAX_CPU_COUNT=1 python -W error -m pytest ...`.

Durations on the reference machine: the tests take 11 to 16 seconds. The two benchmark commands take about 4 minutes together. The build takes about 30 seconds. The execution takes about 20 seconds.

### 4.7 Fixes already in the code

Do not redo these fixes:

1. The overlay standardizes its flow path on scored stocks with all three flows. The 0.5 weight stays fixed. The code publishes `overlay_flow_cohort`.
2. `comparison_report` uses non-overlapping holding periods from the first date with flow information.
3. One definition-label policy applies on every path, as Section 4.4 item 13 states.
4. `quantity_growth` and the demand quadrant need one quantity definition and one fee definition across the change window.
5. Each decision ranks features only among scored stocks. The output has `eligible_members`, `delivered_members` and `scored_members`.
6. The input gates in Section 4.4 items 8, 11, 12 and 16 are in place. `run_local_exports` checks the benchmark multiplier and records it in the result metadata.
7. AR rows have `zero_change_share` and `structure_changed_in_delta_window`.
8. Identical window values give a standard deviation of exactly 0. Research models use an explicit list of feature names.

### 4.8 Rules that keep the build passing

1. The build refuses to run unless the benchmark records carry the current SHA-256 of `work/notebook_source.py`. After any change to that file, run both benchmark commands before the build.
2. Run the two benchmark commands one after the other. Do not run other heavy work while they run. They measure wall-clock time.
3. The benchmark compares the archived and current outputs with `rtol` 1e-10, `atol` 1e-12, strict dtypes and strict missingness. It skips the columns in `METHODOLOGY_CHANGED_COLUMNS` in `work/benchmark_code_quality.py`. `METHODOLOGY_ADDED_COLUMNS` must equal exactly the set of columns that the current runtime outputs and the archived runtime does not, on the benchmark fixture. Removing an output column fails the benchmark.
4. If your change adds an output column, add it to `METHODOLOGY_ADDED_COLUMNS`. If your change deliberately changes a compared column, add it to `METHODOLOGY_CHANGED_COLUMNS`. Derive both sets from your change, not from observed differences.
5. The benchmark also checks that `asdict(Config())` is equal in both runtimes. A new `Config` field fails this check. Ask the user before you change `Config` or this check.
6. After the benchmarks, delete the folders that match `work/code_quality_benchmark/*stocks_*` and `work/code_quality_benchmark_full/*stocks_*`. Git ignores their `.pkl` files but not their `measurement.json` files.
7. Keep each file's line endings. Check with `git ls-files --eol <path>` before and after each edit. `README.md`, `work/notebook_source.py` and `tests/test_stock_optimization_candidate.py` use CRLF. The `work/*.md` notes, `work/build_notebook.py`, `work/benchmark_code_quality.py` and the other five test files use LF. The scripts write the notebook and the JSON records with the line endings of the platform: CRLF on Windows, LF on other systems. On a system other than Windows, expect line-ending changes in those files, and report them.
8. Each notebook code cell must have at most 99 lines. The build adds one header line to each cell. Comment lines directly above a top-level function go into the cell of that function. So a function and its comment lines can have at most 98 lines. `evaluate_legs` has 97 lines and 1 comment line, so it has no room. `aggregate_initial_stock_vintages` has 94 lines and 1 comment line. `absorption_ratio` has 92 lines. Put new logic for these functions into a new private helper function.
9. Format the runtime with Ruff 0.16.4. Ruff keeps CRLF line endings.
10. The build embeds every module-level function of the six test files, helpers included. It does not embed module-level constants, module-level imports, pytest fixtures or tests inside classes. A new test cannot use a new fixture. Helper names must be unique across the six files.
11. In the notebook, each test runs as `test(notebook_globals, **params)`. So the first parameter of each test must be `api`, and tests reach runtime functions through `api`. Tests in `tests/test_formula_catalogue.py` take `catalogue_module` instead. Use `np`, `pd`, `pytest` and `Path`, or import inside the test.
12. The notebook injects only `tmp_path`, and all tests share one folder. Use unique file names.
13. Use at most one `pytest.mark.parametrize` per test. Give its argument names as one comma-separated string. Its values must be Python literals.
14. All embedded checks run in one notebook cell with a 90-second timeout. Keep new tests fast. After execution, `embedded_checks` in `work/verification.json` must equal the pytest count.
15. Warnings are failures in the test command and in the notebook execution. Do not suppress warnings broadly.
16. Refer to functions by name. Line numbers change.

## 5. Phase 1: map the registry

1. Read the structure of `<REGISTRY>`. List its tables or files and their fields.
2. For each canonical column in Section 4.3, find the registry field that supplies it.
3. For each match, record these attributes:
   - source table and field
   - unit
   - definition or version field
   - population
   - corporate-action basis
   - availability timestamp and its timezone
   - revision behaviour
4. For each attribute, give its status: "documented" (with the registry reference), "measured" (with the Phase 2 item) or "unknown".
5. Write the result as Table R in the report.

## 6. Phase 2: measure the data

Save each query as `<REPORT_DIR>/queries/<item id>.sql` or as a script in `<REPORT_DIR>/scripts/`. In the report, give the file name next to each result.

### 6.1 Table A: field semantics

| Id | Question | Measure exactly |
|---|---|---|
| A1 | Is U a percentage from 0 to 100? | Minimum, 1st, 50th and 99th percentiles, maximum. Share of rows with U ≤ 1, U > 100 and U = 0. |
| A2 | Do Q and S use the same share unit? | 1st, 50th and 99th percentiles of S. 50th and 99th percentiles and maximum of Q/S. Share of rows with Q/S > 1. |
| A3 | Are Q and S histories on one corporate-action basis? | Count stock-days where S changes by a factor of at least 1.5, or of at most 1/1.5, from the previous session. For these, give the share where Q changes by the same factor within 5%. For 10 such events, check whether later vintages restate the earlier Q and S. |
| A4 | Are the counts entity counts? | Share of non-integer values in `broker_count`, `active_lenders` and `total_lenders`. Share of rows where `active_lenders` exceeds `total_lenders`. |
| A5 | What is the fee field? | Registry definition: borrower fee, lender fee, rebate or score. Unit. 1st, 50th and 99th percentiles and maximum. |
| A6 | How are definitions labelled? | For each definition field: distinct values, with first and last dates. Share of delivered rows with a null label, and with an empty-string label. Share of (`security_id`, session) pairs in the universe with no delivered row. |
| A7 | How do vintages behave? | Share of (`security_id`, `event_date`) keys with more than one vintage. For later vintages, share with at least one mapped field missing. 50th, 90th and 99th percentiles of the delay from `event_date` to `available_at`. |
| A8 | What are the calendar and the delivery lag? | Source of the ASX session calendar. Sessions missing against the exchange calendar. The delay from `event_date` to the first vintage. A proposed rule for `expected_event_date`. |
| A9 | Is the universe point in time? | Source of eligibility and GICS sector with their availability times. Share of universe rows without a sector. Whether delisted stocks stay in the universe. |
| A10 | How do index memberships change? | Whether membership files are complete snapshots or lists of changes. Whether a correction excludes a stock by omission or by a flag. Source of the point-in-time ADV mask. For supplied index metrics: definition, unit and weighting. |
| A11 | What event data exists? | Dividend record dates and meeting dates, with announcement times and cancellation status. Coverage by year. |
| A12 | Which execution inputs exist? | Document only: borrowable flag, borrow availability, ADV shares, round-trip cost and the price adjustment basis. Do not compute returns. |

### 6.2 Table C: feature diagnostics on real data

Prepare the inputs this way:

1. Build the sessions table yourself. Take `session` from the A8 calendar.
2. Make `open_at`, `close_at` and `decision_at` timezone-aware. Put each `decision_at` on the same Australia/Sydney date as its `session`.
3. Set `expected_event_date` from the A8 proposed rule. Each value must be a listed session no later than its `session`. A missing value gives that decision no rows.
4. Label every Table C result as provisional on that rule.
5. Export the full lending history. Every lending `event_date` must be a listed session.
6. Add sessions after the end of `<SCOPE>` so that the sessions table lists every active event's `event_date`. Drop events dated before the first session.
7. Map `annual_fee` only if A5 documents a non-negative annual borrower rate with a known unit. Convert it with `canonical_fee`. Without a mapped `fee_definition`, `fee_valid_count` is 0.
8. To limit run time, restrict only the universe table to at most 400 decision sessions at the end of `<SCOPE>`. Keep the full lending history and the full sessions table. A flow needs 250 valid observations up to the source session. Event masks (13 sessions per dividend, 23 per meeting), missing rows, invalid values, definition changes and the `expected_event_date` lag reduce that count. Give each decision session at least 350 earlier sessions. If `<SCOPE>` is too short for that, use fewer decision sessions and say so in the report. On the reference machine, 330 sessions × 200 stocks took about 54 seconds. Time grows with the length of the history.
9. Export the tables to Parquet in `<REPORT_DIR>/exports/`. Write `<REPORT_DIR>/scripts/run_features.py`. Load each table with `load_table`.
10. Call `build_feature_panel(lending, sessions, universe, api["Config"](), events)`.

| Id | Measure exactly |
|---|---|
| C1 | Per decision: `eligible_members`, `delivered_members` and `scored_members`. The 1st and 50th percentiles of `scored_members` / `eligible_members`. The ten sessions with the lowest ratio. The number of decision sessions with no output rows. |
| C2 | Per decision: `overlay_flow_cohort` / `scored_members`. The 1st and 50th percentiles. |
| C3 | Share of rows with `blank_definition_label_count` > 0. The 50th percentile of `fee_valid_count`. |
| C4 | For each row with `current_session_masked` True, take the same stock's latest earlier row with `current_session_masked` False. Compute the absolute change of `raw_scarcity_level` and of `raw_positioning_level`. Divide each change by the interquartile range of that column across stocks at the masked decision. Give the 50th and 90th percentiles. |
| C5 | Share of rows with `positioning_above_one_count` > 0. |
| C6 | Per decision: share of scored stocks whose `raw_participation_level` equals that of another scored stock. Give the 50th and 90th percentiles across decisions. |
| C7 | If index data exists, give the share of AR rows with a value and the distributions of `N` and `zero_change_share`. Give the share of rows with `structure_changed_in_delta_window`. Use `run_index_exports`, or `aggregate_initial_stock_vintages` followed by `index_crowding_scores` and `absorption_ratio`. Check `status` first when you use `run_index_exports`. Count AR rows only for sessions inside `<SCOPE>`. The first 61 sessions of the sessions table never get an AR value. |
| C8 | Count late releases in your own script, because no function outputs this count. Take the earliest `available_at` per (`index_id` or `security_id`, `event_date`). Join it on `expected_event_date`. Count the rows later than the earlier of `decision_at` and `close_at`. |

Never fill missing `adv_eligible` values. If no point-in-time ADV mask exists, skip the `aggregate_initial_stock_vintages` route.

## 7. Phase 3: write the report, then stop

Write `<REPORT_DIR>/data_contract_report.md` with these sections:

1. The results of the baseline checks in Section 4.1.
2. Table R from Phase 1.
3. The Table A results, with query file names.
4. The Table C results, with script names. Mark them provisional on the `expected_event_date` rule.
5. For each decision item in Section 8: the evidence, the options, your recommended option, the code places to change and the tests to add.
6. A proposed settings dictionary for `run_local_exports` and for `run_index_exports`, with real paths and real column names. Leave each of the five declarations False unless the evidence establishes it. Save the dictionary as `<REPORT_DIR>/settings_proposal.json`. Never save it in the repository.
7. Each unknown, with the person or document that can resolve it.

Then write a summary of at most 15 lines in chat. Stop. Wait for the user's approval. Continue without approval only for items in `<AUTHORIZED_ITEMS>`.

## 8. Phase 4: implement approved items

### 8.1 Decision items

| Id | Decision | Options | Code places |
|---|---|---|---|
| B1 | Declared units | (a) Add a setting for the U scale and convert a fraction to a percentage. (b) Document the units only. | `run_local_exports`, the `STOCK_EXPORTS` template in `work/build_notebook.py`, notes |
| B2 | Definition of a supplied index metric | (a) Require a declared metric definition and reject other definitions. (b) Document only. | `initial_index_metrics`, `run_index_exports`, the `INDEX_EXPORTS` template |
| B3 | Scoring in an event window | (a) Keep scoring and the flag. (b) Treat a masked source session as a missing mandatory input, so the stock gets no score. | `_stock_feature_row`, `_normalize_feature_rows`, `build_scores` |
| B4 | Minimum coverage | (a) No gate. (b) A declared minimum of `scored_members` / `eligible_members`. Below it, the decision gets no scores. | `_normalize_feature_rows` or `build_scores`. A new `Config` field triggers Section 4.8 rule 5. |
| B5 | Capacity by side | (a) Report a long capacity without the borrow limit and a short capacity with it. (b) Keep one column. | `capacity_report` |
| B6 | Ties across legs | (a) Exclude names of the long leg from the short leg. (b) Form no legs when scores have no spread. (c) Keep the current rule. | `evaluate_legs` (no room left: use a helper) |
| B7 | Input types | (a) Declare a type per canonical column, reject booleans as quantities, and treat text missing-value tokens the same in CSV and Parquet. (b) Keep the current readers. | `load_table` |
| B8 | Membership corrections | (a) Use the latest complete snapshot per (`session`, `index_id`) as a set. (b) Accept explicit removal rows. (c) Keep the current rule. | `aggregate_initial_stock_vintages` (use a helper), `universe_at`, `_prepared_universe_selector` |
| B9 | Vintage granularity | (a) Keep whole rows. (b) Use the latest complete record. (c) Select each field separately. | `history_asof`, `_prepared_history_selector`, `initial_index_metrics`, `aggregate_initial_stock_vintages` |

The stock path has two loops. The reference loop is `_build_feature_panel_reference`, with `features_at`, `history_asof`, `universe_at` and `event_mask`. The prepared loop is `build_feature_panel`, with `_prepare_stock_context`, `_canonical_stock_data`, `_prepare_event_groups` and the `_prepared_*` functions. Tests compare the two loops. Change both loops in the same way.

### 8.2 Procedure for each approved item

1. Write a failing test in one of the six test files. Prefer `tests/test_quality_regressions.py`. Follow Section 4.8 rules 10 to 14.
2. Run the test. Check that it fails for the expected reason.
3. Make the smallest change in `work/notebook_source.py`.
4. Format the runtime. Check the line endings.
5. Run the full test command. All tests must pass.
6. If a contract changes, update `work/packaging_notes.md` and the `README.md` text. Never write real-data statistics into the repository.

### 8.3 Procedure after the last approved item

1. Update the two declared column sets, as Section 4.8 rules 3 and 4 state.
2. If `work/notebook_source.py` changed, run both benchmark commands, one after the other. Then delete the intermediate benchmark folders, as Section 4.8 rule 6 states.
3. Update the timings and the counts of compared, added and changed columns in `README.md`, `work/packaging_notes.md` and `work/code_quality_notes.md`. Use the new benchmark records only. The current counts are 38 compared, 7 added and 32 changed.
4. Run the build. The build embeds the notes, so the notes must be final before this step.
5. Run the execution command.
6. Run the full test command again.
7. Open `work/verification.json`. Check that `status` is `executed`, that `error_outputs` and `warning_outputs` are 0, that `embedded_sources_match` is true and that `embedded_checks` equals the pytest count.
8. Check that the SHA-256 of `ASX_Crowding_Framework.ipynb` equals `notebook_sha256` in `work/verification.json`.
9. Do not commit. List every changed file with its new SHA-256.

## 9. Final report

After Phase 4, report:

1. The items that you implemented, with their tests.
2. The test count, the number of executed cells and embedded checks, and the new SHA-256 values.
3. The new benchmark timings.
4. The items that stay open, with the reason.

## 10. When to stop and ask

Stop and ask the user when one of these conditions occurs:

1. A Value cell in Section 0 is empty.
2. A baseline check in Section 4.1 fails.
3. The registry and the data disagree about a unit or a definition.
4. An item needs a decision that `<AUTHORIZED_ITEMS>` does not give.
5. The evidence contradicts an option in `<AUTHORIZED_ITEMS>`.
6. A query needs write access or data outside `<SCOPE>`.
7. Real data fails an `InputError` gate. Do not change the values. Report the gate, the message and the number of rows.
8. A required column has no registry field. Mark Table C as blocked, and finish Table A.
9. A code change gives a compared benchmark column different values, and no approved item explains this.
10. The build or the execution fails twice for the same reason.
