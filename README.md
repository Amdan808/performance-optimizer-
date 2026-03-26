# Performance Under Pressure — Executable Model

This project started as an observation linking psychology and mathematics:
performance can become sharper and more reliable under high-stakes, time-pressured conditions.

The repository turns that idea into a runnable system you can inspect, simulate, and discuss.

## Origin & intent

The model formalizes a self-observed pattern:

- Output often stays low during long horizons.
- Performance rises steeply near hard deadlines.
- Stakes, perceived consequence, and focus dynamics shape this curve.

The practical goal is intervention design: engineering conditions that trigger peak performance earlier, rather than waiting for last-minute pressure.

## What this repository includes

- **Python backend (FastAPI):** simulation, evaluation, calibration, sensitivity ranking, intervention comparison.
- **TypeScript web app (React + Vite):**
  - `Dashboard` mode for interactive analysis.
  - `Public presentation` mode for non-technical explanation.
- **Conceptual reference:** `performance_model.md` (original model narrative and symbols).

## Repository layout

```text
.
├── performance_model.md          # conceptual model and assumptions
├── src/performance_model/        # backend package
├── tests/                        # backend tests
└── web/                          # frontend dashboard
```

## Model at a glance

The system is two-layer:

- **Layer 1: Output engine** models instantaneous performance `P(t)` from stress, capability, focus, fatigue, and noise.
- **Layer 2: Outcome evaluation** maps accumulated output `Ω` against threshold `θ` into success probability `p`.

This implementation closes key logical gaps from the original conceptual draft:

- Explicit bounded `g(Ω, θ, X)` and `ψ = f(p, ρ, ι)` forms.
- Smooth terminal gate near deadline (no hard discontinuity).
- Integrated distraction-focus coupling (`D_max -> D(S) -> F(S)`).
- Parameter governance split between scenario inputs and fitted parameters.

## Tech stack

- Backend: Python 3.12+, FastAPI, Pydantic v2, NumPy, SciPy, Pytest
- Frontend: React 19, TypeScript, Vite, Recharts, `openapi-typescript`

## Quick start

### 1) Start backend

From repository root:

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

Run the API server:

```bash
uvicorn performance_model.main:app --app-dir src --host 127.0.0.1 --port 8000
```

Open API docs:

- Swagger UI: `http://127.0.0.1:8000/docs`
- OpenAPI JSON: `http://127.0.0.1:8000/openapi.json`

### 2) Start frontend

From `web/`:

```bash
npm install
npm run dev
```

The web app defaults to API base URL `http://127.0.0.1:8000`.

Open the app and choose either:

- `Dashboard` for simulation, sensitivity, and intervention testing.
- `Public presentation` for a concise narrative suitable for GitHub/public readers.

## API overview

- `GET /health` — health check
- `GET /parameter-governance` — fixed vs fitted parameter roles
- `POST /simulate` — run deterministic or stochastic scenario simulation
- `POST /evaluate` — compute outcome probability from `omega`, `theta`, and extrinsic factors
- `POST /calibrate` — fit allowed behavioral parameters (`alpha`, `beta`, `lambda0`, `gamma`)
- `POST /sensitivity` — rank local parameter impact
- `POST /compare-interventions` — baseline vs intervention deltas

## Model governance notes

- Scenario inputs (for conditions/introspection): includes values such as `iota`, `rho`, `R_s`, `R_f`.
- Fitted parameters (for calibration): `alpha`, `beta`, `lambda0`, `gamma`.
- Unsupported fit parameters are rejected with HTTP 400 by design.

## Development checks

### Backend

From repository root:

```bash
. .venv/bin/activate
pytest -q
```

### Frontend

From `web/`:

```bash
npm run lint
npm run build
```

## Regenerate frontend API types

If backend request/response schemas change:

```bash
# from repository root
. .venv/bin/activate
PYTHONPATH=src python - <<'PY'
import json
from performance_model.main import app
with open('web/src/api/openapi.json', 'w', encoding='utf-8') as f:
    json.dump(app.openapi(), f, indent=2)
PY

# from web/
npm run generate:api
```

## Notes

- Frontend build is split into `react`, `charts`, and `vendor` chunks (`web/vite.config.ts`) to keep initial payload smaller.
