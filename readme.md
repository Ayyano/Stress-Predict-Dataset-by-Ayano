# Stress Prediction Research Project

![Stress prediction research workflow](assets/stress-project-banner.svg)

This repository contains a Jupyter-based machine-learning workflow for exploring stress prediction from synthetic wearable-style and questionnaire data. It also includes two synthetic dataset artifacts and a separately packaged local desktop assessment application, NeuroPulse AI.

The notebook, datasets, and desktop app are distinct deliverables. The app's source tree is stored inside `NeuroPulse_AI_Pathology_Pro_v9.zip`; it is not installed as a root-level web service.

> **Research and privacy notice:** The supplied datasets are synthetic and intended for education, prototyping, and pipeline testing. Model output and generated reports are not medical diagnoses or clinically validated advice. The desktop app can store manually entered assessment data locally; do not enter real patient information unless you have an appropriate, secured environment and authorization.

## Contents

- [Project overview](#project-overview)
- [Features and modules](#features-and-modules)
- [Technology stack](#technology-stack)
- [Architecture](#architecture)
- [Repository structure](#repository-structure)
- [Prerequisites](#prerequisites)
- [Installation and setup](#installation-and-setup)
- [Running the research notebook](#running-the-research-notebook)
- [Running the NeuroPulse desktop app](#running-the-neuropulse-desktop-app)
- [Environment variables](#environment-variables)
- [Notebook workflow and outputs](#notebook-workflow-and-outputs)
- [Desktop app: frontend and backend](#desktop-app-frontend-and-backend)
- [Desktop app API](#desktop-app-api)
- [Dataset and database structure](#dataset-and-database-structure)
- [Reports and PDF generation](#reports-and-pdf-generation)
- [UI, animation, and audio](#ui-animation-and-audio)
- [Security and privacy](#security-and-privacy)
- [Validation and error handling](#validation-and-error-handling)
- [Testing](#testing)
- [Build and deployment](#build-and-deployment)
- [Troubleshooting](#troubleshooting)
- [Limitations and future work](#limitations-and-future-work)

## Project overview

The research workflow in [`finally.ipynb`](finally.ipynb) loads or creates a 600-row synthetic dataset, engineers baseline-relative heart-rate and respiration features, compares classifiers, and produces evaluation visualizations and PDF reports. The companion [`synthetic_stress_predict_600_participants.csv`](synthetic_stress_predict_600_participants.csv) contains 17 columns and balanced binary labels.

[`NeuroPulse_AI_Pathology_Pro_v9.zip`](NeuroPulse_AI_Pathology_Pro_v9.zip) contains a separate local-first assessment application. After extraction, it combines a FastAPI/Python inference and reporting backend with a static HTML/CSS/JavaScript interface, SQLite storage, and an optional Electron desktop shell.

## Features and modules

### Research notebook

- Finds a suitable CSV or generates the deterministic 600-participant dataset.
- Resolves dataset columns, handles missing values, audits outliers, and engineers changes relative to resting heart rate and respiration.
- Compares multiple scikit-learn models and optional XGBoost/LightGBM models using stratified splits and cross-validation.
- Generates model comparison tables, cohort analysis, plots, and PDF reports.
- Includes notebook cells for a Dash-based interactive dashboard and an optional local Ollama integration.

### NeuroPulse app (source inside the ZIP)

- A guided 14-step assessment, review screen, results dashboard, live report, and saved-records view.
- Full multimodal and wearable-only model predictions, reference comparisons, and participant-specific prior-visit summaries.
- Local SQLite persistence and downloadable PDF reports.
- A paced-breathing guide, three visual themes, and optional synthesized ambient audio.
- Browser-based operation or an Electron window on macOS, Linux, and Windows.

## Technology stack

| Area | Technologies found in the repository |
| --- | --- |
| Research | Python 3 / Jupyter, NumPy, pandas, SciPy, scikit-learn, XGBoost, LightGBM |
| Notebook charts and reports | Matplotlib, Seaborn, Plotly, ReportLab, Dash |
| Desktop app backend | Python, FastAPI, Uvicorn, Pydantic, SQLite, NumPy, pandas, SciPy |
| Desktop app models | scikit-learn, with optional XGBoost and LightGBM; a NumPy/SciPy fallback is implemented |
| Desktop app frontend | HTML, CSS, vanilla JavaScript, Canvas charts, Web Audio API |
| Desktop shell | Electron and electron-builder |

There is no root-level `package.json` or Python dependency manifest. The desktop app has its own files inside the ZIP at `neuropulse-ai/backend/requirements.txt` and `neuropulse-ai/electron/package.json`.

## Architecture

```text
Repository root
├── finally.ipynb ─────────────── research, model evaluation, charts, and notebook reports
├── synthetic_stress_predict_600_participants.csv
├── synthetic_stress_predict_raw_data_S001_S600.zip
└── NeuroPulse_AI_Pathology_Pro_v9.zip
    └── neuropulse-ai/
        ├── frontend/ ─────────── static assessment UI
        ├── backend/ ──────────── FastAPI, training/inference, SQLite, PDF generation
        ├── electron/ ────────── optional desktop window and packaging
        └── tests/ ────────────── API and end-to-end checks
```

The notebook is an independent analysis pipeline. The desktop app initializes its own model engine at backend startup using a CSV in its configured data directory, if available, or otherwise generates its built-in synthetic dataset. It does not import the notebook as a Python module.

## Repository structure

```text
.
├── assets/
│   └── stress-project-banner.svg
├── finally.ipynb
├── live preview and download clinical pdfs reports
├── NeuroPulse_AI_Pathology_Pro_v9.zip
├── synthetic_stress_predict_600_participants.csv
├── synthetic_stress_predict_raw_data_S001_S600.zip
└── readme.md
```

The extensionless `live preview and download clinical pdfs reports` file is a notebook-style code export (it begins with an IPython `!pip` command), not a standard standalone Python module. The raw-data ZIP contains per-participant Empatica E4-style signal folders (`ACC`, `BVP`, `EDA`, `HR`, `IBI`, `TEMP`, tags, and participant info); the separate root CSV is the 17-column processed dataset. The archive's README states that its file layout is modeled after the open [Stress-Predict-Dataset](https://github.com/italha-d/Stress-Predict-Dataset); the values in this repository remain synthetic.

After extracting the app ZIP, its source layout is:

```text
neuropulse-ai/
├── backend/
│   ├── dataset.py
│   ├── ml_engine.py
│   ├── pdf_report.py
│   ├── requirements.txt
│   ├── server.py
│   └── storage.py
├── electron/
│   ├── main.js
│   ├── package.json
│   └── preload.js
├── frontend/
│   ├── app.js
│   ├── audio.js
│   ├── charts.js
│   ├── index.html
│   └── styles.css
├── tests/
│   └── test_e2e.py
├── start_mac_linux.sh
└── start_windows.bat
```

The package also includes platform launcher scripts and app icons.

## Prerequisites

- Python 3.10 or later is recommended for the notebook. Its recorded kernel is Python 3.10; the notebook's first cell installs some, but not all, of its imports.
- Python 3.9 or later for the NeuroPulse app, as stated by its included setup instructions and launch scripts.
- Node.js 18 or later only if using the Electron shell or building its installers.
- A modern browser for the web interface. Internet access is needed for the initial Python/npm package installs; the desktop app itself defaults to a loopback address.

## Installation and setup

Clone the repository and enter it:

```bash
git clone https://github.com/Ayyano/Stress-Predict-Dataset-by-Ayano.git
cd Stress-Predict-Dataset-by-Ayano
```

### Set up a notebook environment

On macOS/Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install jupyterlab numpy pandas scipy scikit-learn matplotlib seaborn reportlab xgboost lightgbm dash plotly openpyxl
```

On Windows PowerShell, create and activate the environment with:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install jupyterlab numpy pandas scipy scikit-learn matplotlib seaborn reportlab xgboost lightgbm dash plotly openpyxl
```

The notebook's first setup cell also runs `pip install -q reportlab xgboost lightgbm`. Run notebook cells from top to bottom in the environment where the dependencies are installed.

## Running the research notebook

With the notebook environment activated:

```bash
jupyter lab finally.ipynb
```

The notebook searches common runtime locations for a CSV and falls back to generating its synthetic dataset when one is not found. On a local checkout, confirm the selected CSV in the notebook output if you have additional CSV files in the working tree. Notebook-generated tables, figures, and reports are written under `results/` (or `/content/results` in a Colab-style environment). The later interactive dashboard cell writes report artifacts under `clinical_reports/`.

The notebook includes a cell that starts a Dash server and attempts to create a public Cloudflare quick tunnel. See [Security and privacy](#security-and-privacy) before executing that cell.

## Running the NeuroPulse desktop app

Extract the bundled source:

```bash
unzip -q NeuroPulse_AI_Pathology_Pro_v9.zip -d neuropulse-app
cd neuropulse-app/neuropulse-ai
```

On macOS/Linux, run the launcher:

```bash
./start_mac_linux.sh
```

On Windows, run `start_windows.bat` from the extracted `neuropulse-ai` directory. The launchers create a local `.venv` and install `backend/requirements.txt` on first launch, then start the backend at `http://127.0.0.1:8765`. If Electron's dependencies are installed, the launcher opens the desktop window instead of the browser.

For browser mode without a launcher, install the backend dependencies and start the server:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r backend/requirements.txt
python -m backend.server
```

Then open `http://127.0.0.1:8765`. On Windows, use the equivalent `.venv\Scripts\activate` command.

To run the Electron shell directly, first set up the Python environment above, then:

```bash
cd electron
npm install
npm start
```

## Environment variables

The app backend reads these variables; all are optional:

| Variable | Default | Purpose |
| --- | --- | --- |
| `NEUROPULSE_HOST` | `127.0.0.1` | Backend bind address. |
| `NEUROPULSE_PORT` | `8765` | Backend port. |
| `NEUROPULSE_DATA_DIR` | `neuropulse-ai/local_data` | Directory for the generated/loaded dataset and SQLite records. The Electron shell sets this to its application user-data directory. |

The notebook's interactive dashboard cell separately reads `OLLAMA_URL` (default `http://localhost:11434/api/generate`) and `OLLAMA_MODEL` (default `mistral`) for its optional local Ollama integration. These are not backend app settings. No API keys or credentials are required by the checked-in project.

Example:

```bash
NEUROPULSE_PORT=8766 python -m backend.server
```

## Notebook workflow and outputs

The notebook is organized into data handling and preprocessing, split/cross-validation and ablation, model fitting/tuning and ensembles, visual evaluation, and PDF reporting. It includes a further multi-agent-style orchestration example and Dash dashboard code. The optional local Ollama calls target the configured Ollama endpoint; they are not required for the packaged app's FastAPI inference.

The notebook evaluates synthetic data, so its metrics describe this generated sample and experimental pipeline only. Do not interpret them as evidence of real-world or clinical predictive performance.

## Desktop app: frontend and backend

### Frontend

`frontend/index.html` defines the UI views: welcome, 14-step assessment questionnaire, review, results, live report, and saved reports. `frontend/app.js` handles navigation, draft autosave, input validation, API calls, and view rendering. `frontend/charts.js` draws the dashboard charts; `frontend/styles.css` provides responsive styling, theme variants, animations, and reduced-motion handling; `frontend/audio.js` synthesizes optional sound locally in the browser.

### Backend and model

`backend/server.py` serves the static frontend and FastAPI routes. At startup, `backend/ml_engine.py` loads a compatible CSV from the data directory or generates the deterministic 600-row dataset, prepares features, and trains models. The full feature set contains 23 features (19 wearable-derived and 4 questionnaire features); a separate wearable-only model is also used. The preferred scikit-learn path includes a soft-voting ensemble and model comparisons; XGBoost and LightGBM are optional. If scikit-learn is unavailable, the module includes a NumPy/SciPy model fallback.

`backend/dataset.py` implements the generated dataset and column matching. `backend/storage.py` persists assessments in SQLite. `backend/pdf_report.py` builds the downloadable report using ReportLab and Matplotlib.

The Electron shell in `electron/main.js` starts the local Python backend and loads it in a BrowserWindow. `electron/preload.js` exposes the PDF-save bridge with context isolation enabled and Node integration disabled.

## Desktop app API

The backend exposes interactive OpenAPI documentation at `http://127.0.0.1:8765/api/docs`.

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/api/health` | Backend status, model mode, loaded-model count, and dataset source. |
| `POST` | `/api/analyze` | Validate input, run inference, attach prior visits matching the participant name, and optionally save the assessment. |
| `POST` | `/api/report/pdf` | Generate a PDF from a result object. |
| `GET` | `/api/records` | List up to 100 saved assessment summaries. |
| `GET` | `/api/records/{record_id}` | Retrieve a saved assessment and its matching prior-visit summary. |
| `DELETE` | `/api/records/{record_id}` | Delete one saved assessment. |

There is no login, API token, or authorization layer in the current backend. `/api/analyze` accepts JSON fields `participant_name`, `age`, `gender`, `pss_score`, `stai_state`, `stai_trait`, `hr`, `stroop_hr`, `interview_hr`, `hypervent_hr`, `rr`, `stress_rr`, `mean_eda`, `skin_temp`, and optional boolean `save_record` (defaults to `true`). Values are range-checked by the Pydantic request model; invalid input returns HTTP 422.

Example health check:

```bash
curl http://127.0.0.1:8765/api/health
```

Example assessment using fictional values:

```bash
curl -X POST http://127.0.0.1:8765/api/analyze \
  -H 'Content-Type: application/json' \
  -d '{
    "participant_name": "Example_Research_001",
    "age": 29,
    "gender": "Female",
    "pss_score": 18,
    "stai_state": 42,
    "stai_trait": 40,
    "hr": 72,
    "stroop_hr": 81,
    "interview_hr": 87,
    "hypervent_hr": 84,
    "rr": 14,
    "stress_rr": 19,
    "mean_eda": 2.1,
    "skin_temp": 33.1,
    "save_record": false
  }'
```

The analysis response separates measured values, derived features, backend predictions, and interpretive observations. A missing record returns HTTP 404. PDF-generation failures are returned as HTTP 500.

## Dataset and database structure

The processed CSV has 600 synthetic records, 17 columns, and 300 rows per label. Its schema is: participant ID and assigned name; age and gender; PSS, STAI-State, and STAI-Trait scores; resting and task heart rates; resting and stress respiration rates; mean EDA; skin temperature; label; and timestamp. The generator creates six 100-row demographic cohorts.

The separate raw-data ZIP contains `Raw_data/S001` through `Raw_data/S600`, with synthetic wearable-style signal files. The NeuroPulse app creates its own dataset in `local_data/dataset` by default; it does not automatically ingest the root archive.

The app creates a local SQLite database named `neuropulse_records.sqlite3` with an `assessments` table. Its columns are `id`, `created_at`, `participant_name`, `anon_id`, `age`, `gender`, `verdict`, `stress_probability`, `triage_severity`, `payload_json`, and `result_json`. Browser use stores this by default under `neuropulse-ai/local_data/`; Electron sets the data directory under the app's OS-specific user-data path. Values are stored as ordinary local SQLite data, not encrypted by the app.

## Reports and PDF generation

The notebook creates evaluation figures and research/viva PDFs in its output directories. The desktop app's `POST /api/report/pdf` endpoint turns an analysis result into a downloadable PDF with assessment values, model interpretation, charts, reference comparisons, and a non-diagnostic disclaimer. `frontend/app.js` calls this endpoint; the Electron bridge can present a native save dialog for the returned PDF bytes.

## UI, animation, and audio

The desktop UI includes Sage, Mist, and Evening themes, responsive layouts, animated decorative elements, Canvas charts, and a guided breathing modal. CSS includes a `prefers-reduced-motion` override. The optional sound dock offers rain, ocean, felt piano, and singing-bowl modes generated with the browser Web Audio API; it starts muted until enabled and includes a volume control.

## Security and privacy

- The desktop backend defaults to `127.0.0.1` and the app code does not require cloud inference. This is a local-use design, not a security boundary.
- The backend has no authentication or authorization, enables permissive CORS, and stores participant names, measurements, and report data in a local SQLite database without application-level encryption. Do not bind it to a public or untrusted network; changing `NEUROPULSE_HOST` away from loopback can expose the unauthenticated API.
- The browser frontend autosaves an in-progress assessment draft in `localStorage`; clear the browser's site data when using a shared device.
- The notebook's final interactive dashboard cell starts a server bound to `0.0.0.0` and attempts to download/run `cloudflared` to create a public quick tunnel. Anyone with the resulting URL may be able to access the dashboard for the duration of the tunnel. Avoid that cell with private data, and inspect its behavior before running it in any environment.
- The repository's supplied records are synthetic. Use only data you are authorized to process, and secure the host filesystem and any exported reports.

## Validation and error handling

The desktop API validates assessment values and required fields with Pydantic. Invalid requests return HTTP 422; missing record IDs return HTTP 404. The frontend displays validation and request status in the UI, and the backend logs errors such as PDF-generation failures. The research notebook is an interactive workflow rather than a separately packaged service, so cell errors appear in notebook output.

## Testing

The app bundle includes an end-to-end test script. From the extracted `neuropulse-ai` directory, install backend requirements and run:

```bash
python3 -m pip install -r backend/requirements.txt
python3 -m pip install httpx
python3 tests/test_e2e.py
```

The test uses FastAPI's `TestClient` (which requires `httpx`); `httpx` is not listed in the app's runtime requirements. The script exercises health and analysis endpoints, input validation, PDF responses, local record operations, and frontend asset serving. It writes verification PDF files under `tests/`. There is no root-level automated test configuration.

## Build and deployment

The Electron project defines these npm scripts in `electron/package.json`:

| Command (from `neuropulse-ai/electron/`) | Result |
| --- | --- |
| `npm start` | Run the Electron app. |
| `npm run build:win` | Build Windows NSIS and MSI targets. |
| `npm run build:mac` | Build macOS DMG and directory targets. |
| `npm run build:all` | Build for macOS and Windows. |

Build example:

```bash
cd electron
npm install
npm run build:win
```

The configured output directory is `neuropulse-ai/dist/`. The builder packages the backend, frontend, and assets, but does not bundle a Python interpreter or a prepared Python environment; a compatible Python runtime and backend dependencies must be available to the packaged app. The repository does not include a production deployment configuration for hosting the API remotely. Keep the backend on loopback unless authentication, access controls, transport security, and data protections are added.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Backend does not start | Confirm Python 3.9+ is available, dependencies installed, and port `8765` is free; override with `NEUROPULSE_PORT` if needed. |
| Browser reports backend offline | Wait for model initialization, then check `http://127.0.0.1:8765/api/health` and the backend terminal output. |
| Notebook import errors | Activate the notebook environment and install the listed notebook dependencies; restart the Jupyter kernel afterward. |
| Dataset differs from expected | Check which CSV the notebook selected; it searches available runtime paths and may use a user-provided CSV instead of generating its fallback. |
| Electron cannot find backend | Confirm Python and backend dependencies are installed and that the desktop app can bind to the configured local port. |
| Ambient sound is silent | Enable audio from the sound dock; browsers require a user gesture before starting Web Audio. |

## Limitations and future work

- Dataset records are synthetic and class-balanced; they are not a substitute for representative, consented, clinically governed data.
- The notebook and app train/evaluate on synthetic data. Reported prediction probabilities and metrics are educational outputs, not validated clinical conclusions.
- The desktop app has no user authentication, encryption-at-rest, multi-user access controls, or hardened remote deployment setup.
- The app's prior-visit lookup is based on a case-insensitive, trimmed participant-name match; it does not verify identity.
- The notebook's optional Ollama and public-tunnel paths require separate local/runtime setup. The tunnel path can expose the dashboard.
- No formal roadmap or clinical validation plan is included in the repository.
