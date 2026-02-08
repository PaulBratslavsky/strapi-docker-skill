---
name: dockerize-strapi
description: >-
  Generate Docker and docker-compose configuration for a Strapi project.
  Trigger when the user says "dockerize strapi", "add Docker support",
  "containerize", or wants Docker/docker-compose files for Strapi.
  Supports PostgreSQL, MySQL, MariaDB. Detects Strapi version (v4/v5)
  and adapts configuration automatically.
compatibility: Requires Docker. Works with Strapi v4 and v5 projects.
allowed-tools: Bash Read Write Edit Glob Grep AskUserQuestion
metadata:
  author: strapi-community
  version: "1.0.0"
---

# Dockerize Strapi

Generate production-ready Docker configuration for Strapi projects.

## Step 1: Detect Project Configuration

Read these files from the current working directory:

1. `package.json` — extract Strapi version from `dependencies["@strapi/strapi"]`
2. Check for `yarn.lock` (Yarn) vs `package-lock.json` (npm) to detect package manager
3. Check for `tsconfig.json` → TypeScript (`.ts`), otherwise JavaScript (`.js`)

Determine **Strapi major version**:
- `5.x.x` → **Strapi 5**: Node 20+, `mysql2` driver, requires `APP_KEYS`, `API_TOKEN_SALT`, `TRANSFER_TOKEN_SALT`
- `4.x.x` → **Strapi 4**: Node 18+, `mysql` driver

Report detected configuration to the user before continuing.

## Step 2: Gather User Preferences

Use `AskUserQuestion` to ask all of the following at once:

**Database** (header: "Database"):
- PostgreSQL (Recommended)
- MySQL
- MariaDB

**Docker Compose** (header: "Compose"):
- Yes, generate docker-compose.yml (Recommended)
- No, Dockerfile only

**Environment** (header: "Environment"):
- Development
- Production
- Both

## Step 3: Gather Database Configuration

Use `AskUserQuestion` to ask:

**Setup mode** (header: "DB Config"):
- Use defaults (Recommended) — applies the defaults below
- Customize — then ask for each value individually

Defaults:

| Setting  | PostgreSQL | MySQL / MariaDB |
|----------|-----------|-----------------|
| Host     | localhost | localhost       |
| Port     | 5432      | 3306            |
| Name     | strapi    | strapi          |
| Username | strapi    | strapi          |
| Password | strapi    | strapi          |

Also ask for the **project name** (default: current directory name).

## Step 4: Install Database Driver

Before generating Dockerfiles, install the required database driver into the project so that `package.json` and the lockfile include it. This ensures the driver is available when `npm install` / `yarn install` runs inside the Docker image.

First remove conflicting packages (ignore errors if not installed):
- Installing `pg` → remove `mysql`, `mysql2`, `better-sqlite3`
- Installing `mysql2` → remove `pg`, `mysql`, `better-sqlite3`
- Installing `mysql` → remove `pg`, `mysql2`, `better-sqlite3`

Then install the correct package:

| Database   | Strapi 5 | Strapi 4 |
|------------|----------|----------|
| PostgreSQL | `pg`     | `pg`     |
| MySQL      | `mysql2` | `mysql`  |
| MariaDB    | `mysql2` | `mysql`  |

Use the detected package manager (`yarn add` / `npm install`).

**This step must happen before generating Dockerfiles** so the updated `package.json` and lockfile are copied into the Docker image during build.

## Step 5: Generate .dockerignore

Write `.dockerignore`:

```
.tmp/
.cache/
.git/
build/
node_modules/
data/
.env
```

## Step 6: Generate Dockerfile(s)

Read the appropriate template from the `templates/` directory relative to this skill file:

- Development → read `templates/Dockerfile.dev`
- Production → read `templates/Dockerfile.prod`
- Both → generate `Dockerfile` from dev template AND `Dockerfile.prod` from prod template

Apply these substitutions:

| Placeholder        | Yarn                                                              | npm                                                                |
|--------------------|-------------------------------------------------------------------|--------------------------------------------------------------------|
| `{{NODE_VERSION}}` | `20` (Strapi 5) or `18` (Strapi 4)                               | same                                                               |
| `{{PKG_FILES}}`    | `package.json yarn.lock`                                          | `package.json package-lock.json`                                   |
| `{{INSTALL}}`      | `yarn config set network-timeout 600000 -g && yarn install`       | `npm config set fetch-retry-maxtimeout 600000 -g && npm install`   |
| `{{INSTALL_PROD}}` | `yarn config set network-timeout 600000 -g && yarn install --production` | `npm config set fetch-retry-maxtimeout 600000 -g && npm install --only=production` |
| `{{BUILD}}`        | `yarn build`                                                      | `npm run build`                                                    |
| `{{DEV_CMD}}`      | `["yarn", "develop"]`                                             | `["npm", "run", "develop"]`                                        |
| `{{START_CMD}}`    | `["yarn", "start"]`                                               | `["npm", "run", "start"]`                                          |

## Step 7: Generate docker-compose.yml

Skip if user chose "No" in Step 2.

Read `templates/docker-compose.yml` from the skill directory and apply substitutions:

| Placeholder               | Value                                                                 |
|---------------------------|-----------------------------------------------------------------------|
| `{{PROJECT_NAME}}`        | project name from Step 3                                              |
| `{{PROJECT_NAME_LOWER}}`  | project name lowercased                                               |
| `{{PROJECT_NAME_CAP}}`    | project name capitalized                                              |
| `{{LOCKFILE}}`            | `yarn.lock` or `package-lock.json`                                    |
| `{{DB_PORT}}`             | database port from Step 3                                             |

### Database-specific sections

**PostgreSQL:**
```yaml
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${DATABASE_USERNAME}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
      POSTGRES_DB: ${DATABASE_NAME}
    volumes:
      - {{PROJECT_NAME_LOWER}}-data:/var/lib/postgresql/data/
```

**MySQL:**
- Strapi 5: `command: "--character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci"`
- Strapi 4: `command: "--character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci --default-authentication-plugin=mysql_native_password"`
```yaml
    image: mysql:latest
    command: "{{MYSQL_COMMAND}}"
    environment:
      MYSQL_USER: ${DATABASE_USERNAME}
      MYSQL_ROOT_PASSWORD: ${DATABASE_PASSWORD}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_ROOT_HOST: '%'
    volumes:
      - {{PROJECT_NAME_LOWER}}-data:/var/lib/mysql
```

**MariaDB:**
```yaml
    image: mariadb:latest
    environment:
      MYSQL_USER: ${DATABASE_USERNAME}
      MYSQL_ROOT_PASSWORD: ${DATABASE_PASSWORD}
      MYSQL_PASSWORD: ${DATABASE_PASSWORD}
      MYSQL_DATABASE: ${DATABASE_NAME}
      MYSQL_ROOT_HOST: '%'
    volumes:
      - {{PROJECT_NAME_LOWER}}-data:/var/lib/mysql
```

### Strapi 5 extra environment variables

If Strapi 5, add these to the strapi service environment block:
```yaml
      APP_KEYS: ${APP_KEYS}
      API_TOKEN_SALT: ${API_TOKEN_SALT}
      TRANSFER_TOKEN_SALT: ${TRANSFER_TOKEN_SALT}
```

## Step 8: Generate Database Configuration

If `config/database.{{ext}}` exists, copy it to `config/database.backup` first.

### Database client value:
- PostgreSQL → `postgres`
- MySQL/MariaDB + Strapi 5 → `mysql2`
- MySQL/MariaDB + Strapi 4 → `mysql`

### TypeScript (`config/database.ts`):
```typescript
export default ({ env }) => ({
  connection: {
    client: '{{CLIENT}}',
    connection: {
      host: env('DATABASE_HOST', '{{DB_HOST}}'),
      port: env.int('DATABASE_PORT', {{DB_PORT}}),
      database: env('DATABASE_NAME', '{{DB_NAME}}'),
      user: env('DATABASE_USERNAME', '{{DB_USER}}'),
      password: env('DATABASE_PASSWORD', '{{DB_PASSWORD}}'),
      ssl: env.bool('DATABASE_SSL', false)
    }
  }
});
```

### JavaScript (`config/database.js`):
```javascript
module.exports = ({ env }) => ({
  connection: {
    client: '{{CLIENT}}',
    connection: {
      host: env('DATABASE_HOST', '{{DB_HOST}}'),
      port: env.int('DATABASE_PORT', {{DB_PORT}}),
      database: env('DATABASE_NAME', '{{DB_NAME}}'),
      user: env('DATABASE_USERNAME', '{{DB_USER}}'),
      password: env('DATABASE_PASSWORD', '{{DB_PASSWORD}}'),
      ssl: env.bool('DATABASE_SSL', false)
    }
  }
});
```

## Step 9: Update .env

Check if `.env` contains `@strapi-community/dockerize variables`. If yes, update existing values in place. If no, append:

```env
# @strapi-community/dockerize variables
DATABASE_HOST={{DB_HOST}}
DATABASE_PORT={{DB_PORT}}
DATABASE_NAME={{DB_NAME}}
DATABASE_USERNAME={{DB_USER}}
DATABASE_PASSWORD={{DB_PASSWORD}}
NODE_ENV={{NODE_ENV_VALUE}}
DATABASE_CLIENT={{CLIENT}}
# @strapi-community/dockerize end variables
```

`NODE_ENV` value: `development` if environment is Development or Both, `production` if Production only.

## Step 10: Print Summary

```
Dockerize Strapi — Complete!

  Strapi Version:  {{version}}
  Database:        {{database}} ({{client}})
  Package Manager: {{packageManager}}
  Environment:     {{environment}}

  Generated files:
    - Dockerfile
    - docker-compose.yml  (if selected)
    - .dockerignore
    - config/database.{{ext}}
    - .env (updated)

  Next steps:
    docker compose up --build
```
