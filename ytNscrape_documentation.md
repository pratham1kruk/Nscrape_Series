# ⚡ ytNscrape

> **Paste a YouTube lecture URL. Get a note-ready PDF and a ZIP of clean slide frames — automatically.**

[![Version](https://img.shields.io/badge/version-v1.0.0-F5C400?style=flat-square)](https://hub.docker.com/r/pratham1uk)
[![Series](https://img.shields.io/badge/series-NScrape-E6B800?style=flat-square)](#nscrape-series)
[![Docker](https://img.shields.io/badge/docker-ready-D4A800?style=flat-square)](https://hub.docker.com/r/pratham1uk)
[![Author](https://img.shields.io/badge/author-pratham1uk-B8960A?style=flat-square)](https://hub.docker.com/r/pratham1uk)

**DockerHub →** [`pratham1uk/ytnscrape-gateway`](https://hub.docker.com/r/pratham1uk) · [`pratham1uk/ytnscrape-frontend`](https://hub.docker.com/r/pratham1uk)

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [What It Does](#1-what-it-does) |
| 2 | [Architecture](#2-architecture) |
| 3 | [Quick Start (Pre-Built Images)](#3-quick-start-pre-built-images) |
| 4 | [Ports at a Glance](#4-ports-at-a-glance) |
| 5 | [Docker Commands Reference](#5-docker-commands-reference) |
| 6 | [Changing Ports](#6-changing-ports) |
| 7 | [Configuration (Environment Variables)](#7-configuration-environment-variables) |
| 8 | [Usage](#8-usage) |
| 9 | [API Reference](#9-api-reference) |
| 10 | [GPU Acceleration](#10-gpu-acceleration) |
| 11 | [Troubleshooting](#11-troubleshooting) |
| 12 | [Known Limitations](#12-known-limitations) |
| 13 | [License](#13-license) |

---

## 1. What It Does

**ytNscrape** is the second tool in the **NScrape Series** — a collection of purpose-built web data extraction tools.

ytNscrape is a **YouTube lecture scraper** that downloads a video, extracts and deduplicates slide frames, syncs captions, and renders everything into a study-ready PDF and ZIP — fully automated, no editing required.

It runs as **nine Docker microservices** — a vanilla JS frontend, a FastAPI gateway, six processing services, and MinIO object storage — all wired together and deployable with a single command.

> 💡 **No coding required.** Just paste a YouTube URL, click Extract, and download your notes.

The full 8-stage pipeline:

| Stage | What Happens |
|:-----:|-------------|
| **1** | **Download** — fetches the YouTube video + caption track via `yt-dlp` |
| **2** | **Extract** — pulls frames at 1 fps using `ffmpeg` |
| **3** | **Stage 1 Dedup** — perceptual hash removes near-identical frames, keeps the one with most text |
| **4** | **Priority Extract** — classifies frames as P1 (code/terminal), P2 (software UI), P3 (slides); deduplicates per tier |
| **5** | **Post-Process** — crops letterbox, sharpens text, saves as lossless PNG |
| **6** | **Caption Sync** — aligns VTT timestamps to surviving frames |
| **7** | **Render ZIP** — clean PNGs + `manifest.json` + `captions.txt` |
| **8** | **Render PDF** — A4, 2 slides/page, captions in small type, ruled blank lines for handwriting |

---

## 2. Architecture

```
Browser
  └── Frontend  (nginx + vanilla JS)          :3000
        └── /api/* → FastAPI Gateway          :3082
                        └── Redis job queue
                              ├── svc-ingest        (yt-dlp + ffmpeg)
                              ├── svc-dedup         (pHash stage 1)
                              ├── svc-priority      (OCR + MiniLM classify/dedup)
                              ├── svc-postprocess   (Pillow crop/resize/sharpen)
                              ├── svc-captions      (VTT parser + sync)
                              ├── svc-render-zip    (zipfile)
                              └── svc-render-pdf    (fpdf2)
                                        └── MinIO S3-compatible storage
                                              API  :9000
                                              UI   :9001
```

| Container | Image | Default Port | Role |
|-----------|-------|:-----------:|------|
| `ytnscrape-frontend` | `pratham1uk/ytnscrape-frontend` | **3000** | Vanilla JS UI + Nginx reverse proxy |
| `ytnscrape-gateway` | `pratham1uk/ytnscrape-gateway` | **3082** | FastAPI job gateway |
| `ytnscrape-redis` | `redis:7-alpine` | **3061** | Async job queue |
| `ytnscrape-minio` | `minio/minio` | **9000 / 9001** | S3-compatible object storage |
| `svc-ingest` | `pratham1uk/ytnscrape-svc-ingest` | — | yt-dlp + ffmpeg frame extraction |
| `svc-dedup` | `pratham1uk/ytnscrape-svc-dedup` | — | Perceptual hash deduplication |
| `svc-priority` | `pratham1uk/ytnscrape-svc-priority` | — | OCR + MiniLM classify/dedup |
| `svc-postprocess` | `pratham1uk/ytnscrape-svc-postprocess` | — | Pillow crop/resize/sharpen |
| `svc-captions` | `pratham1uk/ytnscrape-svc-captions` | — | VTT parser + caption sync |
| `svc-render-zip` | `pratham1uk/ytnscrape-svc-render-zip` | — | ZIP renderer |
| `svc-render-pdf` | `pratham1uk/ytnscrape-svc-render-pdf` | — | PDF renderer (fpdf2) |

> **Design principle:** Each service is an independent Python process. They communicate **only via Redis lists** (push/pop) and share frames via a Docker volume (`tmp_frames`). The gateway streams real-time progress to the browser via **Server-Sent Events**.

---

## 3. Quick Start (Pre-Built Images)

> ⚡ No source code. No build step. Just Docker.

### Requirements

- **Docker Engine** 24+ or Docker Desktop with Compose v2
- **RAM:** minimum 4 GB — 8 GB recommended (semantic model runs inside `svc-priority`)
- **Disk:** ~3 GB for all images combined
- **Internet access** during first pull and at runtime (yt-dlp downloads videos live)

Check your versions:

```bash
docker --version
docker compose version
```

### Step 1 — Create a Working Folder

```bash
mkdir ytnscrape && cd ytnscrape
```

### Step 2 — Download the Compose File

```bash
# Linux / macOS / WSL2
curl -fsSL https://raw.githubusercontent.com/pratham1uk/ytnscrape/main/docker-compose.yml -o docker-compose.yml
```

Or create `docker-compose.yml` manually and paste the contents from the releases page.

### Step 3 — Pull All Images

```bash
docker compose pull
```

This downloads all 9 service images from Docker Hub (~3 GB total on first pull, cached afterwards).

### Step 4 — Start

```bash
docker compose up -d
```

### Step 5 — Open the App

```
http://localhost:3000
```

### Step 6 — Verify Everything is Healthy

```bash
docker compose ps
```

All services should show `healthy` or `running`.

```bash
curl http://localhost:3082/api/health
# Expected: {"status":"ok","redis":"connected"}
```

> ⏳ If `svc-priority` takes a minute to become healthy — that's normal. It loads a ~90 MB embedding model on first start.

---

## 4. Ports at a Glance

The internal container ports are fixed but the **host-side ports you access from your browser are fully configurable**. If any default port is already in use on your machine, change it in `docker-compose.yml` before starting.

| Service | Host Port | What It Is |
|---------|:---------:|------------|
| Frontend | **3000** | The web UI — open this in your browser |
| API / Gateway | **3082** | FastAPI — Swagger docs at `:3082/docs` |
| MinIO API | **9000** | S3-compatible object storage (internal) |
| MinIO Console | **9001** | MinIO web UI (optional, for debugging) |
| Redis | **3061** | Job queue (internal, mapped for debug) |

> **Why these ports?** `3000`, `3082`, and `3061` are chosen to avoid conflicts with common dev servers on `8000`/`8080`. MinIO uses its standard ports.

### How to Check if a Port is Already in Use

**Linux / macOS:**
```bash
lsof -i :3000
lsof -i :3082
lsof -i :3061
```

**Windows:**
```powershell
netstat -ano | findstr :3000
netstat -ano | findstr :3082
netstat -ano | findstr :3061
```

If any port is occupied, edit the left side of its `host:container` mapping in `docker-compose.yml`.

---

## 5. Docker Commands Reference

### Start in the Background

```bash
docker compose up -d
```

### Check That Everything Is Running

```bash
docker compose ps
```

### View Logs

```bash
# All services
docker compose logs -f

# One specific service
docker compose logs -f svc-ingest
docker compose logs -f svc-priority
docker compose logs -f gateway
```

### Stop (Keeps Your Data)

```bash
docker compose down
```

### Stop and Wipe All Data (Full Reset)

```bash
docker compose down -v
```

This removes Redis job history and MinIO stored files. The images themselves are kept — no re-pull needed.

### Update to a Newer Release

```bash
docker compose pull      # download new images
docker compose down      # stop current containers
docker compose up -d     # start with new images
```

### Pull a Specific Version

```bash
# Latest release
docker compose pull

# Or pull a specific version tag directly
docker pull pratham1uk/ytnscrape-gateway:v1.2.0
docker pull pratham1uk/ytnscrape-svc-ingest:v1.2.0
# ... repeat for each service, or use compose with image tags set
```

---

## 6. Changing Ports

All host-side ports are in `docker-compose.yml`. Edit the **left side** of the `host:container` mapping.

### Move the Frontend to Port 8080

Find the `frontend` service block and change `"3000:80"` to `"8080:80"`:

```yaml
frontend:
  image: pratham1uk/ytnscrape-frontend:latest
  ports:
    - "8080:80"     # was 3000:80
```

Then restart:

```bash
docker compose down && docker compose up -d
```

The UI is now at **http://localhost:8080**.

### Move the API to Port 9090

Find the `gateway` service block and change `"3082:8000"` to `"9090:8000"`:

```yaml
gateway:
  image: pratham1uk/ytnscrape-gateway:latest
  ports:
    - "9090:8000"   # was 3082:8000
```

> The frontend proxies `/api/*` to the gateway via nginx. If you change the gateway's host port you also need to update `frontend/nginx.conf` and rebuild the frontend image — so it's easiest to leave the gateway port as-is unless you have a conflict.

### Hide Redis from the Host (Recommended for Production)

Redis is exposed on `3061` for debugging. To stop exposing it, remove or comment out its `ports` block:

```yaml
redis:
  image: redis:7-alpine
  # ports:           # comment this out
  #   - "3061:6379"  # to stop exposing Redis to the host
```

Redis still works internally — all services talk to it via the Docker network.

### Use a Different Port for MinIO

```yaml
minio:
  ports:
    - "19000:9000"   # S3 API on host port 19000
    - "19001:9001"   # Console on host port 19001
```

### Example: Running on Completely Different Ports

```yaml
frontend:
  ports:
    - "9000:80"

gateway:
  ports:
    - "9001:8000"

redis:
  ports:
    - "9002:6379"
```

The containers talk to each other over the internal Docker network regardless of what host ports you choose — only you access the host ports.

---

## 7. Configuration (Environment Variables)

Create a `.env` file in the same folder as `docker-compose.yml` to override defaults without editing the compose file.

| Variable | Default | Description |
|----------|:-------:|-------------|
| `FRAME_FPS` | `1` | Frames sampled per second of video |
| `PHASH_THRESHOLD` | `15` | Stage-1 similarity cutoff (higher = more aggressive dedup) |
| `SIMILARITY_THRESHOLD` | `0.82` | Stage-2 cosine threshold for slide clustering |
| `OUTPUT_WIDTH` | `0` | Output frame width in px — `0` = keep native resolution |
| `FRAMES_PER_PAGE` | `2` | Slides per PDF page (`1`, `2`, or `3`) |
| `NOTE_LINES` | `4` | Ruled lines per slide in the PDF |
| `MAX_WORKERS` | `4` | Worker threads per service |
| `LOG_LEVEL` | `INFO` | `DEBUG` for verbose output |
| `YTDLP_NO_UPDATE` | `0` | Set to `1` to disable yt-dlp auto-update on start |

```bash
# .env
FRAME_FPS=1
PHASH_THRESHOLD=15
SIMILARITY_THRESHOLD=0.82
OUTPUT_WIDTH=0
FRAMES_PER_PAGE=2
NOTE_LINES=4
MAX_WORKERS=4
LOG_LEVEL=INFO
YTDLP_NO_UPDATE=0
```

Then run:

```bash
docker compose up -d
```

Compose picks up `.env` automatically.

---

## 8. Usage

1. Open **http://localhost:3000**
2. Paste any YouTube URL into the input bar
3. Click **EXTRACT**
4. Watch live pipeline progress in the right panel
5. Download your **Notes PDF**, **Caption Reference PDF**, and/or **ZIP** when complete

---

## 9. API Reference

> 📖 Interactive Swagger docs: **`http://localhost:3082/docs`**

### `POST /api/jobs` — Submit a New Job

```http
POST /api/jobs
Content-Type: application/json
```

**Request Body:**

```json
{
  "url": "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
}
```

**Response:**

```json
{
  "job_id": "uuid",
  "status": "pending",
  "message": "Job queued"
}
```

---

### `GET /api/jobs/{job_id}` — Poll Job State

Returns current status including progress, URLs, and frame counts.

**Response while running:**

```json
{
  "job_id": "uuid",
  "status": "running",
  "progress": 45,
  "stage": "svc-priority"
}
```

**Response when complete:**

```json
{
  "job_id": "uuid",
  "status": "done",
  "progress": 100,
  "pdf_url": "...",
  "zip_url": "...",
  "frame_count": 38
}
```

**Status values:** `pending` → `running` → `done` or `error`

---

### `GET /api/jobs/{job_id}/stream` — Live Progress Stream

Server-Sent Events stream — emits JSON on every pipeline stage until `done` or `error`.

---

### `DELETE /api/jobs/{job_id}` — Cancel a Running Job

---

### `GET /api/jobs` — List Recent Jobs

Returns up to 20 recent job IDs.

---

## 10. GPU Acceleration

GPU support is **automatic** — no config changes needed.

| Scenario | What Happens |
|----------|-------------|
| No NVIDIA GPU | `svc-priority` starts in CPU mode — works normally |
| NVIDIA GPU + nvidia-container-toolkit installed | Container auto-detects GPU, runs embedding model on CUDA |
| Apple Silicon | CPU mode (MPS not used in this version) |

> To enable NVIDIA passthrough, install [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html) on the host and restart Docker. The `docker-compose.yml` already has the `deploy.resources.reservations` block — nothing else to change.

---

## 11. Troubleshooting

### 🔴 Container Keeps Restarting

**Cause:** Most commonly `svc-priority` failing to download its embedding model, or insufficient RAM.

**Fix:**
```bash
docker compose logs svc-priority
docker compose logs svc-ingest
```

Ensure at least 4 GB RAM is allocated to Docker. In Docker Desktop: **Settings → Resources → Memory → set to at least 4 GB.**

---

### 🔴 Job Stuck / No Progress

**Cause:** A pipeline service is not consuming from its Redis queue.

**Fix:** Check queue depths:

```bash
docker compose exec redis redis-cli LLEN q:ingest
docker compose exec redis redis-cli LLEN q:priority
docker compose exec redis redis-cli LLEN q:render:pdf
```

If queues are backed up, restart the relevant service:

```bash
docker compose restart svc-priority
```

---

### 🔴 Port Already in Use on Startup

```
Error: bind: address already in use
```

**Fix:** Change the conflicting port in `docker-compose.yml` as described in the [Changing Ports](#6-changing-ports) section. Then restart:

```bash
docker compose down && docker compose up -d
```

---

### 🔴 Reset a Stuck Job Completely

```bash
docker compose down -v   # wipes Redis + MinIO
docker compose up -d
```

---

### 🔴 `svc-priority` Takes a Long Time to Start

**Cause:** It loads a ~90 MB embedding model on first start — this is expected.

**Fix:** Wait 60–90 seconds and monitor with:

```bash
docker compose logs -f svc-priority
```

---

## 12. Known Limitations

| Limitation | Detail |
|-----------|--------|
| **Private / age-gated videos** | Not supported without supplying browser cookies to yt-dlp |
| **No manual captions** | Falls back to YouTube auto-generated English subs when no manual track exists |
| **Single-machine only** | All services share a Docker volume for frames; multi-node requires S3 + Redis Cluster |
| **Apple Silicon GPU** | MPS not used in this version — runs in CPU mode |

---

## NScrape Series

ytNscrape is part of the **NScrape Series** — a growing collection of focused web data extraction tools, each solving a specific scraping problem.

Every tool in the series is independently deployable via Docker, shares the same design philosophy (UI-first, export-ready), and is documented in the NScrape Series public repository.

| Tool | What It Does | Status |
|------|-------------|:------:|
| 🕷️ **LetsNScrape** | General-purpose web scraper — any URL, any data type | `v1.0.0` ✅ |
| ⚡ **ytNscrape** | YouTube lecture scraper — slide frames + captions + PDF notes | `v1.0.0` ✅ |
| 📸 **SnapNScrape** | General web scraping with OCR failsafe | `v1.0.0` ✅ |

---

<div align="center">

---

*ytNscrape v1.0.0 — NScrape Series — [pratham1uk](https://hub.docker.com/r/pratham1uk)*

[![Gateway](https://img.shields.io/badge/DockerHub-Gateway-F5C400?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk)
[![Frontend](https://img.shields.io/badge/DockerHub-Frontend-E6B800?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk)

</div>
