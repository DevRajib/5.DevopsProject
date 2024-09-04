# 3-Tier Kubernetes Deployment: React, DRF, Database on EKS with Traefik SSL

This guide explains how to deploy a 3-tier architecture (React frontend, DRF backend, and PostgreSQL database) to an EKS cluster with Traefik for SSL/HTTPS and Docker Hub for image storage.

## Prerequisites

1. **AWS CLI** and `kubectl` installed and configured.
2. `eksctl` for managing EKS clusters.
3. **AWS IAM authenticator** for Kubernetes.
4. **Docker** installed and configured (for Docker Hub authentication).
5. **Helm** installed for managing Kubernetes charts.

## Directory Structure

```bash
k8s-deployment/
├── backend/                     # Django DRF Backend
│   ├── Dockerfile
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   └── secret.yaml
│   └── src/                     # Your DRF project source code
├── frontend/                    # React Frontend
│   ├── Dockerfile
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   └── public/                  # React public assets
│   └── src/                     # React source code
├── database/
│   ├── k8s/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   └── persistentvolume.yaml
├── traefik/
│   ├── values.yaml              # Traefik Helm values configuration
│   └── k8s/
│       └── ingress.yaml         # Ingress configuration
└── eks-setup/
    └── cluster.yaml             # eksctl cluster config file



