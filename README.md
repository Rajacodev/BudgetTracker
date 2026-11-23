# Budget Tracking Application (ML)

Full-stack personal finance app: Flask + MongoDB API, React dashboard, and two real scikit-learn/numpy ML features
(automatic transaction categorisation, monthly expense forecasting).

## Quick start

**1. Backend** (Python 3.10+, MongoDB running locally or an Atlas URI)

    cd backend
    python -m venv venv && source venv/bin/activate     # Windows: venv\Scripts\activate
    pip install -r requirements.txt
    cp .env.example .env                                 # Windows: copy .env.example .env
    python -m ml.train_model                             # optional: trains + prints metrics (also auto-runs on first use)
    python -m scripts.seed_demo                          # optional: demo user + 12 months of data
    python app.py                                        # http://localhost:5000/api/health

**2. Frontend** (Node 18+)

    cd frontend
    npm install
    npm run dev                                          # http://localhost:5173 (proxies /api to :5000)

Demo login after seeding: `demo@budget.app` / `Demo@1234`.

**3. Tests**

    cd backend
    pytest          # ML + analytics tests run anywhere; tests/test_api.py needs MongoDB and skips if it is unreachable

## Features
- **Auth:** registration/login, bcrypt password hashing, JWT; every query is scoped to the signed-in user.
- **Transactions:** CRUD, search/filter/pagination, CSV import (`date,description,amount[,type][,category]`;
  without a `type` column negative = expense, positive = income).
- **ML categorisation:** TF-IDF (word + character n-grams) + Logistic Regression. Low-confidence predictions are flagged
  "Review"; confirming/editing a category marks it as user-verified, and **Retrain** folds those rows back into the model.
- **ML forecasting:** rolling-origin backtest of last-month / 3-month average / linear trend / Holt smoothing; the lowest-MAE
  model forecasts the next 1-12 months with an approximate range. Only completed months are used.
- **Budgets, analytics, insights:** per-category monthly limits with progress, trends and breakdowns, and rule-based insights
  (budget overruns, category spikes, savings rate, concentration, next-month forecast).

## API
| Area | Endpoints (all but register/login need `Authorization: Bearer <token>`) |
|---|---|
| Auth | `POST /api/auth/register`, `POST /api/auth/login`, `GET /api/auth/me` |
| Transactions | `GET/POST /api/transactions`, `PUT/DELETE /api/transactions/<id>`, `POST /api/transactions/import` |
| ML | `POST /api/ml/categorize`, `GET /api/ml/forecast?months=3`, `GET /api/ml/metrics`, `POST /api/ml/retrain` |
| Budgets | `GET /api/budgets?month=YYYY-MM`, `POST /api/budgets` (upsert), `PUT/DELETE /api/budgets/<id>` |
| Analytics | `GET /api/analytics/summary`, `/by-category`, `/trends` |
| Insights | `GET /api/insights` |

## Honest notes on the ML (worth knowing for interviews)
- The seed dataset (`dataset/seed_transactions.csv`) is **synthetic**, generated from real merchant names with bank-statement
  templates (`python -m ml.generate_dataset`). A random train/test split scores 100% because the model memorises merchants.
  The app therefore reports a **merchant-held-out** split: about **56% accuracy / 0.58 macro-F1 on merchants never seen in
  training**, but about **97% accuracy on the ~54% of cases where confidence is at least 50%**. That is why uncertain cases go
  to user review instead of being trusted blindly.
- Real accuracy depends on real data: import your own statements, confirm categories, retrain.
- Retraining is **global** (all users' confirmed descriptions feed one shared model). Per-user models would be the next step
  for a real multi-tenant product.
- Forecasts with under 12 months of history cannot capture seasonality; the API returns a reliability label and the UI shows
  the backtest table and range rather than a single confident number.

## Layout
    backend/   app.py config.py db.py  routes/ services/ models/ utils/ ml/ scripts/ tests/
    frontend/  src/{pages,components,context,hooks,services,utils}
    dataset/   seed_transactions.csv

## Progress
- [x] 1 Architecture  - [x] 2 Flask + MongoDB  - [x] 3 JWT auth  - [x] 4 Transaction APIs  - [x] 5 Categorisation model
- [x] 6 Forecasting  - [x] 7 React pages  - [x] 8 Frontend/backend integration  - [x] 9 Analytics, budgets, insights
- [x] 10 Tests (see verification status in the chat summary)
