----------------------------------------
Running on local (using Docker Compose)
----------------------------------------
docker compose down           # Stop and remove containers after changing code
docker compose up --build -d  # Rebuild with new code and run

----------------------------------------
Running on cluster (using Docker Swarm)
----------------------------------------
1. Remove current stack (stops running app):
    docker stack rm bookmeup

2. Rebuild and push iamges for each modified service:
    cd C:\Users\alexn\Desktop\BookMeUp\Authentication-Service
    docker build -t alexnv67/auth-service .
    docker push alexnv67/auth-service

    cd C:\Users\alexn\Desktop\BookMeUp\Business-Logic-Service
    docker build -t alexnv67/logic-service .
    docker push alexnv67/logic-service

    cd C:\Users\alexn\Desktop\BookMeUp\Database-Interaction-Service
    docker build -t alexnv67/db-service .
    docker push alexnv67/db-service

3. Deploy the stack:
    docker swarm init (only once per machine)
    docker stack deploy -c stack.yml bookmeup

4. Check status at anytime (optional):
    docker stack ls
    docker service ls

------------------------------------
Adminer: http://localhost:8080/
------------------------------------
System:   PostgreSQL
Server:   postgres
Username: user
Password: password
Database: bookmeup

------------------------------------
Portainer: http://localhost:9000/
------------------------------------
Username: admin
Password: admin.portainer
