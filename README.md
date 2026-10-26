# wp-stack

> WordPress on PostgreSQL, wrapped in Docker Compose, and grown step by step into a production-style stack: nginx in front, Ansible to deploy, HTTPS, n8n on a subdomain, and GitHub Actions for continuous deployment.

![Docker](https://img.shields.io/badge/Docker_Compose-2496ED?logo=docker&logoColor=white)
![WordPress](https://img.shields.io/badge/WordPress_6.5-21759B?logo=wordpress&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL_16-4169E1?logo=postgresql&logoColor=white)
![Status](https://img.shields.io/badge/status-work_in_progress-orange)

This is a hands-on DevOps learning project. Each milestone below is one task, and the repo grows one task at a time.

---

## Why this is interesting

WordPress only speaks MySQL out of the box. This stack runs it on **PostgreSQL** instead, using the [PG4WP](https://github.com/PostgreSQL-For-Wordpress/postgresql-for-wordpress) drop-in, which translates WordPress's MySQL queries into PostgreSQL on the fly.

## Architecture

Current state (task 2):

```
 browser ──► :8080 ──► wordpress (Apache + PHP 8.2 + PG4WP) ──► db (PostgreSQL 16)
```

Target state (after task 6):

```
                    ┌──► wordpress ──► postgres
 internet ──► nginx ┤        (internal network only)
   :80/:443         └──► n8n
   x.com / n8n.x.com
```

## What's inside

| Path | Purpose |
|---|---|
| `docker-compose.yml` | Defines the `db` (Postgres 16) and `wordpress` services, a health check so WordPress waits for the database, and named volumes for persistent data |
| `wordpress/Dockerfile` | Extends `wordpress:6.5-php8.2-apache` with the PHP `pgsql` extension and installs PG4WP as `wp-content/db.php` |
| `.env.example` | Template for database name, user and password |
| `.gitattributes` | Forces LF line endings so files behave the same on Windows and Linux |

## Quick start

**Requirements:** Docker with the Compose plugin (`docker compose version`).

```bash
git clone https://github.com/bawback/wp-stack.git
cd wp-stack

cp .env.example .env        # then set a real DB_PASSWORD
docker compose up -d --build
```

Open **http://localhost:8080** and run the WordPress installer.

Useful commands:

```bash
docker compose ps                 # service status
docker compose logs -f wordpress  # follow WordPress logs
docker compose down               # stop (data is kept)
docker compose down -v            # stop and wipe the database and uploads
```

## Configuration

All settings live in `.env`, which is git-ignored so secrets never get committed.

| Variable | Description | Example |
|---|---|---|
| `DB_NAME` | PostgreSQL database name | `wordpress` |
| `DB_USER` | PostgreSQL user | `wordpress` |
| `DB_PASSWORD` | PostgreSQL password | `change-me` |

## Roadmap

- [x] **1. Public GitHub repo** for the project
- [x] **2. Docker Compose with WordPress + PostgreSQL**, with WordPress reachable locally
- [ ] **3. nginx reverse proxy** that serves WordPress by default, with WordPress no longer exposed directly
- [ ] **4. VPS deployment with Ansible**: one command configures the server and brings the stack up
- [ ] **5. HTTPS** on nginx so a real domain (e.g. `x.com`) works over SSL
- [ ] **6. n8n** added to the stack and served on a subdomain (e.g. `n8n.x.com`)
- [ ] **7. GitHub Actions pipeline** that deploys automatically on every push to the main branch

## Troubleshooting

- **WordPress shows "Error establishing a database connection"**: check that `.env` exists and `docker compose ps` shows `db` as `healthy`.
- **Port 8080 is already in use**: change `"8080:80"` in `docker-compose.yml` to another host port such as `"8081:80"`.
- **Changed `.env` but nothing happened**: Postgres only reads credentials on first start. Run `docker compose down -v` to reset (this deletes data).
