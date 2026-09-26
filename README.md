# Movie Picture Pipeline - CI/CD with GitHub Actions

This repository contains the end-to-end CI/CD automation pipeline for the Movie Picture web application (React frontend and Flask Python backend) deploying to AWS EKS using GitHub Actions, Docker, and Kustomize.

---

## 1. Project Architecture

The architecture consists of two microservices:
- **Frontend**: A React application built with TypeScript and Node.js (`starter/frontend`).
- **Backend API**: A Python Flask application serving movie catalog data via `/movies` (`starter/backend`).

Both services are containerized using Docker, pushed to Amazon Elastic Container Registry (ECR), and deployed to Amazon Elastic Kubernetes Service (EKS).

---

## 2. CI/CD Workflows

All workflows are located under `.github/workflows/`:

| Workflow File | Workflow Name | Trigger Event | Description |
| :--- | :--- | :--- | :--- |
| `frontend-ci.yaml` | `Frontend Continuous Integration` | `pull_request` on `starter/frontend/**`, `workflow_dispatch` | Runs linting and tests in parallel, then builds the Docker image. |
| `backend-ci.yaml` | `Backend Continuous Integration` | `pull_request` on `starter/backend/**`, `workflow_dispatch` | Runs flake8 linting and pytest in parallel, then builds the Docker image. |
| `frontend-cd.yaml` | `Frontend Continuous Deployment` | `push` (merge) to `main` on `starter/frontend/**`, `workflow_dispatch` | Runs lint/test, builds Docker image with `REACT_APP_MOVIE_API_URL`, tags with commit SHA, pushes to ECR, and deploys via Kustomize to EKS. |
| `backend-cd.yaml` | `Backend Continuous Deployment` | `push` (merge) to `main` on `starter/backend/**`, `workflow_dispatch` | Runs lint/test, builds Docker image, tags with commit SHA, pushes to ECR, and deploys via Kustomize to EKS. |

### Pipeline Highlights
- **Parallel Execution**: Linting and testing run concurrently to minimize pipeline execution time.
- **Dependency Control**: Build and deployment stages utilize `needs: [lint, test]` to prevent invalid code from being built or deployed.
- **Secure Secret Handling**: AWS credentials are injected securely through GitHub Secrets.
- **Immutable Tagging**: Images are tagged with `${{ github.sha }}` for version traceability.

---

## 3. Required GitHub Secrets

To allow GitHub Actions to interact with your AWS environment, configure the following secrets in **Settings > Secrets and variables > Actions**:

- `AWS_ACCESS_KEY_ID`: Access key for `github-action-user`
- `AWS_SECRET_ACCESS_KEY`: Secret key for `github-action-user`
- `AWS_REGION`: AWS Region (e.g. `us-east-1`)
- `ECR_FRONTEND_REPOSITORY`: Frontend ECR repository name or URL
- `ECR_BACKEND_REPOSITORY`: Backend ECR repository name or URL
- `EKS_CLUSTER_NAME`: Name of the EKS cluster (default: `cluster`)
- `REACT_APP_MOVIE_API_URL`: Backend service URL (LoadBalancer external IP or DNS)

---

## 4. Local Development and Verification

### Frontend
```bash
cd starter/frontend
npm ci
npm run lint
CI=true npm test
```

### Backend
```bash
cd starter/backend
pipenv install --dev
pipenv run lint
pipenv run test
```

---

## 5. Teardown Instructions

To avoid incurring cloud charges, tear down all provisioned resources after project verification:
```bash
cd setup/terraform
terraform destroy
```
