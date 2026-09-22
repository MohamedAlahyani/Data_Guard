# CommerceGuard

### Intelligent data-quality monitoring for e-commerce pipelines

[![Status: Planning](https://img.shields.io/badge/status-planning-blue)](#project-status)
[![Python 3.12](https://img.shields.io/badge/python-3.12-3776AB?logo=python&logoColor=white)](#technology-stack)
[![Machine Learning](https://img.shields.io/badge/ML-anomaly%20detection-F7931E)](#machine-learning)
[![License: MIT](https://img.shields.io/badge/license-MIT-green)](#license)

CommerceGuard is an end-to-end **data engineering, machine learning, and software engineering** project that monitors the quality of e-commerce data.

It validates daily customer, product, order, and payment data; separates invalid records; calculates quality metrics; and uses anomaly detection to identify unusual pipeline behavior before unreliable data reaches reports or ML systems.

> This repository is being developed as an introductory machine learning portfolio project. The initial dataset is synthetic, allowing data failures to be introduced and evaluated safely.

---

## Table of contents

- [The problem](#the-problem)
- [Proposed solution](#proposed-solution)
- [Key features](#key-features)
- [Architecture](#architecture)
- [Machine learning](#machine-learning)
- [Dataset](#dataset)
- [Data validation](#data-validation)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [Development roadmap](#development-roadmap)
- [Getting started](#getting-started)
- [Evaluation](#evaluation)
- [API design](#api-design)
- [Testing](#testing)
- [Future improvements](#future-improvements)
- [Learning objectives](#learning-objectives)
- [License](#license)

---

## The problem

E-commerce companies depend on accurate data for revenue reporting, inventory management, payment processing, and customer analytics. A pipeline can complete successfully while still producing unreliable data.

Examples include:

- Duplicate orders that inflate revenue
- Missing customer or product identifiers
- Negative prices or quantities
- Payments that do not match order totals
- Orders linked to nonexistent customers
- Sudden drops in daily order volume
- Unexpected increases in failed payments
- Schema, date-format, or currency changes

Traditional validation rules find known errors, but they may miss new or unexpected behavior.

## Proposed solution

CommerceGuard combines two detection methods:

| Method | Question answered | Example |
| --- | --- | --- |
| Rule-based validation | Is this individual record valid? | A product price must be greater than zero. |
| ML anomaly detection | Is this entire daily batch behaving normally? | Today's order volume is unusually low. |

Invalid records are preserved in a quarantine area with their failure reasons. Valid records are loaded into PostgreSQL, while daily quality metrics are analyzed by an anomaly-detection model.

## Key features

- Ingest daily e-commerce data from CSV or JSON files
- Preserve an unchanged copy of every raw batch
- Clean and standardize values through an ETL pipeline
- Validate customers, products, orders, order items, and payments
- Quarantine rejected records instead of deleting them
- Track quality metrics for every pipeline execution
- Detect unusual batches with an Isolation Forest model
- Compare ML results with statistical and rule-based baselines
- Expose results through a FastAPI backend
- Visualize pipeline health in a Streamlit dashboard
- Test pipeline, validation, feature, and API behavior

## Architecture

```mermaid
flowchart TD
    A[Data generator or source] --> B[Raw data layer]
    B --> C[ETL pipeline]
    C --> D{Validation}
    D -->|Valid| E[(PostgreSQL)]
    D -->|Invalid| F[Quarantine]
    E --> G[Quality metrics]
    F --> G
    G --> H[Anomaly model]
    H --> I[FastAPI]
    I --> J[Dashboard and alerts]
```

### Processing flow

1. A source provides one daily data batch.
2. The original files are stored in the raw layer.
3. The ETL pipeline cleans and standardizes each entity.
4. Validation rules separate valid and invalid records.
5. Valid data, rejected records, and run metadata are stored.
6. Daily metrics are calculated and passed to the ML model.
7. Predictions and possible causes appear in the dashboard.

## Machine learning

### Problem definition

The first ML task is **unsupervised anomaly detection**. The model learns patterns from normal historical pipeline runs and assigns an anomaly score to each new daily batch.

### Input features

| Feature | Meaning |
| --- | --- |
| `row_count` | Total orders received |
| `unique_customer_count` | Customers who placed orders |
| `missing_value_ratio` | Proportion of missing required values |
| `duplicate_ratio` | Proportion of duplicate records |
| `rejected_record_ratio` | Proportion rejected by validation |
| `average_order_value` | Mean order total |
| `order_value_std` | Variation in order values |
| `refund_ratio` | Proportion of refunded orders |
| `payment_failure_ratio` | Proportion of failed payments |
| `pipeline_duration_seconds` | Batch-processing time |

### Models

| Model | Purpose |
| --- | --- |
| Fixed thresholds | Rule-based baseline |
| Z-score or IQR | Statistical baseline |
| Isolation Forest | Primary anomaly-detection model |
| Local Outlier Factor | Optional comparison |

The project compares simple and ML-based methods instead of assuming the most complex model is automatically the best.

### Example output

```json
{
  "pipeline_run_id": 152,
  "status": "anomaly",
  "anomaly_score": 0.87,
  "possible_causes": [
    "Order volume is 51% below its historical average",
    "Missing customer identifiers increased to 14%"
  ]
}
```

## Dataset

The first version uses generated e-commerce data. Synthetic data protects privacy, makes the project reproducible, and allows known anomalies to be injected for evaluation.

### Entities

| Entity | Important fields |
| --- | --- |
| Customers | `customer_id`, `full_name`, `email`, `country`, `created_at` |
| Products | `product_id`, `name`, `category`, `unit_price`, `stock_quantity` |
| Orders | `order_id`, `customer_id`, `order_date`, `status`, `currency`, `order_total` |
| Order items | `order_item_id`, `order_id`, `product_id`, `quantity`, `unit_price` |
| Payments | `payment_id`, `order_id`, `payment_method`, `payment_status`, `amount` |

### Injected anomalies

- Duplicate order or payment IDs
- Missing customer IDs
- Negative prices or quantities
- Invalid dates, currencies, or statuses
- Payments that do not match order totals
- References to nonexistent customers or products
- Unusually high transaction values
- Sudden changes in daily order volume
- Increased payment-failure rates

The generator records which anomalies were injected. These labels are used for evaluation but are not provided to the unsupervised model during training.

## Data validation

| Entity | Rule | Severity |
| --- | --- | --- |
| Customer | Customer ID is present and unique | Critical |
| Customer | Email has a valid format | Warning |
| Product | Unit price is greater than zero | Critical |
| Product | Stock quantity is not negative | Critical |
| Order | Order ID is present and unique | Critical |
| Order | Referenced customer exists | Critical |
| Order | Status belongs to the accepted list | Warning |
| Order item | Quantity is greater than zero | Critical |
| Order item | Referenced order and product exist | Critical |
| Payment | Amount is greater than zero | Critical |
| Payment | Amount matches the related order total | Critical |
| All entities | Required timestamps are valid | Critical |

## Technology stack

| Layer | Technology |
| --- | --- |
| Language | Python 3.12 |
| Data processing | Pandas |
| Machine learning | Scikit-learn |
| Database | PostgreSQL |
| ORM and migrations | SQLAlchemy and Alembic |
| Backend API | FastAPI |
| Dashboard | Streamlit |
| Validation | Pandera or custom validators |
| Testing | Pytest |
| Code quality | Ruff and Black |
| CI | GitHub Actions |
| Scheduling | Prefect after the MVP |

> Spark, Kafka, Airflow, and cloud infrastructure are intentionally excluded from the MVP. They may be explored only after the core pipeline and model work correctly.

## Project structure

```text
commerceguard/
├── data/
│   ├── raw/
│   ├── processed/
│   └── rejected/
├── notebooks/
│   ├── 01_exploratory_data_analysis.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_model_experiments.ipynb
├── src/commerceguard/
│   ├── api/
│   ├── database/
│   ├── generator/
│   ├── ingestion/
│   ├── validation/
│   ├── pipelines/
│   ├── features/
│   ├── models/
│   └── monitoring/
├── dashboard/
├── tests/
│   ├── unit/
│   └── integration/
├── models/
├── alembic/
├── .github/workflows/
├── .env.example
├── alembic.ini
├── pyproject.toml
└── README.md
```

## Development roadmap

### Project status

The project is currently in the **planning and data-design stage**.

### MVP

- [ ] Define database entities and relationships
- [ ] Build a reproducible e-commerce data generator
- [ ] Generate normal and anomalous daily batches
- [ ] Perform exploratory data analysis
- [ ] Create the PostgreSQL schema and migrations
- [ ] Implement the ETL pipeline
- [ ] Add critical validation rules
- [ ] Store rejected records and failure reasons
- [ ] Calculate daily quality metrics
- [ ] Build statistical baselines
- [ ] Train and evaluate an Isolation Forest
- [ ] Expose pipeline results through FastAPI
- [ ] Create the monitoring dashboard
- [ ] Add unit and integration tests

### After the MVP

- [ ] Schedule pipeline execution with Prefect
- [ ] Add email, Discord, or Slack alerts
- [ ] Add authentication and role-based access
- [ ] Containerize the application
- [ ] Add experiment tracking and model versioning
- [ ] Deploy the API, database, and dashboard

## Getting started

> The commands below describe the planned project interface and will become usable as each component is implemented.

### Prerequisites

- Python 3.12
- PostgreSQL 15+
- Git

### 1. Clone the repository

```bash
git clone https://github.com/YOUR_USERNAME/commerceguard.git
cd commerceguard
```

### 2. Create a virtual environment

<details>
<summary>Windows PowerShell</summary>

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

</details>

<details>
<summary>Linux or macOS</summary>

```bash
python3 -m venv .venv
source .venv/bin/activate
```

</details>

### 3. Install dependencies

```bash
pip install -e ".[dev]"
```

### 4. Configure the environment

Copy `.env.example` to `.env` and configure the database connection:

```env
DATABASE_URL=postgresql+psycopg://commerceguard:password@localhost:5432/commerceguard
RAW_DATA_PATH=data/raw
PROCESSED_DATA_PATH=data/processed
REJECTED_DATA_PATH=data/rejected
MODEL_PATH=models/isolation_forest.joblib
```

Never commit the `.env` file or real credentials.

### 5. Prepare the database

```bash
alembic upgrade head
```

### 6. Run the project

```bash
# Generate 180 days of sample data
python -m commerceguard.generator --days 180 --inject-anomalies

# Run the ETL and validation pipeline
python -m commerceguard.pipelines.daily_orders

# Train the anomaly model
python -m commerceguard.models.train

# Start the API
uvicorn commerceguard.api.main:app --reload

# Start the dashboard in a second terminal
streamlit run dashboard/app.py
```

## Evaluation

Injected anomalies provide ground truth for evaluating the unsupervised model.

The project reports:

- Precision
- Recall
- F1-score
- Confusion matrix
- False-positive rate
- Percentage of injected anomalies detected
- Batch-processing time

A chronological train/test split is used so future runs are never used to predict past behavior. Features that only become available after an incident is resolved are excluded to prevent data leakage.

## API design

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/health` | Check API availability |
| `GET` | `/pipeline-runs` | List recent runs |
| `GET` | `/pipeline-runs/{id}` | Inspect one run |
| `GET` | `/pipeline-runs/{id}/metrics` | Return quality metrics |
| `GET` | `/pipeline-runs/{id}/failures` | Return validation failures |
| `GET` | `/anomalies` | List detected anomalies |
| `POST` | `/predict` | Score new pipeline metrics |

## Testing

```bash
pytest
```

The test suite will cover:

- Reproducible data generation
- Transformation and validation logic
- Database operations
- Feature calculations
- Model input and output contracts
- API responses
- End-to-end pipeline execution

## Future improvements

- Connect to a public e-commerce dataset or demo API
- Detect schema changes and data drift
- Model weekends, holidays, promotions, and seasonality
- Explain anomalous metrics more precisely
- Support multiple stores and data sources
- Add alert acknowledgment and incident workflows
- Track experiments and model versions with MLflow
- Evaluate streaming tools when the project scale justifies them

## Learning objectives

CommerceGuard is designed to demonstrate:

- Exploratory data analysis and feature engineering
- ML baselines, model training, and evaluation
- Correct evaluation of an unsupervised model
- ETL pipeline and relational database design
- Data validation, observability, and incident investigation
- Backend API development
- Modular architecture and automated testing
- Reproducibility, monitoring, and technical documentation

## Privacy and safety

- Synthetic data is used initially to avoid exposing customer information.
- Real customer data must be anonymized before processing.
- Payment card details must never be collected or stored.
- An anomaly indicates unusual behavior, not proof of fraud or corruption.
- Predictions should support human investigation rather than silently delete data.
- Rejected records remain available for debugging and recovery.

## License

This project is intended for educational and portfolio purposes and can be released under the [MIT License](LICENSE).

---

If you have a suggestion or find a problem, please open an issue or submit a pull request.
