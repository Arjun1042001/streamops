# StreamOps — Secure Cloud-Native Streaming Platform

Secure CI/CD & Kubernetes deployment — GitHub Actions, Docker, Semgrep/Trivy/OWASP ZAP, Helm

![Docker Pipeline](https://github.com/Arjun1042001/streamops/actions/workflows/pipeline.yml/badge.svg)
![Security Pipeline](https://github.com/Arjun1042001/streamops/actions/workflows/security.yml/badge.svg)
![GHCR Pipeline](https://github.com/Arjun1042001/streamops/actions/workflows/publish-ghcr.yml/badge.svg)

## What this project does

StreamOps is a containerized application deployment project designed to demonstrate a practical CI/CD and Kubernetes workflow.

The project stores application configuration, Kubernetes manifests, and Helm templates in a GitHub repository. It provides a structured way to build, configure, and deploy the StreamOps application to a Kubernetes cluster.

The StreamOps web application has a home page and a login page (`index.html` and `login.html`), with ConfigMap-based application settings, a Kubernetes deployment and Service, and a Helm chart for reusable deployments.

## Architecture / How it works

```text
Developer Commit
       |
       v
GitHub / GitLab
       |
       v
CI/CD Pipeline
       |
       +--> Application Testing
       +--> Semgrep SAST
       +--> Trivy Filesystem Scan
       +--> Docker Image Build
       +--> Trivy Image Scan
       +--> OWASP ZAP DAST
       +--> Security Reports
       |
       v
GitHub Container Registry
       |
       v
Kubernetes / Minikube
       |
       +--> Two Application Replicas
       +--> Kubernetes Service
       +--> ConfigMap
       +--> Readiness Probe
       +--> Liveness Probe
       +--> Resource Requests and Limits
       +--> Rolling Deployment
```

The delivery workflow is split across three GitHub Actions workflows in `.github/workflows/`:

- `pipeline.yml` — build and test the application.
- `security.yml` — security scanning and reporting.
- `publish-ghcr.yml` — publish the validated image to GitHub Container Registry.

An equivalent GitLab CI/CD pipeline is provided in `.gitlab-ci.yml`.

Deployment uses the manifests in `k8s/` and the Helm chart in `helm/my-first-pipeline/`:

- `k8s/app.yaml` defines the `my-first-pipeline` Deployment and Service. The Deployment runs 2 replicas of `nginx:alpine` with a RollingUpdate strategy, on container port 80, with readiness and liveness probes, CPU and memory requests of 50m / 32Mi and limits of 200m / 128Mi. It mounts the `website-html` ConfigMap read-only at `/usr/share/nginx/html`. The Service is a NodePort Service.
- `k8s/configmap.yaml` provides the ConfigMap-based application settings.
- `helm/my-first-pipeline/` (`Chart.yaml`, `values.yaml`, `templates/`) packages the same deployment for reusable, environment-specific installs with `helm install` / `helm upgrade`.

## Tech stack

![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-%232088FF.svg?style=for-the-badge&logo=githubactions&logoColor=white)
![GitLab CI/CD](https://img.shields.io/badge/GitLab_CI%2FCD-%23FC6D26.svg?style=for-the-badge&logo=gitlab&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-%232496ED.svg?style=for-the-badge&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-%23326CE5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-%230F1689.svg?style=for-the-badge&logo=helm&logoColor=white)
![Semgrep](https://img.shields.io/badge/Semgrep-1E1E1E.svg?style=for-the-badge)
![Trivy](https://img.shields.io/badge/Trivy-1904DA.svg?style=for-the-badge)
![OWASP ZAP](https://img.shields.io/badge/OWASP_ZAP-00549E.svg?style=for-the-badge)

- Git and GitHub
- GitLab
- GitHub Actions
- GitLab CI/CD
- Docker and Nginx
- GitHub Container Registry
- Kubernetes and Minikube
- Semgrep
- Trivy
- OWASP ZAP
- PowerShell

## Repository structure

```text
.
├── .github/workflows/
│   ├── pipeline.yml
│   ├── security.yml
│   └── publish-ghcr.yml
├── .gitlab-ci.yml
├── Dockerfile
├── index.html
├── login.html
├── k8s/
│   ├── app.yaml
│   └── configmap.yaml
├── helm/my-first-pipeline/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── .helmignore
│   └── templates/
├── docs/images/
│   ├── project-preview.png
│   └── streamops-login.png
├── streamops-home.png.png
├── streamops-login.png
└── README.md
```

## Key work demonstrated

- Implemented equivalent DevSecOps workflows in GitHub Actions (3 workflows) and GitLab CI/CD.
- Automated build, application validation, Semgrep SAST, Trivy filesystem and image scanning, and OWASP ZAP DAST, with security-report and artifact upload.
- Published immutable Docker images to GitHub Container Registry using commit-SHA tags, with SBOM and provenance metadata.
- Deployed the application to Kubernetes with two Nginx replicas, a rolling-update strategy, ConfigMap settings, and readiness and liveness probes, plus CPU and memory requests and limits.
- Resolved CI configuration, reserved-stage, Docker networking, Kubernetes context, and local-cluster issues.

## DevSecOps Pipeline

The automated delivery workflow performs:

1. Source-code checkout
2. Application build and endpoint testing
3. Semgrep static application security testing
4. Trivy filesystem and configuration scanning
5. Docker image creation
6. Trivy container-image vulnerability scanning
7. OWASP ZAP dynamic application security testing
8. Security-report and artifact upload
9. Docker image publishing to GHCR
10. Immutable image tagging using the Git commit SHA

## Security Controls

| Area | Implementation |
|---|---|
| SAST | Semgrep |
| Filesystem scanning | Trivy |
| Container scanning | Trivy |
| DAST | OWASP ZAP |
| Image registry | GitHub Container Registry |
| Image traceability | Commit-SHA tags |
| Supply-chain metadata | SBOM and provenance |
| Runtime health | Liveness and readiness probes |
| Resource governance | Kubernetes requests and limits |

## Run Locally with Docker

```bash
docker build -t my-first-pipeline:local .
docker run -d --name cicd-web -p 8081:80 my-first-pipeline:local
```

Open:

```text
http://localhost:8081
```

## Deploy to Kubernetes

StreamOps can be deployed using standard Kubernetes manifests:

```bash
kubectl apply -f k8s/
```

Or with the detailed commands from this repo:

```bash
minikube start --driver=docker
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/app.yaml
kubectl rollout status deployment/my-first-pipeline
kubectl get pods,services
minikube service my-first-pipeline
```

It can also be deployed using Helm:

```bash
helm install streamops ./helm/my-first-pipeline
```

To update an existing Helm deployment:

```bash
helm upgrade streamops ./helm/my-first-pipeline
```

## Project Preview

![DevSecOps Project Preview](docs/images/project-preview.png)

### StreamOps Home Page

![StreamOps Home Page](streamops-home.png.png)

### StreamOps Login Page

![StreamOps Login Page](docs/images/streamops-login.png)

## Purpose

The purpose of StreamOps is to demonstrate:

- application containerization
- Kubernetes resource management
- Helm-based packaging
- configuration management
- Git-based development workflows
- repeatable application deployment

## Interview Summary

I designed and implemented a production-style DevSecOps delivery workflow for a containerized Nginx application. GitHub Actions and GitLab CI/CD automate application testing, Semgrep SAST, Trivy filesystem and image scanning, and OWASP ZAP DAST. The validated image is published to GitHub Container Registry using immutable commit-SHA tags with SBOM and provenance metadata. I deployed the application to Kubernetes with two replicas, rolling updates, ConfigMaps, resource controls, and liveness and readiness probes.

## Scope

The scope of the StreamOps project is to design, configure, and deploy a containerized web application using Kubernetes and Helm.

The project focuses on creating a repeatable deployment process that can be managed through source control and executed in a local or cloud-based Kubernetes environment.
