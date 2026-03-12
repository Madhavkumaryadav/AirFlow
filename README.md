# AirFlow — Docker-based Apache Airflow Pipeline

A project for spinning up **Apache Airflow** using **Docker Compose**, enabling you to define, schedule, and monitor data pipelines (DAGs) with ease.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Creating DAGs](#creating-dags)
- [Services](#services)
- [Configuration](#configuration)
- [Useful Commands](#useful-commands)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

[Apache Airflow](https://airflow.apache.org/) is an open-source platform to programmatically author, schedule, and monitor workflows. This repository provides a Docker Compose–based setup to run a fully functional Airflow environment locally or in production, including:

- **Webserver** – Airflow's browser-based UI for managing and monitoring DAGs.
- **Scheduler** – Continuously monitors and triggers task instances.
- **Worker** – Executes tasks dispatched by the scheduler (Celery executor).
- **Flower** – Optional Celery monitoring dashboard.
- **PostgreSQL** – Metadata database for Airflow.
- **Redis** – Message broker for the Celery executor.

---

## Architecture

```
┌─────────────┐       ┌───────────────┐       ┌──────────────┐
│   Browser   │──────▶│  Webserver    │       │  Scheduler   │
└─────────────┘       │  (port 8080)  │       │              │
                      └───────┬───────┘       └──────┬───────┘
                              │                       │
                    ┌─────────▼─────────┐   ┌────────▼────────┐
                    │    PostgreSQL DB   │   │   Redis Broker  │
                    └───────────────────┘   └────────┬────────┘
                                                      │
                                             ┌────────▼────────┐
                                             │   Celery Worker │
                                             └─────────────────┘
```

---

## Prerequisites

| Tool | Minimum Version |
|------|----------------|
| [Docker](https://docs.docker.com/get-docker/) | 20.10+ |
| [Docker Compose](https://docs.docker.com/compose/install/) | v2.0+ |
| RAM | 4 GB+ recommended |

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/Madhavkumaryadav/AirFlow.git
cd AirFlow
```

### 2. Initialize the Airflow database

```bash
docker compose up airflow-init
```

### 3. Start all services

```bash
docker compose up -d
```

### 4. Access the Airflow UI

Open [http://localhost:8080](http://localhost:8080) in your browser.

- **Username:** `airflow`
- **Password:** `airflow`

### 5. Stop all services

```bash
docker compose down
```

---

## Project Structure

```
AirFlow/
├── dags/                  # Place your DAG files here
│   └── example_dag.py
├── logs/                  # Airflow task logs (auto-generated)
├── plugins/               # Custom Airflow plugins
├── config/                # Airflow configuration overrides
├── docker-compose.yml     # Docker Compose service definitions
└── README.md
```

---

## Creating DAGs

A **DAG** (Directed Acyclic Graph) defines a workflow in Airflow. Place Python DAG files inside the `dags/` directory.

### Minimal DAG example

```python
from datetime import datetime
from airflow import DAG
from airflow.operators.python import PythonOperator

def print_hello():
    print("Hello from Airflow!")

with DAG(
    dag_id="hello_world",
    start_date=datetime(2024, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:
    task = PythonOperator(
        task_id="say_hello",
        python_callable=print_hello,
    )
```

After saving the file to `dags/`, it will appear in the Airflow UI within a few seconds.

---

## Services

| Service | Port | Description |
|---------|------|-------------|
| `airflow-webserver` | `8080` | Airflow Web UI |
| `airflow-scheduler` | — | Schedules and triggers DAGs |
| `airflow-worker` | — | Executes tasks (Celery) |
| `flower` | `5555` | Celery monitoring dashboard |
| `postgres` | `5432` | Airflow metadata database |
| `redis` | `6379` | Message broker for Celery |

---

## Configuration

Environment variables and Airflow settings can be customised in `docker-compose.yml` under the `x-airflow-common` section, or by editing `config/airflow.cfg`.

Common variables:

| Variable | Default | Description |
|----------|---------|-------------|
| `AIRFLOW__CORE__EXECUTOR` | `CeleryExecutor` | Task executor |
| `AIRFLOW__DATABASE__SQL_ALCHEMY_CONN` | PostgreSQL URI | Metadata DB connection |
| `AIRFLOW__CELERY__BROKER_URL` | Redis URI | Celery broker URL |
| `AIRFLOW__CORE__FERNET_KEY` | — | Encryption key for secrets |

---

## Useful Commands

```bash
# List all DAGs
docker compose exec airflow-webserver airflow dags list

# Trigger a DAG run manually
docker compose exec airflow-webserver airflow dags trigger <dag_id>

# View task logs
docker compose exec airflow-webserver airflow tasks logs <dag_id> <task_id> <execution_date>

# Pause / unpause a DAG
docker compose exec airflow-webserver airflow dags pause <dag_id>
docker compose exec airflow-webserver airflow dags unpause <dag_id>

# Check Airflow version
docker compose exec airflow-webserver airflow version
```

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m "Add my feature"`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request.

---

## License

This project is licensed under the [MIT License](LICENSE).
