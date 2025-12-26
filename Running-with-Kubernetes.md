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
```

## First-Time Setup After Installation
```bash
# 1. Start Docker Desktop (from Start Menu)

# 2. Create Minikube cluster
minikube start --driver=docker --cpus=4 --memory=4096

# 3. Verify cluster is running
kubectl cluster-info
kubectl get nodes

# 4. Install Ingress addon
minikube addons enable ingress

# 5. Wait for Ingress controller (1-2 minutes)
kubectl get pods -n ingress-nginx -w
# Press Ctrl+C when ingress-nginx-controller shows Running

# 6. Patch Ingress controller to LoadBalancer (required for tunnel)
kubectl patch svc ingress-nginx-controller -n ingress-nginx -p @'
{"spec":{"type":"LoadBalancer"}}
'@
```

## Running the app with Kubernetes (minikube)
```bash
# 1. Start Minikube
minikube start

# 2. Deploy application (first time only)
cd C:\Users\vladu\Desktop\Development\Slotify\Deployment
kubectl apply -f kubernetes/base/

# 3. Wait for pods to be ready (30-60 seconds)
kubectl get pods -n slotify -w
# Press Ctrl+C when all show 1/1 Running

# 4. Start tunnel (keep this terminal open)
minikube tunnel
```

## Accessing the app
```bash
# Application
Base URL: http://127.0.0.1
Test: curl http://127.0.0.1/auth/health

# Adminer
minikube service adminer -n slotify  # In a separate terminal
# Login: System: PostgreSQL, Server: postgres, User: user, Password: password, DB: slotify
```

## Methods to stop the app
### 1. Quick stop (Everything Preserved)
```bash
# Stop tunnel (Ctrl+C in tunnel terminal)
minikube stop

# Resume later (CPU/memory/addons/deployments all preserved!)
minikube start
minikube tunnel
```

### 2. Delete App Only (Minikube Preserved)
```bash
# Delete apps but keep Minikube cluster
kubectl delete namespace slotify

# Redeploy
kubectl apply -f kubernetes/base/
minikube tunnel
```

### 3. Complete Reset (Everything Lost)
```bash
# Delete entire cluster
minikube delete

# Recreate from scratch (MUST specify settings again!)
minikube start --driver=docker --cpus=4 --memory=4096
minikube addons enable ingress
kubectl get pods -n ingress-nginx -w  # Wait for Running
kubectl patch svc ingress-nginx-controller -n ingress-nginx -p @'
{"spec":{"type":"LoadBalancer"}}
'@
kubectl apply -f kubernetes/base/
minikube tunnel
```

## Quick Health Check
```bash
# Check Docker
docker ps

# Check Minikube
minikube status

# Check kubectl
kubectl get nodes

# Check deployments
kubectl get all -n slotify

# Check current CPU/memory allocation
minikube ssh "nproc && free -m"
# Expected: 4 CPUs, ~3950 MB memory
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

**⚠️ Important:** Never commit secrets.yaml to Git in production! Use `.gitignore` or create secrets manually with:
```bash
kubectl create secret generic slotify-secrets -n slotify \
  --from-literal=POSTGRES_USER=user \
  --from-literal=POSTGRES_PASSWORD=password \
  --from-literal=POSTGRES_DB=slotify \
  --from-literal=JWT_SECRET_KEY=your-secret-key
```

## Other useful commands
```bash
# Check logs from one of the failed pods
kubectl logs logic-service-7f4bf8fc8-ljb4c -n slotify --tail=50
```
