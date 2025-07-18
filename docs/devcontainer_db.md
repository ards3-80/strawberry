# DevContainer Database Health Check System

This document describes the database health check system implemented in `scripts/devcontainer_db_health_check.sh`.

## Overview

The health check script ensures that the PostgreSQL database is properly configured and accessible within the development container environment. It performs a three-stage verification process with comprehensive error handling.

## Prerequisites

The following environment variables must be available to the database container, typically via a `.env` file:

- `POSTGRES_USER`: Database user
- `POSTGRES_PASSWORD`: Database password
- `POSTGRES_DB`: Target database name

> **Note:** In most setups, the `.env` file is automatically generated or populated by GitHub secrets (in CI/CD) or Codespaces secrets. Manual setting is usually not required unless running locally outside these environments.

## Process Stages

### 1. Environment Validation

- Checks for required environment variables
- Constructs the PostgreSQL connection URL
- Exits with code 2 if any required variables are missing
- Shows masked password status for security

### 2. Container Management

- Changes to repository root for consistent path resolution
- Starts Docker containers via docker-compose
- Uses `--force-recreate` flag to ensure clean state
- Container configuration is read from `.devcontainer/docker-compose.yml`

### 3. Database Readiness Check

- Implements a progressive wait system (max 30 seconds)
- 15 iterations with 2-second intervals
- Uses `pg_isready` to verify database availability
- Provides visual feedback during wait period
- Exits with code 1 if database isn't ready after timeout

### 4. Prisma Connection Verification

- Tests database connectivity using Prisma
- Passes environment variables to application container
- Executes `prisma db push` as connection test
- Reports success (exit 0) or failure (exit 1)

## Exit Codes

- 0: Success - Database is ready and Prisma connection verified
- 1: Failure - Database connection or Prisma verification failed
- 2: Configuration Error - Missing environment variables

## Upgrades

### Shell Migration (zsh to bash)

The script is being migrated from zsh to bash for wider compatibility and standardization:

- **Current**: Uses zsh shell (`#!/bin/zsh`)
- **Target**: Will use bash shell (`#!/bin/bash`)
- **Rationale**: bash is more universally available and is the default shell in most container environments
- **Impact**: No functional changes; syntax is compatible with both shells

### DevContainer Integration Improvements

The following improvements will better align the script with devcontainer best practices:

1. **Environment Variable Management**

   - **Current**: Relies on shell environment variables
   - **Target**: Use Docker Compose's `.env` file approach
   - **Rationale**: Align with Docker Compose and devcontainer conventions
   - **Impact**: More consistent environment handling across containers

2. **Container Lifecycle Management**

   - **Current**: Forcefully recreates containers with `--force-recreate`
   - **Target**: Check container state before operations
   - **Rationale**: Respect VS Code's container management
   - **Impact**: More efficient container handling, faster operations

3. **Path Resolution**

   - **Current**: Uses hardcoded relative paths
   - **Target**: Use devcontainer.json configuration
   - **Rationale**: Support flexible docker-compose.yml locations
   - **Impact**: More robust path handling

4. **Workspace Integration**

   - **Current**: Forces repository root
   - **Target**: Use devcontainer's workspaceFolder
   - **Rationale**: Follow VS Code workspace conventions
   - **Impact**: Better integration with VS Code environment

5. **Container Naming**

   - **Current**: Assumes default Docker Compose naming
   - **Target**: Support devcontainer naming conventions
   - **Rationale**: Prevent naming conflicts
   - **Impact**: More reliable container identification

6. **Health Check Implementation**

   - **Current**: Custom health check logic
   - **Target**: Use Docker's built-in health checks
   - **Rationale**: Leverage platform capabilities
   - **Impact**: More standardized health monitoring

7. **Lifecycle Integration**
   - **Current**: Standalone script execution
   - **Target**: Integration with devcontainer hooks
   - **Rationale**: Better automation and initialization
   - **Impact**: Smoother developer experience

## Usage

The script should be run from the repository root:

```bash
./scripts/devcontainer_db_health_check.sh
```

This is typically executed during development container setup or when verifying database connectivity issues.

## Actionables

The following actionable items correspond to the upgrades and improvements outlined above. Check off each item as it is implemented:

- [ ] Migrate script from zsh to bash for compatibility
- [ ] Source environment variables from `.env` file (Docker Compose approach)
- [ ] Check container state before operations (avoid forced recreation)
- [ ] Use devcontainer.json for path resolution (support flexible docker-compose.yml locations)
- [ ] Use devcontainer's workspaceFolder for workspace integration
- [ ] Support devcontainer naming conventions for containers
- [ ] Use Docker's built-in health checks for database readiness
- [ ] Integrate script with devcontainer lifecycle hooks for automation
