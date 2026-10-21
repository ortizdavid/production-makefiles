# ==============================================================================
# Production Makefile for PHP Applications (MySQL & Custom MVC Stack)
# ==============================================================================

SHELL := /bin/bash

# =========================
# Load Environment Variables
# =========================
ifneq (,$(wildcard .env))
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
DB_NAME      ?= avaliacao_desempenho_db
DB_USER      ?= root
DB_HOST      ?= 127.0.0.1
PORT         ?= 8000
TIMESTAMP    := $(shell date +%Y%m%d_%H%M%S)
LATEST_DUMP  := $(shell ls -t database/dumps/*.sql 2>/dev/null | head -n 1)


# =========================
# Core Execution & Lint
# =========================
.PHONY: run lint

run: ## Start the built-in PHP development server
	@echo "$(CYAN)🚀 Starting server at http://localhost:$(PORT)...$(RESET)"
	/usr/bin/php8.4 -S localhost:$(PORT)

lint: ## Verify PHP syntax errors across the project (excluding vendor)
	@echo "$(CYAN)🔍 Checking PHP files for syntax errors...$(RESET)"
	@find . -name "*.php" -not -path "./vendor/*" -exec php -l {} \;


# =========================
# Database Management (MySQL)
# =========================
.PHONY: db-init db-setup db-seed setup-db fresh db-dump db-restore db-connect db-status db-cli

db-init: ## Initialize database and user from init.sql
	@echo "$(CYAN)👤 Initializing database and user...$(RESET)"
	@if [ -f "database/structure/init.sql" ]; then \
		mysql -h $(DB_HOST) -u root $(if $(ROOT_PASS),-p$(ROOT_PASS),) < "database/structure/init.sql"; \
		echo "$(GREEN)✅ Initialization completed!$(RESET)"; \
	fi

db-setup: ## Create table structures from database/structure/*.sql
	@echo "$(CYAN)🛠️ Creating table structures in database $(DB_NAME)...$(RESET)"
	@if [ -f "database/structure/tables.sql" ]; then \
		echo "📂 Executing: database/structure/tables.sql"; \
		mysql -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) $(DB_NAME) < "database/structure/tables.sql"; \
	fi
	@for file in database/structure/*.sql; do \
		if [ "$$file" != "database/structure/tables.sql" ] && [ "$$file" != "database/structure/init.sql" ] && [ -f "$$file" ]; then \
			echo "📂 Executing: $$file"; \
			mysql -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) $(DB_NAME) < "$$file"; \
		fi; \
	done
	@echo "$(GREEN)✅ Structure created successfully!$(RESET)"

db-seed: ## Run development data seeders
	@echo "$(CYAN)🌱 Applying database seeds to $(DB_NAME)...$(RESET)"
	@for file in database/seed/*.sql; do \
		if [ -f "$$file" ]; then \
			echo "📂 Executing: $$file"; \
			mysql -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) $(DB_NAME) < "$$file"; \
		fi; \
	done
	@echo "$(GREEN)✅ Seeds applied successfully!$(RESET)"

setup-db: db-setup db-seed ## Full setup (tables and seeds)
	@echo "$(GREEN)✨ Database fully configured!$(RESET)"

fresh: setup-db ## Alias for full database setup

db-dump: ## Create full database backup (structure, data, routines, events)
	@mkdir -p database/dumps
	@echo "$(CYAN)📦 Generating full backup of database $(DB_NAME)...$(RESET)"
	mysqldump --no-tablespaces --routines --events -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) $(DB_NAME) > database/dumps/database_dump_$(TIMESTAMP).sql
	@echo "$(GREEN)✅ Dump generated successfully at database/dumps/database_dump_$(TIMESTAMP).sql$(RESET)"

db-restore: ## Restore database from latest dump or file (Usage: make db-restore file=...)
	@if [ -z "$(file)" ] && [ -z "$(LATEST_DUMP)" ]; then \
		echo "$(YELLOW)❌ No dump file found in database/dumps/$(RESET)"; \
		exit 1; \
	fi
	@TARGET_FILE=$$([ -n "$(file)" ] && echo "database/dumps/$(file)" || echo "$(LATEST_DUMP)"); \
	echo "$(CYAN)🔍 Selected file for restore: $$TARGET_FILE$(RESET)"; \
	echo "♻️ Restoring database $(DB_NAME)..."; \
	mysql -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) $(DB_NAME) < $$TARGET_FILE; \
	echo "$(GREEN)✅ Database restored successfully!$(RESET)"

db-status: ## Check MySQL server status via mysqladmin
	@echo "$(CYAN)🔍 Checking MySQL status...$(RESET)"
	mysqladmin -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) status

db-connect: ## Connect to MySQL interactive shell
	@echo "$(CYAN)🔌 Connecting to MySQL database...$(RESET)"
	mysql -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) $(DB_NAME)

db-cli: ## Open interactive database CLI using mycli
	mycli -h $(DB_HOST) -u $(DB_USER) $(if $(DB_PASS),-p$(DB_PASS),) $(DB_NAME)


# =========================
# Docker & Services Management
# =========================
.PHONY: docker-up docker-start docker-stop docker-restart docker-status docker-logs mysql-start pmy-start pmy-stop

docker-up: ## Create and start MySQL Docker container via compose
	@echo "$(BLUE)🐳 Creating and starting MySQL Docker container...$(RESET)"
	docker compose up -d
	@echo "$(GREEN)✅ MySQL container is up and running!$(RESET)"

docker-start: ## Start existing MySQL container
	@echo "$(CYAN)▶️ Starting existing MySQL container...$(RESET)"
	docker start avaliacao_mysql

docker-stop: ## Stop MySQL container
	@echo "$(YELLOW)⏹️ Stopping MySQL container...$(RESET)"
	docker stop avaliacao_mysql

docker-restart: ## Restart MySQL container
	@echo "$(CYAN)🔄 Restarting MySQL container...$(RESET)"
	docker restart avaliacao_mysql

docker-status: ## Check MySQL container status
	@echo "$(CYAN)🔍 MySQL container status:$(RESET)"
	docker ps -f name=avaliacao_mysql

docker-logs: ## Tail MySQL container logs
	docker compose logs --tail=50 -f

mysql-start: ## Start native local MySQL service (Linux systemd)
	@echo "$(CYAN)🟢 Starting native MySQL service...$(RESET)"
	sudo service mysql start

pmy-start: ## Start phpMyAdmin container on port 8080
	@echo "$(CYAN)🚀 Starting phpMyAdmin...$(RESET)"
	docker run --name phpmyadmin -d -p 8080:80 -e PMA_HOST=172.17.0.1 phpmyadmin/phpmyadmin || docker start phpmyadmin
	@echo "$(GREEN)✅ phpMyAdmin running on port 8080!$(RESET)"

pmy-stop: ## Stop and remove phpMyAdmin container
	@echo "$(YELLOW)🛑 Stopping phpMyAdmin...$(RESET)"
	docker stop phpmyadmin && docker rm phpmyadmin
	@echo "$(GREEN)✅ phpMyAdmin removed.$(RESET)"


# =========================
# Helper
# =========================
.PHONY: help

help: ## Show this help message with available targets
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'
