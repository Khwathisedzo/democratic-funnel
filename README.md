# The Democratic Funnel
**Where local democracy leaks before 4 November 2026** — Team UL, DIRISA Student Datathon Challenge 2026

We decompose each municipality's participation shortfall into a **registration leak** (eligible adults not registered)
and a **turnout leak** (registered voters not voting), forecast 2026 turnout, and explain the factors associated
with each municipality's risk, delivered through an interactive dashboard.

```
Adults who may vote  ->  People who registered  ->  People who voted
        E                        R                       V
          \___ registration leak ___/ \____ turnout leak ____/
```

---

## Contents
1. [Repository layout](#repository-layout)
2. [Running the dashboard (quick start)](#running-the-dashboard-quick-start)
3. [Running the dashboard, step by step](#running-the-dashboard-step-by-step)
4. [Troubleshooting](#troubleshooting)
5. [Deploying online (Render)](#deploying-online-render)
6. [The notebooks](#the-notebooks)
7. [Where things live](#where-things-live)
8. [Reproducing the results](#reproducing-the-results)
9. [Data and privacy](#data-and-privacy)

---

## Repository layout
```
web/            THE DASHBOARD - Flask + HTML, Chart.js charts, Leaflet map   <- run the app from here
  app.py          the Flask server (routes: /, /api/data, /api/geo, /healthz)
  templates/      index.html (all dashboard pages)
  static/         css/, js/app.js, fonts/, img/ (logos, team photos)
  requirements.txt  only what the dashboard needs (flask, gunicorn, numpy, pandas)
app/artifacts/  the two data files the dashboard reads (written by 09_dashboard_prep, committed to Git)
app/app.py      the OLD Streamlit version of the dashboard - kept for reference, not used any more
src/            shared code for the notebooks: config (paths), io_utils (robust readers), privacy
notebooks/      one notebook per pipeline stage, run in numeric order (00_setup first)
docs/           sources.csv - every dataset, URL, access date, snapshot date
data/           raw -> interim -> processed   (not in Git; lives on the shared Drive)
models/         trained models                (not in Git)
reports/figures charts for slides             (not in Git)
render.yaml     one-click deploy of the dashboard to Render
```

---

## Running the dashboard (quick start)
The dashboard lives in **`web/`** and is started from inside that folder. It only needs the two files in
`app/artifacts/`, which are already in the repo, so **no data download, Google Drive or notebooks are needed to run it**.

**Windows (PowerShell)**
```powershell
git clone https://github.com/Thato20-M/democratic-funnel.git
cd democratic-funnel\web
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python app.py
```

**macOS / Linux**
```bash
git clone https://github.com/Thato20-M/democratic-funnel.git
cd democratic-funnel/web
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python app.py
```

Then open **http://localhost:5000** in your browser. Stop the server with `Ctrl + C`.

---

## Running the dashboard, step by step

### 1. Requirements
- **Python 3.10 or newer** (Render uses 3.12.7). Check with `python --version`.
- **Git**, to clone the repo (or download the ZIP from GitHub and unzip it).
- An internet connection in the browser: Chart.js and Leaflet (and the map tiles) load from a CDN.

### 2. Get the code
```bash
git clone https://github.com/Thato20-M/democratic-funnel.git
cd democratic-funnel
```
Already cloned? Get the newest version with `git pull`.

### 3. Go into the `web` folder
```bash
cd web
```
All the commands below are run **from `web/`**. (The app finds `../app/artifacts/` on its own, relative to
`web/app.py`, so it must stay inside the repo next to the `app/` folder.)

### 4. Create and activate a virtual environment (recommended)
| System | Create | Activate |
|---|---|---|
| Windows PowerShell | `python -m venv .venv` | `.venv\Scripts\Activate.ps1` |
| Windows cmd | `python -m venv .venv` | `.venv\Scripts\activate.bat` |
| macOS / Linux | `python3 -m venv .venv` | `source .venv/bin/activate` |

When it is active, your prompt starts with `(.venv)`. Next time you only need the **Activate** step.

### 5. Install the dashboard packages
```bash
pip install -r requirements.txt
```
This installs only Flask, gunicorn, numpy and pandas. The heavy notebook packages (geopandas, lightgbm, shap, ...)
are in the root `requirements.txt` and are **not** needed for the dashboard.

### 6. Start the app
```bash
python app.py
```
You should see something like:
```
 * Running on http://127.0.0.1:5000
```
Open **http://localhost:5000**. Other devices on the same Wi-Fi can use `http://<your-computer's-IP>:5000`,
because the app listens on `0.0.0.0`.

### 7. Check it is working
| URL | What you get |
|---|---|
| `http://localhost:5000/` | the dashboard |
| `http://localhost:5000/healthz` | `{"ok": true}` - the server is up |
| `http://localhost:5000/api/data` | the JSON bundle with every number the pages show |
| `http://localhost:5000/api/geo` | the municipal boundaries (GeoJSON) for the map |

Dashboard pages: **Overview, Participation, Map, Municipality profile, Priority list, Voter education,
Recommendations, About, Team.**

### 8. Making changes
- `templates/index.html` edits show on refresh (template auto-reload is on).
- CSS / JS files in `static/` are versioned by their last-change time, so a normal refresh loads the new file.
- Changes to `app.py` need a restart (`Ctrl + C`, then `python app.py`).
- New numbers come from the notebooks: re-run `09_dashboard_prep` to rewrite `app/artifacts/`, then restart the app.

### Running it the production way (optional)
Render starts the app with gunicorn, which works on macOS / Linux only (from `web/`):
```bash
gunicorn app:app --bind 0.0.0.0:5000 --workers 2
```
(From the repo root the equivalent is `gunicorn --chdir web app:app`.) On Windows just use `python app.py`.

---

## Troubleshooting
| Problem | Fix |
|---|---|
| `python` is not recognised (Windows) | Use `py` instead of `python`, or reinstall Python with **"Add python.exe to PATH"** ticked. |
| `Activate.ps1 cannot be loaded because running scripts is disabled` | Run once: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, then activate again. Or use cmd and `.venv\Scripts\activate.bat`. |
| `ModuleNotFoundError: No module named 'flask'` | The virtual environment is not active, or step 5 was skipped. Activate it and run `pip install -r requirements.txt`. |
| `FileNotFoundError: ... app/artifacts/dashboard.csv` | You moved `web/` out of the repo, or the artifacts are missing. Keep the repo structure; run `git pull` or re-run `09_dashboard_prep`. |
| `Address already in use` / port 5000 busy | Another app uses port 5000 (on macOS often AirPlay Receiver). Close it, or change `port=5000` at the bottom of `web/app.py`. |
| Page loads but charts or map are blank | The browser has no internet access to the CDNs, or an old cached page: hard refresh with `Ctrl + F5` (`Cmd + Shift + R` on Mac). |
| `requirements.txt` not found | You are not in `web/`. Run `cd web` first. |

---

## Deploying online (Render)
`render.yaml` is a Render Blueprint: on render.com choose **New -> Blueprint** and pick this repo. It runs
```
build:  pip install -r web/requirements.txt
start:  gunicorn --chdir web app:app --bind 0.0.0.0:$PORT --workers 2 --timeout 60
health: /healthz
```
The online app reads only the committed `app/artifacts/`, so no secrets or Drive access are needed. The blueprint
deploys the `nkululeko` branch; switch `branch:` to `main` once the dashboard work is merged.

---

## The notebooks
Run them **in numeric order**, starting with `00_setup` (in Colab: *Runtime -> Run all*). Each one reads the files
the previous ones made. `RAW`, `INTERIM`, `PROCESSED`, `MODELS` and `FIGURES` are folders defined in `src/config.py`.

| # | Notebook | What it does | Reads | Writes |
|---|---|---|---|---|
| 00 | `00_setup` | Finds/clones the repo, connects Drive, creates folders, installs missing packages (Colab), logs package versions, lists the raw data | - | `environment.json`, `data_inventory.csv` (in the work root) |
| 01 | `01_clean_municipal_2011` / `_2016` / `_2021` | Cleans the IEC local-election results for one year: ward ballot only, one row per voting station, summed to municipality, turnout. The three are identical except `YEAR`. | `RAW/iec_lge/<year>/` | `INTERIM/results_municipal_<year>_clean.csv` |
| 01 | `01_results_lge` | Joins 2011, 2016 and 2021 into one table on `muni_code`; flags municipalities whose boundaries changed in 2016 | the three clean files | `INTERIM/results_lge_2011_2021.csv` |
| 02 | `02_registration` | **Updated (Sep 2026):** scrapes the IEC Voter Registration Statistics portal with Selenium / Playwright (Colab), per province and municipality, and parses registration by age group, gender and ward | IEC website (live) | `iec_clean_demographics.csv`, `iec_fully_cleaned_demographics.csv`, `iec_registered_people_per_ward.csv` (in the current folder) |
| 03 | `03_byelections_npe` | Post-COVID voting signal per municipality from by-elections and the 2024 national election | `RAW/iec_byelections/`, `RAW/iec_npe/` | `INTERIM/recent_signals.csv` |
| 04 | `04_census` | Census 2022: adults who may vote (citizens 18+), young adults (18-29), service and education rates | `RAW/census/` | `INTERIM/census_2022.csv` |
| 05 | `05_geography` | Municipal boundaries and areas, neighbour list, the 2011 -> today boundary mapping, IEC Atlas registration activity | `RAW/boundaries/`, IEC Atlas | `INTERIM/boundaries.gpkg`, `muni_crosswalk_2011.csv`, `neighbours.csv`, ... |
| 06 | `06_master` | The master table: one row per municipality per election (2011, 2016, 2021, 2026) | everything in `INTERIM/` | `PROCESSED/master.csv` |
| 07 | `07_eda` | Exploring the data: registration, turnout, where it is lowest, what goes with low turnout, young people | `PROCESSED/master.csv` | `FIGURES/q0N_*.png` |
| 07a | `07a_turnout_trends` | Turnout change 2011 -> 2021 and where it fell most | `INTERIM/results_lge_2011_2021.csv` | `FIGURES/turnout_*.png` |
| 08 | `08_model` | **Updated (Sep 2026):** 2026 forecast = 2021 turnout + national swing + per-municipality adjustment. Main model: hierarchical (mixed-effects) regression, municipalities within provinces; Ridge and LightGBM for comparison; baselines, out-of-sample check, the 2024 rebound (urban vs rural), factor contributions, worse-than-expected municipalities | `PROCESSED/master.csv` | `PROCESSED/predictions_2026.csv`, `PROCESSED/model_results.csv`, `MODELS/turnout_model.joblib`, `FIGURES/model_*.png` |
| 09 | `09_dashboard_prep` | Builds the two files the dashboard reads, with plain column names; simplifies the boundaries so the map loads fast on a phone | `master.csv`, `predictions_2026.csv`, `boundaries.gpkg` | `app/artifacts/dashboard.csv`, `app/artifacts/municipalities.geojson` |

> **Note on `02_registration`:** the September 2026 version (the scraper) saves its CSVs in the current working
> folder and no longer writes `INTERIM/registration_2026.csv`, which `06_master` reads. Until the scraper output is
> wired into that file, keep a copy of `registration_2026.csv` in `data/interim/` (the earlier version of the
> notebook is in the Git history, commit `857cadc`).

---

## Where things live
| | Location |
|---|---|
| Code | This GitHub repo |
| Data, models, figures | Shared Drive folder `DIRISA_SDC` |
| Dashboard data | `app/artifacts/` in this repo |

`src/config.py` resolves the data folder automatically:
1. `DIRISA_ROOT` environment variable, if set (e.g. the Drive-for-Desktop folder `G:/My Drive/DIRISA_SDC`)
2. Colab -> `/content/drive/MyDrive/DIRISA_SDC`
3. Otherwise -> this repo (so judges can run it with data placed in `data/`)

### Notebook setup
**Colab:** open `notebooks/00_setup.ipynb` and run all. (One-time: add a shortcut to `DIRISA_SDC` in My Drive; for a
private repo, add a `GITHUB_TOKEN` in Colab Secrets.)

**Local (PyCharm / VS Code / Jupyter)** - from the repo root, not `web/`:
```bash
python -m venv .venv
.venv\Scripts\activate                     # macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
# optional: point at the shared data synced by Google Drive for Desktop
setx DIRISA_ROOT "G:\My Drive\DIRISA_SDC"  # macOS/Linux: export DIRISA_ROOT="..."  (restart the terminal after setx)
jupyter lab                                # or open the notebooks in your editor
```

---

## Reproducing the results
1. Download each source listed in `docs/sources.csv` into `data/raw/<raw_folder>/`
2. Run the notebooks in numeric order (see [The notebooks](#the-notebooks))
3. `09_dashboard_prep` refreshes `app/artifacts/`; restart the dashboard to see the new numbers
4. Package versions used are logged in `environment.json` (in the work root) by `00_setup`

## Data and privacy
All data is publicly available; sources and access dates are in `docs/sources.csv`.
Candidate lists contain personal information: personal columns are dropped on load (`src/privacy.py`),
only aggregate counts are kept, and the raw files are never committed.
