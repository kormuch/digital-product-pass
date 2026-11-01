# PAUL — Product Attribute Unified Layer

A Digital Product Pass platform that ingests heterogeneous product data, uses a two-stage AI pipeline to classify and extract structured information, and lets humans review before persisting.

---

## The Problem

Product information in manufacturing companies is scattered across ERP systems, PLM tools, external portals, and dozens of file formats — CSV, XLSX, JSON, XML, PDF, scanned images. There is no single source of truth, which makes EU Digital Product Passport compliance practically impossible without manual data wrangling at scale.

## The Solution

PAUL accepts any product-related file (datasheets, lab reports, certificates, catalogs, marketing material), runs it through a two-stage AI pipeline that classifies the document type and extracts structured product attributes with source citations, then presents the results for human review before committing to the database. Confirmed data feeds into a lakehouse for analytics and a knowledge graph for relationship queries.

---

## Architecture

```
                         +---------------------------------------------+
                         |              PAUL Frontend                   |
                         |         React / Vite / Tailwind              |
                         +--------------------+------------------------+
                                              |
                         +--------------------v------------------------+
                         |            FastAPI Backend                   |
                         |                                             |
                         |  +---------------+    +------------------+  |
  Drop files ---------->|  |  Classifier    |--->|    Extractor     |  |
  (any format)          |  |  Agent         |    |    Agent         |  |
                         |  |  (doc type +   |    |  (attributes +   |  |
                         |  |   confidence)  |    |   citations)     |  |
                         |  +---------------+    +--------+---------+  |
                         |                                |            |
                         |              Human Review <----+            |
                         |                   |                         |
                         +-------------------+-------------------------+
                                             |
                    +------------------------+------------------------+
                    v                        v                        v
            +--------------+    +--------------------+    +--------------+
            | PostgreSQL   |    | Lakehouse          |    | Neo4j        |
            | (primary DB) |    | MinIO > Iceberg >  |    | Knowledge    |
            |              |    | Trino > OpenMetadata|   | Graph        |
            +--------------+    +--------------------+    +--------------+
```

---

## Key Features

- **Two-stage AI pipeline** — Classifier Agent determines document type and confidence; Extractor Agent pulls structured attributes with citations back to the source text
- **Confidence gate** — Documents scoring >= 85% are auto-extracted; below that, the user picks or types the document type for a second pass
- **Existing product diff** — When a product already exists, attribute-level changes (new, changed, kept) are shown before confirmation
- **Multi-source detection** — Same article number found across multiple files during bulk upload triggers automatic merge on confirm
- **LLM provider chain** — Ollama (local), DeepSeek, Gemini, Groq — config-driven fallback so the system works offline or with any provider
- **OCR support** — Scanned PDFs and images (PNG, JPG, TIFF, BMP, WEBP) processed via Tesseract OCR (German + English)
- **Lakehouse sync** — Confirmed products are mirrored to Apache Iceberg tables, queryable via Trino SQL
- **Knowledge graph** — Neo4j graph layer for product relationships, lineage, and evidence tracking
- **AI transparency** — Full prompt visibility and model configuration exposed via API endpoints
- **Bulk upload** — Multiple files processed in sequence with rate-limit awareness

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React, TypeScript, Vite, Tailwind CSS |
| Backend | Python, FastAPI, SQLAlchemy |
| Database | PostgreSQL 16 |
| AI / LLM | Ollama, DeepSeek, Gemini, Groq (config-driven) |
| OCR | Tesseract (deu + eng), pdfplumber, pytesseract |
| Lakehouse | Apache Iceberg, Trino, MinIO |
| Data Catalog | OpenMetadata |
| Graph | Neo4j |
| Infrastructure | Docker Compose |

---

## Quick Start

```bash
# Full stack (includes OpenMetadata)
cd data-layer && docker compose --profile openmetadata up -d --build

# Lightweight (without OpenMetadata, ~2 GB RAM)
cd data-layer && docker compose up -d --build
```

| Service | URL |
|---------|-----|
| PAUL UI | `http://localhost:3000` |
| PAUL API (Swagger) | `http://localhost:8000/docs` |
| Trino SQL | `http://localhost:8082` |
| MinIO Console | `http://localhost:9001` |
| OpenMetadata | `http://localhost:8585` |

**Requirements:** Docker + Docker Compose, ~4 GB RAM for full stack. Optional `.env` in `data-layer/` for LLM API keys (defaults to local Ollama).

---

## API Overview

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/analyze/` | Upload file — classify and extract |
| `POST` | `/api/v1/analyze/confirm` | Confirm extracted products and persist |
| `POST` | `/api/v1/analyze/re-extract` | Re-extract with user-chosen document type |
| `GET/POST` | `/api/v1/products/` | List or create products |
| `GET/POST` | `/api/v1/families/` | List or create product families |
| `GET` | `/api/v1/export/products.json` | Export in product-layer format |
| `GET` | `/api/v1/analyze/prompts` | AI pipeline transparency |
| `GET` | `/api/v1/lakehouse/products` | Query Iceberg tables via Trino |

---

## SBOM & Vulnerability Analysis

PAUL includes a supply chain security module that connects to external REST APIs, generates Software Bills of Materials, and checks components against known vulnerabilities.

- **SBOM generation** — Builds CycloneDX 1.5 SBOMs from Bill of Materials data retrieved via REST API
- **CVE detection** — Checks all BOM components against the NIST National Vulnerability Database
- **VEX output** — Generates Vulnerability Exploitability eXchange documents for discovered CVEs
- **Certificate extraction** — Decodes and stores CE certificates (EMC 2014/30/EU, LVD 2014/35/EU, RoHS 2011/65/EU)

Sample results from test data:
- 41 components identified across 2 products
- 19 real CVEs found (BLE vulnerabilities, access control bypasses, DoS vectors)
- Full CycloneDX 1.5 SBOM and VEX documents generated

---

## Supported File Formats

PDF, PNG, JPG, TIFF, BMP, WEBP, CSV, TSV, XLSX, XLS, JSON, XML, TXT, MD, LOG

## Recognized Document Types

Datasheet, Lab Report, Certificate, Software Documentation, Bill of Materials, Marketing Material, Compliance Declaration, Safety Data Sheet, Product Specification, Test Report
