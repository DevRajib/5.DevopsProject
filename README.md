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
```

## Step-by-Step Instructions
### 1. Set Up an EKS Cluster
Create a cluster using eksctl. Create a configuration file named cluster.yaml inside the eks-setup/ directory.


```apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: my-cluster
  region: us-west-2
  version: "1.24"

nodeGroups:
  - name: ng-1
    instanceType: t3.medium
    desiredCapacity: 3
```

Run the following command to create the cluster:

```
eksctl create cluster -f eks-setup/cluster.yaml

```

## 2. Install Traefik Using Helm
Add the Traefik Helm chart repository and install Traefik.


```
helm repo add traefik https://traefik.github.io/charts
helm repo update

helm install traefik traefik/traefik --namespace kube-system --values traefik/values.yaml

```

Your `values.yaml` file should enable ACME for SSL certificates.

```
additionalArguments:
  - "--entryPoints.web.address=:80"
  - "--entryPoints.websecure.address=:443"
  - "--certificatesResolvers.myresolver.acme.tlsChallenge=true"
  - "--certificatesResolvers.myresolver.acme.email=your-email@example.com"
  - "--certificatesResolvers.myresolver.acme.storage=/data/acme.json"

```





















