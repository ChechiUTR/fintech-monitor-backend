**# Advanced Financial Market & Crypto Asset Monitor — Backend

> Real-time data ingestion, concurrent processing, and risk-indicator engine for the Advanced Financial Market & Crypto Asset Monitor.

This is the **backend / processing** repository. It is one half of a two-repository project. The user-facing interface and dashboards live in the companion repository, [`fintech-monitor-frontend`](#related-repositories).

---

## Project Description

The Advanced Financial Market & Crypto Asset Monitor tracks **50 stocks and crypto assets in near real time**, computes volatility and risk indicators, and surfaces the results through live dashboards. The system runs **without a database**: market data is pulled directly from public APIs and held in in-memory Python structures (dictionaries, lists, and pandas DataFrames), then pushed to a Google Sheet that powers the visualization layer.

**This repository contains the backend**, which is responsible for:

- Fetching live market data from free, key-free public APIs.
- Running concurrent threads to query all 50 assets simultaneously, simulating a high-frequency environment.
- Calculating volatility and financial-risk indicators in parallel.
- Writing the processed results to a Google Sheet on a scheduled cadence (this sheet is the data bridge consumed by Looker Studio in the frontend).
- Running as a cloud-hosted, scheduled job on **AWS**.

---

## Related Repositories

This project is split into two coordinated repositories. They are designed to work together and share the Google Sheet as their integration contract.

| Repository | Responsibility | Link |
|---|---|---|
| **`fintech-monitor-backend`** _(this repo)_ | Data ingestion, concurrent processing, risk calculation, Google Sheet updates, AWS deployment | — |
| **`fintech-monitor-frontend`** | User-facing portal, Looker Studio dashboards, client-side controls, static hosting on AWS | `https://github.com/<org>/fintech-monitor-frontend` |

**Integration boundary:** the backend is the sole writer of the Google Sheet; the frontend (via Looker Studio) is a read-only consumer of it. Neither repo calls the other directly.

---

## Team Members

The same three-person team owns both repositories. Primary focus areas are noted to clarify ownership, but the team collaborates across the full stack.

| Name | GitHub | Primary Focus |
|---|---|---|
| _[Member 1 — Full Name]_ | `@handle` | Backend & Data Engineering Lead |
| _[Member 2 — Full Name]_ | `@handle` | Cloud Infrastructure & DevOps (AWS) |
| _[Member 3 — Full Name]_ | `@handle` | Frontend & Visualization Engineer |

> Replace the placeholders above with real names and GitHub handles before the first sprint.

---

## Defined Objectives

The backend is responsible for the following deliverables. Each maps to one of the project's three integration pillars.

### Concurrency & High Performance (CAR)
- Implement concurrent threads (`threading` / `concurrent.futures`) to query the prices of **50 assets at once** rather than sequentially.
- Compute volatility indicators **in parallel** to emulate a high-frequency monitoring environment.
- Handle API rate limits, timeouts, and partial failures gracefully so one slow asset never blocks the batch.

### Cloud Services (SEN)
- Deploy the processing job to **AWS** and run it on a recurring schedule (no manual execution).
- Manage credentials and configuration through AWS-native secrets/parameter storage — no secrets committed to the repo.
- Emit logs and run metrics to a centralized AWS location for observability.

### Data Pipeline → Visualization (VDB)
- Structure all results in clean pandas DataFrames with a stable schema.
- Write/refresh the **Google Sheet** that Looker Studio reads, on every scheduled run.
- Guarantee the sheet schema stays backward-compatible so frontend dashboards never break.

### Definition of Done
- 50 assets refreshed concurrently within the target run window.
- Volatility/risk indicators computed and validated for accuracy.
- Google Sheet updated automatically on schedule with zero manual steps.
- Job runs unattended on AWS with logging and alerting in place.

---

## Technology Stack

**Language & Core Libraries**
- Python 3.11+
- [`yfinance`](https://pypi.org/project/yfinance/) — Yahoo Finance market data (free, no API key)
- `requests` — CoinGecko public API (free, no auth for basic data)
- `pandas`, `numpy` — data structuring and indicator math
- `threading` / `concurrent.futures` — parallel asset querying (CAR requirement)
- `gspread` + Google Sheets API — writing the data bridge sheet

**Cloud & Infrastructure (AWS)**
- **AWS Lambda** or **EC2** — runtime for the processing job
- **Amazon EventBridge** — scheduled triggering of the job
- **AWS Secrets Manager** / **Parameter Store** — Google service-account credentials and config
- **Amazon CloudWatch** — logs, metrics, and alarms
- **IAM** — least-privilege roles for the job

**Tooling**
- `python-dotenv` for local development config
- `pytest` for unit tests
- `black` / `ruff` for formatting and linting

---

## Repository Structure

```
fintech-monitor-backend/
├── src/
│   ├── ingestion/        # API clients for yfinance and CoinGecko
│   ├── concurrency/      # threading / worker-pool logic (CAR)
│   ├── indicators/       # volatility & risk calculations
│   ├── sheets/           # Google Sheets writer (data bridge)
│   └── main.py           # entry point / orchestration
├── infra/                # AWS deployment config (IaC, scheduling)
├── tests/                # unit tests
├── config/               # asset lists, schema definitions
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Getting Started

```bash
# 1. Clone and enter the repo
git clone https://github.com/<org>/fintech-monitor-backend.git
cd fintech-monitor-backend

# 2. Create a virtual environment and install dependencies
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

# 3. Add local config (Google service-account credentials, asset list)
cp config/example.env .env   # then fill in values

# 4. Run the processor locally
python src/main.py
```

> **Note:** the asset list (the 50 tickers/coins) lives in `config/`. Keep it in sync with the frontend so dashboards and data always match.**
