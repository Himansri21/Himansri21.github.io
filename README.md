# Portfolio Website - Containerized Deployment

[![Docker](https://img.shields.io/badge/Docker-Enabled-2496ED?logo=docker)](https://www.docker.com/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-Ready-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?logo=github-actions)](https://github.com/features/actions)
[![Website](https://img.shields.io/badge/Live-Website-success)](http://34.93.198.75)

A containerized portfolio website deployed on Google Kubernetes Engine (GKE) with automated CI/CD pipeline using GitHub Actions.

## 📋 Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Local Development](#local-development)
- [Docker](#docker)
- [Kubernetes Deployment](#kubernetes-deployment)
- [CI/CD Pipeline](#cicd-pipeline)
- [Live Deployment](#live-deployment)

## 🎯 Overview

This is a personal portfolio website built with HTML/CSS/JavaScript, containerized using Docker, and deployed to a GKE cluster using GitOps principles with ArgoCD. The project demonstrates modern DevOps practices including:

- Containerization with Docker
- Kubernetes orchestration
- Automated CI/CD with GitHub Actions
- GitOps deployment with ArgoCD
- Infrastructure as Code with Terraform

## 🛠️ Tech Stack

### Frontend
- HTML5
- CSS3
- JavaScript
- Responsive Design

### DevOps & Infrastructure
- **Containerization**: Docker
- **Orchestration**: Kubernetes (GKE)
- **CI/CD**: GitHub Actions
- **GitOps**: ArgoCD
- **IaC**: Terraform
- **Cloud Provider**: Google Cloud Platform (GCP)
- **Container Registry**: Docker Hub / GCR

## 📁 Project Structure

```
.
├── .github/
│   └── workflows/
│       └─ main.yml      # CI/CD pipeline configuration
├── k8s/
│   ├── deployment.yaml           # Kubernetes Deployment manifest
│   ├── service.yaml              # Kubernetes Service manifest
│   └── ingress.yaml              # Kubernetes Ingress manifest (optional)
├── index.html                    # Main HTML file
├── website.html                  # Additional page
├── Dockerfile                    # Docker container configuration
└── README.md                     # This file
```

## 💻 Local Development

### Prerequisites

- Web browser (Chrome, Firefox, Safari, etc.)
- Text editor or IDE (VS Code recommended)

### Running Locally

1. **Clone the repository**:
   ```bash
   git clone https://github.com/Himansri21/Himansri21.github.io.git
   cd Himansri21.github.io
   ```

2. **Open in browser**:
   ```bash
   # Simply open index.html in your browser
   open index.html  # macOS
   xdg-open index.html  # Linux
   start index.html  # Windows
   ```

3. **Or use a local server** (recommended):
   ```bash
   # Python 3
   python3 -m http.server 8000
   
   # Python 2
   python -m SimpleHTTPServer 8000
   
   # Node.js (with http-server)
   npx http-server -p 8000
   ```

   Then visit: `http://localhost:8000`

## 🐳 Docker

### Building the Docker Image

The application is containerized using an optimized Nginx-based Docker image.

**Build locally**:
```bash
docker build -t portfolio-website:latest .
```

**Run locally**:
```bash
docker run -d -p 8080:80 portfolio-website:latest
```

Visit: `http://localhost:8080`

### Dockerfile Overview

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/
COPY website.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Optimizations**:
- ✅ Uses Alpine Linux for minimal image size
- ✅ Nginx for efficient static file serving
- ✅ Multi-stage build ready
- ✅ Non-root user configuration
- ✅ Health checks included

### Push to Container Registry

**Docker Hub**:
```bash
docker tag portfolio-website:latest himansri21/portfolio-website:latest
docker push himansri21/portfolio-website:latest
```

**Google Container Registry**:
```bash
docker tag portfolio-website:latest gcr.io/PROJECT_ID/portfolio-website:latest
docker push gcr.io/PROJECT_ID/portfolio-website:latest
```

## ☸️ Kubernetes Deployment

### Prerequisites

- kubectl configured to access your cluster
- Kubernetes cluster running (GKE, EKS, AKS, or local)

### Deploy to Kubernetes

**Manual deployment**:
```bash
# Create namespace
kubectl create namespace website

# Apply manifests
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Verify deployment
kubectl get pods -n website
kubectl get svc -n website
```

**Get external IP**:
```bash
kubectl get svc portfolio-service -n website -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

The project uses GitHub Actions for automated CI/CD.

**Workflow file**: `.github/workflows/docker-build.yml`

#### Pipeline Stages

1. **Trigger**: Push to `main` branch
2. **Build**: Build Docker image
3. **Push**: Push to container registry
4. **Deploy**: Update Kubernetes deployment

#### Workflow Configuration

```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  IMAGE_NAME: portfolio-website
  REGISTRY: docker.io/himansri21

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
    
  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Set up Cloud SDK
      uses: google-github-actions/setup-gcloud@v1
      with:
        service_account_key: ${{ secrets.GCP_SA_KEY }}
        project_id: ${{ secrets.GCP_PROJECT_ID }}
    
    - name: Configure kubectl
      run: |
        gcloud container clusters get-credentials ${{ secrets.GKE_CLUSTER_NAME }} \
          --zone ${{ secrets.GKE_ZONE }} \
          --project ${{ secrets.GCP_PROJECT_ID }}
    
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/portfolio-website \
          website=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }} \
          -n website
        kubectl rollout status deployment/portfolio-website -n website
```

### Required Secrets

Configure these secrets in GitHub repository settings:

| Secret Name | Description | Example |
|-------------|-------------|---------|
| `DOCKER_USERNAME` | Docker Hub username | `himansri21` |
| `DOCKER_PASSWORD` | Docker Hub password/token | `dckr_pat_xxxxx` |
| `GCP_SA_KEY` | GCP Service Account JSON key | `{...}` |
| `GCP_PROJECT_ID` | GCP Project ID | `my-project-123` |
| `GKE_CLUSTER_NAME` | GKE Cluster name | `my-gke-cluster` |
| `GKE_ZONE` | GKE Cluster zone | `asia-south1-a` |

### Setting Up Secrets

```bash
# Navigate to GitHub repo → Settings → Secrets and variables → Actions → New repository secret

# Or use GitHub CLI
gh secret set DOCKER_USERNAME
gh secret set DOCKER_PASSWORD
gh secret set GCP_SA_KEY < service-account-key.json
gh secret set GCP_PROJECT_ID
gh secret set GKE_CLUSTER_NAME
gh secret set GKE_ZONE
```

## 🌐 Live Deployment

### Current Deployment

- **Live URL**: http://34.93.198.75
- **Platform**: Google Kubernetes Engine (GKE)
- **Deployment Method**: GitOps with ArgoCD
- **Auto-scaling**: Enabled (1-3 replicas)
- **Health Monitoring**: Liveness & Readiness probes

### Accessing the Application

```bash
# Get service details
kubectl get svc portfolio-service -n website

# Get pods status
kubectl get pods -n website

# View logs
kubectl logs -f deployment/portfolio-website -n website
```

## 🔍 Monitoring & Debugging

### View Application Logs

```bash
# Real-time logs
kubectl logs -f -l app=portfolio -n website

# Logs from specific pod
kubectl logs <pod-name> -n website
```

### Check Deployment Status

```bash
# Deployment status
kubectl get deployment portfolio-website -n website

# Pod status
kubectl get pods -n website -o wide

# Events
kubectl get events -n website --sort-by='.lastTimestamp'
```

### Rollback Deployment

```bash
# View rollout history
kubectl rollout history deployment/portfolio-website -n website

# Rollback to previous version
kubectl rollout undo deployment/portfolio-website -n website

# Rollback to specific revision
kubectl rollout undo deployment/portfolio-website --to-revision=2 -n website
```

## 🚀 Development Workflow

### Making Changes

1. **Develop locally**:
   ```bash
   # Make changes to HTML/CSS/JS
   # Test locally in browser
   ```

2. **Commit and push**:
   ```bash
   git add .
   git commit -m "feat: update portfolio section"
   git push origin main
   ```

3. **Automated deployment**:
   - GitHub Actions triggers automatically
   - Builds new Docker image
   - Pushes to registry
   - Updates Kubernetes deployment
   - ArgoCD syncs changes

4. **Verify deployment**:
   ```bash
   # Check if new version is deployed
   kubectl get pods -n website
   kubectl describe pod <pod-name> -n website
   ```

## 🏗️ Infrastructure

The complete infrastructure is managed using Terraform and includes:

- **GKE Cluster** with auto-scaling node pools
- **VPC Network** with proper segmentation
- **ArgoCD** for GitOps deployment
- **Load Balancer** for external access
- **Monitoring & Logging** integration

Infrastructure repository: [GCP-cluster-terraform](https://github.com/Himansri21/GCP-cluster-terraform)

## 📊 Architecture

```
┌─────────────────────────────────────────────┐
│          GitHub Repository                  │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │  Source Code (HTML/CSS/JS)           │   │
│  │  Dockerfile                          │   │
│  │  Kubernetes Manifests (k8s/)         │   │
│  │  GitHub Actions (.github/workflows/) │   │
│  └──────────────────────────────────────┘   │
└──────────────┬──────────────────────────────┘
               │
               │ Push to main branch
               ▼
┌─────────────────────────────────────────────┐
│       GitHub Actions CI/CD Pipeline         │
│                                             │
│  1. Checkout code                           │
│  2. Build Docker image                      │
│  3. Push to Docker Hub                      │
│  4. Deploy to GKE using kubectl             │
└──────────────┬──────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────┐
│          Docker Hub Registry                │
│                                             │
│  Image: himansri21/portfolio-website        │
└──────────────┬──────────────────────────────┘
               │
               │ ArgoCD syncs
               ▼
┌─────────────────────────────────────────────┐
│       Google Kubernetes Engine (GKE)        │
│                                             │
│  ┌─────────────────────────────────────┐    │
│  │  Namespace: website                 │    │
│  │                                     │    │
│  │  ┌────────────────────────────────┐ │    │
│  │  │  Deployment: portfolio-website │ │    │ 
│  │  │  Replicas: 2                   │ │    │
│  │  │  Image: latest                 │ │    │
│  │  └────────────────────────────────┘ │    │
│  │                                     │    │
│  │  ┌────────────────────────────────┐ │    │
│  │  │  Service: LoadBalancer         │ │    │
│  │  │  External IP: 34.93.198.75     │ │    │
│  │  └────────────────────────────────┘ │    │
│  └─────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
               │
               ▼
         End Users (Web Browser)
```

## 🔐 Security Best Practices

- ✅ Non-root container user
- ✅ Resource limits configured
- ✅ Health checks implemented
- ✅ Secrets managed via GitHub Secrets
- ✅ Network policies enabled
- ✅ Image scanning in CI/CD

## 📝 License

This project is open source and available under the MIT License.

## 👤 Author

**Himanshu Srivastava**
- GitHub: [@Himansri21](https://github.com/Himansri21)
- LinkedIn: [Himanshu Srivastava](https://www.linkedin.com/in/himansri21/)
- Email: himansrivastava2003@gmail.com

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!

## ⭐ Show your support

Give a ⭐️ if you like this project!

---

**Built with ❤️ using modern DevOps practices**
