---
title: "Source Code Preparation"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---
# Source Code Preparation

## Objective

Prepare the application repository for repeatable validation and deployment without exposing secrets or modifying production resources accidentally.

## Architecture context

The repository is a multi-service application. CI/CD builds separate backend, chat, and optional AI images, deploys Kubernetes manifests, and builds the frontend separately for S3 and CloudFront.

## Repository structure

| Path | Purpose |
|---|---|
| `backend/` | FastAPI application, SQLAlchemy models, Alembic migrations, worker code, tests |
| `frontend/` | React/Vite application, API clients, route components, frontend tests |
| `chat-service/` | Node.js, Express, Socket.IO, DynamoDB repositories, Redis adapter |
| `ai_service/` | FastAPI SageMaker adapter and AI parsing/scoring code |
| `k8s/app/` | Base Kubernetes namespace, services, deployments, jobs, HPA, PDB |
| `k8s/eks/` | EKS-specific ConfigMap, ServiceAccount, Ingress, secret example |
| `k8s/observability/` | ServiceMonitor and PrometheusRule manifests |
| `scripts/k8s/` | Local and EKS deployment scripts |
| `scripts/ci/` | GitHub Actions helper scripts |
| `scripts/aws/` | AWS resource helper scripts for CloudFront, ALB, SQS, rollout, node group |
| `.github/workflows/cicd.yml` | Validation and production deployment workflow |
| `docs/architecture/` | ADRs, concurrency policy, event consumer contract |
| `Dockerfile.ai-service` | AI adapter image build file |
| `docker-compose.yml` | Local dependency stack |

## Obtain authorized project source

Use authorized repository access only:

```bash
git clone https://github.com/Temp-orgo/AWS-Internship.git
cd AWS-Internship
git status --short --branch
```

I do not publish private tokens, clone credentials, or secret-bearing remote URLs in the documentation.

## Review branch and workflow state

```bash
git remote -v
git status --short --branch
git log --oneline -5
```

Expected result:

- The remote points to the authorized project repository.
- The working tree is understood before any deployment action.
- No unrelated local edits are overwritten.

## Identify service dependencies

### Backend

```bash
cd backend
python -m pip install -r requirements.txt
python -m ruff format --check .
python -m ruff check .
python -m mypy app tests
python -m pytest -m "not postgres"
```

The backend uses Python 3.12 in CI, FastAPI, SQLAlchemy, Alembic, Pydantic settings, boto3, Prometheus client, and OpenTelemetry libraries.

### Chat service

```bash
cd chat-service
npm ci
npm run check
npm run lint
npm run format:check
npm run test:ci
```

The chat service uses Node.js, Express, Socket.IO, DynamoDB, Redis adapter, OpenTelemetry, and Prometheus metrics.

### Frontend

```bash
cd frontend
npm ci
npm run test
npm run build
```

The frontend uses React 18, Vite, Tailwind CSS, Socket.IO client, Axios, and route-level API clients.

### AI service

```bash
docker build -f Dockerfile.ai-service -t internship-ai:ci .
```

The AI adapter exposes `/health/ready`, `/parse-job`, `/parse-cv`, `/match-applications`, `/rerank`, `/cv-job/group-score`, and related scoring routes. In production, it calls SageMaker Runtime instead of loading a local model inside the worker.

## Environment templates

Use example files as references only:

| File | Purpose |
|---|---|
| `k8s/app/secret.local.example.yaml` | Local Kubernetes secret example |
| `k8s/app/secret.local.yaml.example` | Local secret template |
| `k8s/eks/secret.example.yaml` | EKS secret key shape without real values |
| `backend/requirements.txt` | Backend runtime dependencies |
| `frontend/package.json` | Frontend scripts and dependencies |
| `chat-service/package.json` | Chat scripts and dependencies |

Do not commit real `.env`, `.env.production`, kubeconfig, database URL, JWT secret, AWS key, GitHub token, private key, or SES credential.

## Local validation commands

The CI workflow performs these major checks:

```bash
bash -n $(git ls-files '*.sh')
python scripts/ci/infrastructure.py
python -m ruff format --check backend
python -m ruff check backend
python -m mypy backend/app backend/tests
python -m pytest -m "not postgres" backend
npm run test --prefix frontend
npm run build --prefix frontend
npm run test:ci --prefix chat-service
docker build -t internship-backend:ci ./backend
docker build -t internship-chat:ci ./chat-service
docker build -f Dockerfile.ai-service -t internship-ai:ci .
git diff --check
```

If Docker is unavailable locally, report Docker build validation as not run instead of claiming success.

## Deployment preparation

Before running production workflows:

1. Confirm GitHub production variables and secrets are configured.
2. Confirm OIDC role `internship-github-deploy` can assume the intended AWS account.
3. Confirm ECR repositories exist for backend, chat, and AI if AI deployment is enabled.
4. Confirm RDS, Redis, DynamoDB, SQS, S3, CloudFront, ALB, Lambda, SES, and SageMaker readiness.
5. Confirm `PROCESSING_WORKER_ENABLED` is false unless AI dependencies are ready.
6. Use `deploy-app` for EKS application deployment and `deploy-frontend` for frontend deployment.

## GitHub Actions workflow modes

The inspected workflow supports:

| Mode | Purpose |
|---|---|
| `validate` | Run validation without production deployment |
| `deploy` | Legacy application deploy path still accepted by the workflow |
| `rollout` | Restart existing EKS workloads without building images |
| `restore-compute` | Ensure EKS managed node group capacity |
| `deploy-app` | Build/push images and deploy EKS workloads |
| `deploy-public` | Apply public ALB Ingress |
| `deploy-frontend` | Build frontend, sync to S3, invalidate CloudFront |
| `full` | Run the full production sequence |

## Expected result

The source tree is ready when:

- service dependencies install successfully
- backend, frontend, and chat tests pass in the target validation environment
- Docker images can be built
- Kubernetes manifests pass repository infrastructure checks
- required GitHub environment variables and secrets exist
- deployment is split correctly between EKS app workloads and S3/CloudFront frontend

## Common errors

| Symptom | Cause | Resolution |
|---|---|---|
| `Cannot find build-and-push-image.sh` | Helper script path not found | Use repository `scripts/build-and-push-image.sh` or set `BUILD_IMAGE_SCRIPT` |
| Node header download failure during chat CI | Native dependency build needs local headers | CI sets `npm_config_nodedir` to verified toolcache headers |
| Frontend build points at localhost | Missing `VITE_API_BASE_URL=/api` for production | Set Vite variables during production build |
| Docker build unavailable | Docker daemon not running | Start Docker or report build validation as not run |
| Secrets appear in diff | `.env` or generated secret file accidentally staged | Remove secret-bearing files from the commit and rotate exposed values |

## Outcome

The repository is prepared for validated deployment. Static frontend build and Kubernetes workloads are treated as separate delivery paths, while GitHub Actions and OIDC provide the controlled production deployment surface.
