# dockerize-strapi

A Claude Code skill that generates production-ready Docker configuration for Strapi v4 and v5 projects. It auto-detects your project setup (Strapi version, package manager, TypeScript/JavaScript) and generates Dockerfiles, docker-compose.yml, database config, and environment variables.

## Features

- Supports **PostgreSQL**, **MySQL**, and **MariaDB**
- Detects **Strapi v4** and **v5** automatically
- Works with **Yarn** and **npm**
- Generates **development**, **production**, or **both** Dockerfile configurations
- Creates `docker-compose.yml` with the selected database service
- Updates `config/database.ts|js` and `.env` with proper connection settings

## Installation

Copy or clone this repository into your Claude Code skills directory:

```
~/.claude/skills/dockerize-strapi/
```

## Usage

Open a terminal in your Strapi project directory and invoke the skill in Claude Code:

```
dockerize my strapi project
```

Other trigger phrases that work:

```
add docker to strapi
```

```
dockerize strapi
```

```
containerize my strapi app
```

The skill will walk you through a few prompts to choose your database, environment, and configuration, then generate all the necessary files.

## Example Session

```
You: dockerize strapi

Claude: Detected configuration:
  - Strapi 5.x (Node 20)
  - Package manager: yarn
  - Language: TypeScript

  [Prompts for database, environment, and config preferences]

Claude: Dockerize Strapi — Complete!

  Strapi Version:  5.x
  Database:        PostgreSQL (postgres)
  Package Manager: yarn
  Environment:     Both

  Generated files:
    - Dockerfile
    - Dockerfile.prod
    - docker-compose.yml
    - .dockerignore
    - config/database.ts
    - .env (updated)

  Next steps:
    docker compose up --build
```

## Generated Files

| File | Description |
|------|-------------|
| `Dockerfile` | Development Dockerfile with hot-reload |
| `Dockerfile.prod` | Multi-stage production Dockerfile (if selected) |
| `docker-compose.yml` | Compose config with Strapi + database service |
| `.dockerignore` | Excludes node_modules, .git, .env, etc. |
| `config/database.ts\|js` | Database connection config using env vars |
| `.env` | Updated with database connection variables |
