# 🍽️ Restaurant Table Reservation System

A full-stack, microservices-based restaurant table reservation system with CI/CD and Kubernetes deployment.

## Architecture

```
                    ┌─────────────┐
                    │   Nginx UI  │ :3000
                    │  (React)    │
                    └──────┬──────┘
                           │
              ┌────────────┴────────────┐
              │                         │
    ┌─────────▼─────────┐   ┌──────────▼──────────┐
    │  Table Service     │   │ Reservation Service  │
    │  (Flask) :5001     │   │ (Express) :5002      │
    └─────────┬─────────┘   └──────────┬──────────┘
              │                         │
              └────────────┬────────────┘
                           │
                    ┌──────▼──────┐
                    │ PostgreSQL  │ :5432
                    │   16        │
                    └─────────────┘
```

## Repositories

| Repo | Description | Tech |
|------|-------------|------|
| [restaurant-table-service](https://github.com/ashwinberyl/restaurant-table-service) | Table management API | Python, Flask, SQLAlchemy |
| [restaurant-reservation-service](https://github.com/ashwinberyl/restaurant-reservation-service) | Reservation booking API | Node.js, Express, Sequelize |
| [restaurant-ui](https://github.com/ashwinberyl/restaurant-ui) | Frontend SPA | React, Vite |
| [restaurant-infra](https://github.com/ashwinberyl/restaurant-infra) | Infrastructure & deployment | Docker Compose, K8s, ArgoCD |

## Quick Start (Docker Compose)

```bash
# Clone all repos into the same parent directory
git clone https://github.com/ashwinberyl/restaurant-table-service.git
git clone https://github.com/ashwinberyl/restaurant-reservation-service.git
git clone https://github.com/ashwinberyl/restaurant-ui.git
git clone https://github.com/ashwinberyl/restaurant-infra.git

# Start everything
cd restaurant-infra
cp .env.example .env    # Edit with your secrets
docker compose up --build -d

# Access the app
open http://localhost:3000
```

## Kubernetes Deployment (Minikube)

```bash
# Start Minikube
minikube start

# Apply manifests
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/secrets.yaml
kubectl apply -f k8s/postgres.yaml
kubectl apply -f k8s/table-service.yaml
kubectl apply -f k8s/reservation-service.yaml
kubectl apply -f k8s/ui.yaml

# Get UI URL
minikube service restaurant-ui -n restaurant --url
```

## ArgoCD Deployment

```bash
# Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Apply the application
kubectl apply -f argocd/application.yaml

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## API Documentation

| Service | Swagger UI |
|---------|------------|
| Table Service | http://localhost:5001/api-docs/ |
| Reservation Service | http://localhost:5002/api-docs |

## CI/CD Pipeline

```
Push → GitHub Actions → Lint → Test → Build Image → Push to GHCR → ArgoCD Sync → Minikube
```

Each service repo has a CI workflow that:
1. **Lints** the code (flake8 / eslint)
2. **Runs tests** (pytest / jest)
3. **Scans for secrets** (gitleaks)
4. **Builds & pushes** Docker image to GHCR

## Secrets Management

| Environment | Method |
|-------------|--------|
| Local | `.env` files (gitignored) |
| GitHub Actions | GitHub Secrets |
| Kubernetes | K8s Secrets |

## Business Rules

- Fixed **2-hour** reservation time slots
- Tables cannot be booked beyond their seating capacity
- No double-booking (same table, date, and time slot)
- Cancellation only allowed **1+ hour** before reserved time
- Customer details and special requests are recorded
