# 🚀 Enterprise Multi-Cloud GitOps Platform with DevSecOps
## Part 1: Infrastructure, Kubernetes Clusters & GitOps Foundation

---

## 📋 Table of Contents - Part 1

1. [Project Overview](#project-overview)
2. [Architecture Design](#architecture-design)
3. [Prerequisites & Tools](#prerequisites--tools)
4. [Phase 1: Local Development Environment](#phase-1-local-development-environment)
5. [Phase 2: Cloud Infrastructure Setup](#phase-2-cloud-infrastructure-setup)
6. [Phase 3: Kubernetes Cluster Provisioning](#phase-3-kubernetes-cluster-provisioning)
7. [Phase 4: GitOps with Argo CD](#phase-4-gitops-with-argo-cd)
8. [Repository Structure](#repository-structure)

---

## 🎯 Project Overview

### What We're Building

An enterprise-grade, production-ready platform that combines:

| Component | Technology | Purpose |
|-----------|------------|---------|
| **Multi-Cloud K8s** | EKS + AKS + GKE | High availability, vendor independence |
| **GitOps** | Argo CD | Declarative, auditable deployments |
| **Service Mesh** | Istio | Zero-trust security, traffic management |
| **DevSecOps** | Trivy, OPA, Snyk | Shift-left security |
| **Observability** | Prometheus, Grafana, Jaeger | Full-stack monitoring |
| **Autoscaling** | HPA, VPA, Cluster Autoscaler | Cost optimization |

### Project Timeline

```
Week 1-2: Infrastructure & Clusters
Week 3-4: GitOps & Service Mesh
Week 5-6: Security & Observability
Week 7-8: Testing, Optimization & Documentation
```

---

## 🏗️ Architecture Design

### High-Level Architecture

```
                                    ┌─────────────────────────────────────┐
                                    │         GitHub Repository           │
                                    │    (Infrastructure + App Configs)   │
                                    └──────────────────┬──────────────────┘
                                                       │
                                    ┌──────────────────▼──────────────────┐
                                    │          GitHub Actions             │
                                    │   (CI Pipeline + Security Scans)    │
                                    └──────────────────┬──────────────────┘
                                                       │
                         ┌─────────────────────────────┼─────────────────────────────┐
                         │                             │                             │
              ┌──────────▼──────────┐      ┌──────────▼──────────┐      ┌──────────▼──────────┐
              │     AWS (EKS)       │      │    Azure (AKS)      │      │     GCP (GKE)       │
              │  ┌───────────────┐  │      │  ┌───────────────┐  │      │  ┌───────────────┐  │
              │  │   Argo CD     │◄─┼──────┼──│   Argo CD     │◄─┼──────┼──│   Argo CD     │  │
              │  │   (GitOps)    │  │      │  │   (GitOps)    │  │      │  │   (GitOps)    │  │
              │  └───────┬───────┘  │      │  └───────┬───────┘  │      │  └───────┬───────┘  │
              │          │          │      │          │          │      │          │          │
              │  ┌───────▼───────┐  │      │  ┌───────▼───────┐  │      │  ┌───────▼───────┐  │
              │  │    Istio      │  │      │  │    Istio      │  │      │  │    Istio      │  │
              │  │ Service Mesh  │  │      │  │ Service Mesh  │  │      │  │ Service Mesh  │  │
              │  └───────┬───────┘  │      │  └───────┬───────┘  │      │  └───────┬───────┘  │
              │          │          │      │          │          │      │          │          │
              │  ┌───────▼───────┐  │      │  ┌───────▼───────┐  │      │  ┌───────▼───────┐  │
              │  │ Applications  │  │      │  │ Applications  │  │      │  │ Applications  │  │
              │  │  (Microsvcs)  │  │      │  │  (Microsvcs)  │  │      │  │  (Microsvcs)  │  │
              │  └───────────────┘  │      │  └───────────────┘  │      │  └───────────────┘  │
              └─────────────────────┘      └─────────────────────┘      └─────────────────────┘
                         │                             │                             │
                         └─────────────────────────────┼─────────────────────────────┘
                                                       │
                                    ┌──────────────────▼──────────────────┐
                                    │      Centralized Observability      │
                                    │  Prometheus │ Grafana │ Jaeger      │
                                    └─────────────────────────────────────┘
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

| Provider | What You Need | Free Tier Available |
|----------|---------------|---------------------|
| AWS | AWS Account with admin access | Yes (12 months) |
| Azure | Azure Subscription | Yes ($200 credit) |
| GCP | GCP Project with billing | Yes ($300 credit) |
| GitHub | GitHub account | Yes |
| Docker Hub | Docker Hub account | Yes |

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
choco install -y azure-cli
choco install -y gcloudsdk
choco install -y argocd-cli
choco install -y istioctl
choco install -y k9s
choco install -y lens
choco install -y git
choco install -y vscode

# Verify Installations
kubectl version --client
helm version
terraform version
aws --version
az version
gcloud version
argocd version --client
istioctl version --remote=false
```

#### macOS (Terminal)

```bash
# Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Required Tools
brew install kubectl helm terraform awscli azure-cli
brew install --cask google-cloud-sdk docker
brew install argocd istioctl k9s
brew install --cask lens

# Verify Installations
kubectl version --client
helm version
terraform version
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
├── 📁 infrastructure/                    # Terraform IaC
│   ├── 📁 aws/
│   │   ├── 📁 eks/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   ├── outputs.tf
│   │   │   └── versions.tf
│   │   ├── 📁 vpc/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   └── 📁 ecr/
│   │       └── main.tf
│   │
│   ├── 📁 azure/
│   │   ├── 📁 aks/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   ├── 📁 vnet/
│   │   │   ├── main.tf
│   │   │   └── variables.tf
│   │   └── 📁 acr/
│   │       └── main.tf
│   │
│   ├── 📁 gcp/
│   │   ├── 📁 gke/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf
│   │   ├── 📁 vpc/
│   │   │   └── main.tf
│   │   └── 📁 gcr/
│   │       └── main.tf
│   │
│   └── 📁 modules/
│       ├── 📁 kubernetes-addons/
│       └── 📁 networking/
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
│   ├── 📁 overlays/                     # Kustomize overlays
│   │   ├── 📁 aws/
│   │   │   └── kustomization.yaml
│   │   ├── 📁 azure/
│   │   │   └── kustomization.yaml
│   │   └── 📁 gcp/
│   │       └── kustomization.yaml
│   │
│   └── 📁 apps/                         # Application manifests
│       ├── 📁 frontend/
│       ├── 📁 backend/
│       └── 📁 database/
│
├── 📁 gitops/                           # Argo CD configurations
│   ├── 📁 argocd/
│   │   ├── install.yaml
│   │   ├── argocd-cm.yaml
│   │   ├── argocd-rbac-cm.yaml
│   │   └── 📁 projects/
│   │       ├── infrastructure.yaml
│   │       ├── applications.yaml
│   │       └── security.yaml
│   │
│   ├── 📁 applications/
│   │   ├── app-of-apps.yaml
│   │   ├── 📁 aws/
│   │   │   └── applications.yaml
│   │   ├── 📁 azure/
│   │   │   └── applications.yaml
│   │   └── 📁 gcp/
│   │       └── applications.yaml
│   │
│   └── 📁 applicationsets/
│       └── multi-cluster-apps.yaml
│
├── 📁 service-mesh/                     # Istio configurations
│   ├── 📁 istio-install/
│   │   ├── istio-operator.yaml
│   │   └── istio-profile.yaml
│   │
│   ├── 📁 gateway/
│   │   ├── gateway.yaml
│   │   └── virtual-services.yaml
│   │
│   ├── 📁 traffic-management/
│   │   ├── destination-rules.yaml
│   │   ├── canary-release.yaml
│   │   └── circuit-breaker.yaml
│   │
│   └── 📁 security/
│       ├── peer-authentication.yaml
│       ├── authorization-policy.yaml
│       └── request-authentication.yaml
│
├── 📁 security/                         # DevSecOps
│   ├── 📁 trivy/
│   │   ├── trivy-operator.yaml
│   │   └── scan-policies.yaml
│   │
│   ├── 📁 opa-gatekeeper/
│   │   ├── gatekeeper-install.yaml
│   │   └── 📁 constraints/
│   │       ├── require-labels.yaml
│   │       ├── container-limits.yaml
│   │       └── allowed-repos.yaml
│   │
│   ├── 📁 falco/
│   │   └── falco-daemonset.yaml
│   │
│   └── 📁 vault/
│       ├── vault-install.yaml
│       └── secret-policies.yaml
│
├── 📁 observability/                    # Monitoring Stack
│   ├── 📁 prometheus/
│   │   ├── prometheus-operator.yaml
│   │   ├── prometheus-rules.yaml
│   │   └── alertmanager-config.yaml
│   │
│   ├── 📁 grafana/
│   │   ├── grafana-install.yaml
│   │   └── 📁 dashboards/
│   │       ├── cluster-overview.json
│   │       ├── istio-mesh.json
│   │       └── security-dashboard.json
│   │
│   ├── 📁 jaeger/
│   │   └── jaeger-operator.yaml
│   │
│   └── 📁 loki/
│       └── loki-stack.yaml
│
├── 📁 .github/                          # CI/CD Pipelines
│   └── 📁 workflows/
│       ├── ci-pipeline.yaml
│       ├── security-scan.yaml
│       ├── infrastructure-deploy.yaml
│       └── argocd-sync.yaml
│
├── 📁 applications/                     # Sample Applications
│   ├── 📁 demo-frontend/
│   │   ├── Dockerfile
│   │   ├── src/
│   │   └── k8s/
│   │
│   ├── 📁 demo-backend/
│   │   ├── Dockerfile
│   │   ├── src/
│   │   └── k8s/
│   │
│   └── 📁 demo-api/
│       ├── Dockerfile
│       ├── src/
│       └── k8s/
│
├── 📁 scripts/                          # Automation scripts
│   ├── setup-cluster.sh
│   ├── install-argocd.sh
│   ├── install-istio.sh
│   └── cleanup.sh
│
├── 📁 docs/                             # Documentation
│   ├── architecture.md
│   ├── runbook.md
│   └── troubleshooting.md
│
├── .gitignore
├── README.md
└── Makefile
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

# Create initial structure
mkdir -p infrastructure/{aws,azure,gcp}/{vpc,eks,aks,gke,ecr,acr,gcr}
mkdir -p kubernetes/{base,overlays/{aws,azure,gcp},apps}
mkdir -p gitops/{argocd,applications,applicationsets}
mkdir -p service-mesh/{istio-install,gateway,traffic-management,security}
mkdir -p security/{trivy,opa-gatekeeper/constraints,falco,vault}
mkdir -p observability/{prometheus,grafana/dashboards,jaeger,loki}
mkdir -p .github/workflows
mkdir -p applications/{demo-frontend,demo-backend,demo-api}
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

#### Azure Configuration

```bash
# Login to Azure
az login

# Set subscription
az account list --output table
az account set --subscription "YOUR_SUBSCRIPTION_ID"

# Create Service Principal for Terraform
az ad sp create-for-rbac --name "terraform-sp" --role="Contributor" \
  --scopes="/subscriptions/YOUR_SUBSCRIPTION_ID" \
  --sdk-auth > azure-credentials.json

# Set environment variables
export ARM_CLIENT_ID="appId from above"
export ARM_CLIENT_SECRET="password from above"
export ARM_SUBSCRIPTION_ID="YOUR_SUBSCRIPTION_ID"
export ARM_TENANT_ID="tenant from above"
```

#### GCP Configuration

```bash
# Login to GCP
gcloud auth login
gcloud auth application-default login

# Set project
gcloud projects list
gcloud config set project YOUR_PROJECT_ID

# Enable required APIs
gcloud services enable container.googleapis.com
gcloud services enable compute.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com

# Create Service Account for Terraform
gcloud iam service-accounts create terraform \
  --display-name="Terraform Service Account"

gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
  --member="serviceAccount:terraform@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/editor"

gcloud iam service-accounts keys create gcp-credentials.json \
  --iam-account=terraform@YOUR_PROJECT_ID.iam.gserviceaccount.com

export GOOGLE_APPLICATION_CREDENTIALS="$(pwd)/gcp-credentials.json"
```

---

## 🌐 Phase 2: Cloud Infrastructure Setup

### 2.1 Terraform Backend Configuration

Create remote state storage for Terraform:

#### AWS S3 Backend

```hcl
# infrastructure/aws/backend/main.tf

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "terraform_state" {
  bucket = "multicloud-gitops-tfstate-${random_id.bucket_suffix.hex}"

  lifecycle {
    prevent_destroy = true
  }

  tags = {
    Name        = "Terraform State Bucket"
    Environment = "infrastructure"
    Project     = "multi-cloud-gitops"
  }
}

resource "random_id" "bucket_suffix" {
  byte_length = 4
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "aws:kms"
    }
  }
}

resource "aws_s3_bucket_public_access_block" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "multicloud-gitops-tfstate-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name        = "Terraform State Lock Table"
    Environment = "infrastructure"
    Project     = "multi-cloud-gitops"
  }
}

output "bucket_name" {
  value = aws_s3_bucket.terraform_state.id
}

output "dynamodb_table_name" {
  value = aws_dynamodb_table.terraform_locks.name
}
```

### 2.2 AWS VPC & EKS Configuration

```hcl
# infrastructure/aws/vpc/main.tf

terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "multicloud-gitops-tfstate-XXXX"
    key            = "aws/vpc/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "multicloud-gitops-tfstate-locks"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = {
      Project     = "multi-cloud-gitops"
      Environment = var.environment
      ManagedBy   = "terraform"
    }
  }
}

# Variables
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "production"
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0.0/16"
}

variable "cluster_name" {
  description = "EKS cluster name"
  type        = string
  default     = "multicloud-eks"
}

# Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

# VPC Module
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "${var.cluster_name}-vpc"
  cidr = var.vpc_cidr

  azs             = slice(data.aws_availability_zones.available.names, 0, 3)
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]

  enable_nat_gateway     = true
  single_nat_gateway     = false
  one_nat_gateway_per_az = true
  enable_vpn_gateway     = false

  enable_dns_hostnames = true
  enable_dns_support   = true

  # EKS requires specific tags
  public_subnet_tags = {
    "kubernetes.io/role/elb"                    = 1
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb"           = 1
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
  }

  tags = {
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
  }
}

# Outputs
output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "private_subnets" {
  description = "Private subnet IDs"
  value       = module.vpc.private_subnets
}

output "public_subnets" {
  description = "Public subnet IDs"
  value       = module.vpc.public_subnets
}
```

```hcl
# infrastructure/aws/eks/main.tf

terraform {
  required_version = ">= 1.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
    helm = {
      source  = "hashicorp/helm"
      version = "~> 2.11"
    }
  }

  backend "s3" {
    bucket         = "multicloud-gitops-tfstate-XXXX"
    key            = "aws/eks/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "multicloud-gitops-tfstate-locks"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region
}

# Variables
variable "aws_region" {
  default = "us-east-1"
}

variable "cluster_name" {
  default = "multicloud-eks"
}

variable "cluster_version" {
  default = "1.28"
}

variable "environment" {
  default = "production"
}

# Data sources
data "terraform_remote_state" "vpc" {
  backend = "s3"
  config = {
    bucket = "multicloud-gitops-tfstate-XXXX"
    key    = "aws/vpc/terraform.tfstate"
    region = "us-east-1"
  }
}

data "aws_caller_identity" "current" {}

# EKS Cluster
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  vpc_id     = data.terraform_remote_state.vpc.outputs.vpc_id
  subnet_ids = data.terraform_remote_state.vpc.outputs.private_subnets

  cluster_endpoint_public_access  = true
  cluster_endpoint_private_access = true

  # Enable IRSA
  enable_irsa = true

  # Cluster addons
  cluster_addons = {
    coredns = {
      most_recent = true
    }
    kube-proxy = {
      most_recent = true
    }
    vpc-cni = {
      most_recent              = true
      before_compute           = true
      service_account_role_arn = module.vpc_cni_irsa.iam_role_arn
      configuration_values = jsonencode({
        env = {
          ENABLE_PREFIX_DELEGATION = "true"
          WARM_PREFIX_TARGET       = "1"
        }
      })
    }
    aws-ebs-csi-driver = {
      most_recent              = true
      service_account_role_arn = module.ebs_csi_irsa.iam_role_arn
    }
  }

  # Node groups
  eks_managed_node_groups = {
    # System node group for critical workloads
    system = {
      name = "system-ng"

      instance_types = ["t3.large"]
      capacity_type  = "ON_DEMAND"

      min_size     = 2
      max_size     = 4
      desired_size = 2

      labels = {
        role = "system"
      }

      taints = [{
        key    = "CriticalAddonsOnly"
        value  = "true"
        effect = "PREFER_NO_SCHEDULE"
      }]

      update_config = {
        max_unavailable_percentage = 33
      }

      tags = {
        NodeGroup = "system"
      }
    }

    # Application node group
    applications = {
      name = "apps-ng"

      instance_types = ["t3.xlarge", "t3.large"]
      capacity_type  = "SPOT"

      min_size     = 2
      max_size     = 10
      desired_size = 3

      labels = {
        role = "applications"
      }

      update_config = {
        max_unavailable_percentage = 33
      }

      tags = {
        NodeGroup = "applications"
      }
    }
  }

  # Security group rules
  node_security_group_additional_rules = {
    ingress_self_all = {
      description = "Node to node all ports/protocols"
      protocol    = "-1"
      from_port   = 0
      to_port     = 0
      type        = "ingress"
      self        = true
    }
    ingress_istio_webhook = {
      description                   = "Istio Webhook"
      protocol                      = "tcp"
      from_port                     = 15017
      to_port                       = 15017
      type                          = "ingress"
      source_cluster_security_group = true
    }
  }

  # aws-auth configmap
  manage_aws_auth_configmap = true

  aws_auth_roles = [
    {
      rolearn  = "arn:aws:iam::${data.aws_caller_identity.current.account_id}:role/Admin"
      username = "admin"
      groups   = ["system:masters"]
    }
  ]

  tags = {
    Environment = var.environment
    Project     = "multi-cloud-gitops"
  }
}

# IRSA for VPC CNI
module "vpc_cni_irsa" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  version = "~> 5.0"

  role_name_prefix      = "VPC-CNI-IRSA"
  attach_vpc_cni_policy = true
  vpc_cni_enable_ipv4   = true

  oidc_providers = {
    main = {
      provider_arn               = module.eks.oidc_provider_arn
      namespace_service_accounts = ["kube-system:aws-node"]
    }
  }
}

# IRSA for EBS CSI Driver
module "ebs_csi_irsa" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  version = "~> 5.0"

  role_name_prefix      = "EBS-CSI-IRSA"
  attach_ebs_csi_policy = true

  oidc_providers = {
    main = {
      provider_arn               = module.eks.oidc_provider_arn
      namespace_service_accounts = ["kube-system:ebs-csi-controller-sa"]
    }
  }
}

# Cluster Autoscaler IRSA
module "cluster_autoscaler_irsa" {
  source  = "terraform-aws-modules/iam/aws//modules/iam-role-for-service-accounts-eks"
  version = "~> 5.0"

  role_name_prefix                       = "Cluster-Autoscaler-IRSA"
  attach_cluster_autoscaler_policy       = true
  cluster_autoscaler_cluster_names       = [module.eks.cluster_name]

  oidc_providers = {
    main = {
      provider_arn               = module.eks.oidc_provider_arn
      namespace_service_accounts = ["kube-system:cluster-autoscaler"]
    }
  }
}

# Outputs
output "cluster_name" {
  value = module.eks.cluster_name
}

output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_certificate_authority_data" {
  value = module.eks.cluster_certificate_authority_data
}

output "oidc_provider_arn" {
  value = module.eks.oidc_provider_arn
}

output "configure_kubectl" {
  value = "aws eks update-kubeconfig --region ${var.aws_region} --name ${module.eks.cluster_name}"
}
```

### 2.3 Azure VNet & AKS Configuration

```hcl
# infrastructure/azure/vnet/main.tf

terraform {
  required_version = ">= 1.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.75"
    }
  }

  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "multicloudtfstate"
    container_name       = "tfstate"
    key                  = "azure/vnet/terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
}

variable "location" {
  default = "eastus"
}

variable "environment" {
  default = "production"
}

variable "resource_group_name" {
  default = "multicloud-gitops-rg"
}

variable "vnet_cidr" {
  default = "10.1.0.0/16"
}

# Resource Group
resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location

  tags = {
    Environment = var.environment
    Project     = "multi-cloud-gitops"
    ManagedBy   = "terraform"
  }
}

# Virtual Network
resource "azurerm_virtual_network" "main" {
  name                = "multicloud-vnet"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  address_space       = [var.vnet_cidr]

  tags = {
    Environment = var.environment
    Project     = "multi-cloud-gitops"
  }
}

# Subnets
resource "azurerm_subnet" "aks_nodes" {
  name                 = "aks-nodes-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.1.1.0/24"]
}

resource "azurerm_subnet" "aks_pods" {
  name                 = "aks-pods-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.1.2.0/22"]

  delegation {
    name = "aks-delegation"
    service_delegation {
      name = "Microsoft.ContainerService/managedClusters"
      actions = [
        "Microsoft.Network/virtualNetworks/subnets/join/action"
      ]
    }
  }
}

resource "azurerm_subnet" "application_gateway" {
  name                 = "appgw-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.1.10.0/24"]
}

# Network Security Group
resource "azurerm_network_security_group" "aks" {
  name                = "aks-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "AllowHTTPS"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "AllowHTTP"
    priority                   = 110
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }

  tags = {
    Environment = var.environment
  }
}

resource "azurerm_subnet_network_security_group_association" "aks" {
  subnet_id                 = azurerm_subnet.aks_nodes.id
  network_security_group_id = azurerm_network_security_group.aks.id
}

# Outputs
output "resource_group_name" {
  value = azurerm_resource_group.main.name
}

output "vnet_id" {
  value = azurerm_virtual_network.main.id
}

output "aks_nodes_subnet_id" {
  value = azurerm_subnet.aks_nodes.id
}

output "aks_pods_subnet_id" {
  value = azurerm_subnet.aks_pods.id
}
```

```hcl
# infrastructure/azure/aks/main.tf

terraform {
  required_version = ">= 1.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.75"
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.45"
    }
  }

  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "multicloudtfstate"
    container_name       = "tfstate"
    key                  = "azure/aks/terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
}

variable "location" {
  default = "eastus"
}

variable "cluster_name" {
  default = "multicloud-aks"
}

variable "kubernetes_version" {
  default = "1.28"
}

variable "environment" {
  default = "production"
}

# Data sources
data "terraform_remote_state" "vnet" {
  backend = "azurerm"
  config = {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "multicloudtfstate"
    container_name       = "tfstate"
    key                  = "azure/vnet/terraform.tfstate"
  }
}

data "azurerm_resource_group" "main" {
  name = data.terraform_remote_state.vnet.outputs.resource_group_name
}

# Log Analytics Workspace
resource "azurerm_log_analytics_workspace" "aks" {
  name                = "${var.cluster_name}-logs"
  location            = data.azurerm_resource_group.main.location
  resource_group_name = data.azurerm_resource_group.main.name
  sku                 = "PerGB2018"
  retention_in_days   = 30

  tags = {
    Environment = var.environment
    Project     = "multi-cloud-gitops"
  }
}

# AKS Cluster
resource "azurerm_kubernetes_cluster" "main" {
  name                = var.cluster_name
  location            = data.azurerm_resource_group.main.location
  resource_group_name = data.azurerm_resource_group.main.name
  dns_prefix          = var.cluster_name
  kubernetes_version  = var.kubernetes_version

  # System node pool
  default_node_pool {
    name                = "system"
    node_count          = 2
    vm_size             = "Standard_D2s_v3"
    vnet_subnet_id      = data.terraform_remote_state.vnet.outputs.aks_nodes_subnet_id
    enable_auto_scaling = true
    min_count           = 2
    max_count           = 4
    os_disk_size_gb     = 50
    os_disk_type        = "Managed"

    node_labels = {
      "nodepool-type" = "system"
      "environment"   = var.environment
    }

    tags = {
      Environment = var.environment
      NodePool    = "system"
    }
  }

  identity {
    type = "SystemAssigned"
  }

  # Network configuration
  network_profile {
    network_plugin     = "azure"
    network_policy     = "azure"
    load_balancer_sku  = "standard"
    service_cidr       = "10.100.0.0/16"
    dns_service_ip     = "10.100.0.10"
  }

  # Azure AD integration
  azure_active_directory_role_based_access_control {
    managed            = true
    azure_rbac_enabled = true
  }

  # Monitoring
  oms_agent {
    log_analytics_workspace_id = azurerm_log_analytics_workspace.aks.id
  }

  # Security
  azure_policy_enabled = true

  # Auto-upgrade channel
  automatic_channel_upgrade = "patch"

  # Maintenance window
  maintenance_window {
    allowed {
      day   = "Sunday"
      hours = [0, 1, 2, 3, 4, 5]
    }
  }

  tags = {
    Environment = var.environment
    Project     = "multi-cloud-gitops"
    ManagedBy   = "terraform"
  }
}

# Application node pool
resource "azurerm_kubernetes_cluster_node_pool" "apps" {
  name                  = "apps"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.main.id
  vm_size               = "Standard_D4s_v3"
  node_count            = 2
  vnet_subnet_id        = data.terraform_remote_state.vnet.outputs.aks_nodes_subnet_id

  enable_auto_scaling = true
  min_count           = 2
  max_count           = 10

  priority        = "Spot"
  eviction_policy = "Delete"
  spot_max_price  = -1 # Market price

  node_labels = {
    "nodepool-type"                    = "applications"
    "kubernetes.azure.com/scalesetpriority" = "spot"
  }

  node_taints = [
    "kubernetes.azure.com/scalesetpriority=spot:NoSchedule"
  ]

  tags = {
    Environment = var.environment
    NodePool    = "applications"
  }
}

# Container Registry
resource "azurerm_container_registry" "main" {
  name                = "multicloudgitopsacr"
  resource_group_name = data.azurerm_resource_group.main.name
  location            = data.azurerm_resource_group.main.location
  sku                 = "Premium"
  admin_enabled       = false

  georeplications {
    location                = "westus"
    zone_redundancy_enabled = true
  }

  tags = {
    Environment = var.environment
    Project     = "multi-cloud-gitops"
  }
}

# ACR Pull role assignment
resource "azurerm_role_assignment" "acr_pull" {
  principal_id                     = azurerm_kubernetes_cluster.main.kubelet_identity[0].object_id
  role_definition_name             = "AcrPull"
  scope                            = azurerm_container_registry.main.id
  skip_service_principal_aad_check = true
}

# Outputs
output "cluster_name" {
  value = azurerm_kubernetes_cluster.main.name
}

output "kube_config_raw" {
  value     = azurerm_kubernetes_cluster.main.kube_config_raw
  sensitive = true
}

output "cluster_fqdn" {
  value = azurerm_kubernetes_cluster.main.fqdn
}

output "acr_login_server" {
  value = azurerm_container_registry.main.login_server
}

output "configure_kubectl" {
  value = "az aks get-credentials --resource-group ${data.azurerm_resource_group.main.name} --name ${azurerm_kubernetes_cluster.main.name}"
}
```

### 2.4 GCP VPC & GKE Configuration

```hcl
# infrastructure/gcp/vpc/main.tf

terraform {
  required_version = ">= 1.0"

  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }

  backend "gcs" {
    bucket = "multicloud-gitops-tfstate"
    prefix = "gcp/vpc"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  default = "us-central1"
}

variable "environment" {
  default = "production"
}

# VPC Network
resource "google_compute_network" "main" {
  name                    = "multicloud-vpc"
  auto_create_subnetworks = false
  routing_mode            = "GLOBAL"
}

# Subnet for GKE
resource "google_compute_subnetwork" "gke" {
  name          = "gke-subnet"
  ip_cidr_range = "10.2.0.0/20"
  region        = var.region
  network       = google_compute_network.main.id

  secondary_ip_range {
    range_name    = "gke-pods"
    ip_cidr_range = "10.3.0.0/16"
  }

  secondary_ip_range {
    range_name    = "gke-services"
    ip_cidr_range = "10.4.0.0/20"
  }

  private_ip_google_access = true
}

# Cloud Router for NAT
resource "google_compute_router" "main" {
  name    = "multicloud-router"
  region  = var.region
  network = google_compute_network.main.id
}

# Cloud NAT
resource "google_compute_router_nat" "main" {
  name                               = "multicloud-nat"
  router                             = google_compute_router.main.name
  region                             = var.region
  nat_ip_allocate_option             = "AUTO_ONLY"
  source_subnetwork_ip_ranges_to_nat = "ALL_SUBNETWORKS_ALL_IP_RANGES"

  log_config {
    enable = true
    filter = "ERRORS_ONLY"
  }
}

# Firewall rules
resource "google_compute_firewall" "allow_internal" {
  name    = "allow-internal"
  network = google_compute_network.main.name

  allow {
    protocol = "icmp"
  }

  allow {
    protocol = "tcp"
    ports    = ["0-65535"]
  }

  allow {
    protocol = "udp"
    ports    = ["0-65535"]
  }

  source_ranges = ["10.2.0.0/16"]
}

resource "google_compute_firewall" "allow_ssh" {
  name    = "allow-ssh"
  network = google_compute_network.main.name

  allow {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = ["0.0.0.0/0"]
  target_tags   = ["ssh"]
}

# Outputs
output "network_name" {
  value = google_compute_network.main.name
}

output "network_id" {
  value = google_compute_network.main.id
}

output "subnet_name" {
  value = google_compute_subnetwork.gke.name
}

output "subnet_id" {
  value = google_compute_subnetwork.gke.id
}
```

```hcl
# infrastructure/gcp/gke/main.tf

terraform {
  required_version = ">= 1.0"

  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "~> 5.0"
    }
  }

  backend "gcs" {
    bucket = "multicloud-gitops-tfstate"
    prefix = "gcp/gke"
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

provider "google-beta" {
  project = var.project_id
  region  = var.region
}

variable "project_id" {
  description = "GCP Project ID"
  type        = string
}

variable "region" {
  default = "us-central1"
}

variable "cluster_name" {
  default = "multicloud-gke"
}

variable "environment" {
  default = "production"
}

# Data sources
data "terraform_remote_state" "vpc" {
  backend = "gcs"
  config = {
    bucket = "multicloud-gitops-tfstate"
    prefix = "gcp/vpc"
  }
}

# Service Account for GKE nodes
resource "google_service_account" "gke_nodes" {
  account_id   = "gke-nodes-sa"
  display_name = "GKE Nodes Service Account"
}

resource "google_project_iam_member" "gke_nodes_roles" {
  for_each = toset([
    "roles/logging.logWriter",
    "roles/monitoring.metricWriter",
    "roles/monitoring.viewer",
    "roles/stackdriver.resourceMetadata.writer",
    "roles/storage.objectViewer",
    "roles/artifactregistry.reader"
  ])

  project = var.project_id
  role    = each.value
  member  = "serviceAccount:${google_service_account.gke_nodes.email}"
}

# GKE Cluster
resource "google_container_cluster" "main" {
  provider = google-beta

  name     = var.cluster_name
  location = var.region

  # We can't create a cluster with no node pool defined, but we want to only use
  # separately managed node pools. So we create the smallest possible default
  # node pool and immediately delete it.
  remove_default_node_pool = true
  initial_node_count       = 1

  network    = data.terraform_remote_state.vpc.outputs.network_name
  subnetwork = data.terraform_remote_state.vpc.outputs.subnet_name

  # IP allocation policy for VPC-native cluster
  ip_allocation_policy {
    cluster_secondary_range_name  = "gke-pods"
    services_secondary_range_name = "gke-services"
  }

  # Private cluster config
  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  # Master authorized networks
  master_authorized_networks_config {
    cidr_blocks {
      cidr_block   = "0.0.0.0/0"
      display_name = "All networks"
    }
  }

  # Workload Identity
  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  # Binary Authorization
  binary_authorization {
    evaluation_mode = "PROJECT_SINGLETON_POLICY_ENFORCE"
  }

  # Security posture
  security_posture_config {
    mode               = "BASIC"
    vulnerability_mode = "VULNERABILITY_ENTERPRISE"
  }

  # Addons
  addons_config {
    http_load_balancing {
      disabled = false
    }

    horizontal_pod_autoscaling {
      disabled = false
    }

    gce_persistent_disk_csi_driver_config {
      enabled = true
    }

    gcs_fuse_csi_driver_config {
      enabled = true
    }

    dns_cache_config {
      enabled = true
    }
  }

  # Cluster autoscaling
  cluster_autoscaling {
    enabled = true

    resource_limits {
      resource_type = "cpu"
      minimum       = 2
      maximum       = 100
    }

    resource_limits {
      resource_type = "memory"
      minimum       = 4
      maximum       = 400
    }

    auto_provisioning_defaults {
      service_account = google_service_account.gke_nodes.email
      oauth_scopes = [
        "https://www.googleapis.com/auth/cloud-platform"
      ]
    }
  }

  # Maintenance window
  maintenance_policy {
    daily_maintenance_window {
      start_time = "03:00"
    }
  }

  # Release channel
  release_channel {
    channel = "REGULAR"
  }

  # Logging and monitoring
  logging_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
  }

  monitoring_config {
    enable_components = ["SYSTEM_COMPONENTS"]

    managed_prometheus {
      enabled = true
    }
  }

  resource_labels = {
    environment = var.environment
    project     = "multi-cloud-gitops"
    managed_by  = "terraform"
  }
}

# System node pool
resource "google_container_node_pool" "system" {
  name       = "system-pool"
  location   = var.region
  cluster    = google_container_cluster.main.name
  node_count = 2

  autoscaling {
    min_node_count = 2
    max_node_count = 4
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }

  node_config {
    preemptible  = false
    machine_type = "e2-standard-2"
    disk_size_gb = 50
    disk_type    = "pd-ssd"

    service_account = google_service_account.gke_nodes.email
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    workload_metadata_config {
      mode = "GKE_METADATA"
    }

    shielded_instance_config {
      enable_secure_boot          = true
      enable_integrity_monitoring = true
    }

    labels = {
      pool        = "system"
      environment = var.environment
    }

    taint {
      key    = "CriticalAddonsOnly"
      value  = "true"
      effect = "PREFER_NO_SCHEDULE"
    }
  }
}

# Application node pool (preemptible/spot)
resource "google_container_node_pool" "apps" {
  name     = "apps-pool"
  location = var.region
  cluster  = google_container_cluster.main.name

  autoscaling {
    min_node_count = 2
    max_node_count = 10
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }

  node_config {
    preemptible  = true
    machine_type = "e2-standard-4"
    disk_size_gb = 100
    disk_type    = "pd-standard"

    service_account = google_service_account.gke_nodes.email
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    workload_metadata_config {
      mode = "GKE_METADATA"
    }

    shielded_instance_config {
      enable_secure_boot          = true
      enable_integrity_monitoring = true
    }

    labels = {
      pool        = "applications"
      environment = var.environment
    }
  }
}

# Artifact Registry
resource "google_artifact_registry_repository" "main" {
  location      = var.region
  repository_id = "multicloud-gitops"
  description   = "Docker repository for multi-cloud GitOps"
  format        = "DOCKER"

  cleanup_policies {
    id     = "keep-minimum-versions"
    action = "KEEP"

    most_recent_versions {
      keep_count = 10
    }
  }
}

# Outputs
output "cluster_name" {
  value = google_container_cluster.main.name
}

output "cluster_endpoint" {
  value     = google_container_cluster.main.endpoint
  sensitive = true
}

output "cluster_ca_certificate" {
  value     = google_container_cluster.main.master_auth[0].cluster_ca_certificate
  sensitive = true
}

output "artifact_registry" {
  value = "${var.region}-docker.pkg.dev/${var.project_id}/${google_artifact_registry_repository.main.repository_id}"
}

output "configure_kubectl" {
  value = "gcloud container clusters get-credentials ${google_container_cluster.main.name} --region ${var.region} --project ${var.project_id}"
}
```

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

    # Application health customizations
    resource.customizations.health.argoproj.io_Application: |
      hs = {}
      hs.status = "Progressing"
      hs.message = ""
      if obj.status ~= nil then
        if obj.status.health ~= nil then
          hs.status = obj.status.health.status
          if obj.status.health.message ~= nil then
            hs.message = obj.status.health.message
          end
        end
      end
      return hs

  rbacConfig:
    policy.default: role:readonly
    policy.csv: |
      p, role:admin, applications, *, */*, allow
      p, role:admin, clusters, *, *, allow
      p, role:admin, repositories, *, *, allow
      p, role:admin, logs, *, *, allow
      p, role:admin, exec, *, */*, allow
      p, role:developer, applications, get, */*, allow
      p, role:developer, applications, sync, */*, allow
      p, role:developer, logs, get, */*, allow
      g, admin-group, role:admin
      g, dev-group, role:developer

controller:
  replicas: 2
  
  metrics:
    enabled: true
    serviceMonitor:
      enabled: true

repoServer:
  replicas: 2
  
  extraContainers:
    - name: cmp-kustomize-with-helm
      command: [/var/run/argocd/argocd-cmp-server]
      image: quay.io/argoproj/argocd:v2.9.3
      securityContext:
        runAsNonRoot: true
        runAsUser: 999
      volumeMounts:
        - mountPath: /var/run/argocd
          name: var-files
        - mountPath: /home/argocd/cmp-server/plugins
          name: plugins
        - mountPath: /home/argocd/cmp-server/config/plugin.yaml
          subPath: plugin.yaml
          name: kustomize-with-helm

applicationSet:
  enabled: true
  replicas: 2

notifications:
  enabled: true
  
  secret:
    create: true
    items:
      slack-token: <your-slack-token>

  notifiers:
    service.slack: |
      token: $slack-token
      
  subscriptions:
    - recipients:
        - slack:devops-alerts
      triggers:
        - on-deployed
        - on-health-degraded
        - on-sync-failed

  templates:
    template.app-deployed: |
      message: |
        Application {{.app.metadata.name}} is now running new version.
      slack:
        attachments: |
          [{
            "title": "{{.app.metadata.name}}",
            "color": "good",
            "fields": [
              { "title": "Sync Status", "value": "{{.app.status.sync.status}}", "short": true },
              { "title": "Repository", "value": "{{.app.spec.source.repoURL}}", "short": true }
            ]
          }]

  triggers:
    trigger.on-deployed: |
      - when: app.status.operationState.phase in ['Succeeded'] and app.status.health.status == 'Healthy'
        send: [app-deployed]
    trigger.on-health-degraded: |
      - when: app.status.health.status == 'Degraded'
        send: [app-health-degraded]
    trigger.on-sync-failed: |
      - when: app.status.operationState.phase in ['Failed']
        send: [app-sync-failed]

redis-ha:
  enabled: true

dex:
  enabled: true
  
  config:
    connectors:
      - type: github
        id: github
        name: GitHub
        config:
          clientID: $dex.github.clientID
          clientSecret: $dex.github.clientSecret
          orgs:
            - name: your-github-org
```

### 4.2 Argo CD Application Configurations

```yaml
# gitops/applications/app-of-apps.yaml

apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: app-of-apps
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  
  source:
    repoURL: https://github.com/YOUR_ORG/multi-cloud-gitops-platform.git
    targetRevision: main
    path: gitops/applications
    
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
    
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

```yaml
# gitops/applicationsets/multi-cluster-apps.yaml

apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: multi-cluster-applications
  namespace: argocd
spec:
  generators:
    - matrix:
        generators:
          # Cluster generator
          - clusters:
              selector:
                matchLabels:
                  environment: production
          # Application list generator
          - list:
              elements:
                - app: frontend
                  namespace: apps
                  path: applications/demo-frontend/k8s
                - app: backend
                  namespace: apps
                  path: applications/demo-backend/k8s
                - app: api
                  namespace: apps
                  path: applications/demo-api/k8s

  template:
    metadata:
      name: '{{.app}}-{{.name}}'
      labels:
        app: '{{.app}}'
        cluster: '{{.name}}'
    spec:
      project: default
      
      source:
        repoURL: https://github.com/YOUR_ORG/multi-cloud-gitops-platform.git
        targetRevision: main
        path: '{{.path}}/overlays/{{.metadata.labels.cloud}}'
        
      destination:
        server: '{{.server}}'
        namespace: '{{.namespace}}'
        
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
          - ApplyOutOfSyncOnly=true
        retry:
          limit: 3
          backoff:
            duration: 5s
            maxDuration: 2m
            factor: 2

  strategy:
    type: RollingSync
    rollingSync:
      steps:
        - matchExpressions:
            - key: cloud
              operator: In
              values:
                - aws
        - matchExpressions:
            - key: cloud
              operator: In
              values:
                - azure
        - matchExpressions:
            - key: cloud
              operator: In
              values:
                - gcp
```

### 4.3 Multi-Cluster Registration

```bash
# scripts/register-clusters.sh

#!/bin/bash

set -e

ARGOCD_SERVER="argocd.yourdomain.com"

# Login to Argo CD
argocd login $ARGOCD_SERVER --username admin --password $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)

# Register AWS EKS cluster
echo "Registering AWS EKS cluster..."
aws eks update-kubeconfig --region us-east-1 --name multicloud-eks --alias aws-eks
argocd cluster add aws-eks --name aws-production --label cloud=aws --label environment=production

# Register Azure AKS cluster
echo "Registering Azure AKS cluster..."
az aks get-credentials --resource-group multicloud-gitops-rg --name multicloud-aks --admin --context azure-aks
argocd cluster add azure-aks --name azure-production --label cloud=azure --label environment=production

# Register GCP GKE cluster
echo "Registering GCP GKE cluster..."
gcloud container clusters get-credentials multicloud-gke --region us-central1 --project YOUR_PROJECT_ID
kubectl config rename-context gke_YOUR_PROJECT_ID_us-central1_multicloud-gke gcp-gke
argocd cluster add gcp-gke --name gcp-production --label cloud=gcp --label environment=production

# List registered clusters
argocd cluster list
```

---

## ⏭️ Continue to Part 2

Part 1 covers the foundational infrastructure. **Continue to [IMPLEMENTATION_PART2.md](IMPLEMENTATION_PART2.md)** for:

- ✅ Istio Service Mesh Configuration
- ✅ DevSecOps with Trivy & OPA
- ✅ Observability Stack (Prometheus, Grafana, Jaeger)
- ✅ Autoscaling (HPA/VPA)
- ✅ CI/CD Pipelines with GitHub Actions
- ✅ Sample Applications
- ✅ Testing & Validation
- ✅ Production Checklist

---

**[➡️ Continue to Part 2: Service Mesh, Security & Observability](IMPLEMENTATION_PART2.md)**
