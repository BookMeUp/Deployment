# Slotify

Microservices-based booking platform for service businesses built with a modern microservices architecture.

## Architecture

This is a **meta-repository** that uses Git submodules to organize three independent microservices. Each service is maintained in its own repository and can be developed independently.

### Repository Structure

```
slotify-backend/                    # Meta-repository
├── auth-service/                   # Submodule: Authentication Service
├── business-service/               # Submodule: Business Logic Service
├── database-service/               # Submodule: Database Service
└── deployment/                     # Deployment configurations
    ├── docker/                     # Docker Compose files
    ├── kubernetes/                 # Kubernetes manifests
    └── helm/                       # Helm charts
```

### Clone the Repository

```bash
# Clone with all submodules
git clone --recurse-submodules https://github.com/BookMeUp/slotify-backend.git

# Or if already cloned without submodules
git submodule update --init --recursive
```

## Microservices

### Authentication Service (Port 5001)
User authentication and JWT token management. Handles registration, login, and token generation with bcrypt password hashing.

📖 **[Full Documentation](auth-service/README.md)**

### Business Logic Service (Port 5002)
Core business logic including smart time slot calculation, role-based access control, appointment conflict prevention, and service/availability management.

📖 **[Full Documentation](business-service/README.md)**

### Database Interaction Service (Port 5003)
SQLAlchemy-based CRUD operations for all entities. Internal use only, not exposed via Ingress.

📖 **[Full Documentation](database-service/README.md)**

## Deployment Options

### Option 1: Docker Compose (Recommended for Local Development)

```bash
cd deployment/docker
docker-compose up -d
```

📖 **[Docker Deployment Guide](deployment/docker/Running-with-Docker.md)**

### Option 2: Helm (Recommended for Production)

```bash
cd deployment
helm install slotify ./helm -n slotify --create-namespace
```

📖 **[Helm Deployment Guide](deployment/helm/Running-with-Helm.md)**

### Option 3: Kubernetes (Manual)

```bash
cd deployment
kubectl apply -f kubernetes/base/
```

📖 **[Kubernetes Deployment Guide](deployment/kubernetes/Running-with-Kubernetes.md)**

## Quick Start (Helm)

```bash
# Prerequisites
choco install minikube kubernetes-cli kubernetes-helm -y
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Deploy
minikube start --driver=docker --cpus=4 --memory=4096
minikube addons enable ingress
cd deployment
helm install slotify ./helm -n slotify --create-namespace
minikube tunnel  # Keep running
```

## Development Workflow

### Working on Individual Services

When developing a specific service, open the **standalone service repository** directly:

```bash
# Clone and work on auth-service independently
git clone https://github.com/BookMeUp/slotify-auth-service.git
cd slotify-auth-service
# Make changes, commit, push
```

This is recommended because:
- Simpler git operations
- CI/CD workflows work naturally
- Better IDE support

### Working on Deployment Configurations

Open the **meta-repository** when:
- Updating deployment configs
- Working across multiple services
- Testing full system integration

```bash
cd slotify-backend
# Make changes to deployment/
git add deployment/
git commit -m "Update deployment configs"
git push
```

### Updating Submodules

After pushing changes to a service repository, update the meta-repo:

```bash
cd slotify-backend
git submodule update --remote auth-service
git add auth-service
git commit -m "Update auth-service to latest"
git push
```

Or update all submodules:

```bash
git submodule update --remote
git add .
git commit -m "Update all submodules"
git push
```

## Deployment Options

- **Helm**: [Deployment/helm/Running-with-Helm.md](Deployment/helm/Running-with-Helm.md) - Single command, includes Rancher & Monitoring
- **Kubernetes**: [Deployment/kubernetes/Running-with-Kubernetes.md](Deployment/kubernetes/Running-with-Kubernetes.md) - Manual `kubectl apply`

## Access Services

| Service | URL | Credentials |
|---------|-----|-------------|
| **API** | http://127.0.0.1 | JWT tokens |
| **Adminer** | http://127.0.0.1:8081 | postgres/user/password/slotify |
| **Rancher** | https://127.0.0.1:8443 | admin/admin |
| **Grafana** | http://127.0.0.1:3000 | admin/admin |

## API Examples

```bash
# Register & Login
curl -X POST http://127.0.0.1/auth/register -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com","password":"pass123","role":"customer"}'

curl -X POST http://127.0.0.1/auth/login -H "Content-Type: application/json" \
  -d '{"email":"john@example.com","password":"pass123"}'
# Save token: export TOKEN="<access_token>"

# Create Service (Staff)
curl -X POST http://127.0.0.1/services -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" -d '{"name":"Haircut","price":"80 RON","duration":30}'

# Add Availability (Staff)
curl -X POST http://127.0.0.1/availability -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"date":"2026-01-15","start_time":"09:00","end_time":"17:00"}'

# Check Available Slots
curl "http://127.0.0.1/available-timeslots?date=2026-01-15&service_id=1"

# Book Appointment (Customer)
curl -X POST http://127.0.0.1/appointments -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"service_id":1,"date":"2026-01-15","time":"10:00"}'

# View Appointments
curl http://127.0.0.1/appointments -H "Authorization: Bearer $TOKEN"
```

## Technology Stack

- **Backend**: Python 3.13 + Flask 2.3.2
- **Authentication**: Flask-JWT-Extended + bcrypt
- **Database**: PostgreSQL + SQLAlchemy ORM
- **Monitoring**: Prometheus + Grafana
- **API Gateway**: Kong (in infrastructure)
- **Orchestration**: Kubernetes + Helm
- **Container Registry**: Docker Hub

## Project Links

- **Meta Repository**: https://github.com/BookMeUp/slotify-backend
- **Auth Service**: https://github.com/BookMeUp/slotify-auth-service
- **Business Service**: https://github.com/BookMeUp/slotify-business-service
- **Database Service**: https://github.com/BookMeUp/slotify-database-service

## Contributing

1. Work on individual service repositories for feature development
2. Open the meta-repo for deployment configuration changes
3. Each service has its own CI/CD pipeline
4. Update submodule references after merging service changes

## License

Part of the Slotify platform.
