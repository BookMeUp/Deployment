------------------------------------
Play with Docker: https://labs.play-with-docker.com/
------------------------------------
1. Create 3 instances/nodes
2. Run the following command on node 1 (it will output a new command):
    docker swarm init --advertise-addr eth0
3. Run the previously generated command on nodes 2 and 3.
4. (Optional) Check node status:
    docker node ls
5. Clone stack.yml and kong.yml on node 1:
    git clone https://github.com/BookMeUp/Deployment.git
6. Deploy using classic Docker Swarm commands.
7. (Optional) Check which services run on the current node:
    docker ps
8. Open port :8000 on node 1 and copy public URL to Postman.

Useful ports:
- Main application :8000
- Adminer          :8080
- Portainer        :9000
