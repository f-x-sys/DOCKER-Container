# Docker, Kubernetes & CI/CD — MNIST Application

## Overview

This project explores containerization, orchestration, and CI/CD through a Python Flask application that trains an MNIST digit classifier and exposes predictions through a REST API.

The project progresses through five iterations, from running a single Docker container to automatically testing and publishing container images through GitHub Actions.

**Technologies:** Python, Flask, scikit-learn, pytest, Docker, Docker Compose, PostgreSQL, Kubernetes, GitHub Actions, GHCR.

## Project Structure

```text
DOCKER-Container/
├── .github/
│   └── workflows/
│       └── ci.yml
├── .dockerignore
├── .gitignore
├── Dockerfile
├── app.py
├── deployment.yaml
├── docker-compose.yml
├── requirements.txt
└── test_app.py
```

## Iteration 1 — Docker: Containerization

**Objective:** Package and execute a Python ML application inside an isolated container.

**Workflow:**
1. Build a Docker image containing the application and dependencies.
2. Start a container and publish port 5050.
3. Verify the Flask API through HTTP requests.
4. Submit image pixel values to the prediction endpoint.

```powershell
docker build -t mnist-app .
docker run -p 5050:5050 mnist-app
```

In a second terminal:

```powershell
curl.exe http://localhost:5050/health
curl.exe http://localhost:5050/sample/0
```

The `/predict` endpoint accepts a POST request with 64 pixel values representing an 8×8 digit image.

**Outcome:** The ML application runs in a reproducible containerized environment.

## Iteration 2 — Docker Compose: Multi-Container Application

**Objective:** Coordinate the Flask application and PostgreSQL database.

**Components:**
- `app`: Python Flask application.
- `db`: PostgreSQL database.
- `pgdata`: Persistent Docker volume.

```powershell
docker compose up --build -d
docker compose ps
```

Test the database-backed endpoint:

```powershell
curl.exe http://localhost:5050/visits
curl.exe http://localhost:5050/visits
```

Inspect stored records:

```powershell
docker compose exec db psql -U postgres -c "SELECT * FROM visits;"
```

**Persistence experiment:**

```powershell
docker compose down
docker compose up -d
```

The visit count survives container recreation because the database uses persistent storage.

Running `docker compose down -v` also removes the Compose-managed volume and its stored data.

**Outcome:** Demonstrates service coordination, networking, and persistent database storage.

## Iteration 3 — Kubernetes: Container Orchestration

**Objective:** Deploy multiple application replicas and observe Kubernetes orchestration.

Deploy:

```powershell
kubectl apply -f deployment.yaml
kubectl get pods
```

### Load Balancing

```powershell
1..6 | ForEach-Object {
    curl.exe -s http://localhost:30080/health
}
```

Observe whether requests are served by different Pods.

### Self-Healing

```powershell
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods -w
```

Kubernetes creates a replacement Pod to maintain the desired replica count.

### Scaling

```powershell
kubectl scale deployment mnist-app --replicas=6
kubectl get pods

kubectl scale deployment mnist-app --replicas=1
kubectl get pods
```

**Outcome:** Demonstrates replication, service load distribution, self-healing, and horizontal scaling within a local Kubernetes cluster.

## Iteration 4 — GitHub Actions: Continuous Integration

**Objective:** Automatically validate application changes after pushing code to GitHub.

The workflow is defined in `.github/workflows/ci.yml`.

**CI pipeline:**

1. Checkout repository code.
2. Install Python dependencies.
3. Execute automated tests with pytest.
4. Build the Docker image.
5. Verify container startup and HTTP responses.

Run tests locally:

```powershell
python -m pip install -r requirements.txt
python -m pip install pytest
python -m pytest -v
```

Push changes:

```powershell
git add .
git commit -m "Update application"
git push
```

GitHub Actions automatically executes the configured jobs.

**Outcome:** Application changes are tested and validated in an automated environment, reducing environment-specific failures.

## Iteration 5 — GHCR: Continuous Delivery

**Objective:** Automatically publish a versioned Docker image after successful CI checks.

The `publish` job in `ci.yml`:

1. Runs after the required validation jobs succeed.
2. Authenticates with GitHub Container Registry.
3. Builds and publishes the container image.
4. Tags the image with the commit SHA and `latest`.

**Published image:**

```text
ghcr.io/f-x-sys/docker-container:latest
```

Run the published image:

```powershell
docker pull ghcr.io/f-x-sys/docker-container:latest
docker run --rm -p 5050:5050 ghcr.io/f-x-sys/docker-container:latest
```

Verify:

```powershell
curl.exe http://localhost:5050/health
```

The image must be accessible to the user performing the pull. Private packages require appropriate authentication.

**Outcome:** A validated, versioned application image can be distributed and executed without rebuilding from source.

This implements continuous delivery of a container artifact, not automatic deployment to a production environment.

## Key Concepts

| Technology | Responsibility |
|---|---|
| Docker | Containerization |
| Docker Compose | Multi-container coordination |
| PostgreSQL volumes | Persistent storage |
| Kubernetes | Orchestration and scaling |
| pytest | Automated application testing |
| GitHub Actions | Continuous Integration |
| GHCR | Container image distribution |
| CI/CD | Automated validation and delivery |

## Cleanup

Stop the Docker Compose stack:

```powershell
docker compose down
```

Remove the Kubernetes deployment and service resources defined in the manifest:

```powershell
kubectl delete -f deployment.yaml
```

To remove Compose volumes as well, use `docker compose down -v` only when stored database data is no longer needed.

## Lessons Learned

- Docker packages an application and its dependencies into a portable image.
- Containers are disposable; persistent data requires separate storage.
- Docker Compose simplifies coordination between services.
- Kubernetes manages application replicas and replaces failed Pods.
- GitHub Actions automates testing and build verification.
- GHCR distributes versioned container images.
- CI/CD principles are applicable to ML applications and MLOps workflows.

## Environment

- **OS:** Windows 11
- **Shell:** PowerShell
- **Container platform:** Docker Desktop
- **Orchestration:** Local Kubernetes cluster (kind)
- **Application:** Python Flask / MNIST classification

This repository is an educational infrastructure project intended to demonstrate the progression from local ML development to automated container delivery.
