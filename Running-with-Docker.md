### Development Environment

Run the application by building images from local source code:
```bash
docker compose -p slotify-dev up --build -d     # Rebuild with new code and run
docker compose -p slotify-dev down              # Stop and remove containers (images and volumes stay unchanged)
docker compose -p slotify-dev down -v           # Stop and remove containers and volumes (images stay unchanged)
```

----------------------------------------

### Test Environment

Run the application using pre-built images from Docker Hub (no source code needed):
```bash
docker compose -f docker-compose.test.yml -p slotify-test up -d     # Start all services
docker compose -f docker-compose.test.yml -p slotify-test down      # Stop all services
```

----------------------------------------

### Production Environment (Swarm)

Deploy to Docker Swarm cluster for production:
```bash
docker swarm init                                  # Init swarm on machine (run only once)
docker stack deploy -c stack.yml slotify-local    # Deploy stack
docker stack rm slotify-local                     # Remove stack and stop app
```

Other useful commands:
```bash
docker stack ls      # View active stacks
docker service ls    # View active services
```

----------------------------------------

### PORTS

- Main application (http://localhost:8000/)
- Adminer (http://localhost:8080/)
    - System:   PostgreSQL
    - Server:   postgres
    - Username: user
    - Password: password
    - Database: slotify
- Portainer (http://localhost:9000/)
    - Username: admin
    - Password: admin.portainer
- Prometheus (http://localhost:9090/)
    - Test query: flask_http_request_total
- Grafana (http://localhost:3000/)
    - Username: admin
    - Password: admin
