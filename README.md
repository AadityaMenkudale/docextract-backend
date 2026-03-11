# DocExtract AI – Flask Backend

Complete REST API backend for the DocExtract AI frontend.

## Tech Stack
- **Flask 3** – lightweight WSGI framework
- **Flask-CORS** – cross-origin headers for the React frontend
- **Anthropic SDK** – Claude claude-opus-4-5 for document extraction (PDF / image / DOCX)
- **JSON flat-file storage** – zero-dependency persistence (swap for PostgreSQL in production)

---

## Quick Start

```bash
# 1. Clone / place this folder next to your frontend
cd backend

# 2. Create a virtual environment
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Set your Anthropic API key
cp .env.example .env
# Edit .env and paste your key

# 5. Run
python app.py
# → http://localhost:5000
```

---

## API Reference

### Health
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/health` | Liveness check |
| GET | `/api/stats` | Dashboard summary counts |

### Documents
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/documents/upload` | Upload a file (multipart `file` field) |
| GET | `/api/documents` | Paginated list with filters |
| GET | `/api/documents/:id` | Full detail + extracted fields |
| DELETE | `/api/documents/:id` | Delete document & file |

### Extraction & Review
| Method | Path | Description |
|--------|------|-------------|
| POST | `/api/documents/:id/extract` | Re-run AI extraction |
| PATCH | `/api/documents/:id/fields` | Save human-edited field values |
| POST | `/api/documents/:id/approve` | Mark document as approved |

### File & Export
| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/documents/:id/file` | Stream original file |
| GET | `/api/documents/:id/export?format=json\|csv` | Export extracted data |

---

## Query Parameters – GET /api/documents
| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `page` | int | 1 | Page number |
| `per_page` | int | 10 | Items per page (max 100) |
| `status` | string | – | Filter: Processing / Success / Error / Review Required / Failed |
| `type` | string | – | Filter: Invoice / Contract / Receipt / Identity |
| `search` | string | – | Substring match on filename |

---

## Document Status Flow
```
Upload → Processing → Success
                    → Review Required  (confidence < 80 or manual check)
                    → Error            (AI or file error)
```

---

## Connecting the Frontend

In your React app set:
```
VITE_API_URL=http://localhost:5000
```

Then replace hard-coded mock arrays with `fetch` calls to the API.

Example – upload:
```js
const form = new FormData();
form.append('file', file);
const res = await fetch(`${import.meta.env.VITE_API_URL}/api/documents/upload`, {
  method: 'POST',
  body: form,
});
const doc = await res.json();
```

---

## Production Deployment

```bash
# Using Gunicorn (already in requirements.txt)
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

For production, replace `storage/documents.json` with a proper database
(PostgreSQL + SQLAlchemy recommended) and use a task queue (Celery + Redis)
for async extraction so uploads return immediately.
