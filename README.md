# Broken Link Checker

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](https://github.com/your-org/broken-link-checker/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

A high-performance, asynchronous background domain crawler, HTTP link validation engine, and interactive CLI audit suite. Designed to recursively traverse web domains, detect dead hyperlinks, validate internal page anchors, catch unhandled redirect loops, and generate structured site health reports in JSON, CSV, and HTML formats.

---

## 🎯 Problem Statement & Target Audience

### The Problem
Modern web applications and content-rich sites continuously evolve. Over time, external sites go offline, internal route paths change, and URL structures shift. This leads to **"link rot"**—a build-up of dead links, invalid page fragments, protocol mismatches, and endless redirect loops. 

Unmonitored broken links cause:
* **Degraded User Experience (UX):** Frustrated users encountering HTTP 404 pages or infinite redirect chains.
* **SEO Penalties:** Search engine crawlers lower index ranks for sites with high error rates and broken internal references.
* **Wasted Crawl Budget:** Search engines waste resources navigating invalid paths instead of indexing actual content.

### Target Audience
* **Web Developers & SREs:** Integrate link checks directly into CI/CD pipelines to catch broken routes before production deployment.
* **SEO Specialists & Technical Marketers:** Audit whole site domain graphs to preserve link equity and optimize search indexing.
* **QA Engineers & Content Managers:** Schedule automated domain scans and review actionable failure reports via interactive terminal interfaces or REST APIs.

---

## ✨ Key Features

* **Recursive Concurrent Domain Crawler:** Asynchronously traverses websites up to configurable depth limits. Includes strict domain scoping controls (same-domain, cross-domain, subdomain inclusion) and token-bucket rate limiting to prevent overwhelming target hosts.
* **Robust HTML & Fragment Parsing:** Leverages `BeautifulSoup4` for parsing relative paths, protocol-relative URLs (`//example.com`), inline base tags (`<base href="...">`), and internal hash anchors (`#target-element`).
* **Rich Terminal Interface:** Interactive command-line client built with `Rich` featuring real-time crawling progress bars, status distribution charts, and filterable broken link tables.
* **Structured Multi-Format Data Export:** Uses strict `Pydantic` models for data validation and exports actionable site audit reports to `JSON`, `CSV`, or standalone styled `HTML` documents.
* **FastAPI REST Service & Background Worker:** Run the engine as a headless REST API with async job execution, status polling endpoints, and periodic scan scheduling using `APScheduler`.

---

## 🏗️ Architecture & System Design

The system employs a decoupled, asynchronous architecture separating link extraction, validation, state management, and user interfaces (CLI/API).

```
                      +-----------------------------------+
                      |         User Interfaces           |
                      |  [ Rich CLI ]    [ FastAPI REST ] |
                      +-----------------+-----------------+
                                        |
                                        v
                      +-----------------------------------+
                      |      Async Crawler Engine         |
                      |          (src/core.py)            |
                      +--------+-----------------+--------+
                               |                 |
            +------------------+                 +------------------+
            v                                                       v
+-----------------------+                               +-----------------------+
|  HTML Parser & URL    |                               |  Async HTTP Fetcher   |
|  Normalizer (BS4)     |                               |  (httpx / asyncio)    |
+-----------------------+                               +-----------------------+
            |                                                       |
            +------------------+                 +------------------+
                               |                 |
                               v                 v
                      +-----------------------------------+
                      |    State & Deduplication Registry  |
                      |          (src/models.py)          |
                      +-----------------+-----------------+
                                        |
                                        v
                      +-----------------------------------+
                      |     Report Generation Engine      |
                      |     (JSON / CSV / HTML Exporter)  |
                      +-----------------------------------+
```

### Module Breakdown

| Module | Purpose |
| :--- | :--- |
| `src/models.py` | Pydantic domain models defining crawl configurations, scoping rules, link verification states, HTTP status classifications, and report export schemas. |
| `src/core.py` | Asynchronous crawler engine, BeautifulSoup4 HTML link extractor/normalizer, concurrent HTTP fetcher using `asyncio`/`httpx`, deduplication registry, and report exporters. |
| `src/api.py` | FastAPI REST interface allowing users to trigger crawl jobs asynchronously, poll job metrics, retrieve broken link reports, and check service health. |
| `src/main.py` | Rich-based CLI command suite for interactive terminal domain audits with real-time progress indicators, status breakdown tables, and API server entrypoint. |

---

## 🛠️ Technology Stack

| Component | Technology | Description |
| :--- | :--- | :--- |
| **Language** | Python 3.10+ | Core language execution environment |
| **Async Engine** | `asyncio` / `httpx` | Non-blocking I/O operations and concurrent HTTP requests |
| **HTML Parser** | `BeautifulSoup4` / `lxml` | DOM parsing, link extraction, and path normalization |
| **Web Framework** | `FastAPI` / `Uvicorn` | RESTful API server for headless execution |
| **CLI & Terminal UI** | `Rich` | Real-time progress bars, tables, and colored terminal formatting |
| **Data Validation** | `Pydantic v2` | Type enforcement, settings management, and report schemas |
| **Job Scheduling** | `APScheduler` | Automated background recurring audits |
| **Testing** | `pytest` / `pytest-asyncio` | Unit testing and async integration test suite |

---

## ⚡ Installation & Virtual Environment Setup

### Prerequisites
* Python 3.10 or higher
* `pip` package manager

### Steps

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-org/broken-link-checker.git
   cd broken-link-checker
   ```

2. **Create and activate a virtual environment:**
   ```bash
   # On macOS / Linux
   python3 -m venv venv
   source venv/bin/activate

   # On Windows (PowerShell)
   python -m venv venv
   .\venv\Scripts\Activate.ps1
   ```

3. **Install dependencies:**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

---

## ⚙️ Environment Variables Configuration

The application reads configuration from environment variables or a `.env` file. 

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Configure settings according to your environment:

```ini
# Core Crawler Settings
MAX_CONCURRENT_REQUESTS=20
CRAWL_TIMEOUT_SECONDS=10.0
DEFAULT_USER_AGENT=BrokenLinkChecker/1.0 (+https://github.com/your-org/broken-link-checker)
MAX_CRAWL_DEPTH=3
ALLOW_SUBDOMAINS=false

# API Server Configuration
API_HOST=127.0.0.1
API_PORT=8000
API_DEBUG=false

# Scheduler Settings
ENABLE_SCHEDULER=false
CRON_SCHEDULE="0 0 * * *"
```

---

## 🚀 Usage

### 1. Interactive CLI Client

Audit a target website directly from your terminal:

```bash
# Run a standard domain audit
python -m src.main audit --url https://example.com

# Crawl with custom depth, concurrency limit, and output format
python -m src.main audit \
  --url https://example.com \
  --depth 4 \
  --workers 15 \
  --format html \
  --output report.html
```

#### CLI Options Summary

```text
Usage: python -m src.main audit [OPTIONS]

Options:
  --url TEXT             Target URL to crawl [required]
  --depth INTEGER        Maximum depth to crawl [default: 3]
  --workers INTEGER      Maximum concurrent HTTP workers [default: 10]
  --format [json|csv|html] Export format for findings [default: json]
  --output PATH          Output destination filepath
  --help                 Show this message and exit.
```

### 2. REST API Server

Start the background service and REST API using the CLI runner:

```bash
python -m src.main server --host 127.0.0.1 --port 8000
```

#### Primary API Endpoints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/health` | Check service operational status |
| `POST` | `/api/v1/crawls` | Trigger an asynchronous domain crawl job |
| `GET` | `/api/v1/crawls/{job_id}` | Poll current crawl job progress and status |
| `GET` | `/api/v1/crawls/{job_id}/report` | Fetch structured broken link report (`JSON`) |

#### Example: Initiating a Crawl via `curl`

```bash
curl -X POST "http://127.0.0.1:8000/api/v1/crawls" \
     -H "Content-Type: application/json" \
     -d '{
           "target_url": "https://example.com",
           "max_depth": 2,
           "allow_subdomains": false
         }'
```

Response:
```json
{
  "job_id": "a9c3d4e1-7890-4b2a-8c11-92e1047faef2",
  "status": "queued",
  "created_at": "2023-10-24T14:32:00Z"
}
```

---

## 🧪 Automated Testing Instructions

The repository includes comprehensive unit and integration tests written using `pytest`.

### Run All Tests
```bash
pytest
```

### Run Core and API Tests Separately
```bash
# Core crawler and parser engine tests
pytest tests/test_core.py

# API endpoints and FastAPI server tests
pytest tests/test_api.py
```

### Run Tests with Coverage Report
```bash
pytest --cov=src --cov-report=term-missing
```

---

## 📁 Project Structure

```text
broken-link-checker/
│
├── .env.example           # Reference environment variables definition
├── .gitignore              # Git ignore rules for virtualenvs, caches, and build artifacts
├── requirements.txt        # Production dependencies listing
│
├── src/                    # Primary application package
│   ├── __init__.py         # Package initialization marker
│   ├── models.py           # Pydantic schemas, ENUMs, domain models
│   ├── core.py             # Crawler logic, HTML parser, async fetcher, report generators
│   ├── api.py              # FastAPI application server and REST endpoints
│   └── main.py             # Rich CLI entrypoint and server runner
│
└── tests/                  # Test suite
    ├── __init__.py         # Test package initialization
    ├── test_core.py        # Unit tests for crawler engine and parsing logic
    └── test_api.py         # Integration tests for FastAPI endpoints
```

---

## 🛠 Roadmap & Future Enhancements

- [ ] **Headless Browser Integration:** Optional rendering engine support via Playwright to detect links rendered dynamically via client-side JavaScript.
- [ ] **Automated Sitemap XML Ingestion:** Support parsing `sitemap.xml` directly to seed initial crawl targets.
- [ ] **Notification Webhooks:** Webhook alerts (Slack, Discord, PagerDuty, Teams) triggered when broken links exceed threshold ratios.
- [ ] **Prometheus Metrics:** Standard `/metrics` endpoint to monitor crawl queue latency, active HTTP client worker counts, and failure rate distributions.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.