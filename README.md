# Campus Ride — Distributed Transportation Platform

A real-time green taxi and transportation sharing platform built for university campus, demonstrating distributed systems concepts across containers, Kubernetes, and cloud deployment.

## Live URLs
- Frontend: https://frontend-production-cb5a.up.railway.app
- Backend API: https://backend-production-a93d.up.railway.app

## Architecture Diagram

┌─────────────────────────────────────────────────┐

│              TIER 1 - PRESENTATION               │

│         Next.js 16 + Tailwind CSS + Leaflet     │

│              http://localhost:3000               │

└──────────────────┬──────────────────────────────┘

│ HTTP REST API

┌──────────────────▼──────────────────────────────┐

│              TIER 2 - APPLICATION                │

│           FastAPI + Uvicorn (Port 8000)         │

│         SQLAlchemy ORM + asyncpg driver         │

└──────────────────┬──────────────────────────────┘

│ asyncpg TCP:5432

┌──────────────────▼──────────────────────────────┐

│                TIER 3 - DATA                     │

│    PostgreSQL (local) + Supabase (cloud BaaS)   │

└─────────────────────────────────────────────────┘
Infrastructure:

Docker → Kubernetes → Railway (Cloud PaaS)

Auth: Supabase JWT + RLS

Network Analysis: Wireshark

Load Testing: Artillery.io

## Prerequisites

| Tool | Version | Link |
|------|---------|------|
| Git | Latest | https://git-scm.com |
| Node.js | 18+ | https://nodejs.org |
| Python | 3.11+ | https://python.org |
| Docker Desktop | 4.25+ | https://docker.com |
| kubectl | 1.28+ | https://kubernetes.io |
| Wireshark | Latest | https://wireshark.org |
| Artillery | Latest | npm install -g artillery |

## Setup Instructions

### Part 1 — Local Development

```bash
# Backend
cd backend
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
python server.py

# Frontend (new terminal)
cd frontend
npm install
npm run dev
```

Verify:
- Backend: http://localhost:8000/api/v1/students/
- Frontend: http://localhost:3000

### Part 2 — Supabase Authentication

1. Create project at supabase.com → Select Singapore region
2. Go to Settings → API → copy Project URL and anon key
3. Create `frontend/.env.local`:
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co

NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
4. Run migration in Supabase SQL Editor: `supabase/migrations/001_university_ride.sql`

### Part 3 — Docker

```bash
docker-compose build
docker-compose up -d
docker ps
docker-compose logs --tail=30
```

### Part 4 — Kubernetes

```bash
kubectl apply -f k8s/
kubectl get pods -w
kubectl get services
kubectl get hpa
kubectl describe deployment campus-backend
kubectl port-forward svc/campus-frontend-service 3000:3000
```

### Part 5 — Railway Cloud Deployment

```bash
npm install -g @railway/cli
railway login
railway init
railway add --service backend
railway add --service frontend
railway up --service backend
railway up --service frontend
railway domain --service frontend
```

### Part 6 — Wireshark Filters

| Filter | Purpose |
|--------|---------|
| `host 127.0.0.1 and port 8000` | Local backend traffic |
| `http` | HTTP packets only |
| `http.request.method == "POST"` | POST requests |
| `tcp.flags.fin == 1` | TCP FIN packets |
| `tls` | HTTPS/TLS packets |
| `tls.handshake` | TLS handshake only |
| `host 172.19.0.2 or host 172.19.0.3` | Docker container traffic |

### Part 7 — Load Testing

```bash
artillery run load-test.yml
```

## Environment Variables

| Variable | Service | Description | Source |
|----------|---------|-------------|--------|
| `NEXT_PUBLIC_SUPABASE_URL` | Frontend | Supabase project URL | Supabase Dashboard |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Frontend | Supabase public key | Supabase Dashboard |
| `DATABASE_URL` | Backend | PostgreSQL connection string | Railway PostgreSQL |
| `NEXT_PUBLIC_API_URL` | Frontend | Backend API URL | Railway Domain |

## Kubernetes Manifests

| File | Description |
|------|-------------|
| `k8s/configmap.yaml` | Non-sensitive config (backend_host, port, node_env) |
| `k8s/secret.yaml` | Sensitive credentials (Supabase keys, DATABASE_URL) |
| `k8s/postgres.yaml` | PostgreSQL Deployment + ClusterIP Service |
| `k8s/backend.yaml` | FastAPI Deployment (2 replicas) + liveness/readiness probes |
| `k8s/frontend.yaml` | Next.js Deployment (2 replicas) + ClusterIP Service |
| `k8s/ingress.yaml` | NGINX Ingress routing / to frontend, /api to backend |
| `k8s/hpa.yaml` | HPA min=2, max=10 pods, CPU threshold 70% |

## Team Members

| Name | Matric No | Role | Parts |
|------|-----------|------|-------|
| [Leader] | | Team Leader | 1, 2, 3 |
| [Member 2] | | Developer | 4, 5 |
| [Member 3] | | Developer | 6, 7, 8 |

## Troubleshooting

**1. psycopg2 build error in Docker**
Error: pg_config executable not found

Solution: Change psycopg2 to psycopg2-binary in requirements.txt

**2. Port 3000 already in use**
Error: bind: Only one usage of each socket address

Solution: netstat -ano | findstr :3000

taskkill /PID <pid> /F

**3. Kubernetes ErrImageNeverPull**
Error: Failed to pull image

Solution:

docker save image:latest -o image.tar

docker cp image.tar desktop-control-plane:/root/

docker exec desktop-control-plane ctr -n k8s.io images import /root/image.tar

