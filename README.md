# 🎮 Tetris — GCP CI/CD Deployment

A containerized HTML5 Tetris application deployed on **Google Cloud Platform (GCP)** using an automated **CI/CD pipeline**.

The project demonstrates how a code change pushed to GitHub can automatically trigger a pipeline that builds a Docker image, stores it in Artifact Registry, and deploys the updated application to Cloud Run.

---

## 🌐 Live Demo

**Play the deployed Tetris application:**

https://gaming-tetris-602589641033.us-central1.run.app

---

## 🏗️ Architecture

```text
                     Developer
                         │
                         │ git push
                         ▼
                  ┌──────────────┐
                  │    GitHub    │
                  │              │
                  │ main branch  │
                  └──────┬───────┘
                         │
                         │ Push event
                         ▼
              ┌──────────────────────┐
              │   Cloud Build        │
              │   Trigger            │
              │   gcp-tetris-cicd    │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │   Cloud Build        │
              │                      │
              │  1. Docker Build     │
              │  2. Push Image       │
              │  3. Deploy           │
              └──────────┬───────────┘
                         │
                         ▼
              ┌──────────────────────┐
              │  Artifact Registry   │
              │                      │
              │  gaming-repo/tetris │
              └──────────┬───────────┘
                         │
                         │ Container Image
                         ▼
                  ┌──────────────┐
                  │  Cloud Run   │
                  │              │
                  │ gaming-tetris│
                  └──────┬───────┘
                         │
                         ▼
                  🌐 Live Website
                    Tetris Game
```

---

## 🚀 Project Overview

This project takes a simple HTML5/JavaScript Tetris game and deploys it as a Docker container on Google Cloud.

The main objective is to demonstrate a practical DevOps workflow:

**Code → GitHub → Cloud Build → Docker → Artifact Registry → Cloud Run**

The deployment is automated. After a change is pushed to the `main` branch, the Cloud Build trigger automatically starts the CI/CD pipeline.

---

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| Git | Version control |
| GitHub | Source code repository |
| Docker | Application containerization |
| Nginx | Web server for the static application |
| Google Cloud Build | CI/CD pipeline execution |
| Cloud Build Triggers | Automatic pipeline execution |
| Artifact Registry | Docker image storage |
| Cloud Run | Container deployment |
| IAM | Identity and access management |
| Cloud Logging | Build log management |

---

## 🔄 CI/CD Workflow

### 1. Developer changes the application

Application code is modified and committed to the Git repository.

### 2. Push to GitHub

The changes are pushed to the `main` branch.

```bash
git add .
git commit -m "Update application"
git push origin main
```

### 3. Cloud Build Trigger

The **`gcp-tetris-cicd`** Cloud Build trigger monitors the GitHub repository.

When a push occurs on the `main` branch, the trigger automatically starts a Cloud Build.

### 4. Docker Image Build

Cloud Build reads the project's `Dockerfile` and creates a Docker container image.

```dockerfile
FROM nginx

COPY . /usr/share/nginx/html/
```

Nginx serves the HTML, JavaScript, audio, and other static application files.

### 5. Push to Artifact Registry

The newly built Docker image is pushed to Google Artifact Registry.

```text
us-central1-docker.pkg.dev/<PROJECT_ID>/gaming-repo/tetris:latest
```

Artifact Registry provides managed storage for the container image.

### 6. Deploy to Cloud Run

Cloud Build deploys the container image to the Cloud Run service:

```text
gaming-tetris
```

Cloud Run creates a new revision from the deployed container image and routes traffic to the new revision.

### 7. Application becomes available

The updated Tetris application becomes available through the Cloud Run URL.

---

## 🐳 Docker Configuration

The application is packaged using Nginx.

```dockerfile
FROM nginx

COPY . /usr/share/nginx/html/
```

The application files are copied into Nginx's default web directory:

```text
/usr/share/nginx/html/
```

Nginx listens on port `80`, so Cloud Run is configured to use container port `80`.

---

## ⚙️ Cloud Build Pipeline

The CI/CD pipeline is defined in:

```text
cloudbuild.yaml
```

The pipeline performs three main operations:

```text
1. Build Docker Image
          ↓
2. Push Image to Artifact Registry
          ↓
3. Deploy Image to Cloud Run
```

The pipeline also uses Cloud Logging for build logs.

### Pipeline Configuration

The Docker image is built and pushed using:

```text
us-central1-docker.pkg.dev/$PROJECT_ID/gaming-repo/tetris:latest
```

The Cloud Run service deployed by the pipeline is:

```text
gaming-tetris
```

---

## 🔐 IAM and Service Account

A dedicated Google Cloud service account is used by Cloud Build:

```text
gaming-cloud-build@<PROJECT_ID>.iam.gserviceaccount.com
```

The service account provides the permissions required by the CI/CD pipeline to:

- Push container images to Artifact Registry
- Deploy the application to Cloud Run
- Write build logs
- Perform deployment actions using the required service identity

This demonstrates the use of **IAM-based machine identities** for automated cloud deployments instead of relying on a personal user account.

---

## 📦 Project Structure

```text
gcp-tetris-cicd-deployment/
│
├── audio/
├── chart/
├── screenshots/
├── cloudbuild.yaml
├── Dockerfile
├── index.html
├── tetris.js
├── package.json
├── CHANGELOG.md
└── README.md
```

---

## 📸 Project Screenshots

### GitHub Repository

![GitHub Repository](screenshots/github-repository.png)

### Cloud Build — Successful Pipeline

![Cloud Build Success](screenshots/cloud-build-success.png)

### Artifact Registry

![Artifact Registry](screenshots/artifact-registry.png)

### Cloud Run

![Cloud Run](screenshots/cloud-run.png)

### Live Tetris Application

![Live Tetris Application](screenshots/tetris-live.png)

---

## 🧪 Automated Deployment Demonstration

The CI/CD pipeline was tested by modifying the application and pushing the change to the GitHub repository.

The change was committed to the `main` branch.

The GitHub push automatically triggered Cloud Build through the `gcp-tetris-cicd` trigger.

Cloud Build then executed the following workflow:

```text
GitHub Commit
      ↓
Cloud Build Trigger
      ↓
Docker Build
      ↓
Push to Artifact Registry
      ↓
Deploy to Cloud Run
      ↓
New Cloud Run Revision
      ↓
Updated Live Application
```

The final automated deployment test completed successfully without manually starting the Cloud Build or manually deploying the Cloud Run service.

---

## 🧩 Challenges & Solutions

### Cloud Run Container Port

The initial Cloud Run deployment failed because the Nginx container listens on port `80`, while Cloud Run was initially configured to expect port `8080`.

The issue was resolved by configuring Cloud Run to use:

```text
Port: 80
```

This matched the default Nginx configuration used by the container.

---

### Cloud Build Service Account

A dedicated service account was configured for the CI/CD pipeline.

The required IAM permissions were assigned so that Cloud Build could interact with:

- Artifact Registry
- Cloud Run
- Cloud Logging

This separates automated deployment permissions from the personal user account.

---

### Cloud Build Logging

Because a user-managed service account was used, Cloud Build required an explicit logging configuration.

The pipeline was configured with:

```yaml
options:
  logging: CLOUD_LOGGING_ONLY
```

This allowed Cloud Build to use Cloud Logging while using the dedicated service account.

---

## 📚 DevOps Concepts Demonstrated

This project demonstrates practical understanding of:

- Continuous Integration
- Continuous Deployment
- Git-based CI/CD
- Docker containerization
- Docker image management
- Artifact Registry
- Google Cloud Build
- Cloud Build Triggers
- Cloud Run
- IAM
- Service Accounts
- Cloud Logging
- Container ports
- Cloud Run revisions
- Automated deployments
- GitHub-based deployment workflows

---

## 🔮 Future Improvements

Possible improvements for a larger production-oriented implementation:

- Add automated application testing
- Add container vulnerability scanning
- Use immutable image tags instead of `latest`
- Add Terraform for Infrastructure as Code
- Add monitoring and alerting
- Add a staging environment
- Add deployment approval gates
- Implement rollback strategies
- Add observability with metrics and dashboards

---

## 🎯 Learning Outcome

This project demonstrates the complete lifecycle of a containerized application:

```text
Source Code
     ↓
Version Control
     ↓
CI/CD Automation
     ↓
Docker Container Build
     ↓
Container Registry
     ↓
Cloud Deployment
     ↓
Cloud Run Revision
     ↓
Live Application
```

The project provided hands-on experience with **Docker, GitHub, Google Cloud Build, Artifact Registry, Cloud Run, IAM, service accounts, and automated CI/CD deployment**.
