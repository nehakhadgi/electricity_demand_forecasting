# Electricity Demand Forecasting Pipeline with SARIMAX

An end-to-end time-series engineering and forecasting pipeline that ingests high-frequency smart meter readings, aggregates records into clean hourly sequences, and evaluates a seasonal-exogenous **SARIMAX** model against a robust baseline.

This project implements a rigorous chronological holdout validation strategy and establishes operational rules for production fallback, retraining, and automated data monitoring.

---

## 📌 Project Overview
Electricity grid management requires highly accurate demand forecasts to balance power generation, avoid grid overloads, and optimize wholesale trading. Since power consumption follows highly recurring, multi-seasonal patterns (daily rhythms paired with weekly industrial loops), standard machine learning splits can lead to extreme data leakage.

This project builds and deploys:
1. **A robust data ingestion pipeline** that converts 10-minute Tetouan City power grid readings into unified, hourly time-series metrics.
2. **A baseline benchmark** using a **Weekly Seasonal-Naive** model to set a realistic, honest threshold.
3. **A multivariate SARIMAX forecaster** utilizing temperature and humidity as exogenous features to model non-linear demand shifts.
4. **An out-of-time chronological evaluation framework** over a strict 7-day future horizon.
5. **Operational run-rate guardrails** including fallbacks, health metrics, and automated monitoring thresholds.

---

## 📂 Repository Structure

*   `forecast_pipeline.py`: Production-grade python pipeline executing data ingestion, resample aggregations, feature alignment, and model fitting.
*   `Tetouan_City_power_consumption.csv`: Raw, high-frequency public energy consumption dataset from Tetouan City (Sourced from the UCI ML Repository).
*   `forecast_evaluation_metrics.csv`: Auto-generated matrix tracking Holdout RMSE, MAPE, and MAE across all models.
*   `operational_guardrails.md`: Formulated operational playbook detailing production fallback routines and alert thresholds.
*   `README.md`: This comprehensive overview and documentation homepage.

---

## 🛠️ Data Pipeline & Architecture

### 1. Ingestion & Resampling
The source data consists of high-frequency (10-minute interval) readings across three distinct distribution zones in Tetouan City. To eliminate high-frequency signal noise and align with standard utility planning timelines, the pipeline:
*   Aggregates data up to the **hourly level** using a mean-resampling approach.
*   Handles missing timestamps and gaps using chronological linear interpolation.
*   Constructs cyclical features (sine/cosine transformations of hours and days) to capture continuous temporal positions.

### 2. Chronological Holdout Strategy (Zero Leak)
Traditional randomized k-fold cross-validation is completely disabled. In time-series modeling, predicting past records using future data is a critical form of **temporal leakage**. 
*   **Training Set**: Historical hourly sequences up to the last 7 days.
*   **Chronological Test Holdout**: The final, untouched **7 days (168 hourly steps)**. All models are evaluated strictly on their ability to forecast ahead into this unseen future window.

---

## 📊 Models & Benchmarks

### 1. Weekly Seasonal-Naive Baseline
Before training complex statistical models, the pipeline fits a simple, highly competitive baseline:
*   **Logic**: The forecasted demand for any hour in the next week is assumed to be exactly equal to the demand recorded at that same hour in the previous week (t - 168 hours).
*   Any advanced model *must* beat this baseline to prove its economic utility.

### 2. Multivariate SARIMAX Forecaster
To improve on the naive baseline, we fit a **SARIMAX** model with the following parameters:
*   **Autoregressive (AR) & Moving Average (MA) Terms**: Modeled to capture short-term residual auto-correlations.
*   **Seasonal Order**: Set to 24-hour daily seasonality.
*   **Exogenous Variables**: Integrates outdoor temperature, wind speed, and humidity to adjust baseline predictions dynamically during extreme weather changes.

---

## 🚨 Production MLOps & Operational Playbook

To ensure this pipeline operates safely in a live utility environment, we define three explicit operating rules:

### 1. Automated Monitoring (Data Drift)
If the rolling average deviation between forecasted and actual demand exceeds a **20% MAPE threshold** over any 6-hour sequence, the system flags a **Critical Prediction Alert**. This indicates an unexpected event (e.g., sudden storm, holiday, or equipment failure) and shifts downstream services away from live inference.

### 2. Fallback Routine
If the SARIMAX service fails to converge, experiences high input latency, or encounters API timeout issues, the production wrapper immediately defaults to the **Seasonal-Naive baseline**. This ensures that downstream grids always receive a stable, safe forecast without service interruption.

### 3. Retraining Triggers
The SARIMAX parameters are scheduled to automatically retrain on a **weekly rolling window** every Sunday at 12:00 AM UTC. This absorbs latest consumer trends, changing weather seasons, and holiday behaviors without manual intervention.

---

## 🚀 How to Run the Pipeline

1.  **Clone the Repository**:
    ```bash
    git clone https://github.com/your-username/electricity-demand-forecaster.git
    cd electricity-demand-forecaster
    ```
2.  **Install Dependencies**:
    ```bash
    pip install pandas numpy statsmodels scikit-learn matplotlib
    ```
3.  **Execute Ingestion & Inferences**:
    ```bash
    python forecast_pipeline.py
    ```
