# Backend - FastAPI Application

## Setup

1. Create virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Run development server:
```bash
uvicorn main:app --reload
```

API will be available at: http://localhost:8000
API documentation: http://localhost:8000/docs

## Project Structure

```
backend/
├── main.py              # FastAPI application entry point
├── requirements.txt     # Python dependencies
├── models/              # SQLAlchemy models
├── routes/              # API route handlers
├── schemas/             # Pydantic schemas
├── services/            # Business logic
├── database.py          # Database configuration
└── config.py            # Application configuration
```
