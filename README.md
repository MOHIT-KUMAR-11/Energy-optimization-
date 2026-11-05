# Energy Optimization — AI-Based Smart Energy Management System

A college project: households and public buildings enter or upload an electricity
reading, and the app explains consumption, flags wastage, predicts future usage,
and recommends concrete actions to cut costs — all from the user's own numbers.

**Stack:** plain HTML/CSS/JS frontend + a pure Python standard-library backend
(`http.server`, `sqlite3`, `hashlib`). **Zero pip installs required.**

## Run it

```bash
python run.py
```

Then open **http://127.0.0.1:8000/** in a browser. Demo Mode auto-logs you into
a pre-seeded demo account (`demo@energyai.local` / `demo1234`) with 5 months of
realistic history already loaded, so the dashboard is populated from the first
click — no setup needed for a live presentation.

To use a different port: `PORT=5000 python run.py`.

## What's implemented

- **Analyze Energy** (`/analyze`): manual meter-reading entry, or upload a
  photo (`/upload`) with an editable "verify before analysis" step.
- **AI analysis** (`/insights`): trend vs. previous reading, per-person
  consumption level, wastage heuristic — all numbers-backed, not generic tips.
- **Recommendations** (`/recommendations`) and **Optimization** (`/optimization`):
  priority-ranked, data-driven cards and a Today/This Week/This Month action plan.
- **Predictions** (`/predictions`): a transparent least-squares trend line over
  your own reading history (says "insufficient data" below 3 readings — never
  fakes confidence it doesn't have).
- **Dashboard** (`/dashboard`): summary cards, consumption/cost trend charts,
  Energy Score, and a prediction preview.
- **Energy History** (`/history`): full reading table + trend chart.
- **AI Energy Advisor** (`/advisor` + the floating chat widget on every page):
  answers questions using your own latest reading/insights/recommendations.
- **Settings** (`/settings`): profile view + "Reset Demo Data".

## Project structure

```
energy-optimization/
├── backend/
│   ├── server.py          # http.server routing + all API endpoints
│   ├── db.py              # sqlite3 connection + schema init
│   ├── repository.py      # all SQL lives here
│   ├── auth.py            # pbkdf2 password hashing + cookie sessions
│   ├── config.py
│   ├── engine/            # calculations, analysis, recommendations,
│   │                       # optimization, prediction, energy_score, advisor, context
│   ├── ocr/extractor.py   # OCR seam (honest fallback — see below)
│   └── utils/             # validators, multipart parser, demo-data seeder
├── frontend/
│   ├── *.html             # one file per page (11 pages total)
│   └── static/{css,js}    # design system + api client + per-page logic
└── database/schema.sql    # sqlite schema (users, energy_readings, bills,
                            # uploaded_documents, appliances, recommendations,
                            # predictions — energy_readings rows ARE the history)
```

## Design decisions worth knowing about

- **No Flask/Django, no pip installs.** The whole backend is `http.server` +
  `sqlite3` + `hashlib`, matching the "standard library only" constraint used
  across this project's other assignments. `backend/server.py` hand-rolls
  routing, JSON responses, cookie sessions, and multipart file uploads.
- **SQLite, not MySQL, by default.** Zero-config for local dev/demo. Set
  `DATABASE_URL` if you wire in MySQL later (`repository.py`/`db.py` are the
  only files that would need a `pymysql`-based connector swapped in).
- **No fake AI.** Predictions use a real (if simple) least-squares trend line
  over your own data, explicitly labeled as a statistical estimate and
  refusing to predict below 3 data points. Recommendations are a rule engine
  driven by the actual reading, not a static tip list. `engine/context.py` is
  the single seam where a trained ML model could later replace the rule-based
  analysis/prediction without touching any API or frontend code.
- **No fake OCR.** This build has no OCR engine (Tesseract) installed, so
  `ocr/extractor.py` honestly reports that and routes the user to manual entry
  with "please verify before analysis" — exactly the required fallback
  behavior. Install `pytesseract` + `Pillow` + the Tesseract binary later and
  it activates automatically, no code changes needed.
