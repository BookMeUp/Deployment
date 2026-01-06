# Slotify

Microservices-based booking platform for service businesses.

## Microservices

### Authentication Service (Port 5001)
User authentication and JWT token management. Handles registration, login, and token generation with bcrypt password hashing.

### Business Logic Service (Port 5002)
Core business logic including smart time slot calculation, role-based access control, appointment conflict prevention, and service/availability management.

### Database Interaction Service (Port 5003)
SQLAlchemy-based CRUD operations for all entities. Internal use only, not exposed via Ingress.

## Quick Start with Helm

```bash
# Prerequisites
choco install minikube kubernetes-cli kubernetes-helm -y
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Deploy
minikube start --driver=docker --cpus=4 --memory=4096
minikube addons enable ingress
cd Deployment
helm install slotify ./helm -n slotify --create-namespace
minikube tunnel  # Keep running
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
