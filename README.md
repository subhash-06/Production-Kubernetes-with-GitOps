# Kubernetes GitOps Demo with ArgoCD

Production-ready Kubernetes deployment using GitOps principles with ArgoCD on Oracle Cloud.

## Features
- GitOps automated deployments with ArgoCD
- Multi-environment setup (dev/prod)
- Resource quotas for environment isolation
- NGINX Ingress for external access
- Automated sync from GitHub

## Architecture
- **Cluster:** Oracle Kubernetes Engine (OKE)
- **GitOps:** ArgoCD
- **App:** 2048 Game (demo application)
- **Environments:** Dev (limited resources) + Prod (full resources)

## Project Structure
```
k8s-gitops-demo/
├── app/2048/              # Application manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── environments/          # Environment configs
    ├── dev/              # Dev namespace with small quota
    └── prod/             # Prod namespace with larger quota
```

## Setup
Deployed via ArgoCD - any push to main branch automatically syncs to cluster.

## Live Demo
Coming soon...
