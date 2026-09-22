CommerceGuard

Intelligent Data Quality Monitoring for E-commerce Pipelines

CommerceGuard is an end-to-end data engineering and machine learning project that monitors the quality of e-commerce data. It detects missing, duplicated, invalid, or unusual records before unreliable data reaches business dashboards, analytics systems, or machine learning models.

The project simulates a real e-commerce platform that processes customers, products, orders, order items, and payments every day. A data pipeline validates each batch, stores clean and rejected records separately, calculates quality metrics, and uses anomaly detection to identify unexpected behavior.

Project status: Planning / early development

Table of Contents

Why This Project?

Problem Statement

Project Goals

System Overview

Main Features

Machine Learning Problem

Dataset

Data Quality Rules

Technology Stack

Project Architecture

Database Design

Project Structure

Implementation Roadmap

Evaluation

Installation

Usage

API Endpoints

Testing

Ethical and Technical Considerations

Future Improvements

Learning Outcomes

License

Why This Project?

E-commerce companies depend on accurate data to calculate revenue, manage inventory, process payments, understand customer behavior, and make business decisions. A small pipeline failure can create incorrect reports or hide important operational problems.

Examples include:

Duplicate orders that inflate revenue

Missing customer or product identifiers

Negative prices or quantities

Payments that do not match order totals

Sudden and unexpected drops in daily order volume

Changes in column names or data types

Invalid timestamps, currencies, or order statuses

CommerceGuard combines deterministic validation rules with machine learning. Rules detect known problems, while anomaly detection helps discover unusual patterns that were not explicitly programmed.

Problem Statement

Traditional data-validation systems only detect conditions that developers already know about. However, an e-commerce dataset can pass all predefined rules and still behave abnormally.

CommerceGuard answers two questions:

Is every individual record valid?

Does the complete daily dataset behave as expected?

The first question is handled by validation rules. The second is handled through statistical analysis and machine learning.

Project Goals

Build a reproducible ETL pipeline for e-commerce data.

Preserve raw data before applying transformations.

Validate orders, customers, products, and payments.

Separate valid records from rejected records.

Calculate data-quality metrics for every pipeline run.

Detect unusual daily batches using anomaly detection.

Expose results through an API and monitoring dashboard.

Apply software-engineering practices such as testing, logging, modular design, and version control.

System Overview

flowchart TD
    A[Data generator or source API] --> B[Raw data layer]
    B --> C[ETL and validation pipeline]
    C --> D[Clean records]
    C --> E[Rejected records]
    D --> F[(PostgreSQL)]
    E --> F
    F --> G[Quality metrics]
    G --> H[Anomaly detection model]
    H --> I[FastAPI service]
    I --> J[Monitoring dashboard]
    H --> K[Alerts]

Main Features

1. E-commerce data ingestion

Read daily CSV or JSON batches.

Optionally collect data from an external API.

Store an unchanged copy of every source batch.

Record the ingestion time and source.

2. ETL pipeline

Extract raw customers, products, orders, order items, and payments.

Standardize dates, currencies, categories, and identifiers.

Validate relationships between entities.

Load valid records into PostgreSQL.

Move invalid records into a quarantine area with rejection reasons.

3. Rule-based data validation

Detect missing required fields.

Detect duplicated identifiers.

Verify accepted values and data types.

Validate prices, quantities, totals, and timestamps.

Check relationships such as whether an order references an existing customer.

4. Machine learning anomaly detection

Build one feature vector for each daily pipeline run.

Learn the typical behavior of daily e-commerce data.

Assign an anomaly score to new batches.

Flag suspicious batches for investigation.

5. Dashboard and reporting

Display the current pipeline status.

Visualize quality metrics over time.

Show failed validation rules and rejected records.

Display anomaly scores and possible causes.

Compare current metrics with historical values.

Machine Learning Problem

Primary task

The initial ML task is unsupervised anomaly detection. The model learns from historical pipeline metrics and determines whether a new daily batch is normal or unusual.

Each pipeline run can be represented using features such as:

Feature

Description

row_count

Number of orders received during the run

unique_customer_count

Number of customers placing orders

missing_value_ratio

Percentage of missing required values

duplicate_ratio

Percentage of duplicated records

rejected_record_ratio

Percentage of records rejected by validation

average_order_value

Mean order total for the batch

order_value_std

Variation in order totals

refund_ratio

Percentage of refunded orders

payment_failure_ratio

Percentage of unsuccessful payments

pipeline_duration_seconds

Time required to process the batch

Models to compare

Rule-based baseline: fixed thresholds defined by the developer

Statistical baseline: Z-score or interquartile-range detection

Isolation Forest: primary unsupervised ML model

Local Outlier Factor: optional comparison model

The project will compare these approaches instead of assuming that the most complex model is automatically the best.

Example prediction

{
  "pipeline_run_id": 152,
  "status": "anomaly",
  "anomaly_score": 0.87,
  "possible_causes": [
    "Order volume is 51% lower than its historical average",
    "Missing customer identifiers increased to 14%"
  ]
}

Dataset

The first version uses a synthetic e-commerce dataset so that data failures can be introduced intentionally and labeled precisely. This avoids exposing real customer information and makes model evaluation reproducible.

Main entities

Customers

customer_id

full_name

email

country

created_at

Products

product_id

name

category

unit_price

stock_quantity

Orders

order_id

customer_id

order_date

status

currency

order_total

Order items

order_item_id

order_id

product_id

quantity

unit_price

Payments

payment_id

order_id

payment_method

payment_status

amount

payment_date

Simulated anomalies

The data generator will occasionally introduce controlled problems:

Duplicate order or payment IDs

Missing customer IDs

Negative prices or quantities

Unsupported currencies

Invalid order statuses

Payment totals that do not match order totals

Orders referencing nonexistent products

Extreme transaction values

Sudden changes in order volume

Increased payment-failure rates

Incorrect date formats

Each generated batch will contain metadata indicating which anomalies were injected. These labels are used only for evaluation and are not provided to the unsupervised model during training.

Data Quality Rules

Entity

Validation rule

Severity

Customer

customer_id must be present and unique

Critical

Customer

Email must follow a valid format

Warning

Product

Price must be greater than zero

Critical

Product

Stock quantity cannot be negative

Critical

Order

Order ID must be present and unique

Critical

Order

Customer must exist

Critical

Order

Status must belong to the accepted status list

Warning

Order item

Quantity must be greater than zero

Critical

Order item

Product and order must exist

Critical

Payment

Amount must be greater than zero

Critical

Payment

Payment total must match the related order total

Critical

All entities

Required timestamps must be valid

Critical

Technology Stack

Area

Technology

Programming language

Python 3.12

Data processing

Pandas

Machine learning

Scikit-learn

Database

PostgreSQL

ORM and migrations

SQLAlchemy and Alembic

Backend API

FastAPI

Dashboard

Streamlit

Data validation

Pandera or custom validation functions

Scheduling

Prefect, added after the MVP

Testing

Pytest

Code quality

Ruff and Black

CI

GitHub Actions

Containerization

Docker, optional after local development

The MVP intentionally avoids adding Spark, Kafka, Airflow, and multiple cloud services. Those technologies can be explored later after the core pipeline and ML model work correctly.

Project Architecture

CommerceGuard uses a layered design:

Source layer: generates or receives e-commerce data.

Raw layer: preserves the original input without modification.

Processing layer: cleans, standardizes, and validates records.

Storage layer: stores clean records, rejected records, and quality metrics.

ML layer: trains the anomaly detector and produces anomaly scores.

Service layer: exposes pipeline and prediction results through FastAPI.

Presentation layer: displays operational information in Streamlit.

Database Design

The initial database contains the following groups of tables:

Business tables

customers

products

orders

order_items

payments

Monitoring tables

pipeline_runs

data_quality_metrics

validation_failures

rejected_records

anomaly_predictions

alert_history

Every monitoring record is connected to a pipeline_run_id, making it possible to reproduce and investigate a specific run.

Project Structure

commerceguard/
├── data/
│   ├── raw/
│   ├── processed/
│   └── rejected/
├── notebooks/
│   ├── 01_exploratory_data_analysis.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_model_experiments.ipynb
├── src/
│   └── commerceguard/
│       ├── api/
│       ├── database/
│       ├── generator/
│       ├── ingestion/
│       ├── validation/
│       ├── pipelines/
│       ├── features/
│       ├── models/
│       └── monitoring/
├── dashboard/
│   └── app.py
├── tests/
│   ├── unit/
│   └── integration/
├── models/
├── alembic/
├── .github/
│   └── workflows/
├── .env.example
├── alembic.ini
├── docker-compose.yml
├── pyproject.toml
└── README.md

Implementation Roadmap

Phase 1: Data generation and exploration

Define the e-commerce entities and relationships.

Generate several months of normal daily data.

Inject controlled anomalies into selected batches.

Explore distributions, correlations, missing values, and outliers.

Deliverable: reproducible dataset and exploratory notebook.

Phase 2: ETL and database

Create the PostgreSQL schema.

Save original batches in the raw layer.

Transform fields into consistent formats.

Load valid records into the business tables.

Record metadata for every pipeline execution.

Deliverable: repeatable raw-to-database pipeline.

Phase 3: Data validation

Implement entity-level validation rules.

Store failure reasons and severity levels.

Quarantine rejected records.

Calculate a data-quality score for each run.

Deliverable: validation report for every processed batch.

Phase 4: Machine learning

Create one metric vector per daily pipeline run.

Split historical runs into training and testing periods.

Establish rule-based and statistical baselines.

Train and tune an Isolation Forest.

Evaluate detection performance using injected anomaly labels.

Save the selected preprocessing pipeline and model.

Deliverable: evaluated anomaly-detection model.

Phase 5: API and dashboard

Build endpoints for pipeline runs, failures, metrics, and predictions.

Display recent pipeline health and historical trends.

Allow users to inspect rejected records.

Explain why a batch was flagged when possible.

Deliverable: working local monitoring application.

Phase 6: Engineering improvements

Add automated tests and continuous integration.

Add structured logging and robust error handling.

Schedule daily executions with Prefect.

Send email, Discord, or Slack alerts.

Add Docker support and deployment documentation.

Deliverable: portfolio-ready project with a reproducible setup.

Evaluation

Although Isolation Forest is unsupervised, the injected anomalies provide ground truth for evaluating the project.

The following metrics will be reported:

Precision

Recall

F1-score

Confusion matrix

False-positive rate

Percentage of injected anomalies detected

Average time required to process a batch

Recall is important because missing a serious data problem can affect reports and decisions. Precision is also important because excessive false alarms can cause users to ignore the monitoring system.

To prevent data leakage, training and evaluation use a chronological split. Future pipeline runs are never used to predict past runs. Metrics that become available only after an incident is resolved are not included as model inputs.

Installation

The commands below describe the planned local setup and may change while the project is under development.

Prerequisites

Python 3.12

PostgreSQL 15 or newer

Git

1. Clone the repository

git clone https://github.com/YOUR_USERNAME/commerceguard.git
cd commerceguard

2. Create a virtual environment

On Windows PowerShell:

py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1

On Linux or macOS:

python3 -m venv .venv
source .venv/bin/activate

3. Install dependencies

pip install -e ".[dev]"

4. Configure environment variables

cp .env.example .env

Example configuration:

DATABASE_URL=postgresql+psycopg://commerceguard:password@localhost:5432/commerceguard
RAW_DATA_PATH=data/raw
PROCESSED_DATA_PATH=data/processed
REJECTED_DATA_PATH=data/rejected
MODEL_PATH=models/isolation_forest.joblib

Never commit the real .env file or database credentials.

5. Apply database migrations

alembic upgrade head

Usage

Generate sample data:

python -m commerceguard.generator --days 180 --inject-anomalies

Run the ETL and validation pipeline:

python -m commerceguard.pipelines.daily_orders

Train the anomaly-detection model:

python -m commerceguard.models.train

Start the API:

uvicorn commerceguard.api.main:app --reload

Start the dashboard:

streamlit run dashboard/app.py

API Endpoints

Planned endpoints include:

Method

Endpoint

Description

GET

/health

Check whether the API is running

GET

/pipeline-runs

List recent pipeline runs

GET

/pipeline-runs/{id}

Inspect one pipeline run

GET

/pipeline-runs/{id}/metrics

Get quality metrics for a run

GET

/pipeline-runs/{id}/failures

List failed validation rules

GET

/anomalies

List detected anomalies

POST

/predict

Score a new set of pipeline metrics

Testing

Run all tests:

pytest

The test suite should cover:

Data-generation reproducibility

Transformation logic

Individual validation rules

Database operations

Feature calculations

Model input and output formats

API responses

End-to-end pipeline execution

Ethical and Technical Considerations

Synthetic data is used initially to protect customer privacy.

Real customer data should be anonymized before processing.

Payment card details must never be collected or stored.

An anomaly is not automatically proof of corruption or fraud.

Predictions should assist investigation, not silently delete data.

Rejected records are preserved for debugging and possible recovery.

Model performance can degrade as customer behavior and business activity change.

Future Improvements

Connect to a real public e-commerce dataset or demo API.

Monitor schema changes and data drift.

Add seasonal features for weekends, holidays, and promotions.

Compare Isolation Forest with additional anomaly-detection models.

Add alert acknowledgment and incident-management workflows.

Add role-based authentication.

Support multiple stores and data sources.

Introduce model versioning and experiment tracking with MLflow.

Deploy the API, database, pipeline, and dashboard.

Explore Kafka or Spark only when data scale justifies them.

Learning Outcomes

By completing CommerceGuard, I aim to demonstrate an understanding of:

The complete machine learning lifecycle

Exploratory data analysis and feature engineering

Supervised evaluation of an unsupervised model

ETL pipeline design

Relational database modeling

Data-quality validation and observability

Backend API development

Testing and continuous integration

Reproducibility, monitoring, and technical documentation

License

This project is intended for educational and portfolio purposes. It can be released under the MIT License.

If you find this project useful or have suggestions, feel free to open an issue or submit a pull request.
