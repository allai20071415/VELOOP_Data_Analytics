# VELoop-data-analytics-

**VELOOP Rewards: Product Analytics, Data Readiness & Data Management Blueprint** (Data Analyst Internship, Task 01)
Repository: https://github.com/allai20071415/VELoop-data-analytics-

![Functional demo dashboard on simulated data](figures/dashboard_functional.png)

> **Data authenticity.** All data here is fictional/simulated (*SIMULATED DATA - CREATED FOR ANALYTICAL TESTING*). Nothing describes actual VELOOP users, earnings, withdrawals, retention or performance. Proposed events, fields and KPIs stay **Proposed** or **To Be Verified** until engineering confirms them.

## Data provenance: what is supplied, generated, proxy, proposed and real
| Label | Meaning | Examples in this repo |
|---|---|---|
| **SUPPLIED SIMULATED** | Company-supplied test files. Fictional, not VELOOP. | The two workbooks (`data/raw/`, not committed): users, sessions, events, tasks, milestones |
| **GENERATED TEST** | Created by this project because the pack lacked these tables. Formula tests only; supports no insight. | `data/synthetic/*` (rewards, wallet, withdrawals, referrals), methodology in `data/synthetic/METHODOLOGY.md` |
| **PROXY** | Approximation from supplied data when the real table is missing. | credited-task counts (K18), `wallet_viewed` events (K25), `referral_created` events (K39), users sourced "Referral" (K40), Help/Support sessions (K43, K45) |
| **PROPOSED** | Design for engineering review; not claimed to exist in VELOOP. | events, 100-field dictionary, ER entities not supplied, pipeline, dashboards 1-7 (wireframes), governance |
| **PRODUCTION** | Actual VELOOP data. | **None used. No metric here is production-ready.** |

Functional versus proposed: `dashboard/index.html` is the only **functional** dashboard (interactive, simulated data, aggregates only). Wireframes `figures/wireframe_*.png` are **proposed**. User master (A016) is an assumption: `Users_1000_Clean` is the master; the behaviour workbook is supplementary and never joined to it.

## Main results (simulated data only; none is production-ready)
- **35 data quality issues** (3 Critical, 11 High, 16 Medium, 5 Low). 31 have record-level evidence files in `outputs/evidence/`; a test asserts each file's row count equals the reported count. 4 are table-level.
- **46 KPIs validated by executed SQL** (`outputs/VELOOP_KPI_Validation.xlsx`): formula, source fields, limitation, status for every KPI. 38 SQL cases and 6 reconciliations pass (SQL equals independent pandas). Availability: 17 calculable, 11 proxy-only, 18 not calculable on the supplied data. All 46 are marked **not production-ready**.
- **11 insights** (`outputs/VELOOP_Insights.xlsx`). Investigated corrections: finished/started event ratios above 1.0 are **not** a duplicate artefact (event_id is unique) and **not** a valid metric: only 8.5% to 13.8% of finish events have an earlier start in the same session. The weekly active-user chart's late drop is a **partial week** (2 of 7 days; the first week has 5), and the rise partly follows the growing user base (active share 16.2% to 50.9%).
- **ER relationships tested** (`outputs/VELOOP_ER_Validation.xlsx`, line colours in the diagram): 7 pass on supplied data, 8 on generated data only, 2 fail (no derivable tasks catalogue; 15 duplicate credits), 1 not testable. Assumptions E1-E7 documented.
- The two Users tables share 1,000 IDs but disagree on every user's attributes.

![Insights](figures/insights_overview.png)

## Deploy the demo site (Netlify, Vercel or GitHub Pages)
`site/` is a static, self-contained demo site: landing page, the functional dashboard (`site/dashboard/`), figures and report PDFs. Everything is labelled simulated data. To publish it, drag the `site/` folder onto https://app.netlify.com/drop (or import the repo in Vercel or Netlify with publish directory `site`). It needs no build step and no login to view.

## Using this after real production data exists
See `docs/PRODUCTION_USAGE_GUIDE.md`: go-live gates, operating rhythm, a decision playbook mapping signals to owners and actions, and the rule that targets come from real baselines, not from this simulation.

## Project structure
```
README.md  requirements.txt  run.py
src/veloop_analytics/   config, loaders, profiling, relationships, quality (+evidence), kpi_test, kpi_engine, insights, synthetic, cleaning, report, cli
tests/                  test_quality.py, test_unit.py, test_analytics.py, test_validation.py   (23 tests; 16 need the raw data and skip without it)
scripts/                design_workbook/build.py, make_figures.py, make_insight_figures.py, make_demo_dashboard.py, build_dashboard.py, build_report_docx.py, build_presentation.py
dashboard/index.html    functional demo dashboard
site/                   static demo site for deployment (landing page, dashboard, figures, report PDFs)
data/raw/               put the two supplied workbooks here (not committed)
data/synthetic/         generated test tables + METHODOLOGY.md
data/processed/         cleaned CSVs and quarantine (generated, not committed)
outputs/                workbooks (design, profiling+DQ, KPI review, insights, scorecard), evidence/, run_manifest.json
figures/                ER, pipeline, wireframes, demo dashboards, insights, scorecard
docs/                   final report (docx+pdf), executive summary (pdf), presentation, review response, checklist
```

## Reproduce everything (about 1-2 minutes)
```bash
pip install -r requirements.txt
# 1. copy the two supplied workbooks into data/raw/ (see data/raw/README.md)
python scripts/design_workbook/build.py     # design workbook (events, dictionary, KPI catalogue...)  -> outputs/
python run.py all                           # clean, profile, evidence, synthetic, kpis, validate, er, insights, manifest
python -m pytest -q tests                   # 23 tests
python scripts/run_tests_with_evidence.py   # test log, JUnit XML and docs/TEST_EVIDENCE.md
python scripts/make_figures.py && python scripts/make_insight_figures.py && python scripts/make_demo_dashboard.py && python scripts/build_dashboard.py
python scripts/build_report_docx.py && python scripts/build_presentation.py   # report and slides
```
Individual steps: `python run.py {clean|profile|evidence|synthetic|kpis|validate|er|insights|manifest}`. `outputs/run_manifest.json` records input SHA-256 hashes and library versions. Runtime on the supplied data is about 15 s for profiling; the largest sheet has 35,714 rows and all sheets are read as text so malformed values are never coerced silently. For much larger data, read in chunks or push the checks into SQL (the KPI formulas are already SQL).

## Deliverable map
| # | Deliverable | File |
|---|---|---|
| 01 | Executive summary (2 pages) | `docs/VELOOP_Executive_Summary.pdf` |
| 21 | Final report | `docs/VELOOP_Final_Report.docx` (+ pdf) |
| 02-03, 04-09 | Audit, coverage, specs, events, dictionary (100 fields) | `outputs/VELOOP_Analytics_Foundation_Workbook.xlsx` |
| 10 | Logical data model | `figures/er_diagram.png`; workbook sheet 07 |
| 11 | KPI catalogue and review (46) | workbook sheet 08; `outputs/VELOOP_KPI_Review.xlsx` |
| 12-13 | Profiling and data quality audit | `outputs/VELOOP_Data_Profiling_and_DQ_Audit.xlsx`, `outputs/evidence/` |
| - | KPI validation, ER validation, test evidence | `outputs/VELOOP_KPI_Validation.xlsx`, `outputs/VELOOP_ER_Validation.xlsx`, `docs/TEST_EVIDENCE.md` |
| 14 | Dashboard blueprint and functional demo | `figures/wireframe_*.png`, `dashboard/index.html` |
| 15-17 | Handoff, pipeline, governance | workbook sheets 12-13; `figures/pipeline_architecture.png` |
| 18-20 | Readiness scorecard, assumptions, backlog | `outputs/VELOOP_Readiness_Scorecard.xlsx`; workbook sheets 16-17 |
| - | Presentation | `docs/VELOOP_Handover_Presentation.pptx` |
| - | Production usage guide | `docs/PRODUCTION_USAGE_GUIDE.md` |
| - | Reproducibility evidence | `docs/REPRODUCIBILITY_EVIDENCE.md` |
| - | Response to review feedback (2 rounds) | `docs/REVIEW_RESPONSE.md` |

## Limitations
- Rewards, Wallet, Withdrawals and Referrals datasets were not supplied: those areas are design-only; KPIs for them are verified on generated test data only.
- The behaviour workbook calls itself "Task 02"; it is used as supplementary simulated data.
- Engineering verification is needed for all 18 assumptions (workbook sheet 16). When production data exists, replace simulated data with governed production data; never mix them.
