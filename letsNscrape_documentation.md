# 🕷️ LetsNScrape

> **The web is your database. LetsNScrape is your query.**

[![Version](https://img.shields.io/badge/version-v1.0.0-0066CC?style=flat-square)](https://hub.docker.com/r/pratham1uk/letsnscape-backend)
[![Series](https://img.shields.io/badge/series-NScrape-0099FF?style=flat-square)](#17-nscrape-series)
[![Docker](https://img.shields.io/badge/docker-ready-0080FF?style=flat-square)](https://hub.docker.com/r/pratham1uk/letsnscape-backend)
[![Author](https://img.shields.io/badge/author-pratham1uk-003399?style=flat-square)](https://hub.docker.com/r/pratham1uk)

**DockerHub →** [`pratham1uk/letsnscape-backend`](https://hub.docker.com/r/pratham1uk/letsnscape-backend) · [`pratham1uk/letsnscape-frontend`](https://hub.docker.com/r/pratham1uk/letsnscape-frontend)

---

## 📋 Table of Contents

| # | Section |
|---|---------|
| 1 | [What is LetsNScrape](#1-what-is-letsnscape) |
| 2 | [What Problem It Solves](#2-what-problem-it-solves) |
| 3 | [How It Works](#3-how-it-works) |
| 4 | [Architecture](#4-architecture) |
| 5 | [Prerequisites](#5-prerequisites) |
| 6 | [Quick Start with Docker](#6-quick-start-with-docker) |
| 7 | [Port Configuration](#7-port-configuration) |
| 8 | [Environment Variables](#8-environment-variables) |
| 9 | [Filters — What You Can Extract](#9-filters--what-you-can-extract) |
| 10 | [Scraping Engines](#10-scraping-engines) |
| 11 | [Multi-Page Crawling](#11-multi-page-crawling) |
| 12 | [API Reference](#12-api-reference) |
| 13 | [Exporting Results](#13-exporting-results) |
| 14 | [Docker Compose (Recommended)](#14-docker-compose-recommended) |
| 15 | [Running Individual Containers](#15-running-individual-containers) |
| 16 | [Common Problems & Solutions](#16-common-problems--solutions) |
| 17 | [NScrape Series](#17-nscrape-series) |

---

## 1. What is LetsNScrape

**LetsNScrape** is the first tool in the **NScrape Series** — a collection of purpose-built web data extraction tools.

LetsNScrape is a **general-purpose web scraper** that lets you point it at any URL and pull out exactly the data you need: text, images, links, metadata, headings, tables, or everything at once.

It runs as **three Docker microservices** — a React frontend UI, a FastAPI backend API, and a Redis job store — all wired together and deployable with a single command.

> 💡 **No coding required.** Just paste a URL, pick your filters, and extract.

---

## 2. What Problem It Solves

### ❌ The Problem with Most Scrapers

Most scraping tools fall into one of two traps:

**Too Simple**
> Browser extensions or basic scripts that only work on plain HTML pages. The moment a site uses React, Vue, Angular, or any JavaScript framework to render content, they return an empty page or miss most of the data.

**Too Complex**
> Developer-only tools that require writing code, setting up environments, managing dependencies, and knowing XPath or CSS selectors just to get a paragraph of text out of a page.

---

### ✅ What LetsNScrape Does Differently

| Feature | Description |
|---------|-------------|
| 🔄 **Dual Engine** | Fast HTTP scraper for static sites + headless Chromium (Playwright) for JS-heavy sites — auto-switched, no config |
| 🎯 **Filter-Based Extraction** | Choose *what* you want (text, images, links, tables...) — no selectors needed |
| 🖥️ **UI-First** | Operated entirely through a web interface. No terminal, no code |
| ⚡ **Async Jobs** | Scraping runs in the background. Submit, watch the progress bar, get results |
| 📄 **Export Ready** | One click → downloadable PDF report or structured Word document |

---

## 3. How It Works

```
┌─────────────────────────────────────────────┐
│  You paste a URL into the UI                │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│  Frontend sends a scrape request to the     │
│  Backend API                                │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│  Backend stores a job in Redis and starts   │
│  scraping in the background                 │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
         ┌─────────────────────────┐
         │  Auto-detect engine     │
         └──────┬──────────────────┘
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
   Static site?     JS-heavy site?
   Scrapy/HTTP      Playwright
   (fast)           (headless Chromium,
                     scrolls + waits)
        │                │
        └───────┬────────┘
                │
                ▼
┌─────────────────────────────────────────────┐
│  Data extracted by your selected filters    │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│  Results stored in Redis                    │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│  Frontend polls → displays results          │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────┐
│  Download as PDF or DOCX (optional)         │
└─────────────────────────────────────────────┘
```

> **🔍 JS Detection Heuristics:** The auto-detect checks for React (`__NEXT_DATA__`), Vue (`ng-app`), Angular, and Next.js fingerprints in the raw HTML. It also checks the text-to-script ratio — very little visible text with many `<script>` tags signals a JS-rendered page.

---

## 4. Architecture

```
╔══════════════════════════════════════════════════════════════════╗
║             Docker Network: letsnscape-net                       ║
║                                                                  ║
║  ┌─────────────────┐   ┌─────────────────┐   ┌───────────────┐  ║
║  │  letsnscape-    │   │  letsnscape-    │   │ letsnscape-   │  ║
║  │  frontend       │──▶│  backend        │──▶│ redis         │  ║
║  │                 │   │                 │   │               │  ║
║  │ React 18 +Nginx │   │ FastAPI+Uvicorn │   │ Redis 7.2     │  ║
║  │ Port: 3071      │   │ Port: 3070      │   │ Port: 6380    │  ║
║  └─────────────────┘   └────────┬────────┘   └───────────────┘  ║
║                                 │                                ║
║                        ┌────────┴────────┐                       ║
║                        │                 │                       ║
║                  Scrapy/HTTP        Playwright                   ║
║                  (static sites)    (JS-heavy SPAs)               ║
╚══════════════════════════════════════════════════════════════════╝
```

| Container | Image | Default Port | Role |
|-----------|-------|:-----------:|------|
| `letsnscape-frontend` | `pratham1uk/letsnscape-frontend` | **3071** | React UI + Nginx reverse proxy |
| `letsnscape-backend` | `pratham1uk/letsnscape-backend` | **3070** | FastAPI scraping API |
| `letsnscape-redis` | `redis:7.2-alpine` | **6380** | Async job queue + result cache |

> **Why port 6380?** Redis is intentionally on port `6380` (not `6379`) to avoid conflicts with any existing Redis instance already running on your machine.

---

## 5. Prerequisites

| Requirement | Minimum |
|------------|---------|
| Docker Engine | `20.10` or newer |
| Docker Compose | `v2` or newer |
| RAM | `2 GB` available to Docker (Playwright/Chromium needs it) |
| Disk | `~1.5 GB` for all images |
| Internet | Required on first pull (~800 MB total) |

**Verify your setup:**

```bash
docker --version
docker-compose --version
```

---

## 6. Quick Start with Docker

> ⚡ Three files. One command. Done.

### Step 1 — Download the compose files

```bash
# Create a directory
mkdir letsnscape && cd letsnscape
mkdir redis-config

# Download compose file
curl -O https://raw.githubusercontent.com/pratham1uk/letsnscape-public/main/docker-compose.prod.yml

# Download env template
curl -O https://raw.githubusercontent.com/pratham1uk/letsnscape-public/main/.env.example

# Download Redis config
curl -o redis-config/redis.conf https://raw.githubusercontent.com/pratham1uk/letsnscape-public/main/redis-config/redis.conf
```

### Step 2 — Configure

```bash
cp .env.example .env
```

The defaults work out of the box:

```env
DOCKERHUB_USER=pratham1uk
FRONTEND_PORT=3071
BACKEND_PORT=3070
REDIS_PORT_HOST=6380
```

### Step 3 — Pull and Run

```bash
docker-compose -f docker-compose.prod.yml up -d
```

### Step 4 — Open the App

```
http://localhost:3071
```

### Step 5 — Verify Health

```bash
docker-compose -f docker-compose.prod.yml ps
```

All three containers should show `healthy` or `running`.

```bash
curl http://localhost:3070/api/v1/health
# Expected: {"status":"ok","redis":"connected"}
```

---

## 7. Port Configuration

Internal container ports are fixed (`3071`, `3070`, `6380`) but **host-side ports are fully configurable**.

### Change Ports in `.env`

```env
FRONTEND_PORT=8080      # Access UI at http://localhost:8080
BACKEND_PORT=8000       # API at http://localhost:8000
REDIS_PORT_HOST=6399    # Redis at localhost:6399
```

Then restart:

```bash
docker-compose -f docker-compose.prod.yml up -d
```

### Check if a Port is Already in Use

**Linux / macOS:**
```bash
lsof -i :3071
lsof -i :3070
lsof -i :6380
```

**Windows:**
```powershell
netstat -ano | findstr :3071
netstat -ano | findstr :3070
netstat -ano | findstr :6380
```

### Example: Custom Port Set

```env
FRONTEND_PORT=9000
BACKEND_PORT=9001
REDIS_PORT_HOST=9002
```

> 💡 Containers communicate with each other over the internal Docker network regardless of host ports — only you access host ports.

---

## 8. Environment Variables

All configuration lives in `.env`. Copy `.env.example` to `.env` and edit.

| Variable | Default | Description |
|----------|:-------:|-------------|
| `DOCKERHUB_USER` | `pratham1uk` | DockerHub username for pulling images |
| `FRONTEND_PORT` | `3071` | Host port for the web UI |
| `BACKEND_PORT` | `3070` | Host port for the API |
| `REDIS_PORT_HOST` | `6380` | Host port for Redis |
| `REDIS_HOST` | `redis` | Internal Redis hostname |
| `REDIS_PORT` | `6380` | Internal Redis port (must match redis.conf) |
| `REDIS_DB` | `0` | Redis database index |
| `MAX_PAGES` | `50` | Maximum pages per scrape job (hard cap) |
| `REQUEST_TIMEOUT` | `30` | HTTP request timeout in seconds |
| `PLAYWRIGHT_TIMEOUT` | `30000` | Playwright page load timeout in milliseconds |
| `DEBUG` | `false` | Enable FastAPI debug mode (`true` for verbose logs) |

---

## 9. Filters — What You Can Extract

Choose one or more filters when submitting a scrape job.

| Filter | What It Extracts |
|--------|-----------------|
| ⭐ **Everything** | All of the below combined — full extraction |
| 📝 **Text** | Clean body text, paragraphs, readable content (scripts/nav/footer stripped) |
| 🖼️ **Images** | Image URLs, alt text, width, height for every `<img>` tag found |
| 🔗 **Links** | All hyperlinks with anchor text and absolute URLs |
| 🏷️ **Metadata** | `<meta>` tags: title, description, keywords, author, Open Graph tags, canonical URL, robots, charset, viewport |
| 📰 **Headings** | H1–H6 structure with level and text |
| 📊 **Tables** | All `<table>` elements as structured row/column data |

> **Combining Filters:** Select multiple filters together — e.g., Images + Links + Metadata — without pulling the full text. **Everything** equals all filters simultaneously.

---

## 10. Scraping Engines

### 🤖 Auto-Detect (Default — Recommended)

The backend fetches the page with a plain HTTP request first, then analyzes:

- React, Vue, Angular, Next.js fingerprints in source
- Presence of `__NEXT_DATA__`, `ng-app`, or framework attributes
- Ratio of visible text to `<script>` tags

JS-rendered page detected → automatically switches to Playwright.

---

### ⚡ Scrapy / HTTP Engine

- Uses `httpx` for async HTTP requests
- Parses HTML with **BeautifulSoup4 + lxml**
- Extremely fast — no browser overhead

**Works on:** news sites, blogs, Wikipedia, e-commerce product pages, documentation, any server-rendered HTML.

---

### 🌐 Playwright Engine

- Launches **headless Chromium** inside the container
- Navigates to page, waits for `networkidle`
- Scrolls the full page to trigger lazy-loaded content
- Captures fully-rendered DOM after JavaScript execution

**Works on:** React SPAs, Vue apps, Angular apps, Next.js sites, dashboards, any client-side rendered site.

---

### When to Force an Engine Manually

| Situation | Use |
|-----------|:---:|
| You know the site is static | ⚡ **Scrapy** |
| Auto-detect chose wrong engine | 🌐 **Playwright** |
| Site uses infinite scroll | 🌐 **Playwright** |
| Scraping APIs / JSON endpoints | ⚡ **Scrapy** |
| Site blocks headless browsers | ⚡ **Scrapy** |

---

## 11. Multi-Page Crawling

LetsNScrape can follow links and scrape multiple pages in one job.

### Settings

| Option | Default | Description |
|--------|:-------:|-------------|
| **Max Pages** | `1` | How many pages to scrape total (max 50) |
| **Follow Links** | `Off` | Whether to crawl links found on each page |
| **Same Domain Only** | `On` | Only follow links on the same domain |
| **Crawl Depth** | `1` | How many levels deep to follow links (max 3) |

### How It Works

Starting from your URL at **depth 0**, LetsNScrape extracts all links on that page, then visits each one (depth 1), and so on — until reaching your depth limit or max pages limit, whichever comes first.

> 🕐 A **500ms polite delay** is added between requests to avoid hammering the server.

### Example: Scrape an Entire Documentation Site

```
URL:              https://docs.example.com
Max Pages:        20
Follow Links:     On
Same Domain Only: On
Depth:            2
```

This scrapes the homepage, then follows up to 20 links across 2 levels deep — all staying within `docs.example.com`.

---

## 12. API Reference

> 📖 Interactive Swagger docs: **`http://localhost:3070/docs`**

---

### `POST /api/v1/scrape` — Start a Scrape Job

```http
POST /api/v1/scrape
Content-Type: application/json
```

**Request Body:**

```json
{
  "url": "https://example.com",
  "filters": ["everything"],
  "engine": "auto",
  "max_pages": 1,
  "follow_links": false,
  "same_domain_only": true,
  "depth": 1
}
```

**Field Reference:**

| Field | Type | Options | Default |
|-------|------|---------|:-------:|
| `url` | string | Any valid URL | *required* |
| `filters` | array | `everything` `text` `images` `links` `metadata` `headings` `tables` | `["everything"]` |
| `engine` | string | `auto` `scrapy` `playwright` | `auto` |
| `max_pages` | integer | 1–50 | `1` |
| `follow_links` | boolean | `true` `false` | `false` |
| `same_domain_only` | boolean | `true` `false` | `true` |
| `depth` | integer | 1–3 | `1` |

**Response:**

```json
{
  "job_id": "3f8a2c1d-...",
  "status": "queued",
  "message": "Scrape job started"
}
```

---

### `GET /api/v1/scrape/{job_id}` — Poll Job Status

**While running:**

```json
{
  "job_id": "3f8a2c1d-...",
  "status": "running",
  "progress": 45,
  "url": "https://example.com",
  "filters": ["text", "links"]
}
```

**When complete:**

```json
{
  "job_id": "3f8a2c1d-...",
  "status": "completed",
  "progress": 100,
  "result": {
    "url": "https://example.com",
    "pages_scraped": 1,
    "engine_used": "scrapy_http",
    "filters": ["text", "links"],
    "summary": {
      "total_pages": 1,
      "total_words": 1842,
      "total_images": 0,
      "total_links": 34
    },
    "pages": [
      {
        "url": "https://example.com",
        "title": "Example Domain",
        "status_code": 200,
        "engine_used": "scrapy_http",
        "word_count": 1842,
        "text": "...",
        "links": ["..."],
        "images": null,
        "metadata": null,
        "headings": null,
        "tables": null
      }
    ]
  }
}
```

**Status flow:** `queued` → `running` → `completed` | `failed`

---

### `POST /api/v1/export` — Export Results

```http
POST /api/v1/export
Content-Type: application/json
```

```json
{
  "job_id": "3f8a2c1d-...",
  "format": "pdf"
}
```

`format` accepts `pdf` or `docx`. Returns a **binary file download**.

---

### `GET /api/v1/health` — Health Check

```json
{
  "status": "ok",
  "service": "LetsNScrape API",
  "redis": "connected"
}
```

> If `redis` shows an error, the backend is running but cannot store jobs — check the Redis container.

---

## 13. Exporting Results

After a scrape completes, export the full results directly from the UI or via API.

### 📄 PDF Export

Generated with **ReportLab**. Includes:

- Cover with scraped URL and timestamp
- Summary table: pages scraped, total words, images, links, engine used, filters applied
- Per-page sections: metadata grid, extracted text, headings, image list, link list, tables
- Clean typography with color-coded section headers

### 📝 DOCX Export (Word Document)

Generated with **python-docx**. Includes:

- Heading hierarchy matching H1–H6 from the scraped page
- Bullet lists for images and links
- Tables for extracted table data
- Formatted metadata section

> Both formats contain all data from all pages scraped, in order.

---

## 14. Docker Compose (Recommended)

Docker Compose handles startup order (Redis → Backend → Frontend), shared networking, and port mapping automatically.

### Compose File Explained

```yaml
services:
  redis:
    image: redis:7.2-alpine
    ports:
      - "${REDIS_PORT_HOST:-6380}:6380"   # host:container
    # Runs on internal port 6380 (avoids conflict with default Redis)

  backend:
    image: pratham1uk/letsnscape-backend:v1.0.0
    ports:
      - "${BACKEND_PORT:-3070}:3070"
    depends_on:
      redis:
        condition: service_healthy         # waits for Redis ping

  frontend:
    image: pratham1uk/letsnscape-frontend:v1.0.0
    ports:
      - "${FRONTEND_PORT:-3071}:3071"
    depends_on:
      backend:
        condition: service_healthy         # waits for backend /health
```

### Useful Compose Commands

```bash
# Start all services
docker-compose -f docker-compose.prod.yml up -d

# Stop all services
docker-compose -f docker-compose.prod.yml down

# View all logs
docker-compose -f docker-compose.prod.yml logs -f

# View logs for one service
docker-compose -f docker-compose.prod.yml logs -f backend

# Check status
docker-compose -f docker-compose.prod.yml ps

# Pull latest images then restart
docker-compose -f docker-compose.prod.yml pull
docker-compose -f docker-compose.prod.yml up -d

# Stop and remove everything including volumes
docker-compose -f docker-compose.prod.yml down -v
```

---

## 15. Running Individual Containers

Prefer manual container control? Here's how.

### Step 1 — Create a Shared Network

```bash
docker network create letsnscape-net
```

### Step 2 — Start Redis

```bash
docker run -d \
  --name letsnscape-redis \
  --network letsnscape-net \
  --restart unless-stopped \
  -p 6380:6380 \
  redis:7.2-alpine \
  redis-server --port 6380
```

Verify:

```bash
docker exec letsnscape-redis redis-cli -p 6380 ping
# Expected: PONG
```

### Step 3 — Start the Backend

```bash
docker run -d \
  --name letsnscape-backend \
  --network letsnscape-net \
  --restart unless-stopped \
  -p 3070:3070 \
  -e REDIS_HOST=letsnscape-redis \
  -e REDIS_PORT=6380 \
  -e MAX_PAGES=50 \
  pratham1uk/letsnscape-backend:v1.0.0
```

Verify:

```bash
curl http://localhost:3070/api/v1/health
```

### Step 4 — Start the Frontend

```bash
docker run -d \
  --name letsnscape-frontend \
  --network letsnscape-net \
  --restart unless-stopped \
  -p 3071:3071 \
  pratham1uk/letsnscape-frontend:v1.0.0
```

Open `http://localhost:3071` 🎉

### Using Different Host Ports

```bash
# Frontend on port 8080
docker run -d --name letsnscape-frontend --network letsnscape-net \
  -p 8080:3071 \
  pratham1uk/letsnscape-frontend:v1.0.0

# Backend on port 8000
docker run -d --name letsnscape-backend --network letsnscape-net \
  -p 8000:3070 \
  -e REDIS_HOST=letsnscape-redis -e REDIS_PORT=6380 \
  pratham1uk/letsnscape-backend:v1.0.0
```

---

## 16. Common Problems & Solutions

### 🔴 UI loads but scrape requests fail immediately

**Cause:** Backend container is not running or not reachable.

```bash
docker ps | grep letsnscape-backend
curl http://localhost:3070/api/v1/health
```

- Container not running → `docker start letsnscape-backend`
- Health check fails → `docker logs letsnscape-backend`

---

### 🔴 Health check shows `redis: error`

**Cause:** Backend cannot reach the Redis container.

```bash
docker network inspect letsnscape-net
```

Both `letsnscape-backend` and `letsnscape-redis` should appear. If Redis is missing, restart it with `--network letsnscape-net`.

---

### 🔴 Port already in use on startup

```
Error: bind: address already in use
```

**Fix:** Change the conflicting port in `.env`:

```env
FRONTEND_PORT=8080
BACKEND_PORT=8001
REDIS_PORT_HOST=6399
```

Then: `docker-compose -f docker-compose.prod.yml up -d`

---

### 🔴 Scrape returns empty text on a JS-heavy site

**Cause:** Auto-detect chose the HTTP engine but the site requires JavaScript.

**Fix:** Change the engine dropdown from **Auto Detect** to **Playwright** and re-run.

---

### 🔴 Playwright scrape times out

**Cause:** The site never reaches `networkidle` (keeps making background requests — polling, WebSockets).

**Fix:** Try switching to the **Scrapy/HTTP** engine to get the pre-render HTML. This is a site behavior issue, not a timeout configuration issue.

---

### 🔴 Container exits immediately after start

**Cause:** Insufficient memory for Playwright/Chromium.

**Fix:** In Docker Desktop: **Settings → Resources → Memory → set to at least 2 GB.** Then restart the backend container.

---

### 🔴 Scrape job stuck at 0% for a long time

**Cause:** Redis is unreachable — job was never stored.

```bash
curl http://localhost:3070/api/v1/health
```

If `redis` is not `connected`, restart the Redis container, then restart the backend.

---

### 🔴 Images not showing in results

**Cause:** Site uses lazy-loading (`data-src` instead of `src`) and the HTTP engine was used.

**Fix:** Use the **Playwright** engine — it scrolls the page triggering lazy-load, and also checks `data-src` / `data-lazy-src` attributes.

---

### 🔴 Export download doesn't start

**Cause:** Job hasn't completed yet, or job ID has expired (jobs expire after 24 hours).

**Fix:** Check job status first. If expired, re-run the scrape and export immediately after completion.

---

### 🔴 `docker-compose` command not found

**Fix:** Use the newer syntax (no hyphen):

```bash
docker compose -f docker-compose.prod.yml up -d
```

---

## 17. NScrape Series

LetsNScrape is part of the **NScrape Series** — a growing collection of focused web data extraction tools, each solving a specific scraping problem.

Every tool in the series is:
- 🐳 Independently deployable via Docker
- 🎨 UI-first, filter-based, export-ready
- 📚 Documented in the NScrape Series public repository

| Tool | What It Does | Status |
|------|-------------|:------:|
| 🕷️ **LetsNScrape** | General-purpose web scraper — any URL, any data type | `v1.0.0` ✅ |
| ⚡ **ytNscrape** | YouTube lecture scraper — slide frames + captions + PDF notes | `v1.0.0` ✅ |
| 📸 **SnapNScrape** | General web scraping with OCR failsafe | `v1.0.0` ✅ |

---

<div align="center">

---

*LetsNScrape v1.0.0 — NScrape Series — [pratham1uk](https://hub.docker.com/r/pratham1uk)*

[![LetsNScrape Backend](https://img.shields.io/badge/DockerHub-letsnscrape--backend-0066CC?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/letsnscrape-backend)
[![LetsNScrape Frontend](https://img.shields.io/badge/DockerHub-letsnscrape--frontend-0099FF?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/letsnscrape-frontend)

</div>
