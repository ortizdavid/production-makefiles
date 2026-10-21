# 🛠️ Go Production Makefile

A feature-complete, production-ready `Makefile` template tailored for Go applications and microservices. It standardizes common tasks such as running, building, database migrations, code quality analysis, and multi-architecture CI/CD builds.

---

## 📋 Prerequisites

To make full use of all targets included in this Makefile, ensure you have the following installed on your machine:

- **Go** (1.20+)
- **Docker** & **Docker Compose**
- *(Optional)* [Atlas CLI](https://atlasgo.io/) (for database migrations)

> **Note:** Development tools like `golangci-lint`, `govulncheck`, and `swag` will be automatically installed by the Makefile if they are missing from your environment.

---

## 🚀 Quick Start

1. Copy the `Makefile` into the root directory of your Go project:
```bash
   cp templates/go/Makefile ./Makefile

```

2. Create or verify your `.env` file in the project root (the Makefile automatically loads environment variables from it).
3. Run `make help` to display all available commands with descriptions:
```bash
make help

```



---

## ⚡ Core Command Categories

### 🛠️ Execution & Build

* `make run`: Runs the application locally (`./cmd/api/main.go`).
* `make build`: Compiles an optimized, lightweight binary into `bin/`.
* `make clean`: Removes generated binaries and build artifacts.

### 🧪 Code Quality & Security

* `make lint`: Runs `golangci-lint` for static analysis and code checks.
* `make vuln-check`: Performs security vulnerability analysis on Go dependencies using `govulncheck`.
* `make format`: Automatically formats code and optimizes imports (`gofmt` & `goimports`).
* `make test`: Runs unit tests with race detection and coverage.
* `make swagger`: Generates OpenAPI/Swagger documentation using `swag`.

### 🗄️ Database & Migrations

* `make setup-dev`: Bootstraps the local dev environment (applies migrations and seeds).
* `make migrate`: Generates a new migration diff based on GORM models using Atlas.
* `make migrate-apply`: Applies all pending database migrations.
* `make seed`: Executes Go-based database seeders.
* `make rebuild-dev`: **[Warning]** Drops the database schema and re-runs setup from scratch.

### 🐳 Docker Operations

* `make docker-build`: Builds the production Docker image.
* `make docker-up`: Starts containerized services in detached mode.
* `make docker-down`: Stops and removes running containers/networks.
* `make docker-logs`: Tails real-time logs from Docker Compose.

### 🔄 CI/CD Automation

* `make ci-prepare`: Downloads and verifies Go modules.
* `make ci-lint`: Runs linter with GitHub Actions output formatting.
* `make ci-test`: Runs tests with atomic coverage reporting for CI pipelines.
* `make ci-build`: Cross-compiles Linux `amd64` and `arm64` binaries with stripped debug flags (`-ldflags="-s -w"`).

---

## 💡 Customization

Feel free to adapt the following variables at the top of the `Makefile` to match your project structure:

```makefile
APP_NAME     := my-service
APP_PORT     := 8080
SWAG_DIR     := ./docs

```
