# Docker Guide

This guide runs the main frontend, main backend, PostgreSQL BI, and Apache Superset.

## Required Files

Create deployment env:

```bash
cd medical-chatbot-deployment
cp .env.example .env
```

Update sensitive values in `medical-chatbot-deployment/.env`, especially:

```env
BI_POSTGRES_PASSWORD=change_me_bi_password
BI_DATABASE_URL=postgresql://bi_user:change_me_bi_password@bi-postgres:5432/medical_chatbot_bi
SUPERSET_SECRET_KEY=change_me_superset_secret
```

Create `../back-end/.env` from `../back-end/.env.example`.

Docker files used by this deployment setup:

```txt
medical-chatbot-deployment/docker-compose.yml  # Full stack orchestration
back-end/Dockerfile                            # Node.js backend image
front-end/Dockerfile                           # Vite frontend image
back-end/superset/Dockerfile                   # Superset image with project config
```

The backend image is built from `../back-end`, installs dependencies from
`package.json` and `yarn.lock`, copies `src/` plus BI SQL files, exposes port
`3000`, and defaults to `yarn dev`. In Docker Compose, the backend source is
mounted into `/app` and the command remains `yarn dev`.

Important values:

```env
MONGODB_URI=your_mongodb_uri
JWT_SECRET=your_secret
JWT_ACCESS_TOKEN_SECRET_ADMIN=your_admin_access_secret
JWT_REFRESH_TOKEN_SECRET_ADMIN=your_admin_refresh_secret
JWT_ACCESS_TOKEN_SECRET_CLIENT=your_client_access_secret
JWT_REFRESH_TOKEN_SECRET_CLIENT=your_client_refresh_secret
AI_SERVER_URL=your_ai_server_url
AI_API_KEY=your_ai_api_key

SUPERSET_USERNAME=admin
SUPERSET_PASSWORD=admin
SUPERSET_DASHBOARD_SYSTEM=bi-system
SUPERSET_DASHBOARD_CHATBOT=bi-chatbot
SUPERSET_DASHBOARD_SAFETY=bi-safety
SUPERSET_DASHBOARD_MODELS=bi-models
```

Create `../front-end/.env`:

```env
VITE_API_ROOT=http://localhost:3000
```

## Start Full Stack

From the deployment folder:

```bash
docker compose up -d --build
```

If your machine uses legacy Compose:

```bash
docker-compose up -d --build
```

URLs:

```txt
Frontend: http://localhost:5173
Backend:  http://localhost:3000
Superset: http://localhost:8088
BI DB:    localhost:5432
```

## Initialize Superset

Run once after first start:

```bash
docker compose exec superset superset db upgrade
docker compose exec superset superset fab create-admin \
  --username admin \
  --firstname Admin \
  --lastname User \
  --email admin@example.com \
  --password admin
docker compose exec superset superset init
docker compose exec superset superset set-database-uri \
  -d medical_chatbot_bi \
  -u postgresql+psycopg2://$BI_POSTGRES_USER:$BI_POSTGRES_PASSWORD@bi-postgres:5432/$BI_POSTGRES_DB
```

With legacy Compose, replace `docker compose` by `docker-compose`.

## Initialize BI Data

The backend container uses the internal PostgreSQL hostname `bi-postgres`.

```bash
docker compose exec backend yarn bi:schema
docker compose exec backend yarn bi:sync
docker compose exec backend yarn bi:views
docker compose exec backend yarn bi:superset
docker compose exec backend yarn bi:dashboards
```

With legacy Compose, replace `docker compose` by `docker-compose`.

## Open Embedded BI

Open the admin site:

```txt
http://localhost:5173/admin/bi
```

## Refresh BI Data Later

When MongoDB data changes:

```bash
docker compose exec backend yarn bi:sync
docker compose exec backend yarn bi:views
```

## Stop

```bash
docker compose down
```

Stop and delete volumes:

```bash
docker compose down -v
```
