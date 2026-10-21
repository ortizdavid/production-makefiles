# 🛠️ PHP Production Makefile

A clean, production-ready `Makefile` template tailored for custom PHP MVC applications and microservices using MySQL. It standardizes local server execution, syntax checking, modular database management (tables and seeds), automated backups, and Docker/phpMyAdmin orchestration.

---

## 📋 Prerequisites

Ensure you have the following installed on your machine:

- **PHP** (8.4+)
- **MySQL / MariaDB client tools** (`mysql`, `mysqldump`, `mysqladmin`)
- **Docker** & **Docker Compose** *(optional)*

---

## 🚀 Quick Start

1. Copy the `Makefile` into the root directory of your PHP project:
```bash
   cp templates/php/Makefile ./Makefile

```

2. Create or verify your `.env` file in the root directory (Database credentials and configuration variables are loaded automatically).
3. Run `make help` to view all available commands:
```bash
make help

```



---

## ⚡ Core Command Categories

### 🚀 Execution & Quality

* `make run`: Starts the built-in PHP development server on the configured port.
* `make lint`: Scans all PHP source files across the project for syntax errors (excluding `vendor/`).

### 🗄️ Database Modular Management

* `make db-init`: Initializes the database and user from `init.sql`.
* `make db-setup`: Automatically executes all table creation scripts inside `database/structure/`.
* `make db-seed`: Applies development seed data from `database/seed/`.
* `make setup-db` (or `make fresh`): Runs the full provisioning pipeline in sequence (tables + seeds).

### 📦 Backup & Recovery

* `make db-dump`: Generates a timestamped full database backup (including structure, data, routines, and events) inside `database/dumps/`.
* `make db-restore`: Restores the database from the latest available dump (or specify a file using `make db-restore file=database_dump_YYYYMMDD_HHMMSS.sql`).

### 🔌 Database Consoles & Utilities

* `make db-connect`: Connects directly to the MySQL interactive shell.
* `make db-cli`: Opens the advanced CLI interface using `mycli`.
* `make db-status`: Checks the current MySQL server status.

### 🐳 Docker & Services

* `make docker-up`: Spins up MySQL via Docker Compose in the background.
* `make docker-logs`: Streams MySQL container logs in real time.
* `make pmy-start` / `make pmy-stop`: Quick management for a local phpMyAdmin container instance running on port 8080.

```
