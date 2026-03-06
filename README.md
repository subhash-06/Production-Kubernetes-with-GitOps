# Production Kubernetes with GitOps

Enterprise-grade Kubernetes deployment demonstrating GitOps principles with ArgoCD on Azure Kubernetes Service (AKS).

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![ArgoCD](https://img.shields.io/badge/argocd-EF7B4D.svg?style=for-the-badge&logo=argo&logoColor=white)

## 🎯 Project Overview

A production-ready Kubernetes platform implementing modern DevOps best practices including GitOps automation, multi-environment isolation, resource governance, and comprehensive observability.

### Key Features
- **GitOps Continuous Deployment** - Automated sync from GitHub to cluster via ArgoCD
- **Multi-Environment Architecture** - Isolated dev/prod environments with different resource allocations
- **Path-Based Ingress Routing** - Single LoadBalancer serving multiple environments
- **Resource Governance** - Kubernetes quotas preventing resource contention
- **High Availability** - Production environment with 3 replicas for fault tolerance
- **Observability Stack** - Prometheus metrics collection with Grafana dashboards

## 🏗️ Architecture

```
                                    Internet
                                       ↓
                            [Azure Load Balancer]
                                       ↓
                        [NGINX Ingress Controller]
                                       ↓
                 ┌─────────────────────┼─────────────────────┐
                 ↓                     ↓                     ↓
           / (default)             /dev (dev)           /prod (prod)
           2 replicas              2 replicas            3 replicas
           No quotas               CPU: 500m             CPU: 2000m
                                   MEM: 1Gi              MEM: 4Gi
                                   Pods: 5               Pods: 20

                                       ↓
                          [Prometheus + Grafana]
                    (Accessible via kubectl port-forward)
```

## 📂 Repository Structure

```
Production-Kubernetes-with-GitOps/
├── app/
│   └── 2048/                      # Main application
│       ├── deployment.yaml        # 2 replicas, standard resources
│       ├── service.yaml           # ClusterIP service
│       └── ingress.yaml           # Root path (/)
│
└── environments/
    ├── dev/                       # Development environment
    │   ├── namespace.yaml         # Dev namespace
    │   ├── resourcequota.yaml     # Limited: 500m CPU, 1Gi RAM, 5 pods
    │   ├── deployment.yaml        # 2 replicas, minimal resources
    │   ├── service.yaml           # ClusterIP service
    │   └── ingress.yaml           # /dev path with rewrite
    │
    └── prod/                      # Production environment
        ├── namespace.yaml         # Prod namespace
        ├── resourcequota.yaml     # Full: 2000m CPU, 4Gi RAM, 20 pods
        ├── deployment.yaml        # 3 replicas, production resources
        ├── service.yaml           # ClusterIP service
        └── ingress.yaml           # /prod path with rewrite
```

## 🚀 Live Demo

### Application Endpoints
- **Default Environment**: http://INGRESS_IP/
- **Dev Environment**: http://INGRESS_IP/dev
- **Prod Environment**: http://INGRESS_IP/prod

### ArgoCD Dashboard
- **URL**: http://LOADBALANCER_IP
- **Username**: admin
- *(Password available via: `kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`)*

### Grafana Monitoring
Following security best practices, Grafana is not exposed to public internet. Access via secure port-forward:

```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Then open: http://localhost:3000
- **Username**: admin
- **Password**: `kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d`

## 🛠️ Technologies & Tools

| Technology | Purpose | Version |
|------------|---------|---------|
| **Kubernetes (AKS)** | Container orchestration | 1.28+ |
| **ArgoCD** | GitOps continuous deployment | Latest |
| **NGINX Ingress** | Layer 7 load balancing | Latest |
| **Prometheus** | Metrics collection | Latest |
| **Grafana** | Metrics visualization | Latest |
| **Helm** | Kubernetes package manager | 3.x |
| **Git** | Source control | Latest |

## 📊 Environment Comparison

| Feature | Default | Dev | Prod |
|---------|---------|-----|------|
| **Replicas** | 2 | 2 | 3 |
| **CPU Request** | 100m | 100m | 500m |
| **CPU Limit** | 200m | 200m | 1000m |
| **Memory Request** | 128Mi | 128Mi | 512Mi |
| **Memory Limit** | 256Mi | 256Mi | 1Gi |
| **Max Pods (Quota)** | - | 5 | 20 |
| **Total CPU (Quota)** | - | 500m | 2000m |
| **Total Memory (Quota)** | - | 1Gi | 4Gi |
| **Public Path** | / | /dev | /prod |

## 🔄 GitOps Workflow

```
Developer → Git Push → GitHub
                         ↓
                    ArgoCD (polls every 3min)
                         ↓
                    Detects changes
                         ↓
                    Syncs to cluster
                         ↓
                    Kubernetes applies manifests
                         ↓
                    Rolling update (zero downtime)
```

### Example: Scaling Dev Environment

```bash
# 1. Update replica count
sed -i 's/replicas: 2/replicas: 3/' environments/dev/deployment.yaml

# 2. Commit and push
git add environments/dev/deployment.yaml
git commit -m "Scale dev to 3 replicas"
git push origin main

# 3. ArgoCD automatically detects and deploys (within 3 minutes)
# 4. Verify
kubectl get pods -n dev
```


## 🚦 Quick Start

### Prerequisites
- Azure account with AKS access
- kubectl configured
- Git installed

### Deployment

```bash
# 1. Clone repository
git clone https://github.com/subhash-06/Production-Kubernetes-with-GitOps.git
cd Production-Kubernetes-with-GitOps

# 2. Install ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. Install NGINX Ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace

# 4. Create ArgoCD Applications
# Via ArgoCD UI or kubectl apply -f argocd-apps.yaml

# 5. Install Monitoring (Optional)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

## 📈 Monitoring & Observability

### Accessing Grafana Dashboards

```bash
# Port-forward Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Get credentials
echo "Username: admin"
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d
```

### Pre-built Dashboards
- **Kubernetes / Compute Resources / Cluster** - Overall cluster metrics
- **Kubernetes / Compute Resources / Namespace (Pods)** - Per-namespace resource usage
- **Kubernetes / Compute Resources / Pod** - Individual pod metrics


## 📝 Future Enhancements

- [ ] Add Horizontal Pod Autoscaling (HPA)
- [ ] Implement cert-manager for SSL/TLS
- [ ] Add custom Prometheus alerts
- [ ] Implement network policies
- [ ] Add Vault for secrets management
- [ ] Implement canary deployments
- [ ] Add service mesh (Istio/Linkerd)

