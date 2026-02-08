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

Open a terminal in your Strapi project directory and run the skill in Claude Code using the slash command:

```
/dockerize-strapi
```

## Example Session

Here's a real example of the skill in action on a Strapi v5 TypeScript project:

**1. Invoke the skill**

```
> /dockerize-strapi
```

**2. The skill auto-detects your project**

It reads `package.json`, checks for lock files, and detects TypeScript — then reports what it found:

```
Detected configuration:
  - Strapi v5 (@strapi/strapi: 5.35.0)
  - TypeScript project
  - NPM package manager
  - Project name: test-docker-skill
```

**3. You answer a few prompts**

The skill asks you to pick your preferences via interactive prompts:

- **Database**: PostgreSQL (Recommended), MySQL, or MariaDB
- **Environment**: Development, Production, or Both
- **DB credentials**: Use defaults (`strapi/strapi/strapi`) or customize

**4. The skill installs the database driver**

It removes conflicting packages (e.g. `better-sqlite3`) and installs the correct driver (`pg` for PostgreSQL) *before* generating Docker files, so the lock file is up to date inside the image.

```
npm uninstall better-sqlite3
npm install pg
```

**5. All files are generated**

```
Dockerize Strapi — Complete!

  Strapi Version:  5.35.0
  Database:        PostgreSQL (postgres)
  Package Manager: NPM
  Environment:     Both (development + production)

  Generated files:
    - Dockerfile              (development)
    - Dockerfile.prod         (production)
    - docker-compose.yml      (with PostgreSQL + Adminer)
    - .dockerignore
    - config/env/development/database.ts
    - config/env/production/database.ts
    - .env                    (updated)
```

**6. Start it up**

```bash
docker compose up -d --build
```

Strapi will be available at `http://localhost:1337` and Adminer (database browser) at `http://localhost:9090`.

## Generated Files

| File | Description |
|------|-------------|
| `Dockerfile` | Development Dockerfile with hot-reload |
| `Dockerfile.prod` | Multi-stage production Dockerfile (if selected) |
| `docker-compose.yml` | Compose config with Strapi + database + Adminer |
| `.dockerignore` | Excludes node_modules, .git, .env, etc. |
| `config/database.ts\|js` | Database connection config using env vars |
| `.env` | Updated with database connection variables |
