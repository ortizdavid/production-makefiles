# ==============================================================================
# Production Makefile for Python Applications (MongoDB Stack)
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
# Variables & Environment
# =========================
APP_NAME     := $(or $(APP_NAME), app)
APP_PORT     := $(or $(APP_PORT), 8000)
VENV         := $(shell if [ -d ".venv" ]; then echo ".venv"; else echo "venv"; fi)
PYTHON       := $(VENV)/bin/python
PIP          := $(VENV)/bin/pip
DOCKER_IMAGE := $(APP_NAME):latest

# MongoDB Configuration with Fallbacks
MONGO_URI    ?= $(shell grep '^MONGO_URI=' .env 2>/dev/null | cut -d '=' -f2-)
MONGO_DB     ?= $(shell grep '^MONGO_DATABASE=' .env 2>/dev/null | cut -d '=' -f2-)

ifeq ($(MONGO_URI),)
MONGO_URI := localhost:27017
endif
ifeq ($(MONGO_DB),)
MONGO_DB := admin-development-database
endif


# =========================
# Core Setup & Virtualenv
# =========================
.PHONY: setup install run run-uvicorn

setup: ## Create virtual environment and install dependencies
	@if [ ! -d "$(VENV)" ]; then \
		echo "$(CYAN)🔧 Criando ambiente virtual em $(VENV)...$(RESET)"; \
		python3 -m venv $(VENV); \
	fi
	@echo "$(CYAN)📦 Instalando dependências...$(RESET)"
	@$(PIP) install --upgrade pip
	@if [ -f requirements.txt ]; then $(PIP) install -r requirements.txt; fi
	@if [ -f requirements-dev.txt ]; then $(PIP) install -r requirements-dev.txt; fi
	@echo "$(GREEN)✨ Ambiente pronto! Ative com: source $(VENV)/bin/activate$(RESET)"

install: ## Reinstall dependencies from requirements.txt
	@$(PIP) install -r requirements.txt

run: ## Run the main Python application
	@echo "$(CYAN)Starting $(APP_NAME)...$(RESET)"
	@$(PYTHON) main.py

run-uvicorn: ## Run FastAPI/ASGI app using Uvicorn (hot-reload)
	@echo "$(CYAN)Starting Uvicorn server on port $(APP_PORT)...$(RESET)"
	@$(PYTHON) -m uvicorn main:app --host 0.0.0.0 --port $(APP_PORT) --reload


# =========================
# Database (MongoDB & Seeds)
# =========================
.PHONY: db-connect db-seed db-dump db-restore seed-dev

seed-dev:
	@PYTHONPATH=. $(PYTHON) ./src/core/seed_dev/seed_faker.py

db-connect: ## Open interactive MongoDB shell (mongosh)
	@echo "$(CYAN)🔌 Conectando ao MongoDB ($(MONGO_URI)/$(MONGO_DB))...$(RESET)"
	@mongosh "mongodb://$(MONGO_URI)/$(MONGO_DB)"

db-seed: seed-dev ## Run development data seeders
	@echo "$(GREEN)🌱 Seed concluído com sucesso!$(RESET)"

db-dump: ## Create a database backup (dump) in database/dumps
	@echo "$(CYAN)📦 Criando backup da base de dados...$(RESET)"
	@mkdir -p database/dumps
	@mongodump --uri="mongodb://$(MONGO_URI)/$(MONGO_DB)" --out="database/dumps/dump-$$(date +%Y%m%d_%H%M%S)"
	@echo "$(GREEN)Backup concluído em database/dumps/$(RESET)"

db-restore: ## Restore database backup (Usage: make db-restore [BACKUP_PATH=...])
	@if [ -z "$(BACKUP_PATH)" ]; then \
		LATEST_BACKUP=$$(ls -d database/dumps/dump-* 2>/dev/null | sort | tail -n 1); \
		if [ -z "$$LATEST_BACKUP" ]; then \
			echo "$(YELLOW)❌ Erro: Nenhum backup encontrado em database/dumps/. Especifica o caminho com BACKUP_PATH=...$(RESET)"; \
			exit 1; \
		fi; \
		BACKUP_PATH="$$LATEST_BACKUP/$(MONGO_DB)"; \
	fi; \
	echo "$(CYAN)🔄 Restaurando base de dados a partir de $$BACKUP_PATH...$(RESET)"; \
	mongorestore --uri="mongodb://$(MONGO_URI)/$(MONGO_DB)" --drop "$$BACKUP_PATH"; \
	echo "$(GREEN)✅ Base de dados restaurada com sucesso!$(RESET)"


# =========================
# Testing & Code Quality
# =========================
.PHONY: test lint format vuln-check

test: ## Run test suite using pytest (or unittest fallback)
	@echo "$(CYAN)Running tests...$(RESET)"
	@if $(PYTHON) -m pytest --version >/dev/null 2>&1; then \
		$(PYTHON) -m pytest -v --capture=no; \
	else \
		$(PYTHON) -m unittest discover -s tests -p "*_test.py"; \
	fi

lint: ## Run code linter (ruff / flake8)
	@echo "$(CYAN)Running linter...$(RESET)"
	@if $(PYTHON) -m ruff --version >/dev/null 2>&1; then \
		$(PYTHON) -m ruff check src .; \
	else \
		$(PYTHON) -m flake8 src --max-line-length=120 || true; \
	fi

format: ## Auto-format code (ruff / autopep8 / black)
	@echo "$(CYAN)Formatting Python code...$(RESET)"
	@if $(PYTHON) -m ruff --version >/dev/null 2>&1; then \
		$(PYTHON) -m ruff format src .; \
	elif $(PYTHON) -m autopep8 --version >/dev/null 2>&1; then \
		$(PYTHON) -m autopep8 --in-place --aggressive --recursive --max-line-length=120 src; \
	fi

vuln-check: ## Check dependencies for known vulnerabilities (pip-audit)
	@echo "$(CYAN)Checking vulnerabilities with pip-audit...$(RESET)"
	@if ! $(VENV)/bin/pip show pip-audit > /dev/null 2>&1; then \
		$(PIP) install pip-audit; \
	fi
	@$(VENV)/bin/pip-audit


# =========================
# Docker Operations
# =========================
.PHONY: docker-build docker-up docker-down docker-logs docker-restart docker-clean

docker-build: ## Build Docker container image
	@echo "$(BLUE)Building Docker image $(DOCKER_IMAGE)...$(RESET)"
	docker build -t $(DOCKER_IMAGE) .

docker-up: ## Start services using Docker Compose
	@echo "$(CYAN)Starting containers in background...$(RESET)"
	docker compose up -d

docker-down: ## Stop and remove Docker Compose containers
	@echo "$(YELLOW)Stopping containers...$(RESET)"
	docker compose down -v --remove-orphans

docker-logs: ## View container logs in real time
	docker compose logs -f

docker-restart: ## Restart all Docker Compose services
	docker compose restart

docker-clean: ## Clean unused Docker resources
	docker system prune -f


# =========================
# Cleaning & Maintenance
# =========================
.PHONY: clean-cache clean-venv clean

clean-cache: ## Remove Python bytecode and cache files (__pycache__, .pyc, etc.)
	@echo "$(YELLOW)🧹 Limpando caches do Python...$(RESET)"
	find . -type d -name "__pycache__" -exec rm -rf {} + 2>/dev/null || true
	find . -type f -name "*.pyc" -delete 2>/dev/null || true
	find . -type f -name "*.pyo" -delete 2>/dev/null || true
	@echo "$(GREEN)Caches removidos.$(RESET)"

clean-venv: ## Remove the virtual environment directory
	@echo "$(YELLOW)🧹 Removendo ambiente virtual ($(VENV))...$(RESET)"
	rm -rf $(VENV)

clean: clean-cache clean-venv ## Perform full cleanup (cache + venv)


# =========================
# Helper
# =========================
.PHONY: help

help: ## Show this help message
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'
