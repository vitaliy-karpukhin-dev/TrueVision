# TrueVision

**Personal finance system** for individuals and small businesses.  
Upload PDFs (bank statements, invoices, budgets) and AI extracts the data. The system then shows transactions, budget plan vs. actual, analytics and a payment calendar.

> Control. Growth. Confidence.

**Kurzbeschreibung (DE):** TrueVision ist ein persönliches Finanzsystem für Privatpersonen und Kleinunternehmen. Dokumente wie Kontoauszüge, Rechnungen und Budgets werden als PDF hochgeladen, die KI extrahiert die Daten, und das System zeigt Transaktionen, Budget Plan/Ist, Analysen und einen Zahlungskalender. Eigenes Full-Stack-Projekt mit FastAPI, PostgreSQL, React und Docker; die Oberfläche ist auf Deutsch und Russisch verfügbar.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 18 + Vite 6, React Router v6, Recharts |
| **Backend** | FastAPI (Python 3.13), SQLAlchemy, Alembic |
| **Database** | PostgreSQL 15 |
| **AI** | Anthropic Claude (Vision, Sonnet) + OpenAI GPT-4o |
| **OCR** | PyMuPDF (fitz) + Tesseract + pdfplumber |
| **Auth** | JWT (python-jose) + bcrypt + email verification |
| **Email** | Gmail SMTP (App Password) |
| **Infrastructure** | Docker + Docker Compose, ngrok (dev) |

---

## Project Structure

```
TrueVision/
├── app/                          # FastAPI backend
│   ├── main.py                   # Entry point, CORS, routers
│   ├── config.py                 # Loads .env + .env.local
│   ├── db/
│   │   └── database.py           # SQLAlchemy engine, SessionLocal
│   ├── models/
│   │   ├── user.py               # User (JWT auth, email verification)
│   │   ├── document.py           # Document (file + extraction_result)
│   │   ├── financial_event.py    # Transactions
│   │   ├── budget.py             # Budget by period (JSONB)
│   │   ├── company.py            # User's company
│   │   ├── notification.py       # Notifications
│   │   └── project.py            # Projects
│   ├── routes/
│   │   ├── auth.py               # /auth: registration, login, verification
│   │   ├── upload.py             # /documents: upload, analysis, deletion
│   │   ├── dashboard.py          # /dashboard: metrics, transactions, forecast
│   │   ├── budget.py             # /budget: budget CRUD, PDF import, plan vs. actual
│   │   ├── chat.py               # /chat: AI assistant
│   │   ├── analysis.py           # /documents/{id}/analyze
│   │   └── notifications.py      # /notifications
│   └── services/
│       ├── email.py              # Sending emails via Gmail SMTP
│       ├── ai.py                 # OpenAI integration
│       ├── extraction.py         # PDF parsing, metric aggregation
│       ├── document_processor.py # Document processing pipeline
│       └── event_builder.py      # Creates FinancialEvent from analysis
│
├── truevision-frontend/          # React frontend
│   ├── src/
│   │   ├── App.jsx               # Routing, ProtectedRoute
│   │   ├── api/client.js         # Axios with JWT, /api proxy
│   │   ├── context/LangContext   # RU / DE localization
│   │   ├── locales/i18n.js       # All translations
│   │   └── pages/
│   │       ├── Login/            # Login + registration
│   │       ├── VerifyEmail/      # Email confirmation
│   │       ├── PathSelection/    # Path selection (employee / entrepreneur)
│   │       ├── Onboarding/       # Onboarding
│   │       ├── Dashboard/        # Home: metrics, chart, notifications
│   │       ├── Finances/         # Overview + budget planner
│   │       ├── Transactions/     # Transaction list, filters, export
│   │       ├── Documents/        # Document upload and analysis
│   │       ├── Analytics/        # Growth charts, financial model
│   │       ├── Calendar/         # Recurring payments
│   │       ├── AIChat/           # AI finance assistant
│   │       └── Profile/          # Profile, company, password change
│   └── vite.config.js            # Proxy /api → localhost:8000
│
├── migrations/                   # Alembic migrations
├── uploads/                      # Uploaded files (not in git)
├── docker-compose.yml
├── Dockerfile
├── requirements.txt
├── .env                          # Environment variables (not in git)
└── .env.local                    # Local variables (not in git)
```

---

## Environment Variables

### `.env`
```env
# Database
DB_USER=<db-user>
DB_PASSWORD=<db-password>
DB_NAME=truevision
DB_HOST=db           # localhost for local runs without Docker
DB_PORT=5432

# JWT
SECRET_KEY=<your-secret-key>
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# AI
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Email (Gmail App Password)
SMTP_EMAIL=your@gmail.com
SMTP_PASSWORD=xxxx xxxx xxxx xxxx

# Misc
UPLOAD_TOKEN=<upload-token>
```

### `.env.local` (local overrides)
```env
DB_HOST=localhost
DB_PORT=5432
FRONTEND_URL=https://your-ngrok-url.ngrok-free.dev
```

---

## Running Locally

### Requirements
- Python 3.13+
- Node.js 18+
- PostgreSQL 15+
- Tesseract OCR (`brew install tesseract tesseract-lang`)

### Backend
```bash
cd TrueVision

# Virtual environment
python3 -m venv venv
source venv/bin/activate

# Dependencies
pip install -r requirements.txt

# Migrations
alembic upgrade head

# Run (with auto-reload)
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### Frontend
```bash
cd truevision-frontend
npm install
npm run dev -- --host
```

### With Docker
```bash
docker compose up --build
```

Services:
- Frontend: http://localhost:5173
- Backend API: http://localhost:8000
- Swagger UI: http://localhost:8000/docs

### With ngrok (public access)
```bash
# In a separate terminal
ngrok http 5173
```
Swagger via ngrok: `https://your-url.ngrok-free.dev/docs`

---

## API: Main Endpoints

Full documentation: **http://localhost:8000/docs**

### Auth `/auth`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/auth/register` | Registration (sends verification email) |
| POST | `/auth/login` | Login → JWT token |
| GET | `/auth/verify?token=` | Email confirmation |
| GET | `/auth/me` | Current user |
| PUT | `/auth/update` | Update profile |
| POST | `/auth/change-password` | Change password |

### Documents `/documents`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/documents/upload` | Upload PDF / JPG / PNG |
| GET | `/documents/` | List the user's documents |
| GET | `/documents/{id}` | Download file |
| POST | `/documents/{id}/analyze` | AI analysis of a document |
| DELETE | `/documents/{id}` | Delete document |

### Budget `/budget`
| Method | Endpoint | Description |
|---|---|---|
| GET | `/budget?period=YYYY-MM` | Get budget for a period |
| PUT | `/budget?period=YYYY-MM` | Save budget |
| POST | `/budget/import/{doc_id}` | Import budget from PDF |
| GET | `/budget/actual?period=YYYY-MM` | Actual spending from transactions |
| GET | `/budget/history` | Budget history by period |

### Dashboard `/dashboard`
| Method | Endpoint | Description |
|---|---|---|
| GET | `/dashboard?period=` | Metrics: income, expenses, score |
| GET | `/dashboard/transactions` | Transaction list with filters |
| POST | `/dashboard/transactions` | Add a transaction manually |
| DELETE | `/dashboard/transactions/{id}` | Delete a transaction |
| GET | `/dashboard/chart-data?period=` | Chart data |
| GET | `/dashboard/forecast` | Forecast based on recurring payments |
| GET | `/dashboard/export` | CSV export |

### Chat `/chat`
| Method | Endpoint | Description |
|---|---|---|
| POST | `/chat/query` | Ask the AI assistant |

---

## How AI Document Analysis Works

```
PDF / JPG  →  pdfplumber (text)
                │
                ├─ text found? → Claude Anthropic (structuring)
                │
                └─ no text (image PDF)? → Claude Vision (OCR + analysis)
                        │
                        └─ result → FinancialEvent in DB
```

### How Budget Import from PDF Works (Haushaltsbudget)

```
PDF  →  pdfplumber (has text?)
          │
          ├─ Yes → Column crop parser (4 columns: Schutzengel/Wohnen/Leben/Sparen)
          │           └─ nothing found → Claude Vision fallback
          │
          └─ No (image PDF) → Column-wise OCR (PyMuPDF + Tesseract)
                                    └─ dynamic detection of column headers
                                           by keywords in the content
```

### Budget Plan vs. Actual

```
GET /budget/actual?period=YYYY-MM
    │
    └─ FinancialEvents for the period
           │
           └─ category mapping:
                rent        → housing
                insurance   → insurance
                expense/
                materials/
                software    → living
                (everything else) → living
```

---

## Email Verification

1. The user registers → `is_verified=False`, the token is stored in the DB
2. Gmail sends an email with a link to `{FRONTEND_URL}/verify-email?token=xxx`
3. The user clicks it → the frontend calls `GET /api/auth/verify?token=xxx`
4. The backend sets `is_verified=True` → redirect to `/login`

> If `SMTP_EMAIL` / `SMTP_PASSWORD` are not set, the user is verified automatically (for the dev environment).

---

## Localization

Two languages are supported: **RU** and **DE**.  
The language can be switched in the app header. All strings are in `truevision-frontend/src/locales/i18n.js`.

---

## Data Models

### User
```
id, email, password (bcrypt), role, first_name, last_name, phone,
avatar_url, is_active, is_verified, verification_token,
path (employee/entrepreneur), created_at, updated_at
```

### Document
```
id, filename, file_path, file_size, status (uploaded/analyzed/processing_failed),
doc_type, extraction_result (JSONB), payment_status, owner_id, created_at
```

### FinancialEvent (transactions)
```
id, user_id, document_id, event_type, vendor, amount, currency,
category, event_date, is_recurring, created_at
```

### Budget
```
id, user_id, period (YYYY-MM), data (JSONB: {income, categories[]}), updated_at
```

### Company
```
id, owner_id, name, position, tax_id, iban, ...
```

### Notification
```
id, user_id, title, body, type (info/warning/error), is_read, created_at
```

---

## Feature Status

| Feature | Status |
|---|---|
| Authentication (JWT + bcrypt + email) | ✅ Done |
| Document upload and AI analysis | ✅ Done |
| Budget planner (import from PDF) | ✅ Done |
| Budget plan vs. actual | ✅ Done |
| Transactions (CRUD, filters, export) | ✅ Done |
| Analytics (charts, financial model) | ✅ Done |
| Payment calendar | ✅ Done |
| AI chat | ✅ Done |
| Profile + company | ✅ Done |
| Responsive design (mobile + desktop) | ✅ Done |
| RU / DE localization | ✅ Done |
| Onboarding ("Entrepreneur" path) | 🔄 Partial |
| Notifications (UI) | 🔄 Partial |
| Bank integrations (Open Banking) | 📋 Planned |
| Multi-currency | 📋 Planned |
| Monetization / subscription | 📋 Planned |
| Mobile app | 📋 Planned |
