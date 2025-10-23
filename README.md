# 🏈 SportsWorldCentral Data App

This project is a full-stack data application designed to ingest, process, serve, and visualize fantasy football data. It features a FastAPI backend, an Apache Airflow data pipeline for ETL, and a Streamlit dashboard for user-facing analytics.

## Dashboard Preview

The Streamlit front-end provides easy access to team rosters and aggregated statistics.

| Team Rosters Page | Team Stats Page |
| :---: | :---: |
| ![Streamlit Roster Page](pictures/streamlitpage1.jpg) | ![Streamlit Stats Page](pictures/streamlitpage2.jpg) |
*(Note: You will need to place your images in a folder like `streamlit/assets/` for them to render)*

---

## Core Architecture

The project is built on a 3-tier architecture:

1.  **Backend (FastAPI):** A high-performance API (`api/`) serves as the system's core. It handles CRUD operations and provides data from a transactional SQLite database (`fantasy_data.db`).
2.  **Data Pipeline (Apache Airflow):** An Airflow instance (`airflow/`) runs scheduled ETL jobs (DAGs). These jobs pull data from the FastAPI backend, process it, and load it into a separate SQLite database (`analytics_database.db`) optimized for analytics.
3.  **Frontend (Streamlit):** A Streamlit application (`streamlit/`) acts as the user-facing dashboard. It fetches data from the FastAPI backend using a simple HTTP client (`swc_simple_client.py`) and presents it in tables and charts.

---

## Tools & Technologies

* **Backend:** FastAPI, Uvicorn, SQLAlchemy (ORM), Pydantic
* **Frontend:** Streamlit
* **Data Pipeline:** Apache Airflow, Docker Compose
* **Database:** SQLite
* **Data Analysis:** Jupyter, Pandas
* **Client:** A custom Python client (`swc_simple_client.py`) is used by the Streamlit app and Jupyter notebooks to interact with the API.

---

## Project Structure

Here is a high-level overview of the project's file structure and the purpose of each component.

## 🧭 Overview

This platform automates the pipeline for fetching, storing, and analyzing player and team data from **SportsWorldCentral** APIs.

### 🔁 Components:
| Component | Description |
|------------|--------------|
| **Airflow** | Manages data ingestion workflows (DAGs) — pulls player data, performs health checks, and updates a SQLite database. |
| **API (FastAPI)** | Provides REST endpoints for querying teams, players, performances, and league data. |
| **Streamlit App** | Interactive dashboard for visualizing team rosters and stats. |
| **Notebooks** | Jupyter notebooks for prototyping, analysis, and client development. |

---

### 1. Backend API (FastAPI)

The API, located in the `api/` folder, is the system's "source of truth." It's built with FastAPI, providing a fast and efficient way to serve data from the `fantasy_data.db` SQLite database. It exposes several endpoints (like `/v0/players/`) that the other services (Airflow and Streamlit) consume.

### 2. Data Pipeline (Apache Airflow)

Airflow manages all data orchestration (ETL). The setup is containerized using Docker, defined in `airflow/airflow/docker-compose.yaml`. The pipelines (DAGs) pull data from the FastAPI backend and load it into the `analytics_database.db`.

#### DAGs Created:

* **`recurring_player_api_insert_update_dag.py`**
    This is the primary pipeline, designed to run on a recurring basis. It ensures the analytics database is kept in sync with the production API. As shown in the Airflow UI, this DAG consists of three main tasks:
    1.  `check_api_health_check_endpoint`: An `HttpOperator` that pings the API's `/` endpoint to ensure it's online.
    2.  `api_player_query`: An `HttpOperator` that fetches player data from the API's `/v0/players/` endpoint.
    3.  `player_sqlite_upsert`: A `PythonOperator` that takes the JSON data from the previous task and "upserts" (updates or inserts) it into the `analytics_database.db` using the `shared_functions.py`.

    ![Airflow DAG Graph](pictures/airflow1.jpg)
    ![Airflow DAG Graph](pictures/airflow2.jpg)
    *(Note: You will need to place your images in a folder like `airflow/assets/` for them to render)*

* **`bulk_player_file_load_dag.py`**
    This DAG is likely used for the initial, large-scale ingestion of data, possibly from a local file, to populate the database for the first time.

### 3. Frontend (Streamlit)

The user-facing dashboard is a multipage Streamlit app defined in the `streamlit/` folder. It is a "thin client" that does not connect directly to any database. Instead, it uses the `swc_simple_client.py` to make HTTP requests to the FastAPI backend, ensuring data is always consistent with the source of truth.

* **Team Rosters Page:** (`page1.py`) Displays a filterable table of all players for a given league and team.
* **Team Stats Page:** (`page2.py`) Fetches data and uses Pandas to aggregate it, displaying a bar chart of team-wide statistics like "Total Touchdowns."

## How to Run

To run the full project, you must start each of the three core components.

1.  **Start the Backend API:**
    ```bash
    cd api
    # Install dependencies
    pip install -r requirements.txt
    # Run the server
    uvicorn main:app --reload
    ```

2.  **Start the Airflow Pipeline:**
    ```bash
    cd airflow/airflow
    # This will build and run all Airflow services
    docker-compose up -d
    ```
    You can access the Airflow UI at `http://localhost:8080` (default) and manually trigger the DAGs to populate the analytics database.

3.  **Start the Streamlit Dashboard:**
    ```bash
    cd streamlit
    # Install dependencies (if not already done)
    # pip install -r requirements.txt
    # Run the app
    streamlit run streamlit_football_app.py
    ```
    You can view the dashboard in your browser, typically at `http://localhost:8501`.

## License

This project is licensed under the terms of the `LICENSE` file.
