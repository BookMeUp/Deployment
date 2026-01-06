## Prerequisites
```bash
# 1. Install Chocolatey (if not installed)
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# 2. Install Docker Desktop (manual download required)
# Download from: https://www.docker.com/products/docker-desktop/

# 3. Install Minikube
choco install minikube -y

# 4. Install kubectl
choco install kubernetes-cli -y

# 5. Install Helm (Kubernetes package manager)
choco install kubernetes-helm -y

# 6. Add Helm repositories
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```


## Deployment with Helm (Recommended)

```bash
# 1. Start Docker Desktop (from Start Menu)

# 2. Create Minikube cluster
minikube start --driver=docker --cpus=4 --memory=4096

# 3. Verify cluster is running
kubectl cluster-info
kubectl get nodes

# 4. Enable Ingress addon
minikube addons enable ingress
kubectl get pods -n ingress-nginx  # Wait for Ingress controller pods to be Running (1-2 minutes)

# 5. Install Slotify (Application + Rancher + Monitoring - all in one!)
helm install slotify ./helm -n slotify --create-namespace # From Deployment folder

# 6. Wait for all pods to be ready (2-4 minutes)
kubectl get pods -n slotify           # Watch application pods
kubectl get pods -n cattle-system     # Watch Rancher pod
kubectl get pods -n monitoring        # Watch monitoring pods

# 7. Start tunnel (keep this terminal open)
minikube tunnel
```


## Accessing the Application

```bash
# Application
Base URL: http://127.0.0.1

# Adminer (Database Management UI)
URL: http://127.0.0.1:8081
Login: System=PostgreSQL, Server=postgres, Username=user, Password=password, Database=slotify

# Rancher (Kubernetes Management UI)
URL: https://127.0.0.1:8443
Username: admin
Password: admin (set CATTLE_BOOTSTRAP_PASSWORD in values.yaml)

# Grafana (Monitoring Dashboard - if monitoring installed)
URL: http://127.0.0.1:3000
Login: admin / admin

# Prometheus (Metrics Database - if monitoring installed)
URL: http://127.0.0.1:9090

# Alertmanager (Alert Management)
URL: http://127.0.0.1:9093
```
