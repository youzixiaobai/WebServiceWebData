# Book Library & Analytics API

Individual coursework project for **XJCO3011 Web Services and Web Data** (University of Leeds): a data-driven REST API with SQL persistence, CRUD, analytics endpoints, API-key authentication on mutating routes, automated tests, and PDF documentation under `docs/`.

## Data source (module brief)

The brief expects use of **publicly available data** (or similar) and **citation** in your technical report.

- **Bundled dataset:** `data/books_gutenberg_catalog_sample.csv` — a small **CSV** derived from the [Project Gutenberg catalog feed](https://www.gutenberg.org/cache/epub/feeds/pg_catalog.csv) (see `scripts/build_gutenberg_csv.py`); columns match `import_csv.py`. Provenance and licence pointers are in `data/DATA_PROVENANCE.txt`.
- **Import into the database:** run:

```bash
python scripts/import_csv.py --replace
```

(`--replace` clears existing `books` rows first; omit it to append only if you manage duplicates yourself.)

You may **replace** this CSV with a download from e.g. **Kaggle**, **data.gov.uk**, or **Open Library** bulk data — keep the same column headers (`title`, `author`, `genre`, `publication_year`, `rating`, `notes`) or adjust `scripts/import_csv.py`. Then cite the **licence and URL** in your report.

## Features (brief alignment)

- **SQL database**: SQLite by default (`books.db`); configurable via `DATABASE_URL` (e.g. PostgreSQL).
- **One resource model (`Book`)** with full **CRUD** over HTTP.
- **≥4 endpoints**: list/create/read/update/delete books, plus **GET `/api/v1/analytics/summary`** for genre aggregates.
- **JSON** request/response bodies; conventional **HTTP status codes** (200, 201, 204, 401, 404, 422).
- **Authentication**: send header `X-API-Key` for `POST`, `PATCH`, `DELETE`. Set `DISABLE_AUTH=true` only for local demos.
- **Runnable locally**; optional deployment to e.g. PythonAnywhere (same ASGI app).

## Quick start

**Requirements:** Python 3.10+ recommended (3.11 tested). Install dependencies:

```bash
pip install -r requirements.txt
```

If you are in mainland China or PyPI is slow/blocked, use a mirror (example: Tsinghua):

```powershell
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
```

Copy environment defaults and adjust the API key for anything beyond local use:

```bash
copy .env.example .env
```

Run the API:

```bash
uvicorn app.main:app --reload
```

**Browser note:** Opening [http://127.0.0.1:8000/](http://127.0.0.1:8000/) shows a short **HTML** landing page with links. The interactive API tester is still **Swagger** at `/docs` (see below).

- Interactive OpenAPI UI (Swagger): [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- OpenAPI JSON: [http://127.0.0.1:8000/openapi.json](http://127.0.0.1:8000/openapi.json)

Optional sample data:

```bash
python scripts/seed_books.py
```

Recommended for coursework: load the **CSV dataset** (see [Data source](#data-source-module-brief) above):

```bash
python scripts/import_csv.py --replace
```

## Troubleshooting: `pip` ProxyError / install fails

### `ERROR: Could not find a version ... (from versions: none)`

This almost always means **pip did not reach PyPI at all** (no index), not that FastAPI is missing. Fix **network / proxy / mirror** first; changing the package version alone will not help until pip can download metadata.

1. **Clear broken proxy** (PowerShell for this session), then retry:

```powershell
$env:HTTP_PROXY=""
$env:HTTPS_PROXY=""
$env:ALL_PROXY=""
pip install -r requirements.txt
```

2. Check **pip’s saved config** (a bad `proxy` here breaks all installs):

```powershell
pip config list
```

Edit or remove proxy entries in `%APPDATA%\pip\pip.ini` if needed.

3. Try **explicit PyPI** (if something hijacked your index URL):

```powershell
pip install -r requirements.txt --index-url https://pypi.org/simple --trusted-host pypi.org --trusted-host files.pythonhosted.org
```

4. If you are in mainland China or international access is unstable, use a **domestic mirror** (see Quick start block above for Tsinghua example).

5. On **Windows**, `uvicorn[standard]` can try to compile **httptools** and fail without **Visual C++ Build Tools**. This repo uses plain **`uvicorn`** in `requirements.txt` to avoid that. (On Linux you may optionally `pip install 'uvicorn[standard]'` for extras.)

### Browser: `/docs` or `/redoc` “connection failed” / cannot open

The docs URLs only work **while the API process is running** and only on the **same machine** that runs Uvicorn.

1. In the project folder, start the server and **leave that window open**:

```powershell
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

2. Use **`http://` not `https://`**, and the port must match (default **8000**):

- [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

3. If it still fails: temporarily disable VPN/proxy for localhost, or allow Python/Uvicorn in Windows Firewall.

### `ProxyError` / `Cannot connect to proxy`

Same as step 1–2 above: your environment points at a proxy that does not exist. Clear env vars and `pip.ini` proxy, then reinstall.

## API documentation (PDF)

The module brief requires API documentation as a **PDF** linked from this README. It must clearly describe endpoints, parameters, responses, examples, and authentication/error behaviour.

- **PDF (submit / link in report):** [`docs/API_DOCUMENTATION.pdf`](docs/API_DOCUMENTATION.pdf) — built from the **same OpenAPI 3 schema** that powers **Swagger UI** (`GET /docs`) and **`GET /openapi.json`**, so it stays aligned with the live API.
- **Machine-readable snapshot (optional hand-in):** [`docs/openapi.json`](docs/openapi.json) — written whenever you regenerate the PDF; identical in content to `/openapi.json` while the app matches this commit.
- **Interactive docs (run server first):** [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) — use **Authorize** and enter `X-API-Key` (Swagger shows the **ApiKeyAuth** scheme on mutating book routes).

**Regenerate** PDF + `openapi.json` (from project root):

```bash
python scripts/generate_docs.py
```

## Technical report

- **Editable source (complete):** [`docs/TECHNICAL_REPORT.md`](docs/TECHNICAL_REPORT.md) — fill in `[Your Name]`, `[Your ID]`, GitHub URL, and the GenAI table/reflection before submission.
- **Generated PDF (for Minerva / GitHub):** run `python scripts/generate_docs.py` to refresh [`docs/TECHNICAL_REPORT.pdf`](docs/TECHNICAL_REPORT.pdf). You may instead export the Markdown to PDF from Word/Google Docs if you prefer nicer typography; **keep within the module limit (5 pages).**

## Testing

```bash
pytest tests -v
```

Tests use an in-memory SQLite database and disable auth via `DISABLE_AUTH=true`.

## Project layout

```
data/
  books_gutenberg_catalog_sample.csv   # PG catalog-derived sample (regenerate: scripts/build_gutenberg_csv.py)
  DATA_PROVENANCE.txt             # how to cite / replace the dataset
app/
  main.py          # FastAPI app, lifespan, validation error handler
  config.py        # Settings (API_KEY, DATABASE_URL, DISABLE_AUTH)
  database.py      # Engine, session, init_db
  models.py        # SQLAlchemy models
  schemas.py       # Pydantic schemas
  crud.py          # Database operations
  deps.py          # API key dependency
  routers/         # books + analytics routes
scripts/
  seed_books.py    # Optional demo rows
  generate_docs.py # Builds PDFs under docs/
tests/
  test_api.py      # API tests
docs/
  API_DOCUMENTATION.pdf    # generated from OpenAPI (Swagger); see scripts/generate_docs.py
  openapi.json             # OpenAPI snapshot (same as /openapi.json when app matches)
  TECHNICAL_REPORT.md      # full technical report (edit before submit)
  TECHNICAL_REPORT.pdf     # generated from the Markdown
```

## Example requests

**Create** (requires key):

```http
POST /api/v1/books/
X-API-Key: dev-api-key-change-in-production
Content-Type: application/json

{
  "title": "Example",
  "author": "Jane Doe",
  "genre": "Fiction",
  "publication_year": 2020,
  "rating": 4.2,
  "notes": null
}
```

**List** (read-only):

```http
GET /api/v1/books/?limit=10
```

**Analytics:**

```http
GET /api/v1/analytics/summary
```

## GitHub submission

Create a **public** repository, push this project with a **visible commit history**, and ensure the Minerva PDF report includes:

- Link to this repo  
- Link to API documentation (this README + `docs/API_DOCUMENTATION.pdf`)  
- Link to presentation slides  
- GenAI declaration and example conversation logs (appendix), if applicable  

---

Module contacts and deadlines are defined in the official assessment brief on Minerva.
