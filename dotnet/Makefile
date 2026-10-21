# ==============================================================================
# Production Makefile for .NET Applications (Entity Framework & Postgres Stack)
# ==============================================================================

SHELL := /bin/bash

# =========================
# Load Environment Variables
# =========================
ifneq (,$(wildcard ./.env))
    include .env
    export
endif

# =========================
# Terminal Colors
# =========================
GREEN  := \033[32m
YELLOW := \033[33m
BLUE   := \033[34m
CYAN   := \033[36m
RESET  := \033[0m

# =========================
# Variables & Configuration
# =========================
PROJECT_NAME   ?= PerformanceEvaluation
PROJECT_FILE   := $(PROJECT_NAME).csproj
ENV            := ASPNETCORE_ENVIRONMENT=Development
MIGRATIONS_DIR := src/Common/Infrastructure/Database/Migrations


# =========================
# Core Build & Execution
# =========================
.PHONY: build run watch clean install

build: ## Build the .NET project
	@echo "$(BLUE)Building $(PROJECT_FILE)...$(RESET)"
	dotnet build $(PROJECT_FILE)

run: ## Run the API application locally in Development mode
	@echo "$(CYAN)Starting $(PROJECT_NAME) API...$(RESET)"
	$(ENV) dotnet run --project $(PROJECT_FILE)

watch: ## Run application with Hot Reload enabled
	@echo "$(CYAN)Starting $(PROJECT_NAME) with Hot Reload (dotnet watch)...$(RESET)"
	$(ENV) dotnet watch --project $(PROJECT_FILE)

install: ## Execute local package installation/automation script
	@echo "$(CYAN)Running package installation script...$(RESET)"
	chmod +x install-package.sh
	./install-package.sh


# =========================
# Database & EF Core Migrations
# =========================
.PHONY: db-migration db-update db-drop

db-migration: ## Create a new EF Core migration (Usage: make db-migration name=AddUserTable)
	@echo "$(CYAN)Creating EF Core migration: $(name)...$(RESET)"
	@dotnet ef --version >/dev/null 2>&1 || dotnet tool install --global dotnet-ef
	$(ENV) dotnet ef migrations add $(name) \
		--project $(PROJECT_FILE) \
		--output-dir $(MIGRATIONS_DIR)

db-update: ## Apply pending EF Core migrations to the database
	@echo "$(CYAN)Applying database migrations...$(RESET)"
	$(ENV) dotnet ef database update --project $(PROJECT_FILE)

db-drop: ## Drop the database (Dangerous)
	@echo "$(YELLOW)Dropping database...$(RESET)"
	$(ENV) dotnet ef database drop --project $(PROJECT_FILE) --force


# =========================
# Testing & Code Quality
# =========================
.PHONY: test lint format

test: ## Run unit and integration tests
	@echo "$(CYAN)Running test suite...$(RESET)"
	dotnet test

lint: ## Run code analysis / formatting check
	@echo "$(CYAN)Analyzing code...$(RESET)"
	dotnet format --verify-no-changes

format: ## Auto-format C# code using dotnet format
	@echo "$(CYAN)Formatting C# code...$(RESET)"
	dotnet format


# =========================
# Docker Operations
# =========================
.PHONY: docker-up docker-down docker-logs docker-clean

docker-up: ## Start infrastructure services (Postgres, etc.) via Docker Compose
	@echo "$(CYAN)Starting Docker services...$(RESET)"
	docker compose up -d

docker-down: ## Stop and remove Docker Compose containers
	@echo "$(YELLOW)Stopping Docker services...$(RESET)"
	docker compose down

docker-logs: ## Tail Docker Compose logs
	docker compose logs -f

docker-clean: ## Clean up unused Docker resources
	docker system prune -f


# =========================
# Cleaning & Maintenance
# =========================
.PHONY: clean

clean: ## Remove build artifacts, bin/ obj/ directories and clean NuGet cache
	@echo "$(YELLOW)Cleaning build artifacts and cache...$(RESET)"
	dotnet clean $(PROJECT_FILE)
	rm -rf bin obj
	find . -type d -name "bin" -exec rm -rf {} +
	find . -type d -name "obj" -exec rm -rf {} +
	dotnet nuget locals all --clear
	@echo "$(GREEN)Cleanup completed successfully!$(RESET)"


# =========================
# Helper
# =========================
.PHONY: help

help: ## Show this help message with available targets
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'
