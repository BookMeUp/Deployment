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


## First-Time Application Setup
```bash
# 1. Start Docker Desktop (from Start Menu)

# 2. Create Minikube cluster
minikube start --driver=docker --cpus=4 --memory=4096

# 3. Verify cluster is running
kubectl cluster-info
kubectl get nodes

# 4. Ingress Setup ('ingress-nginx' namespace)
minikube addons enable ingress      # Install Ingress addon
kubectl get pods -n ingress-nginx   # Wait for Ingress controller (1-2 minutes).
kubectl patch svc ingress-nginx-controller -n ingress-nginx -p '{\"spec\":{\"type\":\"LoadBalancer\"}}' # Patch to LoadBalancer

# 5. Rancher Setup ('cattle-system' namespace)
kubectl apply -f kubernetes/rancher/        # Deploy Rancher (Kubernetes management UI)
kubectl get pods -n cattle-system           # Wait for Rancher pod (1-2 minutes)

# 6. Monitoring Setup ('monitoring' namespace)
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace --set prometheus.prometheusSpec.serviceMonitorSelectorNilUsesHelmValues=false --set grafana.adminPassword=admin   # Install kube-prometheus-stack
kubectl get pods -n monitoring   # Wait for monitoring pods (1-2 minutes)
kubectl patch svc kube-prometheus-stack-grafana -n monitoring -p '{\"spec\":{\"type\":\"LoadBalancer\"}}' # Patch to LoadBalancer
kubectl patch svc kube-prometheus-stack-prometheus -n monitoring -p '{\"spec\":{\"type\":\"LoadBalancer\"}}' # Patch to LoadBalancer
kubectl edit svc kube-prometheus-stack-grafana -n monitoring # Find: port: 80 → Change to: port: 3000 → Save and exit

# 7. Application Deployment ('slotify' namespace)
kubectl apply -f kubernetes/base/namespace.yaml  # From Deployment folder
kubectl apply -f kubernetes/base/                # From Deployment folder
kubectl get pods -n slotify                      # Wait for pods to be ready (30-60 seconds)

# 8. Start tunnel (keep this terminal open)
minikube tunnel
```


## Accessing the app
```bash
# Application
Base URL: http://127.0.0.1

# Adminer (Database Management)
URL: http://127.0.0.1:8081
# Login: System: PostgreSQL, Server: postgres, User: user, Password: password, DB: slotify

# Rancher (Kubernetes Management)
URL: https://127.0.0.1:8443
# Username: admin, Password: admin (changed on first login: Og85gVv4INdGRnCn)

# Grafana (Monitoring Dashboard - if monitoring installed)
URL: http://127.0.0.1:3000
# Login: admin / admin

# Prometheus (Metrics Database - if monitoring installed)
URL: http://127.0.0.1:9090
```


## Update Kubernetes Deployment After Pushing New Image
```bash
# Force Kubernetes to pull the new image and restart pods
kubectl rollout restart deployment/auth-service -n slotify

# Watch pods restart
kubectl get pods -n slotify -w

# Check rollout status
kubectl rollout status deployment/auth-service -n slotify

## Rollback if needed
kubectl rollout undo deployment/auth-service -n slotify
```


## Update Secrets (Passwords, JWT Keys, etc.)
```bash
# 1. Edit the secrets file
code kubernetes/base/secrets.yaml

# 2. Apply the updated secret
kubectl apply -f kubernetes/base/secrets.yaml

# 3. Restart all services that use secrets (pods don't auto-reload)
kubectl rollout restart deployment/auth-service -n slotify
kubectl rollout restart deployment/logic-service -n slotify
kubectl rollout restart deployment/db-service -n slotify
kubectl rollout restart deployment/postgres -n slotify

# 4. Verify all pods are running
kubectl get pods -n slotify
```


## Useful commands
```bash
minikube start                   # Start cluster
minikube tunnel                  # Start tunnel (keep terminal open)
minikube status
minikube ssh "nproc && free -m"  # Check current CPU/memory allocation
minikube stop                    # Stop cluster
minikube delete                  # Delete cluster

kubectl cluster-info
kubectl get nodes
kubectl get all -n {namespace}        # List pods, services, deployments and replica sets
kubectl get pods -n {namespace}       # List pods
kubectl get svc -n {namespace}        # List services
kubectl delete namespace {namespace}  # Delete namespace (doesn't affect cluster)

kubectl logs logic-service-7f4bf8fc8-ljb4c -n slotify --tail=50  # Check logs from one of the failed pods
```
