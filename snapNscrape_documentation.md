<div align="center">

<br/>

```
███████╗███╗   ██╗ █████╗ ██████╗ ███╗   ██╗███████╗ ██████╗██████╗  █████╗ ██████╗ ███████╗
██╔════╝████╗  ██║██╔══██╗██╔══██╗████╗  ██║██╔════╝██╔════╝██╔══██╗██╔══██╗██╔══██╗██╔════╝
███████╗██╔██╗ ██║███████║██████╔╝██╔██╗ ██║███████╗██║     ██████╔╝███████║██████╔╝█████╗  
╚════██║██║╚██╗██║██╔══██║██╔═══╝ ██║╚██╗██║╚════██║██║     ██╔══██╗██╔══██║██╔═══╝ ██╔══╝  
███████║██║ ╚████║██║  ██║██║     ██║ ╚████║███████║╚██████╗██║  ██║██║  ██║██║     ███████╗
╚══════╝╚═╝  ╚═══╝╚═╝  ╚═╝╚═╝     ╚═╝  ╚═══╝╚══════╝ ╚═════╝╚═╝  ╚═╝╚═╝  ╚═╝╚═╝     ╚══════╝
```

### 🔴 Extract Any Web Content. Without Limits.

**HTTP-first scraping · Playwright browser engine · Tesseract OCR failsafe · Redis caching**

<br/>

![Version](https://img.shields.io/badge/version-v1.0.0-e8200e?style=flat-square)
![Docker](https://img.shields.io/badge/docker-pratham1uk-0db7ed?style=flat-square&logo=docker)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![Playwright](https://img.shields.io/badge/Playwright-2EAD33?style=flat-square&logo=playwright&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)

<br/>

> Part of the **NScrape Series** — `snapNscrape` · `letsNscrape` · `ytNscrape`

<br/>

</div>

---

## 📌 What is SnapNScrape?

SnapNScrape is a **containerised web scraping platform** built as a two-microservice stack. It extracts text content from any webpage — including sites that actively block bots, rely on JavaScript rendering, or hide content behind dynamic layers.

Most scrapers fail silently on modern websites. SnapNScrape doesn't.

It runs a **two-stage intelligent pipeline**:

```
Stage 1 — HTTP Scrape         Fast direct request via httpx + BeautifulSoup
               ↓ fails or insufficient?
Stage 2 — OCR Failsafe        Real headless Chromium renders the page fully,
                               screenshots it, Tesseract reads the text from the image
```

This means SnapNScrape can handle virtually any website — regardless of how it is built, protected, or rendered.

---

## 🔴 Problems It Solves

| ❌ The Problem | ✅ How SnapNScrape Handles It |
|---|---|
| Website blocks HTTP bots | Falls back to real Chromium browser via Playwright |
| Content loaded by JavaScript | Browser renders the full page before capturing |
| No public API on the site | Screenshot + OCR reads text directly from the visual page |
| Slow repeated scraping | Redis caches results for 1 hour — instant on repeat requests |
| Need output in a document | Built-in one-click DOCX export on every result |
| Port conflicts on your machine | Fully configurable port via a single environment variable |
| Complex setup requirements | Ships as Docker images — zero local dependencies needed |

---

## 🏗️ Architecture

SnapNScrape runs as **three Docker containers** working in concert:

```
                        ┌─────────────────────────────────┐
                        │         YOU / YOUR APP          │
                        └────────────────┬────────────────┘
                                         │ HTTP request
                                         ▼
                        ┌─────────────────────────────────┐
                        │   🔴  GATEWAY  :${PORT}         │
                        │   Web UI · REST API · DOCX      │
                        └───────┬─────────────┬───────────┘
                                │             │
                         cache miss      cache hit
                                │             ▼
                                │   ┌─────────────────┐
                                │   │  REDIS  :6379   │
                                │   │  Result Cache   │
                                │   └─────────────────┘
                                ▼
                        ┌─────────────────────────────────┐
                        │   SCRAPER SERVICE  :8001        │
                        │   Internal only — never exposed │
                        └───────┬─────────────────────────┘
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
       ┌─────────────────┐           ┌──────────────────────┐
       │  HTTP SCRAPE    │           │  🔴 PLAYWRIGHT + OCR │
       │  httpx + BS4    │           │  Chromium + Tesseract│
       │  Fast path      │           │  Failsafe path       │
       └────────┬────────┘           └──────────┬───────────┘
                └───────────────┬───────────────┘
                                ▼
                    result → cache → response
```

### Container Roles

| Container | Image | Exposed | Role |
|---|---|---|---|
| Gateway | `pratham1uk/snapnscrape-gateway` | ✅ `${PORT}` (configurable) | Web UI, REST API, caching, DOCX export |
| Scraper | `pratham1uk/snapnscrape-scraper` | ❌ Internal only | HTTP scrape + Playwright + OCR pipeline |
| Cache | `redis/redis-stack-server` | ❌ Internal only | Result caching layer |

---

## ✅ Prerequisites

| Requirement | Notes |
|---|---|
| Docker Desktop or Docker Engine | Only thing you need |
| Docker Compose v2+ | Included with Docker Desktop |
| Python / Node / Tesseract / Playwright | ❌ Not needed — all inside the containers |

---

## 🚀 Quick Start

### Step 1 — Get the compose file

Download `docker-compose.public.yml` from the NScrape public repository.

### Step 2 — Run

```bash
docker compose -f docker-compose.public.yml up
```

> ⏳ First run pulls the images automatically. The scraper image is large
> (it ships with a full browser + OCR engine) — allow 5–10 minutes.

### Step 3 — Open

```
http://localhost:3030
```

Paste any URL, hit **Extract**, and get your content.

### Stop

```bash
docker compose -f docker-compose.public.yml down
```

---

## 🔌 Changing the Port

Default port is `3030`. Three ways to change it:

### Option 1 — One-time override
```bash
PORT=9000 docker compose -f docker-compose.public.yml up
```

### Option 2 — Environment file (recommended)

Create a `.env` file next to your compose file:
```env
PORT=9000
```
Then run normally — Docker picks it up automatically.

### Option 3 — Inline on every run
```bash
PORT=8888 docker compose -f docker-compose.public.yml up
```

<br/>

> ### 🪟 Windows / WSL2 — Port Blocked?
>
> Windows reserves large port ranges for Hyper-V. If you see:
> ```
> ports are not available: exposing port TCP 0.0.0.0:3030
> ```
>
> **Check reserved ranges:**
> ```bash
> cmd.exe /c "netsh interface ipv4 show excludedportrange protocol=tcp"
> ```
> Pick any port outside those ranges. Common safe choices: `3000` `5000` `7000` `9000` `9090`
>
> **Or fix it permanently** — open PowerShell as Administrator:
> ```powershell
> net stop winnat
> net start winnat
> ```
> Then retry your original port.

---

## 🖥️ Web UI Walkthrough

Open `http://localhost:3030` in your browser.

```
┌─────────────────────────────────────────────────────────┐
│  SNAPNSCRAPE  v2.0          Docs    Health    Swagger   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ● Gateway Online  ● Scraper Online                     │
│  ● Redis Cache     ● OCR Failsafe                       │
│                                                         │
│  Extract Content              [ POST /api/scrape ]      │
│  ┌─────────────────────────────────────┐  ┌──────────┐  │
│  │ https://  example.com/page          │  │ Extract  │  │
│  └─────────────────────────────────────┘  └──────────┘  │
│                                                         │
│  [ Init ] [ Cache Check ] [ HTTP Scrape ] [ OCR ] ...  │
│                                                         │
│  01 Cache Check    02 HTTP Scrape    03 OCR Fallback   │
└─────────────────────────────────────────────────────────┘
```

**Status indicators** at the top confirm all services are healthy before you submit.

**You do not need to type `https://`** — it is added automatically.

**On the result page you can:**
- Read the full extracted text in a clean terminal-style output panel
- See how the result was obtained — HTTP direct, OCR fallback, or cache hit
- See the exact time it took in milliseconds
- **Copy** the full text to clipboard in one click
- **Open** the original URL directly
- **Download** the result as a `.docx` Word document
- If OCR was used — see the screenshot that was taken of the page

---

## 🔗 REST API Reference

Interactive Swagger docs at:
```
http://localhost:3030/docs
```

<br/>

### `POST /api/scrape` — Scrape a URL

```bash
curl -X POST http://localhost:3030/api/scrape \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}'
```

**Response:**
```json
{
  "url": "https://example.com",
  "content": "Example Domain. This domain is for use in illustrative examples...",
  "source": "http",
  "duration_ms": 843,
  "cached": false
}
```

**The `source` field:**

| Value | Meaning |
|---|---|
| `http` | ⚡ Fast direct HTTP scrape — sub-second |
| `ocr` | 🔴 Playwright screenshot + Tesseract OCR — used when HTTP fails |
| `cache` | ◈ Returned from Redis — no scraping performed |

<br/>

### `POST /api/scrape/export` — Download as DOCX

Same body as above. Returns a `.docx` file download instead of JSON.

```bash
curl -X POST http://localhost:3030/api/scrape/export \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com"}' \
  -o result.docx
```

<br/>

### `GET /api/health` — Service Health

```bash
curl http://localhost:3030/api/health
```

```json
{
  "status": "ok",
  "service": "gateway",
  "scraper_service": "up",
  "cache_keys": 4
}
```

---

## 🏷️ Image Versions

> **Always pin a version tag in production.** Using `latest` means you may
> get a different version than expected when the image is updated.

```yaml
# ✅ Recommended — predictable, pinned
image: pratham1uk/snapnscrape-gateway:v1.0.0
image: pratham1uk/snapnscrape-scraper:v1.0.0

# ⚠️ Convenient but pulls newest on every docker pull
image: pratham1uk/snapnscrape-gateway:latest
image: pratham1uk/snapnscrape-scraper:latest
```

### Changelog

| Version | Date | Notes |
|---|---|---|
| `v1.0.0` | Apr 2026 | Initial release — full microservice stack, HTTP + OCR pipeline, Redis caching, DOCX export |

---

## ⚙️ Environment Variable Reference

### Gateway

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3030` | Host port the app is exposed on |
| `REDIS_URL` | `redis://redis:6379/0` | Redis connection string |
| `SCRAPER_SERVICE_URL` | `http://scraper:8001` | Internal scraper service address |

### Scraper

| Variable | Default | Description |
|---|---|---|
| `TESSERACT_CMD` | `/usr/bin/tesseract` | Path to Tesseract binary inside container |

---

## 📦 Why is the Scraper Image Large?

The scraper image intentionally ships with everything needed to handle any website:

- **Chromium browser** — full headless browser via Playwright
- **Tesseract OCR engine** — text extraction from images
- **English language pack** — Tesseract training data
- **All system libraries** — libnss3, libgbm1, libatk, and others required for headless browser operation

This means **zero additional setup on your machine**. Pull the image and it works — on any OS, any machine, any environment.

---

## 🔴 NScrape Series

SnapNScrape is one tool in a growing collection of focused scraping and data extraction tools — all Docker-first, all available on Docker Hub under `pratham1uk/`.

| Tool | What It Does | Status |
|------|-------------|:------:|
| 🕷️ **LetsNScrape** | General-purpose web scraper — any URL, any data type | `v1.0.0` ✅ |
| ⚡ **ytNscrape** | YouTube lecture scraper — slide frames + captions + PDF notes | `v1.0.0` ✅ |
| 📸 **SnapNScrape** | General web scraping with OCR failsafe | `v1.0.0` ✅ |

---

<div align="center">

<br/>

**Built by pratham1uk · NScrape Series**

[![SnapNScrape Gateway](https://img.shields.io/badge/DockerHub-snapnscrape--gateway-e8200e?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/snapnscrape-gateway)
[![SnapNScrape Scraper](https://img.shields.io/badge/DockerHub-snapnscrape--scraper-111111?style=flat-square&logo=docker)](https://hub.docker.com/r/pratham1uk/snapnscrape-scraper)

<br/>

</div>
