<div align="center">

<br/>

```
███╗   ██╗███████╗ ██████╗██████╗  █████╗ ██████╗ ███████╗
████╗  ██║██╔════╝██╔════╝██╔══██╗██╔══██╗██╔══██╗██╔════╝
██╔██╗ ██║███████╗██║     ██████╔╝███████║██████╔╝█████╗
██║╚██╗██║╚════██║██║     ██╔══██╗██╔══██║██╔═══╝ ██╔══╝
██║ ╚████║███████║╚██████╗██║  ██║██║  ██║██║     ███████╗
╚═╝  ╚═══╝╚══════╝ ╚═════╝╚═╝  ╚═╝╚═╝  ╚═╝╚═╝     ╚══════╝
         S E R I E S
```

### Extract anything. From anywhere. Without limits.

**Three focused tools. One philosophy. Zero setup.**

<br/>

![Series](https://img.shields.io/badge/NScrape_Series-v1.0.0-111111?style=flat-square)
![Docker](https://img.shields.io/badge/docker-pratham1uk-0db7ed?style=flat-square&logo=docker)
![Tools](https://img.shields.io/badge/tools-3_active-22c55e?style=flat-square)
![Author](https://img.shields.io/badge/author-pratham1uk-6366f1?style=flat-square)

<br/>

> Every tool in this series ships as Docker images. No source code. No build step. Pull and run.

<br/>

</div>

---

## What is the NScrape Series?

The **NScrape Series** is a collection of purpose-built data extraction tools — each one solving a specific scraping problem, each deployable with a single `docker compose up`.

They share one design philosophy:

| Principle | What it means |
|---|---|
| 🐳 **Docker-first** | Zero local dependencies — pull the images and run |
| 🎨 **UI-first** | Every tool has a web interface. No terminal required for normal use |
| 📤 **Export-ready** | Every result can be downloaded as PDF, DOCX, ZIP, or structured JSON |
| 🔌 **API-first** | Every tool exposes a REST API with Swagger docs for programmatic use |
| 🔧 **Configurable** | Ports, workers, thresholds — all tunable via environment variables |

---

## The Tools

| Tool | Purpose | Images | UI Port | Status |
|------|---------|--------|:-------:|:------:|
| 🕷️ [**LetsNScrape**](#-letsnscape) | General-purpose web scraper — any URL, any data type | `letsnscape-backend` · `letsnscape-frontend` | **3071** | `v1.0.0` ✅ |
| ⚡ [**ytNscrape**](#-ytnscrape) | YouTube lecture scraper — slide frames + captions + study PDF | 9 service images | **3000** | `v1.0.0` ✅ |
| 📸 [**SnapNScrape**](#-snapnscrape) | Web scraping with OCR failsafe — handles bot-blocked sites | `snapnscrape-gateway` · `snapnscrape-scraper` | **3030** | `v1.0.0` ✅ |

---

## Quick Comparison

| | 🕷️ LetsNScrape | ⚡ ytNscrape | 📸 SnapNScrape |
|---|---|---|---|
| **Input** | Any URL | YouTube URL | Any URL |
| **Engine** | HTTP + Playwright | yt-dlp + ffmpeg | HTTP + Playwright + OCR |
| **Output** | Text, links, images, tables, metadata | PDF notes, ZIP slides, captions | Extracted text |
| **Export** | PDF, DOCX | PDF, ZIP | DOCX |
| **Caching** | Redis job store | Redis job queue | Redis 1-hour cache |
| **Multi-page** | ✅ Up to 50 pages, depth 3 | ❌ Single video | ❌ Single URL |
| **Handles JS sites** | ✅ Auto-detect + Playwright | N/A | ✅ OCR fallback |
| **Handles bot-blocked sites** | ⚠️ Playwright fallback | N/A | ✅ Screenshot + OCR |
| **RAM needed** | 2 GB | 4–8 GB | 2 GB |
| **Containers** | 3 | 9+ | 3 |

---
---

## 🕷️ LetsNScrape

> **The web is your database. LetsNScrape is your query.**

[![Version](https://img.shields.io/badge/version-v1.0.0-0066CC?style=flat-square)](https://hub.docker.com/r/pratham1uk/letsnscape-backend)
[![Docker Pulls](https://img.shields.io/docker/pulls/pratham1uk/letsnscape-backend?style=flat-square&color=0066CC)](https://hub.docker.com/r/pratham1uk/letsnscape-backend)
[![Image Size](https://img.shields.io/docker/image-size/pratham1uk/letsnscape-backend/latest?style=flat-square&color=003399)](https://hub.docker.com/r/pratham1uk/letsnscape-backend)

A **general-purpose web scraper** that extracts exactly the data you need from any webpage. Text, images, links, metadata, headings, tables — individually or all at once. Handles both static HTML sites and JavaScript-rendered SPAs.

### What problem it solves

Most scrapers fail on modern sites. LetsNScrape runs a **dual-engine pipeline** — fast HTTP scrape first, then automatic Playwright fallback for JS-heavy sites. It auto-detects which engine is needed; you don't have to choose.

### Architecture

```
╔══════════════════════════════════════════════════════════════════╗
║  letsnscape-frontend  :3071   React UI + Nginx reverse proxy     ║
║  letsnscape-backend   :3070   FastAPI scraping API               ║
║  letsnscape-redis     :6380   Async job queue + result store     ║
╚══════════════════════════════════════════════════════════════════╝
                          |
              ┌───────────┴───────────┐
              ▼                       ▼
         Scrapy / HTTP           Playwright
         (static sites,         (React, Vue, Angular,
          fast path)             Next.js — auto-switched)
```

### Ports

| Service | Host Port | What it is |
|---------|:---------:|------------|
| Frontend (UI) | **3071** | Open this in your browser |
| Backend (API) | **3070** | REST API — Swagger at `:3070/docs` |
| Redis | **6380** | Job store (non-default port to avoid conflicts) |

### Quick start

```bash
mkdir letsnscape && cd letsnscape

# Download compose file + env template
curl -O https://raw.githubusercontent.com/pratham1uk/letsnscape-public/main/docker-compose.prod.yml
curl -O https://raw.githubusercontent.com/pratham1uk/letsnscape-public/main/.env.example
mkdir redis-config
curl -o redis-config/redis.conf https://raw.githubusercontent.com/pratham1uk/letsnscape-public/main/redis-config/redis.conf

cp .env.example .env

# Pull and start
docker compose -f docker-compose.prod.yml up -d
```

Open **http://localhost:3071**

### Docker commands

```bash
# Start
docker compose -f docker-compose.prod.yml up -d

# Check status
docker compose -f docker-compose.prod.yml ps

# View logs
docker compose -f docker-compose.prod.yml logs -f

# View logs for one service
docker compose -f docker-compose.prod.yml logs -f backend

# Stop (keeps data)
docker compose -f docker-compose.prod.yml down

# Stop and wipe all data
docker compose -f docker-compose.prod.yml down -v

# Update to newer release
docker compose -f docker-compose.prod.yml pull
docker compose -f docker-compose.prod.yml up -d
```

### Changing ports

All ports are controlled by `.env`:

```env
FRONTEND_PORT=8080      # UI at http://localhost:8080
BACKEND_PORT=8000       # API at http://localhost:8000
REDIS_PORT_HOST=6399    # Redis at localhost:6399
```

Then restart: `docker compose -f docker-compose.prod.yml up -d`

### Run without compose (individual containers)

```bash
# 1. Create shared network
docker network create letsnscape-net

# 2. Start Redis
docker run -d \
  --name letsnscape-redis \
  --network letsnscape-net \
  --restart unless-stopped \
  -p 6380:6380 \
  redis:7.2-alpine \
  redis-server --port 6380

# 3. Start backend
docker run -d \
  --name letsnscape-backend \
  --network letsnscape-net \
  --restart unless-stopped \
  -p 3070:3070 \
  -e REDIS_HOST=letsnscape-redis \
  -e REDIS_PORT=6380 \
  pratham1uk/letsnscape-backend:v1.0.0

# 4. Start frontend
docker run -d \
  --name letsnscape-frontend \
  --network letsnscape-net \
  --restart unless-stopped \
  -p 3071:3071 \
  pratham1uk/letsnscape-frontend:v1.0.0
```

To use different host ports, change the left side of `-p host:container`:

```bash
# Frontend on 8080, backend on 8000
docker run -d --name letsnscape-frontend --network letsnscape-net \
  -p 8080:3071 pratham1uk/letsnscape-frontend:v1.0.0

docker run -d --name letsnscape-backend --network letsnscape-net \
  -p 8000:3070 -e REDIS_HOST=letsnscape-redis -e REDIS_PORT=6380 \
  pratham1uk/letsnscape-backend:v1.0.0
```

### What you can extract (filters)

| Filter | What it pulls |
|--------|--------------|
| `everything` | All filters combined |
| `text` | Clean body text, paragraphs (scripts/nav stripped) |
| `images` | Image URLs, alt text, dimensions |
| `links` | All hyperlinks with anchor text |
| `metadata` | Title, description, Open Graph, canonical, robots |
| `headings` | H1–H6 structure |
| `tables` | All `<table>` elements as row/column data |

### Configuration

| Variable | Default | Description |
|----------|:-------:|-------------|
| `FRONTEND_PORT` | `3071` | Host port for the UI |
| `BACKEND_PORT` | `3070` | Host port for the API |
| `REDIS_PORT_HOST` | `6380` | Host port for Redis |
| `MAX_PAGES` | `50` | Maximum pages per crawl job |
| `REQUEST_TIMEOUT` | `30` | HTTP timeout in seconds |
| `PLAYWRIGHT_TIMEOUT` | `30000` | Playwright page load timeout in ms |

### API

```bash
# Submit a scrape job
curl -X POST http://localhost:3070/api/v1/scrape \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com","filters":["text","links"],"engine":"auto"}'

# Poll for results
curl http://localhost:3070/api/v1/scrape/{job_id}

# Export as PDF or DOCX
curl -X POST http://localhost:3070/api/v1/export \
  -H "Content-Type: application/json" \
  -d '{"job_id":"...","format":"pdf"}' -o result.pdf

# Health check
curl http://localhost:3070/api/v1/health
```

Interactive docs: **http://localhost:3070/docs**

---
---

## ⚡ ytNscrape

> **Paste a YouTube lecture URL. Get a note-ready PDF and a ZIP of clean slide frames — automatically.**

[![Version](https://img.shields.io/badge/version-v1.0.0-F5C400?style=flat-square)](https://hub.docker.com/r/pratham1uk/ytnscrape-gateway)
[![Docker Pulls](https://img.shields.io/docker/pulls/pratham1uk/ytnscrape-gateway?style=flat-square&color=D4A800)](https://hub.docker.com/r/pratham1uk/ytnscrape-gateway)
[![Image Size](https://img.shields.io/docker/image-size/pratham1uk/ytnscrape-gateway/latest?style=flat-square&color=B8960A)](https://hub.docker.com/r/pratham1uk/ytnscrape-gateway)

A **YouTube lecture scraper** that downloads a video, extracts and deduplicates slide frames by content type, syncs captions, and renders everything into a study-ready PDF and ZIP — fully automated.

### What problem it solves

Watching a lecture to take notes is slow. Pausing to screenshot slides is tedious. ytNscrape processes the entire video in the background and hands you a structured notes PDF with ruled lines for handwriting, plus a ZIP of every unique slide frame labelled by type (code, software UI, or slide).

### Pipeline

| Stage | What happens |
|:-----:|-------------|
| 1 | **Download** — yt-dlp fetches video + caption track |
| 2 | **Extract** — ffmpeg pulls frames at 1 fps |
| 3 | **Dedup (Stage 1)** — perceptual hash removes near-identical frames |
| 4 | **Priority Extract** — OCR + MiniLM classifies P1 (code), P2 (software UI), P3 (slides); deduplicates per tier |
| 5 | **Post-process** — letterbox crop, sharpen, save as lossless PNG |
| 6 | **Caption sync** — VTT timestamps aligned to surviving frames |
| 7 | **Render ZIP** — clean PNGs + `manifest.json` + `captions.txt` |
| 8 | **Render PDF** — A4, 2 slides/page, captions, ruled lines for notes |

### Architecture

```
Frontend  :3000  (nginx + vanilla JS)
  └── Gateway  :3082  (FastAPI + SSE live progress)
        └── Redis job queue  :3061
              ├── svc-ingest       yt-dlp + ffmpeg
              ├── svc-dedup        pHash dedup
              ├── svc-priority     OCR + MiniLM
              ├── svc-postprocess  Pillow crop/sharpen
              ├── svc-captions     VTT sync
              ├── svc-render-zip   ZIP output
              └── svc-render-pdf   PDF output
                        └── MinIO  :9000 / :9001
```

### Ports

| Service | Host Port | What it is |
|---------|:---------:|------------|
| Frontend (UI) | **3000** | Open this in your browser |
| API / Gateway | **3082** | FastAPI — Swagger at `:3082/docs` |
| MinIO API | **9000** | S3 object storage (internal) |
| MinIO Console | **9001** | MinIO web UI (for debugging) |
| Redis | **3061** | Job queue (mapped for debug) |

### Quick start

```bash
mkdir ytnscrape && cd ytnscrape

curl -fsSL https://raw.githubusercontent.com/pratham1uk/ytnscrape/main/docker-compose.yml \
  -o docker-compose.yml

docker compose pull
docker compose up -d
```

Open **http://localhost:3000**

> ⏳ `svc-priority` loads a ~90 MB embedding model on first start — allow 60–90 seconds.

### Docker commands

```bash
# Start
docker compose up -d

# Check all services are healthy
docker compose ps

# View logs
docker compose logs -f
docker compose logs -f svc-priority    # most likely place for issues

# Stop (keeps your data)
docker compose down

# Full reset — wipes Redis jobs + MinIO files
docker compose down -v

# Update to newer release
docker compose pull && docker compose down && docker compose up -d
```

### Changing ports

Edit the left side of `host:container` in `docker-compose.yml`:

```yaml
# Move frontend to 8080
frontend:
  ports:
    - "8080:80"       # was 3000:80

# Move Redis off host entirely (recommended for production)
redis:
  # ports:            # comment out to stop exposing Redis
  #   - "3061:6379"

# Move MinIO
minio:
  ports:
    - "19000:9000"
    - "19001:9001"
```

Then: `docker compose down && docker compose up -d`

> Note: changing the gateway port also requires updating `frontend/nginx.conf` — easiest to leave it unless you have a conflict.

### Configuration (.env)

```env
FRAME_FPS=1               # frames per second of video
PHASH_THRESHOLD=15        # stage-1 dedup aggressiveness
SIMILARITY_THRESHOLD=0.82 # stage-2 cosine threshold
OUTPUT_WIDTH=0            # output px width — 0 = native resolution
FRAMES_PER_PAGE=2         # slides per PDF page (1, 2, or 3)
NOTE_LINES=4              # ruled lines per slide in PDF
MAX_WORKERS=4             # worker threads per service
LOG_LEVEL=INFO
YTDLP_NO_UPDATE=0         # set 1 to disable yt-dlp auto-update
```

### API

```bash
# Submit a job
curl -X POST http://localhost:3082/api/jobs \
  -H "Content-Type: application/json" \
  -d '{"url":"https://www.youtube.com/watch?v=dQw4w9WgXcQ"}'

# Poll status
curl http://localhost:3082/api/jobs/{job_id}

# Live progress stream (SSE)
curl -N http://localhost:3082/api/jobs/{job_id}/stream

# Cancel
curl -X DELETE http://localhost:3082/api/jobs/{job_id}

# List recent jobs
curl http://localhost:3082/api/jobs
```

Interactive docs: **http://localhost:3082/docs**

### GPU acceleration

Automatic — no config needed. If `nvidia-container-toolkit` is installed and a GPU is present, `svc-priority` runs the embedding model on CUDA. CPU mode works on all machines.

---
---

## 📸 SnapNScrape

> **Extract any web content. Without limits.**

[![Version](https://img.shields.io/badge/version-v1.0.0-e8200e?style=flat-square)](https://hub.docker.com/r/pratham1uk/snapnscrape-gateway)
[![Docker Pulls](https://img.shields.io/docker/pulls/pratham1uk/snapnscrape-gateway?style=flat-square&color=e8200e)](https://hub.docker.com/r/pratham1uk/snapnscrape-gateway)
[![Image Size](https://img.shields.io/docker/image-size/pratham1uk/snapnscrape-scraper/latest?style=flat-square&color=111111)](https://hub.docker.com/r/pratham1uk/snapnscrape-scraper)

A **containerised web scraping platform** with a three-stage intelligent pipeline. Built for sites that actively block bots, rely on JavaScript rendering, or hide content behind dynamic layers. When HTTP fails and Playwright fails, it screenshots the page and reads the text with Tesseract OCR.

### What problem it solves

| Problem | How SnapNScrape handles it |
|---|---|
| Website blocks HTTP bots | Falls back to real Chromium via Playwright |
| Content loaded by JavaScript | Browser renders the full page before capturing |
| No public API on the site | Screenshot + Tesseract OCR reads text from the visual page |
| Slow repeated scraping | Redis caches results for 1 hour |
| Need output in a document | One-click DOCX export on every result |

### Architecture

```
Gateway  :${PORT}  (Web UI · REST API · DOCX export · Redis cache)
  └── Scraper  :8001  (internal only — never exposed)
        ├── Stage 1 — HTTP scrape  (httpx + BeautifulSoup4)  fast path
        └── Stage 2 — OCR failsafe (Playwright Chromium + Tesseract)
Redis  :6379  (result cache — internal only)
```

### Ports

| Service | Host Port | What it is |
|---------|:---------:|------------|
| Gateway (UI + API) | **3030** | Open this in your browser — fully configurable |
| Scraper | ❌ Internal | Never exposed to the host |
| Redis | ❌ Internal | Never exposed to the host |

### Quick start

```bash
# Download the compose file
curl -fsSL https://raw.githubusercontent.com/pratham1uk/snapnscrape-public/main/docker-compose.public.yml \
  -o docker-compose.public.yml

# Start (pulls images automatically on first run)
docker compose -f docker-compose.public.yml up -d
```

Open **http://localhost:3030**

> ⏳ First pull is large — the scraper image ships with a full browser and OCR engine. Allow 5–10 minutes.

### Docker commands

```bash
# Start
docker compose -f docker-compose.public.yml up -d

# Check status
docker compose -f docker-compose.public.yml ps

# View logs
docker compose -f docker-compose.public.yml logs -f

# View logs for one service
docker compose -f docker-compose.public.yml logs -f scraper

# Stop (keeps Redis cache)
docker compose -f docker-compose.public.yml down

# Full reset
docker compose -f docker-compose.public.yml down -v

# Update
docker compose -f docker-compose.public.yml pull
docker compose -f docker-compose.public.yml up -d
```

### Changing the port

The gateway port is controlled by the `PORT` environment variable.

**Option 1 — One-time:**
```bash
PORT=9000 docker compose -f docker-compose.public.yml up -d
```

**Option 2 — `.env` file (recommended):**
```env
PORT=9000
```
Then run normally — Docker picks it up automatically.

**Option 3 — Any port inline:**
```bash
PORT=8888 docker compose -f docker-compose.public.yml up -d
```

> **Windows / WSL2 port blocked?**
> If you see `ports are not available: exposing port TCP 0.0.0.0:3030`, Windows has reserved that range for Hyper-V.
> Check reserved ranges: `cmd.exe /c "netsh interface ipv4 show excludedportrange protocol=tcp"`
> Safe choices: `3000` `5000` `7000` `9000` `9090`
> Or fix permanently (PowerShell as Administrator): `net stop winnat && net start winnat`

### Configuration

| Variable | Default | Description |
|----------|:-------:|-------------|
| `PORT` | `3030` | Host port the gateway is exposed on |
| `REDIS_URL` | `redis://redis:6379/0` | Redis connection string |
| `SCRAPER_SERVICE_URL` | `http://scraper:8001` | Internal scraper address |

### API

The `source` field tells you how the result was obtained:

| Value | Meaning |
|---|---|
| `http` | ⚡ Fast direct HTTP scrape |
| `ocr` | 🔴 Playwright screenshot + Tesseract OCR |
| `cache` | ◈ Returned from Redis cache |

```bash
# Scrape a URL
curl -X POST http://localhost:3030/api/scrape \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com"}'

# Download result as DOCX
curl -X POST http://localhost:3030/api/scrape/export \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com"}' \
  -o result.docx

# Health check
curl http://localhost:3030/api/health
```

Interactive docs: **http://localhost:3030/docs**

---

## All Images at a Glance

| Tool | Image | Tag |
|------|-------|:---:|
| LetsNScrape | `pratham1uk/letsnscape-backend` | `v1.0.0` |
| LetsNScrape | `pratham1uk/letsnscape-frontend` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-gateway` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-frontend` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-ingest` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-dedup` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-priority` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-postprocess` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-captions` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-render-zip` | `v1.0.0` |
| ytNscrape | `pratham1uk/ytnscrape-svc-render-pdf` | `v1.0.0` |
| SnapNScrape | `pratham1uk/snapnscrape-gateway` | `v1.0.0` |
| SnapNScrape | `pratham1uk/snapnscrape-scraper` | `v1.0.0` |

> Always pin a version tag. Using `latest` means you may get a different version than expected when the image is updated.

---

## Troubleshooting

### Port already in use

```
Error: bind: address already in use
```

Change the host port as described in each tool's section above. To find what's using a port:

```bash
# Linux / macOS
lsof -i :3071

# Windows
netstat -ano | findstr :3071
```

### Container exits immediately / keeps restarting

```bash
docker compose logs <service-name>
```

Most common causes: insufficient RAM (especially for LetsNScrape/ytNscrape which run Playwright or a neural model), or a port conflict that wasn't caught at startup.

Minimum RAM per tool: **LetsNScrape** 2 GB · **ytNscrape** 4 GB · **SnapNScrape** 2 GB

In Docker Desktop: **Settings → Resources → Memory**

### Health check fails on Redis

```bash
# LetsNScrape
curl http://localhost:3070/api/v1/health

# ytNscrape
curl http://localhost:3082/health

# SnapNScrape
curl http://localhost:3030/api/health
```

If `redis` is not `connected`, restart the Redis container:

```bash
docker compose restart redis        # letsnscape / ytnscrape
docker compose restart redis        # snapnscrape
```

### Full reset (any tool)

```bash
docker compose down -v   # removes volumes (data), keeps images
docker compose up -d
```

---

<div align="center">

<br/>

**Built by [pratham1uk](https://hub.docker.com/r/pratham1uk) · NScrape Series**

[![LetsNScrape Backend](https://img.shields.io/badge/DockerHub-letsnscrape--backend-0066CC?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/letsnscrape-backend)
[![LetsNScrape Frontend](https://img.shields.io/badge/DockerHub-letsnscrape--frontend-0099FF?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/letsnscrape-frontend)
[![ytNscrape Gateway](https://img.shields.io/badge/DockerHub-ytnscrape--gateway-F5C400?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-gateway)
[![SnapNScrape Gateway](https://img.shields.io/badge/DockerHub-snapnscrape--gateway-e8200e?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/snapnscrape-gateway)
[![SnapNScrape Scraper](https://img.shields.io/badge/DockerHub-snapnscrape--scraper-111111?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/snapnscrape-scraper)
[![ytNscrape Frontend](https://img.shields.io/badge/DockerHub-ytnscrape--frontend-E6B800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-frontend)
[![ytNscrape svc-ingest](https://img.shields.io/badge/DockerHub-ytnscrape--svc--ingest-D4A800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-svc-ingest)
[![ytNscrape svc-dedup](https://img.shields.io/badge/DockerHub-ytnscrape--svc--dedup-D4A800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-svc-dedup)
[![ytNscrape svc-priority](https://img.shields.io/badge/DockerHub-ytnscrape--svc--priority-D4A800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-svc-priority)
[![ytNscrape svc-postprocess](https://img.shields.io/badge/DockerHub-ytnscrape--svc--postprocess-D4A800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-svc-postprocess)
[![ytNscrape svc-captions](https://img.shields.io/badge/DockerHub-ytnscrape--svc--captions-D4A800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-svc-captions)
[![ytNscrape svc-render-zip](https://img.shields.io/badge/DockerHub-ytnscrape--svc--render--zip-D4A800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-svc-render-zip)
[![ytNscrape svc-render-pdf](https://img.shields.io/badge/DockerHub-ytnscrape--svc--render--pdf-D4A800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/ytnscrape-svc-render-pdf)
[![LetsNScrape Backend](https://img.shields.io/badge/DockerHub-letsnscape--backend-0066CC?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/letsnscape-backend)
[![LetsNScrape Frontend](https://img.shields.io/badge/DockerHub-letsnscape--frontend-0099FF?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/letsnscape-frontend)
[![SnapNScrape Gateway](https://img.shields.io/badge/DockerHub-snapnscrape--gateway-e8200e?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/snapnscrape-gateway)
[![SnapNScrape Scraper](https://img.shields.io/badge/DockerHub-snapnscrape--scraper-111111?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/snapnscrape-scraper)

<br/>

</div>
