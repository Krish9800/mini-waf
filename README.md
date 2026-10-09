# Mini Web Application Firewall (`mini-waf`)

[![CI](https://github.com/krish/mini-waf/actions/workflows/ci.yml/badge.svg)](https://github.com/krish/mini-waf/actions/workflows/ci.yml)
[![Python 3.11+](https://img.shields.io/badge/python-3.11+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Code Style: Ruff](https://img.shields.io/badge/code%20style-ruff-000000.svg)](https://github.com/astral-sh/ruff)
[![Type Checked: Mypy](https://img.shields.io/badge/mypy-strict-blue.svg)](https://mypy-lang.org/)
[![Security: Bandit](https://img.shields.io/badge/security-bandit-yellow.svg)](https://github.com/PyCQA/bandit)

Enterprise-Grade Modular Web Application Firewall (WAF) reverse-proxy and ASGI middleware in Python.

---

## Features

- **OWASP CRS-Inspired Security Signatures**:
  - SQL Injection: Union-based, boolean-based, time-based, stacked queries, comment obfuscation (CWE-89).
  - Cross-Site Scripting (XSS): Executable tag injection, inline DOM event handlers, javascript pseudo-protocols (CWE-79).
  - Path Traversal & LFI: Directory climbing (`../`, `..\`), sensitive OS files (`/etc/passwd`, `win.ini`), null-byte poisoning (CWE-22).
  - OS Command Injection: Shell metacharacters (`|`, `;`, `&&`, backticks, `$()`), Unix binary paths (CWE-78).
  - Server-Side Request Forgery (SSRF): Private IPv4 ranges, cloud metadata endpoint access (CWE-918).
  - Automated Scanner Fingerprinting: User-Agent detection for tools like sqlmap, nikto, nuclei (CWE-200).
- **Multi-Pass Deobfuscation & Anti-Evasion**:
  - Recursive URL percentage decoding (dismantles double encoding like `%252e%252e`).
  - Unicode NFKC normalization (collapses full-width character homoglyphs like `＜script＞`).
  - HTML entity unescaping (`&lt;`, `&#x3c;`).
  - RFC 3986 path canonicalization and body size clamping to prevent DoS.
- **Anomaly Scoring & Flexible Controls**:
  - Configurable Paranoia Levels (1 to 4) and score thresholds.
  - Dual modes: `BLOCK` (active prevention) and `MONITOR` (passive detection/IDS mode).
- **Rate Limiting & Dynamic IP Jail**:
  - Sliding token bucket rate limiting per client IP.
  - Automatic temporary banning ("jail") for IPs with repeated security violations.
  - CIDR subnet allowlisting and permanent blocklisting.
- **Outbound Data Leakage Shield**:
  - Strips technology fingerprinting headers (`Server`, `X-Powered-By`).
  - Cloaks 500 server stack traces and database driver errors.
  - Credit card number detection with ISO/IEC 7812 Luhn checksum and middle-digit masking.
- **Dual Deployment Options**:
  - **Reverse Proxy**: High-concurrency async reverse proxy forwarding clean traffic via `httpx`.
  - **ASGI Middleware**: Drop-in middleware for FastAPI, Starlette, or Quart applications.
- **Observability & Management**:
  - Structured JSON and ArcSight CEF audit logging for enterprise SIEM ingestion.
  - Rich interactive CLI with simulation testing, rule filtering, and unjail commands.
  - FastAPI management API with interactive OpenAPI docs at `/docs` on port 8083.
  - Rootless Docker container running as non-privileged user (`UID 10001:10001`).

---

## Installation

```bash
# Clone repository
git clone https://github.com/krish/mini-waf.git
cd mini-waf

# Setup virtual environment
python -m venv .venv
source .venv/bin/activate

# Install dependencies
pip install -e ".[dev]"
```

---

## Quick Start CLI

```bash
# Inspect synthetic or real URL payloads
miniwaf test --url "http://example.com/search?q=1%20UNION%20SELECT%20null"

# Test POST body payloads
miniwaf test --url "/api/comments" --method POST --body "<script>alert(1)</script>"

# List all registered OWASP CRS rules
miniwaf rules --category SQLI

# Release an IP address from temporary jail
miniwaf unjail 192.168.1.55
```

---

## Embedding as ASGI Middleware

Drop MiniWAF directly into any existing FastAPI or Starlette application:

```python
from fastapi import FastAPI
from miniwaf.proxy.middleware import MiniWAFASGIMiddleware

app = FastAPI()

# Mount MiniWAF middleware
app.add_middleware(MiniWAFASGIMiddleware)


@app.get("/")
def home():
    return {"message": "Protected by MiniWAF"}
```

---

## Running the Management REST API

```bash
uvicorn miniwaf.api.app:app --host 0.0.0.0 --port 8083
```

Or using Docker Compose:

```bash
docker compose up -d
```

### Endpoints

- `GET /health`: Operational health check.
- `GET /api/v1/rules`: List all registered detection rules.
- `GET /api/v1/stats`: Live telemetry, mode, threshold, and active jail count.
- `GET /api/v1/jailed`: List currently banned IPs and remaining ban seconds.
- `POST /api/v1/inspect`: Test an HTTP transaction against WAF engine.
- `POST /api/v1/unjail`: Unjail a client IP address.

---

## Verification & Quality Gates

Run the automated test suite with coverage and security checks:

```bash
# Run pytest with 90%+ coverage gate
pytest --cov=miniwaf --cov-report=term-missing --cov-fail-under=90

# Static type checking
mypy src tests

# Code style and formatting
ruff check .
ruff format --check .

# Security AST scanner
bandit -r src/ -c pyproject.toml
```

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
