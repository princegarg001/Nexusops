# 🚀 Enterprise Multi-Cloud GitOps Platform with DevSecOps
## Part 1: Infrastructure, Kubernetes Clusters & GitOps Foundation

> ## 💰 AWS FREE TIER + LOCAL DEVELOPMENT VERSION
> **This guide uses AWS Free Tier + local tools to minimize costs!**
> 
> | Component | Where It Runs | Cost |
> |-----------|---------------|------|
> | Kubernetes Cluster | Local (Minikube) | 💚 FREE |
> | Container Registry | AWS ECR | 💚 FREE (500MB) |
> | Object Storage | AWS S3 | 💚 FREE (5GB) |
> | CI/CD | GitHub Actions | 💚 FREE (2000 mins/mo) |
> | All Platform Tools | Local cluster | 💚 FREE |

---

## 📋 Table of Contents - Part 1

1. [Project Overview](#project-overview)
2. [Architecture Design](#architecture-design)
3. [Prerequisites & Tools](#prerequisites--tools)
4. [Phase 1: Local Development Environment](#phase-1-local-development-environment)
5. [Phase 1.5: AWS Free Tier Setup](#phase-15-aws-free-tier-setup) ✅ *ECR + S3*
6. [Phase 2: Local Kubernetes Setup](#phase-2-local-local-kubernetes-setup-free) ✅ *Minikube*
7. [Phase 4: GitOps with Argo CD](#phase-4-gitops-with-argo-cd)
8. [Repository Structure](#repository-structure)

---

## 🎯 Project Overview

### What We're Building

An enterprise-grade, production-ready platform that combines:

| Component | Technology | Where It Runs | Cost |
|-----------|------------|---------------|------|
| **Kubernetes** | Minikube | Local machine | 💚 FREE |
| **Container Registry** | AWS ECR | AWS Free Tier | 💚 FREE (500MB) |
| **Object Storage** | AWS S3 | AWS Free Tier | 💚 FREE (5GB) |
| **GitOps** | Argo CD | Local cluster | 💚 FREE |
| **Service Mesh** | Istio | Local cluster | 💚 FREE |
| **DevSecOps** | Trivy, OPA | Local cluster | 💚 FREE |
| **Observability** | Prometheus, Grafana | Local cluster | 💚 FREE |
| **CI/CD** | GitHub Actions | GitHub | 💚 FREE (2000 mins) |

> 🎓 **Learning Note**: You get real AWS experience while keeping costs at $0!
>
> ⚠️ **Important**: EKS (managed Kubernetes) costs ~$73/month. We use local Minikube instead!

### Project Timeline (AWS Free Tier + Local)

```
Week 1: Local Kubernetes + AWS Account Setup - FREE
        └─ Minikube cluster + AWS CLI + ECR + S3

Week 2: GitOps with Argo CD - FREE
        └─ Argo CD on local cluster, images from ECR

Week 3: Service Mesh (Istio) - FREE
        └─ Full Istio setup on local cluster

Week 4: Security Tools (Trivy, OPA) - FREE
        └─ Scan images, enforce policies

Week 5: Observability Stack - FREE
        └─ Prometheus, Grafana, Jaeger

Week 6: CI/CD Pipeline + Testing - FREE
        └─ GitHub Actions → ECR → Argo CD
```

### AWS Free Tier Limits (12 months)

| Service | Free Tier Limit | Our Usage |
|---------|-----------------|------------|
| EC2 | 750 hrs t2.micro/month | Not needed (local K8s) |
| ECR | 500 MB storage | ✅ Container images |
| S3 | 5 GB storage | ✅ Terraform state, backups |
| Lambda | 1M requests/month | Optional |
| CloudWatch | 10 metrics | ✅ Basic monitoring |

---

## 🏗️ Architecture Design

### High-Level Architecture (AWS Free Tier + Local Kubernetes)

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                    DEVELOPER WORKFLOW                                    │
└─────────────────────────────────────────────┬───────────────────────────────────────────┘
                                              │
                    ┌─────────────────────────▼─────────────────────────┐
                    │              GitHub Repository                     │
                    │         (Code + K8s Manifests + IaC)              │
                    │                  💚 FREE                           │
                    └─────────────────────────┬─────────────────────────┘
                                              │
                    ┌─────────────────────────▼─────────────────────────┐
                    │            GitHub Actions (CI/CD)                  │
                    │      Build → Test → Scan → Push to ECR            │
                    │           💚 FREE (2000 mins/month)                │
                    └─────────────────────────┬─────────────────────────┘
                                              │
        ┌─────────────────────────────────────┴─────────────────────────────────────┐
        │                                                                           │
        ▼                                                                           ▼
┌───────────────────────────────┐                           ┌───────────────────────────────┐
│      ☁️ AWS FREE TIER         │                           │   🖥️ LOCAL KUBERNETES         │
│                               │                           │      (Minikube/Kind)          │
│  ┌─────────────────────────┐  │                           │                               │
│  │    ECR (Container       │  │    docker pull            │  ┌─────────────────────────┐  │
│  │    Registry)            │──┼───────────────────────────┼─▶│    Applications         │  │
│  │    💚 FREE 500MB        │  │                           │  │    (Your Apps Run Here) │  │
│  └─────────────────────────┘  │                           │  └─────────────────────────┘  │
│                               │                           │                               │
│  ┌─────────────────────────┐  │                           │  ┌─────────────────────────┐  │
│  │    S3 (Terraform        │  │                           │  │    Argo CD              │  │
│  │    State + Backups)     │  │                           │  │    (GitOps Deployments) │  │
│  │    💚 FREE 5GB          │  │                           │  └─────────────────────────┘  │
│  └─────────────────────────┘  │                           │                               │
│                               │                           │  ┌─────────────────────────┐  │
│  ┌─────────────────────────┐  │                           │  │    Istio                │  │
│  │    IAM (Access          │  │                           │  │    (Service Mesh)       │  │
│  │    Management)          │  │                           │  └─────────────────────────┘  │
│  │    💚 FREE              │  │                           │                               │
│  └─────────────────────────┘  │                           │  ┌─────────────────────────┐  │
│                               │                           │  │    Prometheus/Grafana   │  │
└───────────────────────────────┘                           │  │    (Monitoring)         │  │
                                                            │  └─────────────────────────┘  │
                                                            │                               │
                                                            │  💚 ALL FREE (runs locally)   │
                                                            └───────────────────────────────┘

💡 This setup gives you REAL AWS experience while keeping costs at $0!
```

### How It All Connects

```
1. You push code to GitHub
         ↓
2. GitHub Actions builds Docker image
         ↓
3. Image pushed to AWS ECR (free tier)
         ↓
4. Argo CD (on local cluster) detects change
         ↓
5. Argo CD pulls image from ECR
         ↓
6. App deployed to local Kubernetes
         ↓
7. Istio manages traffic, Prometheus monitors
```
                    └──────────────────────────────────┼──────────────────────────────────┘
                                                       │
                                    ┌──────────────────▼──────────────────┐
                                    │        Sample Applications         │
                                    │    (Frontend + Backend + API)      │
                                    │            💚 FREE                 │
                                    └─────────────────────────────────────┘

💡 UPGRADE PATH: When ready, deploy same configs to AWS/Azure/GCP using free credits!
```

### Network Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Global Load Balancer                               │
│                        (Route53 / Azure DNS / Cloud DNS)                    │
└─────────────────────────────────────┬───────────────────────────────────────┘
                                      │
         ┌────────────────────────────┼────────────────────────────┐
         │                            │                            │
┌────────▼────────┐          ┌────────▼────────┐          ┌────────▼────────┐
│   AWS Region    │          │  Azure Region   │          │   GCP Region    │
│   us-east-1     │          │   eastus        │          │  us-central1    │
├─────────────────┤          ├─────────────────┤          ├─────────────────┤
│ VPC: 10.0.0.0/16│          │VNet:10.1.0.0/16 │          │ VPC:10.2.0.0/16 │
├─────────────────┤          ├─────────────────┤          ├─────────────────┤
│ Public Subnet   │          │ Public Subnet   │          │ Public Subnet   │
│ 10.0.1.0/24     │          │ 10.1.1.0/24     │          │ 10.2.1.0/24     │
├─────────────────┤          ├─────────────────┤          ├─────────────────┤
│ Private Subnet  │          │ Private Subnet  │          │ Private Subnet  │
│ 10.0.2.0/24     │          │ 10.1.2.0/24     │          │ 10.2.2.0/24     │
└─────────────────┘          └─────────────────┘          └─────────────────┘
```

---

## 🛠️ Prerequisites & Tools

### Required Accounts

| Provider | What You Need | Cost | Required? |
|----------|---------------|------|---------------|
| **GitHub** | GitHub account | 💚 FREE | ✅ Yes |
| **Docker Hub** | Docker Hub account | 💚 FREE | ✅ Yes |
| **AWS** | AWS Account (Free Tier) | 💚 FREE (12 months) | ✅ Yes |

> 🎯 **Zero-Cost Setup**: GitHub + Docker Hub + AWS Free Tier = Complete learning environment!

### Local Tools Installation

#### Windows (PowerShell as Admin)

```powershell
# Install Chocolatey (Package Manager)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocolType::Tls12
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install Required Tools
choco install -y docker-desktop
choco install -y kubernetes-cli
choco install -y kubernetes-helm
choco install -y terraform
choco install -y awscli
choco install -y minikube
choco install -y argocd-cli
choco install -y istioctl
choco install -y k9s
choco install -y git
choco install -y vscode

# Verify Installations
kubectl version --client
helm version
terraform version
aws --version
minikube version
argocd version --client
istioctl version --remote=false
```

#### macOS (Terminal)

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Required Tools
brew install kubectl helm terraform awscli minikube
brew install --cask docker
brew install argocd istioctl k9s

# Verify Installations
kubectl version --client
helm version
terraform version
minikube version
```

#### Linux (Ubuntu/Debian)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER

# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install Terraform
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Install Cloud CLIs
# AWS
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip && sudo ./aws/install

# Azure
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# GCP
curl https://sdk.cloud.google.com | bash

# Install Argo CD CLI
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd

# Install Istioctl
curl -L https://istio.io/downloadIstio | sh -
sudo mv istio-*/bin/istioctl /usr/local/bin/
```

---

## 📁 Repository Structure

### Complete Project Structure

```
multi-cloud-gitops-platform/
│
├── 📁 infrastructure/                    # Terraform IaC (AWS Free Tier)
│   └── 📁 aws/
│       ├── 📁 ecr/                          # Container Registry (FREE)
│       │   └── main.tf
│       ├── 📁 s3/                           # Terraform State (FREE)
│       │   └── main.tf
│       └── 📁 iam/                          # CI/CD Permissions (FREE)
│           └── main.tf
│
├── 📁 kubernetes/                        # K8s Manifests
│   ├── 📁 base/                         # Base configurations
│   │   ├── 📁 namespaces/
│   │   │   └── namespaces.yaml
│   │   ├── 📁 rbac/
│   │   │   ├── cluster-roles.yaml
│   │   │   └── service-accounts.yaml
│   │   └── 📁 network-policies/
│   │       └── default-deny.yaml
│   │
│   └── 📁 apps/                         # Application manifests
│       ├── 📁 demo-app/
│       ├── 📁 frontend/
│       └── 📁 backend/
│
├── 📁 gitops/                           # Argo CD configurations
│   ├── 📁 argocd/
│   │   ├── install.yaml
│   │   ├── values.yaml
│   │   └── 📁 projects/
│   │       ├── infrastructure.yaml
│   │       └── applications.yaml
│   │
│   └── 📁 applications/
│       └── app-of-apps.yaml
│
├── 📁 service-mesh/                     # Istio configurations
│   ├── 📁 istio-install/
│   │   └── istio-profile.yaml
│   │
│   ├── 📁 gateway/
│   │   ├── gateway.yaml
│   │   └── virtual-services.yaml
│   │
│   ├── 📁 traffic-management/
│   │   └── destination-rules.yaml
│   │
│   └── 📁 security/
│       └── peer-authentication.yaml
│
├── 📁 security/                         # DevSecOps
│   ├── 📁 trivy/
│   │   └── scan-policies.yaml
│   │
│   └── 📁 opa-gatekeeper/
│       └── 📁 constraints/
│           ├── require-labels.yaml
│           └── container-limits.yaml
│
├── 📁 observability/                    # Monitoring Stack
│   ├── 📁 prometheus/
│   │   └── prometheus-values.yaml
│   │
│   ├── 📁 grafana/
│   │   ├── grafana-values.yaml
│   │   └── 📁 dashboards/
│   │       └── cluster-overview.json
│   │
│   └── 📁 jaeger/
│       └── jaeger-values.yaml
│
├── 📁 .github/                          # CI/CD Pipelines
│   └── 📁 workflows/
│       ├── build-push-ecr.yaml           # Build & push to AWS ECR
│       └── security-scan.yaml
│
├── 📁 applications/                     # Sample Applications
│   ├── 📁 demo-app/
│   │   ├── Dockerfile
│   │   └── src/
│   │
│   ├── 📁 demo-frontend/
│   │   ├── Dockerfile
│   │   └── src/
│   │
│   └── 📁 demo-backend/
│       ├── Dockerfile
│       └── src/
│
├── 📁 scripts/                          # Automation scripts
│   ├── setup-local-cluster.ps1
│   ├── install-argocd.ps1
│   └── install-istio.ps1
│
├── 📁 docs/                             # Documentation
│   └── troubleshooting.md
│
├── .gitignore
└── README.md
```

---

## 🔧 Phase 1: Local Development Environment

### 1.1 Initialize Git Repository

```bash
# Create project directory
mkdir multi-cloud-gitops-platform
cd multi-cloud-gitops-platform

# Initialize Git
git init
git branch -M main

# Create .gitignore
cat > .gitignore << 'EOF'
# Terraform
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
*.tfvars
!example.tfvars

# Secrets
*.pem
*.key
secrets/
.env
*.env

# IDE
.idea/
.vscode/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Kubernetes
kubeconfig*
*.kubeconfig

# Logs
*.log
logs/

# Build artifacts
dist/
build/
node_modules/
EOF

# Create initial structure (AWS Free Tier + Local only)
mkdir -p infrastructure/aws/{ecr,s3,iam}
mkdir -p kubernetes/{base,apps}
mkdir -p gitops/{argocd,applications}
mkdir -p service-mesh/{istio-install,gateway,traffic-management,security}
mkdir -p security/{trivy,opa-gatekeeper/constraints}
mkdir -p observability/{prometheus,grafana/dashboards,jaeger}
mkdir -p .github/workflows
mkdir -p applications/{demo-app,demo-frontend,demo-backend}
mkdir -p scripts docs

# Initial commit
git add .
git commit -m "Initial project structure"
```

### 1.2 Configure Cloud CLI Authentication

#### AWS Configuration

```bash
# Configure AWS CLI
aws configure
# Enter: AWS Access Key ID
# Enter: AWS Secret Access Key
# Enter: Default region (us-east-1)
# Enter: Default output format (json)

# Verify
aws sts get-caller-identity

# Create AWS credentials file for Terraform
cat > ~/.aws/credentials << 'EOF'
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY

[terraform]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
EOF
```

---

## ☁️ Phase 1.5: AWS Free Tier Setup

> ### ✅ USE THIS SECTION - All services here are FREE for 12 months!
>
> We'll set up AWS services that stay within free tier limits.

### 1.5.1 Create AWS Account (If you don't have one)

1. Go to [aws.amazon.com](https://aws.amazon.com)
2. Click "Create an AWS Account"
3. Enter email, password, account name
4. Add payment method (required but won't be charged if you stay in free tier)
5. Complete phone verification
6. Select "Basic Support - Free"

### 1.5.2 Configure AWS CLI

```powershell
# Install AWS CLI (if not already installed)
choco install awscli -y

# Configure with your credentials
aws configure
# Enter: AWS Access Key ID (from AWS Console → IAM → Users → Security Credentials)
# Enter: AWS Secret Access Key
# Enter: Default region: us-east-1
# Enter: Default output format: json

# Verify connection
aws sts get-caller-identity
```

### 1.5.3 Create ECR Repository (FREE - 500MB)

```powershell
# Create a private container registry
aws ecr create-repository `
    --repository-name multicloud-gitops/app `
    --image-scanning-configuration scanOnPush=true `
    --region us-east-1

# Get your ECR login command
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $(aws sts get-caller-identity --query Account --output text).dkr.ecr.us-east-1.amazonaws.com

# Your ECR URL will be:
# <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/multicloud-gitops/app
```

### 1.5.4 Create S3 Bucket for Terraform State (FREE - 5GB)

```powershell
# Create S3 bucket for Terraform state (bucket names must be globally unique)
$BUCKET_NAME = "multicloud-gitops-tfstate-$(Get-Random -Maximum 9999)"

aws s3 mb s3://$BUCKET_NAME --region us-east-1

# Enable versioning (best practice)
aws s3api put-bucket-versioning `
    --bucket $BUCKET_NAME `
    --versioning-configuration Status=Enabled

# Block public access (security)
aws s3api put-public-access-block `
    --bucket $BUCKET_NAME `
    --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true

# Save bucket name for later
echo "Your S3 bucket: $BUCKET_NAME"
```

### 1.5.5 Create IAM User for CI/CD (FREE)

```powershell
# Create IAM user for GitHub Actions
aws iam create-user --user-name github-actions-deployer

# Attach ECR policy
aws iam attach-user-policy `
    --user-name github-actions-deployer `
    --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryPowerUser

# Create access keys for GitHub Actions
aws iam create-access-key --user-name github-actions-deployer

# ⚠️ SAVE THESE CREDENTIALS SECURELY!
# You'll add them to GitHub Secrets later
```

### 1.5.6 Set Up AWS Budget Alert (Prevent Surprise Charges!)

```powershell
# Create a $1 budget alert (you'll get emailed if you exceed)
@"
{
    "BudgetName": "FreeTierMonitor",
    "BudgetLimit": {
        "Amount": "1",
        "Unit": "USD"
    },
    "BudgetType": "COST",
    "TimeUnit": "MONTHLY"
}
"@ | Out-File -FilePath budget.json -Encoding utf8

aws budgets create-budget `
    --account-id $(aws sts get-caller-identity --query Account --output text) `
    --budget file://budget.json
```

### 1.5.7 Verify Your Free Tier Usage

```powershell
# Check your current free tier usage
aws ce get-cost-and-usage `
    --time-period Start=$(Get-Date -Format "yyyy-MM-01"),End=$(Get-Date -Format "yyyy-MM-dd") `
    --granularity MONTHLY `
    --metrics "UnblendedCost" `
    --output table
```

### ✅ AWS Free Tier Setup Complete!

You now have:
| Service | What You Created | Free Tier Limit |
|---------|------------------|-----------------|
| **ECR** | Container registry | 500 MB/month |
| **S3** | Terraform state bucket | 5 GB storage |
| **IAM** | CI/CD user | Unlimited |
| **Budget** | $1 alert | Free |

---
Stop cluster (save resources)	minikube stop
Start again	minikube start
Delete cluster	minikube delete
SSH into node	minikube ssh
Check status	minikube status
View logs	minikube logs
Open dashboard	minikube dashboard
Get cluster IP	minikube ip
Expose service	minikube service <name> -n <namespace>

## 🖥️ Phase 2: Local Kubernetes Setup (FREE)

> ### ✅ USE THIS SECTION FOR ZERO-COST LEARNING
> 
> This section sets up a **fully functional Kubernetes cluster on your local machine** at no cost.
> You'll learn the same skills that apply to cloud Kubernetes!

### 2.1 Choose Your Local Kubernetes Tool

| Tool | Best For | Resources Needed | Recommendation |
|------|----------|------------------|----------------|
| **Minikube** | Beginners, Full features | 4+ CPU, 8GB+ RAM | ⭐ Recommended |
| **Kind** | CI/CD, Fast startup | 2+ CPU, 4GB+ RAM | Good alternative |
| **Docker Desktop K8s** | Mac/Windows users | 4+ CPU, 8GB+ RAM | Easiest setup |

### 2.2 Minikube Setup (Recommended)

```powershell
# Windows - Install Minikube via Chocolatey
choco install minikube -y

# Verify installation
minikube version

# Start cluster with adequate resources for Istio & monitoring
minikube start --cpus=4 --memory=8192 --driver=docker --kubernetes-version=v1.28.0

# Enable useful addons
minikube addons enable ingress
minikube addons enable ingress-dns
minikube addons enable metrics-server
minikube addons enable dashboard

# Verify cluster is running
kubectl cluster-info
kubectl get nodes

# Access Kubernetes dashboard (optional)
minikube dashboard
```

### 2.3 Alternative: Kind Setup

```powershell
# Windows - Install Kind via Chocolatey
choco install kind -y

# Create cluster config file
@"
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: multicloud-local
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
"@ | Out-File -FilePath kind-config.yaml -Encoding utf8

# Create cluster
kind create cluster --config kind-config.yaml

# Verify
kubectl cluster-info
kubectl get nodes
```

### 2.4 Alternative: Docker Desktop Kubernetes

```powershell
# 1. Open Docker Desktop
# 2. Go to Settings → Kubernetes
# 3. Check "Enable Kubernetes"
# 4. Click "Apply & Restart"
# 5. Wait for green indicator

# Verify
kubectl cluster-info
kubectl get nodes
```

### 2.5 Create Essential Namespaces

```powershell
# Create namespaces that match the cloud setup
kubectl create namespace argocd
kubectl create namespace istio-system
kubectl create namespace monitoring
kubectl create namespace security
kubectl create namespace apps

# Verify namespaces
kubectl get namespaces
```

### 2.6 Connect to AWS ECR (For pulling images from your Free Tier registry)

```powershell
# Get your AWS Account ID
$AWS_ACCOUNT_ID = aws sts get-caller-identity --query Account --output text
$AWS_REGION = "us-east-1"
$ECR_REGISTRY = "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"

# Login to ECR
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY

# Create Kubernetes secret for pulling images from ECR
$ECR_PASSWORD = aws ecr get-login-password --region $AWS_REGION

kubectl create secret docker-registry ecr-credentials `
    --docker-server=$ECR_REGISTRY `
    --docker-username=AWS `
    --docker-password=$ECR_PASSWORD `
    --namespace=apps

# Verify secret was created
kubectl get secret ecr-credentials -n apps
```

### 2.7 Verify Your Local Cluster

```powershell
# Check everything is running
kubectl get nodes -o wide
kubectl get pods -A
kubectl top nodes  # Requires metrics-server

# Expected output:
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   1m    v1.28.0
```

### 💡 Local Cluster Tips

| Tip | Command |
|-----|---------|
| **Stop cluster** (save resources) | `minikube stop` |
| **Start again** | `minikube start` |
| **Delete cluster** | `minikube delete` |
| **SSH into node** | `minikube ssh` |
| **Check status** | `minikube status` |
| **View logs** | `minikube logs` |

### 🎯 What's Next?

Your local Kubernetes cluster is now ready! Proceed to:
- **Phase 4**: Install Argo CD

---

## 🔄 Phase 4: GitOps with Argo CD

### 4.1 Argo CD Installation

```yaml
# gitops/argocd/install.yaml

apiVersion: v1
kind: Namespace
metadata:
  name: argocd
  labels:
    app.kubernetes.io/name: argocd
---
# Use Helm to install Argo CD
# helm repo add argo https://argoproj.github.io/argo-helm
# helm install argocd argo/argo-cd -n argocd -f values.yaml
```

```yaml
# gitops/argocd/values.yaml

global:
  image:
    tag: "v2.9.3"

server:
  replicas: 2
  
  ingress:
    enabled: true
    ingressClassName: nginx
    annotations:
      cert-manager.io/cluster-issuer: letsencrypt-prod
      nginx.ingress.kubernetes.io/ssl-passthrough: "true"
      nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
    hosts:
      - argocd.yourdomain.com
    tls:
      - secretName: argocd-server-tls
        hosts:
          - argocd.yourdomain.com

  extraArgs:
    - --insecure # Remove in production with proper TLS

  config:
    url: https://argocd.yourdomain.com
    
    # Enable status badge
    statusbadge.enabled: "true"
    
    # Repository credentials template
    repository.credentials: |
      - url: https://github.com/
        passwordSecret:
          name: github-creds
          key: password
        usernameSecret:
          name: github-creds
          key: username

controller:
  replicas: 1

repoServer:
  replicas: 1

applicationSet:
  enabled: true
```

### 4.2 Quick Install Commands (For Local Cluster)

```powershell
# Add Argo CD Helm repository
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Install Argo CD (simplified for local development)
helm install argocd argo/argo-cd -n argocd --create-namespace --set server.service.type=NodePort

# Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s

# Get the initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Access Argo CD UI
# Option 1: Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Option 2: Minikube service (opens in browser)
minikube service argocd-server -n argocd

# Login via CLI
argocd login localhost:8080 --username admin --password <password-from-above> --insecure
```

### 4.3 Create Your First Application

```yaml
# gitops/applications/demo-app.yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: demo-app
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/YOUR_USERNAME/multi-cloud-gitops-platform.git
    targetRevision: main
    path: kubernetes/apps/demo-app
    
  destination:
    server: https://kubernetes.default.svc
    namespace: apps
    
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

```powershell
# Apply the application
kubectl apply -f gitops/applications/demo-app.yaml

# Check status
argocd app list
argocd app get demo-app
```

---

## ⏭️ Continue to Part 2

Part 1 covers the foundational infrastructure. **Continue to [IMPLEMENTATION_PART2.md](IMPLEMENTATION_PART2.md)** for:

- ✅ Istio Service Mesh Configuration
- ✅ DevSecOps with Trivy & OPA
- ✅ Observability Stack (Prometheus, Grafana, Jaeger)
- ✅ CI/CD Pipelines with GitHub Actions
- ✅ Sample Applications
- ✅ Testing & Validation

---

**[➡️ Continue to Part 2: Service Mesh, Security & Observability](IMPLEMENTATION_PART2.md)**
