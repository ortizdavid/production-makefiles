# ==============================================================================
# Production Makefile for Go Applications
# ==============================================================================

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
# Variables
# =========================
APP_NAME     := $(or $(APP_NAME), app)
APP_PORT     := $(or $(APP_PORT), 8080)
SWAG_DIR     := ./docs
DOCKER_IMAGE := $(APP_NAME):latest

DATABASE_URL := postgres://$(DB_USER):$(DB_PASSWORD)@$(DB_HOST):$(DB_PORT)/$(DB_NAME)?sslmode=$(DB_SSLMODE)


# =========================
# Environment Setup
# =========================
.PHONY: setup-dev rebuild-dev

setup-dev: ## Setup development environment (Migrate + Seed)
	@echo "$(CYAN)Setting up DEV environment...$(RESET)"
	@$(MAKE) migrate-apply
	@$(MAKE) seed
	@echo "$(GREEN)DEV environment ready!$(RESET)"

rebuild-dev: ## Dangerous: Drop all tables and run setup-dev from scratch
	@$(MAKE) migrate-drop-all
	@$(MAKE) setup-dev


# =========================
# Run & Build
# =========================
.PHONY: run build clean

run: ## Run the API application locally
	@echo "$(CYAN)Starting $(APP_NAME) on port $(APP_PORT)...$(RESET)"
	go run ./cmd/api/main.go

build: ## Build the optimized application binary
	@echo "$(BLUE)Building binary...$(RESET)"
	CGO_ENABLED=0 go build -ldflags="-s -w" -o bin/$(APP_NAME) ./cmd/api/main.go

clean: ## Remove binaries and temporary build files
	@echo "$(YELLOW)Cleaning build artifacts...$(RESET)"
	@rm -rf bin/ $(APP_NAME)


# =========================
# Database & Migrations
# =========================
.PHONY: migrate migrate-apply migrate-hash migrate-drop-all seed

migrate: ## Create a new migration (Atlas/GORM)
	@$(MAKE) migrate-hash
	@echo "$(CYAN)Creating new migration...$(RESET)"
	atlas migrate diff --env gorm

migrate-apply: ## Apply pending migrations
	@$(MAKE) migrate-hash
	@echo "$(CYAN)Applying migrations...$(RESET)"
	atlas migrate apply \
	  --url "$(DATABASE_URL)" \
	  --dir "file://database/migrations" \
	  --revisions-schema "public" \
	  --allow-dirty

migrate-hash: ## Update the migration directory integrity hash
	@echo "$(YELLOW)Hashing migrations...$(RESET)"
	@atlas migrate hash --env gorm

migrate-drop-all: ## Dangerous: Wipe the entire database schema
	@echo "$(YELLOW)Warning: This will delete EVERYTHING.$(RESET)"
	@read -p "Are you sure? [y/N] " ans && [ $${ans:-N} = y ]
	atlas schema clean --env gorm --url "$(DATABASE_URL)"

seed: ## Run Go database seeder
	@echo "$(CYAN)Running database seeders...$(RESET)"
	go run ./cmd/seed/main.go


# =========================
# Code Quality & Docs
# =========================
.PHONY: lint vuln-check format test swagger

lint: ## Run golangci-lint for static analysis
	@echo "$(CYAN)Checking golangci-lint...$(RESET)"
	@which golangci-lint > /dev/null || go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest
	golangci-lint run --timeout=5m

vuln-check: ## Run security vulnerability analysis on Go dependencies
	@echo "$(CYAN)Checking govulncheck...$(RESET)"
	@which govulncheck > /dev/null || go install golang.org/x/vuln/cmd/govulncheck@latest
	govulncheck ./...

format: ## Format code and optimize imports
	@echo "$(CYAN)Formatting code...$(RESET)"
	goimports -w .
	gofmt -s -w .

test: ## Run unit tests with coverage
	go test -v -race -cover ./...

swagger: ## Generate Swagger/OpenAPI documentation
	@echo "$(CYAN)Generating Swagger documentation...$(RESET)"
	@which swag > /dev/null || go install github.com/swaggo/swag/cmd/swag@latest
	swag init -g ./cmd/api/main.go -o $(SWAG_DIR) --parseDependency


# =========================
# Docker Operations
# =========================
.PHONY: docker-build docker-up docker-down docker-logs docker-clean

docker-build: ## Build the production container image
	docker build -t $(DOCKER_IMAGE) .

docker-up: ## Start containers in background
	docker compose up -d

docker-down: ## Stop and remove containers/networks
	docker compose down --remove-orphans

docker-logs: ## Tail container logs
	docker compose logs -f

docker-clean: ## Remove unused Docker objects and cache
	docker system prune -f


# =========================
# CI/CD Helpers
# =========================
.PHONY: ci-prepare ci-lint ci-test ci-build

ci-prepare: ## Prepare CI environment (download & verify dependencies)
	go mod download
	go mod verify

ci-lint: ## Run linter formatted for CI
	golangci-lint run --timeout=5m --out-format=github-actions

ci-test: ## Run tests with coverage output for CI
	go test -fail-fast -v -race -covermode=atomic -coverprofile=coverage.out ./...

ci-build: ## Build Linux AMD64/ARM64 binaries
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -o bin/api-linux-amd64 ./cmd/api
	CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -ldflags="-s -w" -o bin/api-linux-arm64 ./cmd/api


# =========================
# Helper
# =========================
.PHONY: help

help: ## Show this help message
	@echo "Usage: make [target]"
	@echo ""
	@echo "Targets:"
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "  \033[36m%-20s\033[0m %s\n", $$1, $$2}'
