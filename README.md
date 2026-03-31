# kube_manifest Repository

This repository contains Kubernetes manifests for deploying the Frost App on EKS using GitOps principles with ArgoCD.

## 📦 Project Overview

This project is part of a **complete CI/CD and GitOps pipeline** across three repositories:

### 🔗 Project Repositories

➡️ **[App Code](https://github.com/Ayoub-So/AppCode)** - Application source code and Dockerfile  
➡️ **[Terraform Code](https://github.com/Ayoub-So/terraform_eks)** - Infrastructure as Code (EKS cluster setup)  
➡️ **[Manifest Repo](https://github.com/Ayoub-So/kube_manifest)** - Kubernetes manifests (this repository)  

## 📋 Repository Contents

This repository contains the following Kubernetes manifests:

### Core Manifests

- **`deployment.yaml`** - Frost App deployment configuration
  - Container image reference (auto-updated by CI/CD)
  - Resource requests/limits
  - Health checks (liveness & readiness probes)
  - Environment variables

- **`service.yaml`** - Kubernetes Service
  - LoadBalancer type for external access
  - Port mapping (80 → 80)
  - Service discovery configuration

## 🔄 GitOps Workflow

```
1. Developer pushes code to AppCode repo
   ↓
2. GitHub Actions triggers:
   - Builds Docker image
   - Pushes to AWS ECR
   - Updates kube_manifest/deployment.yaml with new image tag
   ↓
3. Changes pushed to kube_manifest repo (this repo)
   ↓
4. ArgoCD detects changes:
   - Polls every 3 minutes (or webhook triggered)
   - Compares Git state with cluster state
   ↓
5. ArgoCD syncs:
   - Creates/updates Kubernetes resources
   - New pods launched with latest image
   ↓
6. App deployed! ✅
```

## 🚀 Quick Start

### Prerequisites

- EKS cluster running (created via Terraform repo)
- kubectl configured and connected to cluster
- ArgoCD installed on the cluster

### Deploy Application

```bash
# Manual deployment (without GitOps)
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl get services
```

### Using ArgoCD (Recommended)

```bash
# Apply ArgoCD Application manifest (from main project repo)
kubectl apply -f argocd/argocd-application.yaml

# Check ArgoCD status
kubectl get application -n argocd frost-app

# View ArgoCD UI
kubectl port-forward -n argocd svc/argocd-server 8080:443
# Visit: https://localhost:8080
```

## 📝 Manifest Details

### Deployment Configuration

The `deployment.yaml` includes:

```yaml
- Name: frost-app
- Replicas: 2 (for high availability)
- Container port: 80
- Image: Automatically updated by GitHub Actions
- Health checks:
  - Liveness probe: /health endpoint
  - Readiness probe: /health endpoint
- Resource limits:
  - CPU: 500m (limit), 100m (request)
  - Memory: 512Mi (limit), 128Mi (request)
```

### Service Configuration

The `service.yaml` provides:

```yaml
- Type: LoadBalancer (external access via AWS ELB)
- Port mapping: 80:80
- Selector: app=frost-app
```

## 🔐 Access the Application

### Get Service Endpoint

```bash
# Get external IP/DNS
kubectl get service frost-app

# Service will be exposed at: http://<EXTERNAL-IP>
```

### Health Check

```bash
# Test the application
curl http://<EXTERNAL-IP>/health

# Expected response: 200 OK
```

## 📊 Image Updates

The image tag is automatically updated by GitHub Actions CI/CD pipeline:

```bash
# GitHub Actions updates this line:
image: <ECR_REGISTRY>/devops:<GIT_SHA>

# Example:
image: 566074325882.dkr.ecr.us-east-1.amazonaws.com/devops:abc123def456
```

**Never manually edit the image tag** - let the CI/CD pipeline handle it.

## 🔧 Manual Update Process

If you need to manually update the image:

```bash
# Edit the deployment
kubectl set image deployment/frost-app frost-app=<NEW_IMAGE_TAG>

# For GitOps, update deployment.yaml instead
sed -i 's|image: .*|image: <NEW_IMAGE_TAG>|g' deployment.yaml
git add deployment.yaml
git commit -m "Update image to <NEW_IMAGE_TAG>"
git push origin main
```

## 🎯 Best Practices

### DO ✅

- ✅ Use GitOps - all changes through Git
- ✅ Keep manifests in sync with infrastructure
- ✅ Use version control for audit trail
- ✅ Test changes before pushing
- ✅ Use meaningful commit messages

### DON'T ❌

- ❌ Don't manually edit running pods
- ❌ Don't bypass ArgoCD for deployments
- ❌ Don't commit sensitive data (use Secrets)
- ❌ Don't modify image tags manually
- ❌ Don't delete manifests without coordination

## 🔄 Continuous Deployment Flow

### Automatic Updates

```
AppCode commit → GitHub Actions → Build & Push → Update kube_manifest → ArgoCD → EKS
                                                                           (3min poll)
```

### ArgoCD Sync Modes

**Automatic (Current Setup):**
- ArgoCD automatically syncs every 3 minutes
- Prunes old resources
- Self-heals cluster drift

**Manual Sync:**
```bash
# If using manual sync
argocd app sync frost-app
```

## 📈 Monitoring

### Check Deployment Status

```bash
# Watch deployment rollout
kubectl rollout status deployment/frost-app

# View pod logs
kubectl logs -l app=frost-app -f

# Describe deployment
kubectl describe deployment frost-app
```

### ArgoCD Monitoring

```bash
# Check ArgoCD Application status
kubectl get application frost-app -n argocd
kubectl describe application frost-app -n argocd

# View ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller -f
```

## 🐛 Troubleshooting

### Application not deploying

```bash
# Check pod status
kubectl get pods
kubectl describe pod <POD_NAME>

# Check events
kubectl get events

# Check ArgoCD sync status
kubectl get application frost-app -n argocd
```

### Image pull errors

```bash
# Verify ECR credentials
kubectl get secrets
kubectl get secret regcred -o yaml

# Check image exists in ECR
aws ecr describe-images --repository-name devops --region us-east-1
```

### Service not accessible

```bash
# Verify service exists
kubectl get svc frost-app

# Check endpoints
kubectl get endpoints frost-app

# Verify security groups allow traffic
aws ec2 describe-security-groups --group-ids <SG_ID>
```

## 📚 Related Documentation

- [ArgoCD Setup Guide](https://github.com/Ayoub-So/ArgoCD_setup)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)

## 👤 Author

Ayoub Soussi  
Email: ayoubsoussi.2001@gmail.com

## 📄 License

This project is part of the CI/CD EKS with GitOps on AWS project.

---

**Last Updated:** March 31, 2026  
**Status:** Active and monitoring
