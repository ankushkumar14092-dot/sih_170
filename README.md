# SIH26170 — AI-Driven Anomaly Detection in Component Burn-In & Screening
## Indian Space Research Organisation (ISRO)

> **IMPORTANT DISCLAIMER**: This is a prototype/research system developed for Smart India Hackathon (SIH). It is an assistive decision-support tool. It does NOT replace mandatory engineering limits, official ISRO acceptance criteria, or certified screening procedures. All validation results in this repository use synthetic/simulated data unless explicitly stated otherwise.

---

## Quick Start (SIH Demo)

```bash
git clone <repo>
cd sih_170
cp .env.example .env
docker-compose up --build
```

- Frontend: http://localhost:3000
- Backend API: http://localhost:8000
- API Docs: http://localhost:8000/docs
- Grafana: http://localhost:3001
- MLflow: http://localhost:5000

---

## Project Structure

```
sih_170/
├── frontend/          # Next.js engineering dashboard
├── backend/           # FastAPI backend services
├── ml/                # AI/ML models and pipelines
├── data/              # Data schemas, synthetic data, samples
├── models/            # Model registry and artifacts
├── services/          # Microservices (ingestion, alerting)
├── database/          # Migrations, seeds, schemas
├── tests/             # Full test suite
├── infra/             # Docker, monitoring, deployment
├── docs/              # Architecture diagrams, API docs
├── scripts/           # Data generation, training, deployment
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── DOCUMENTATION.md   # Full technical design document
```

---

## See DOCUMENTATION.md for complete system architecture, design decisions, and jury defense.
