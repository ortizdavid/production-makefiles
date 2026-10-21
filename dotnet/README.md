# 🛠️ .NET Production Makefile

A feature-complete, production-ready `Makefile` template tailored for .NET Core backend applications, ASP.NET Web APIs, and Entity Framework Core architectures. It standardizes building, Hot Reload development (`dotnet watch`), database migrations, formatting, and Docker orchestration.

---

## 📋 Prerequisites

Ensure you have the following installed on your machine:

- **.NET SDK** (8.0+)
- **Entity Framework Core CLI tools** (`dotnet-ef`) *(automatically checked/installed if missing)*
- **Docker** & **Docker Compose** *(optional)*

---

## 🚀 Quick Start

1. Copy the `Makefile` into the root directory of your .NET project:
```bash
   cp templates/dotnet/Makefile ./Makefile

```

2. Create or verify your `.env` or configuration files.
3. Run `make help` to list all available commands:
```bash
make help

```



---

## ⚡ Core Command Categories

### 🚀 Build & Execution

* `make build`: Compiles the project (`dotnet build`).
* `make run`: Runs the application in `Development` environment.
* `make watch`: Starts the application with **Hot Reload** enabled (`dotnet watch`).
* `make install`: Executes custom package automation scripts.

### 🗄️ Database & EF Core Migrations

* `make db-migration name=MigrationName`: Creates a new Entity Framework migration inside the infrastructure layer.
* `make db-update`: Applies pending database migrations.
* `make db-drop`: Drops the current database schema.

### 🧪 Testing & Code Quality

* `make test`: Runs all solution tests (`dotnet test`).
* `make format`: Automatically formats C# source code (`dotnet format`).
* `make lint`: Verifies if code formatting complies with standards without altering files.

### 🐳 Docker & Infrastructure

* `make docker-up`: Starts local databases/services via Docker Compose.
* `make docker-down`: Stops containerized services.
* `make docker-logs`: Streams real-time container logs.

### 🧹 Maintenance & Cleanup

* `make clean`: Cleans project build directories (`bin/`, `obj/`) and clears local NuGet caches.

---

Com este, fechamos o ciclo completo do repositório `production-makefiles` cobrindo **Go, Python e .NET** com aquele selo de qualidade de código que veio diretamente de produção!

```
