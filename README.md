----------------------------------------
Running on local (using Docker Compose)
----------------------------------------
docker compose down                              # Stop and remove containers after changing code
docker compose -p bookmeup-dev up --build -d     # Rebuild with new code and run

----------------------------------------
Running on cluster (using Docker Swarm)
----------------------------------------
docker stack rm bookmeup                           # Remove current stack and stop app
docker stack deploy -c stack.yml bookmeup-local    # Deploy with local cluser name
docker stack deploy -c stack.yml bookmeup-play     # Deploy with Play with Docker name

Other commands:
    docker swarm init    # Init swarm on machine (run only once)
    docker stack ls      # View active stacks
    docker service ls    # View active services

--------
PORTS
--------
1. Main application (http://localhost:8000/)
2. Adminer (http://localhost:8080/)
    System:   PostgreSQL
    Server:   postgres
    Username: user
    Password: password
    Database: bookmeup
3. Portainer (http://localhost:9000/)
    Username: admin
    Password: admin.portainer
