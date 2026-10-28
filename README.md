# FestPass Docker & Container Security Assessment

A three-tier registration app for the Docker and Container Security practical assessment. The starter scaffold is set up for the Compose stack and the hardened image stage. Complete the assessment on your own machine and use only your own identity, accounts, keys, screenshots, and image digest.

## Personal values to fill in

- Name: `YOUR NAME`
- REGNO: `202401100200005`
- PORT: `85`
- Docker Hub username: `YOUR_DOCKERHUB_USERNAME`
- Docker Hub repository: `https://hub.docker.com/r/YOUR_DOCKERHUB_USERNAME/fest-backend`
- Docker Hub 1.0.0 digest: `ADD THE DIGEST FROM YOUR PUSH OUTPUT`

Create your `.env` from `.env.example`, then fill in your personal values and a long random `DB_PASSWORD`. `.env` and signing keys are excluded from Git.

## What the app does

- Nginx serves `frontend/index.html` and proxies `/api/` to Flask.
- Flask exposes `GET /api/health`, `GET /api/registrations`, and `POST /api/registrations`.
- PostgreSQL stores registrations in the named `postgres-data` volume.
- Compose places Nginx and the backend on `web`, and the backend and database on `data`. Nginx has no route to the database network.
- The backend image uses a versioned Python slim base and runs as a non-root user.

## Prerequisites

Before the timed portion, install and verify Docker, Trivy, Cosign, and GitHub access as described in the assessment PDF. Pull `python:3.9`, `postgres:17-alpine`, and `nginx:1.30-alpine` in advance. Never use Trivy 0.69.4, 0.69.5, or 0.69.6. Do not commit `.env`, `cosign.key`, registry passwords, or access tokens.

In each new PowerShell terminal, show your own registration number in the prompt (replace the example):

```powershell
function prompt { "[202401100200005] PS $(Get-Location)> " }
```

## Stage 1: backend image

The assessment asks you to start with `FROM python:3.9`, build `fest-backend:0.1`, run it on your PORT with REGNO, show `/api/health` and logs, then remove the container. To earn this stage's exact evidence, temporarily change `backend/Dockerfile` to use `python:3.9` and use the starter dependency pins from the PDF (`Flask==2.2.2`, `Werkzeug==2.2.2`, `psycopg2-binary==2.9.10`). After completing Stage 1, restore the hardened Dockerfile and requirements below.

Capture `S1-2.png` and `S1-3.png` yourself with the full terminal window and your prompt visible.

## Stage 2: Compose stack

```powershell
Copy-Item .env.example .env
# Edit .env with your REGNO, calculated PORT, and a strong DB_PASSWORD.
docker compose up -d --build
docker compose ps
```

Open `http://localhost:<PORT>`, add two registrations, and confirm the page displays your REGNO. To demonstrate isolation, run `docker compose exec frontend ping -c 1 db` (it should fail; if the image has no `ping`, use an equivalent reachability check from inside frontend). Then verify persistence:

```powershell
docker compose down
docker compose up -d
# Reload the page and confirm both entries remain.
docker volume ls
```

Capture `S2-2.png`, `S2-3.png`, and `S2-4.png` with the required visible evidence.

## Stage 3: scan and harden

The current Dockerfile and dependency pins are the hardened configuration for this stage. First build the deliberately old Stage 1 image as `fest-backend:0.1` and save your own Trivy output:

```powershell
New-Item -ItemType Directory -Force reports | Out-Null
trivy image fest-backend:0.1 | Tee-Object reports/trivy-before.txt
```

Build the hardened image and capture the scan result:

```powershell
docker build -t fest-backend:0.2 ./backend
trivy image fest-backend:0.2 | Tee-Object reports/trivy-after.txt
trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
$LASTEXITCODE
```

Use the actual Trivy results and `docker image ls` size values in the README. Do not claim zero findings unless your scan confirms it. Add a before-and-after table:

| Image | CRITICAL | HIGH | Size |
|---|---:|---:|---:|
| `fest-backend:0.1` | fill from report | fill from report | fill from Docker |
| `fest-backend:0.2` | fill from report | fill from report | fill from Docker |

Capture `S3-1.png` and `S3-3.png`. Trivy reports are generated evidence and should be committed after you run the scans.

## Stage 4: version and publish

1. Rebuild with `APP_VERSION=1.0.0`, tag as `YOUR_DOCKERHUB_USERNAME/fest-backend:1.0.0` and `:1.0`, then push both tags.
2. Record the pushed `1.0.0` digest here and capture the public Docker Hub Tags page as `S4-2.png`.
3. For the final Compose release, change the backend service to use `image: YOUR_DOCKERHUB_USERNAME/fest-backend:1.0.0` and remove its local `build:` block. Remove the local backend image, run `docker compose up -d`, then reload the app and confirm version `1.0.0` and your persisted registrations. Capture `S4-1.png` and `S4-3.png`.

## Stage 5: sign and verify

Add `*.key` to `.gitignore` (already included), generate a password-protected Cosign key pair, and commit only `cosign.pub`. Sign the `1.0.0` image by its pushed digest, verify it with `cosign.pub`, and build/push `1.0.1` without signing it to show verification fails. Never put `cosign.key` in any commit. Capture `S5-1.png` through `S5-4.png` with your own terminal prompt visible.

## AI tools used

- OpenAI Codex: used to read the provided assessment PDF and scaffold the application, Docker configuration, and this checklist. All commands, scans, registry operations, signatures, screenshots, and submission evidence must be produced by you on your own machine.


