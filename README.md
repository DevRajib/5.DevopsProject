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

## 3. Build and Push Docker Images
Build your Docker images for the frontend and backend and push them to Docker Hub.


```
# Navigate to frontend directory and build the Docker image
cd frontend
docker build -t your-dockerhub-username/frontend:latest .
docker push your-dockerhub-username/frontend:latest

# Navigate to backend directory and build the Docker image
cd ../backend
docker build -t your-dockerhub-username/backend:latest .
docker push your-dockerhub-username/backend:latest

```

## 4. Create Kubernetes Manifests
Frontend (React)
Create a deployment and service YAML file in the frontend/k8s/ directory.

deployment.yaml


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: your-dockerhub-username/frontend:latest
        ports:
        - containerPort: 80

```

service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: ClusterIP
  ports:
    - port: 80
  selector:
    app: frontend

```

Backend (DRF)
Create a deployment, service, and secret YAML file in the backend/k8s/ directory.

deployment.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: your-dockerhub-username/backend:latest
        ports:
        - containerPort: 8000

```
service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  ports:
    - port: 8000
  selector:
    app: backend


```


## Database (e.g., PostgreSQL)
Create a deployment and service YAML file in the database/k8s/ directory.

deployment.yaml


```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: yourdb
        - name: POSTGRES_USER
          value: youruser
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password


```

## service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  type: ClusterIP
  ports:
    - port: 5432
  selector:
    app: postgres


```
persistentvolume.yaml

Configure persistent storage for the database.

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi



```
## 5. Deploy Resources to EKS
Apply all the Kubernetes manifests.

```
kubectl apply -f frontend/k8s/
kubectl apply -f backend/k8s/
kubectl apply -f database/k8s/
kubectl apply -f traefik/k8s/

```

## 6. Configure Traefik Ingress
Create an ingress YAML file to route traffic to your frontend and backend services.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traefik-ingress
  annotations:
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls.certresolver: myresolver
spec:
  rules:
    - host: decorationbd.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
    - host: api.decorationbd.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: backend
                port:
                  number: 8000
  tls:
    - hosts:
      - decorationbd.com
      - api.decorationbd.com


```
## 7. Verify Deployment
Ensure everything is running smoothly by checking the status of your pods, services, and ingress:


```
kubectl get pods
kubectl get services
kubectl get ingress


```


You should see the frontend served over https://decorationbd.com and the backend API served over https://api.decorationbd.com.

## Conclusion
This step-by-step guide covers setting up a 3-tier React + DRF + Database deployment on an EKS cluster using Traefik for SSL and Docker Hub for image storage. Be sure to replace placeholder values with your specific configurations and credentials.











































