# Data Analytics Dashboard

A full-stack data analytics platform built with React, FastAPI, PostgreSQL, and MongoDB.

## Features (Planned)

- 📤 Upload and manage CSV datasets
- 🔍 SQL query interface with syntax highlighting
- 🔄 Data transformations (dbt-like)
- 📊 Interactive visualizations and dashboards
- ⏰ Scheduled data refreshes
- 🔐 User authentication and authorization

## Tech Stack

### Frontend
- React 18 + TypeScript
- Tailwind CSS
- React Router
- Chart.js / Recharts

### Backend
- FastAPI (Python)
- SQLAlchemy ORM
- JWT Authentication
- Pandas for data processing

### Databases
- PostgreSQL (metadata, users, queries)
- MongoDB (raw data, transformations)

### DevOps
- Docker & Docker Compose
- GitHub Actions CI/CD
- Deployed on Vercel (frontend) + Render (backend)

## Getting Started

### Prerequisites
- Node.js 18+
- Python 3.10+
- Docker & Docker Compose

### Local Development

1. Clone the repository
```bash
git clone https://github.com/Surendar-Sakthivel/data-analytics-dashboard.git
cd data-analytics-dashboard
```

2. Start services with Docker Compose
```bash
docker-compose up -d
```

3. Install frontend dependencies
```bash
cd frontend
npm install
npm start
```

4. Install backend dependencies
```bash
cd backend
pip install -r requirements.txt
uvicorn main:app --reload
```

## Project Structure

```
data-analytics-dashboard/
├── frontend/           # React TypeScript application
├── backend/            # FastAPI application
├── docker-compose.yml  # Local development services
└── README.md
```

## Roadmap

- [x] Phase 1: Project setup
- [ ] Phase 2: Authentication system
- [ ] Phase 3: File upload & preview
- [ ] Phase 4: SQL query interface
- [ ] Phase 5: Data transformations
- [ ] Phase 6: Visualizations & dashboards
- [ ] Phase 7: Scheduling & deployment

## License

MIT

## Author

Surendar Sakthivel
- GitHub: [@Surendar-Sakthivel](https://github.com/Surendar-Sakthivel)
