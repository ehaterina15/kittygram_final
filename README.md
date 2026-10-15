# Kittygram: containerised with CI/CD

**A social app for cat owners, packaged in Docker containers with an automated GitHub Actions pipeline.**

Users register, add their cats with a photo, colour, birth year and achievements, and browse other people's cats. My part of this repository is the DevOps side: Dockerfiles, Docker Compose orchestration, an Nginx gateway and a CI/CD pipeline.

![Python](https://img.shields.io/badge/Python-3.9-3776AB?logo=python&logoColor=white)
![Django REST Framework](https://img.shields.io/badge/DRF-API-A30000?logo=django&logoColor=white)
![React](https://img.shields.io/badge/React-frontend-61DAFB?logo=react&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-gateway-009639?logo=nginx&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-CI%2FCD-2088FF?logo=githubactions&logoColor=white)

## Architecture

```
                ┌──────────────────────────────┐
  :9000  ──▶    │  gateway (Nginx)             │
                │   /api/, /admin/ → backend   │
                │   /media/        → media vol │
                │   /              → React SPA │
                └──────────────┬───────────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                      ▼
  backend (Django +      frontend (React build   db (PostgreSQL 13)
  DRF, Gunicorn)         copied to static vol)
```

Four services share three named volumes: `pg_data`, `static` and `media`.

## CI/CD pipeline

Every push to `main` runs [`.github/workflows/main.yml`](.github/workflows/main.yml):

1. **Backend tests**: lint with `ruff`, then Django tests against a PostgreSQL service container
2. **Frontend tests**: `npm ci` and the Jest test suite
3. **Build & push**: build the backend, frontend and gateway images and push them to Docker Hub
4. **Notify**: send a Telegram message once the images are published

Required repository secrets: `DOCKER_USERNAME`, `DOCKER_PASSWORD`, `TELEGRAM_TO`, `TELEGRAM_TOKEN`.

## Run locally

```bash
git clone https://github.com/ehaterina15/kittygram_final.git
cd kittygram_final
cp .env.example .env            # then edit the values
docker compose up -d --build
docker compose exec backend python manage.py migrate
docker compose exec backend python manage.py collectstatic --noinput
```

The app is then available at http://localhost:9000.

### Environment variables

| Variable | Description |
|---|---|
| `SECRET_KEY`, `DEBUG`, `ALLOWED_HOSTS` | Django settings |
| `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` | Database credentials |
| `DB_HOST`, `DB_PORT` | Database address (`db`, `5432` inside Compose) |

## API overview

| Endpoint | Description |
|---|---|
| `/api/cats/` | CRUD for cats |
| `/api/achievements/` | Cat achievements |
| `/api/users/` | Registration and profiles (Djoser) |
| `/api/token/login/`, `/api/token/logout/` | Token authentication (Djoser) |

## Context

Final DevOps assignment of the Python backend course at [Yandex Practicum](https://practicum.yandex.com/). The Django and React application code was provided as a starting point; I did the containerisation, Compose setup, Nginx configuration and CI/CD.

## Author

**Ekaterina Mashentseva** · [GitHub](https://github.com/ehaterina15)
