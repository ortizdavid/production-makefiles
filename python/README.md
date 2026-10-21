# 🛠️ Python Production Makefile

A feature-complete, production-ready `Makefile` template tailored for Python backend applications and microservices. It standardizes environment setup, virtual environment handling, MongoDB database operations (seeds, backups, and restores), testing, and code quality workflows.

---

## 📋 Prerequisites

Ensure you have the following installed on your machine:

- **Python** (3.9+)
- **pip** and `venv` module
- **Docker** & **Docker Compose** *(optional)*
- **MongoDB CLI tools** (`mongosh`, `mongodump`, `mongorestore`) *(optional for database operations)*

---

## 🚀 Quick Start

1. Copy the `Makefile` into the root directory of your Python project:
   ```bash
   cp templates/python/Makefile ./Makefile

```

2. Create or verify your `.env` file in the project root (the Makefile automatically loads environment variables and detects MongoDB configurations).
3. Run `make setup` to initialize the virtual environment and install dependencies automatically:
```bash
make setup

```


4. Run `make help` to view all available commands:
```bash
make help

```



---

## ⚡ Core Command Categories

### 🐍 Environment & Execution

* `make setup`: Automatically creates `.venv` (or `venv`), upgrades `pip`, and installs requirements.
* `make install`: Re-installs dependencies from `requirements.txt`.
* `make run`: Runs the main Python application (`main.py`).
* `make run-uvicorn`: Starts an ASGI/FastAPI application using Uvicorn with hot-reload.

### 🗄️ Database Operations (MongoDB)

* `make db-connect`: Opens an interactive MongoDB shell (`mongosh`) using URI and database configurations from `.env`.
* `make db-seed`: Runs development database data seeders.
* `make db-dump`: Creates a timestamped database backup in `database/dumps/`.
* `make db-restore`: Restores the database from the latest backup (or specify a path using `make db-restore BACKUP_PATH=database/dumps/...`).

### 🧪 Testing & Code Quality

* `make test`: Runs the test suite using `pytest` (with a fallback to Python's built-in `unittest`).
* `make lint`: Runs static code analysis using `ruff` or `flake8`.
* `make format`: Auto-formats code using `ruff` or `autopep8`.
* `make vuln-check`: Audits dependencies for known security vulnerabilities using `pip-audit`.

### 🐳 Docker Operations

* `make docker-build`: Builds the production Docker image.
* `make docker-up`: Starts services in the background via Docker Compose.
* `make docker-down`: Stops and removes Docker containers and volumes.
* `make docker-logs`: Streams container logs in real time.

### 🧹 Maintenance & Cleanup

* `make clean-cache`: Removes Python bytecode and cache files (`__pycache__`, `.pyc`, etc.).
* `make clean-venv`: Deletes the virtual environment directory.
* `make clean`: Performs a full cleanup (caches + virtual environment).

---

## 💡 Customization

The Makefile automatically detects whether your virtual environment folder is named `.venv` or `venv`. You can override or adjust parameters directly at the top of the file if needed.
