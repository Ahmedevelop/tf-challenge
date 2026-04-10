# Terraform + Kubernetes Take-Home Challenge

## Overview
This project provisions a lightweight AWS infrastructure using Terraform and deploys a simple NGINX application on EKS.

## Demo Video
[Watch the walkthrough on YouTube](https://youtu.be/zbKH0nDm7UA)

## Architecture
- **VPC** with two public subnets across two availability zones
- **EKS cluster** with a single t3.medium node group
- **NGINX deployment** in a `dev` namespace with a ClusterIP service
- **ConfigMap** injected as environment variables into the NGINX container
- **ArgoCD** deployed via Helm as a bonus

## Prerequisites
- Terraform >= 1.5.0
- AWS CLI configured with valid credentials
- kubectl
- helm

## Setup Steps

### 1. Clone the repo
```bash
git clone <repo-url>
cd tf-challenge
```

### 2. Configure AWS credentials
```bash
aws configure
```

### 3. Initialize Terraform
```bash
terraform init
```

### 4. Review the plan
```bash
terraform plan
```

### 5. Apply
```bash
terraform apply
```
EKS cluster creation takes approximately 15-20 minutes.

### 6. Configure kubectl
```bash
aws eks update-kubeconfig --region us-east-1 --name tf-challenge
```

### 7. Verify
```bash
kubectl get nodes
kubectl get pods -n dev
kubectl port-forward service/nginx 8080:80 -n dev
```
Open http://localhost:8080 to see NGINX running.

### 8. Destroy when done
```bash
terraform destroy
```

## Assumptions and Design Decisions

### Flat file structure over nested modules
Initially considered organizing code into separate module folders (`modules/vpc`, `modules/eks`, etc.), but ultimately decided a flat file structure (`vpc.tf`, `eks.tf`, `k8s.tf`) is cleaner and more readable for a single-environment challenge. Nested modules added complexity that is only justified when building reusable infrastructure called from multiple places.

### IAM permissions are intentionally broad
The IAM user used for this challenge has broad permissions for simplicity. In production, these would be scoped to least-privilege using custom policies specific to each resource type.

### S3 backend not configured
Terraform state is stored locally. In prod, state should be stored remotely in S3 with DynamoDB for state locking to enable team collaboration and prevent concurrent modifications.

### Two public subnets required
Initially provisioned a single public subnet ( as outlined in pdf with challenge ), but EKS requires subnets in at least two different availability zones — encountered this as an error during apply and corrected by adding a second subnet in `us-east-1b`.

### ConfigMap usage
A ConfigMap named `nginx-config` is created with environment metadata (`environment=dev`, `app_name=nginx-demo`) and injected into the NGINX container via `env_from`. NGINX itself doesn't consume these variables — this demonstrates the pattern of separating configuration from application code. In a real scenario these values would be consumed by an application reading environment variables.

### ArgoCD and Terraform separation of concerns
ArgoCD is deployed via the Helm provider but does not manage the NGINX deployment. In production, the clean separation would be: Terraform manages infrastructure (VPC, EKS, ArgoCD itself), and ArgoCD manages applications (NGINX manifests in a Git repo). Having both Terraform and ArgoCD manage the same resources would cause conflicts.

## What I learned
- EKS requires subnets in at least two availability zones even for a single-node cluster
- The Kubernetes and Helm providers depend on EKS outputs — Terraform's dependency graph handles ordering automatically
- Got a chance to actually use helm provider 
- `terraform destroy` cleanly removed all provisioned resources, so it is safe to spin up and tear down for testing