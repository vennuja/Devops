# 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?
  To avoid hardcoding sensitive data in version controlled files, environment variables are more secure and flexible.

# 1-2 Why do we need a volume to be attached to our postgres container?
  So that data is saved outside the container and is not lost when the container is deleted.

# 1-3 Document your database container essentials: commands and Dockerfile.
docker build -t tp1-database

docker run -d \
  --name database \
  --network app-network \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=usr \
  -e POSTGRES_PASSWORD=pwd \
  -v $(pwd)/database/pgdata:/var/lib/postgresql/data \
  tp1-database

  Dockerfile:
  
  FROM postgres:17.2-alpine

ENV POSTGRES_DB=db \
    POSTGRES_USER=usr \
    POSTGRES_PASSWORD=pwd

COPY initdb /docker-entrypoint-initdb.d/

# 1-4 Why do we need a multistage build? And explain each step of this dockerfile.
  It allows us to use a full JDK in one stage to compile the code and a lightweight JRE in the second stage to run it. This will keep the final Docker image small and clean.

# 1-5 Why do we need a reverse proxy?
  A reverse proxy acts like a middleman between users and the backend, so instead of users talking directly to the backend API, they send requests to the HTTP server (Apache), and it 
  forwards the requests to the backend.

# 1-6 Why is docker-compose so important?
 Its important beacuse it allows us to run multiple containers (like the backend, database, and web server) with just one command, instead of starting each container manually, i can 
 define everything in one file and Docker Compose handles the setup.

# 1-7 Document docker-compose most important commands.
  docker-compose up --build  - Rebuild and start all services if needed 
  docker-compose up - start all services
  docker-compose down - stop and remove all containers
  docker-compose ps - see all running services
  docker-compose start / stop

# 1-8 Document your docker-compose file.
  My docker-compose.yml defines three services: 
  database: A PostgreSQL container that builds from a custom Dockerfile, uses environment variables for setup, and stores data using a volume.
  backend: A Spring Boot API container that depends on the database and is exposed on port 8085.
  http: An Apache server container that acts as a reverse proxy to the backend and is exposed on port 8086.

# 1-9 Document your publication commands and published images in dockerhub.
Tagging the images

  docker tag tp1-database venujadesilva/tp1-database:1.0
  
  docker tag tp1-backend venujadesilva/tp1-backend:1.0
  
  docker tag tp1-http venujadesilva/tp1-http:1.0

Pushing the images to dockerhub

  docker push venujadesilva/tp1-database:1.0
  
  docker push venujadesilva/tp1-backend:1.0
  
  docker push venujadesilva/tp1-http:1.0

# 1-10 Why do we put our images into an online repo?
  So that other people or systems can pull and run the same container anywhere, it makes it portable, shareable, and ready for deployment on another machine or server.

# Link to the TP2 and TP3 github repository
I did the TP2 and TP3 on the corrected git hub repo, below is the link to that

https://github.com/vennuja/tp-devops-correction-docker

# 2-1 What are testcontainers?
They are Java libraries that help us run real Docker containers in our tests. For example, instead of mocking a database, it allows to spin up a real PostgreSQL container so the integration tests can interact with it.

# 2-2 For what purpose do we need to use secured variables ?
 We use secured variables to store sensitive information like dockerhub passwords or ssh keys and also to prevent exposing secrets in public repos.

# 2-3 Why did we put needs: build-and-test-backend on this job? Maybe try without this and you will see!
This makes sure the Docker images are only built if tests pass. When we remove it, Docker will build even if your tests fail, which can break production.

# 2-4 For what purpose do we need to push docker images?
So others like team members can pull the exact same container anywhere. This makes the application sharable, and also we can deploy it on any server without needing to rebuild.

# 3-1 Document your inventory and base commands
Inventory:
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: ./id_deploy_ansible
  children:
    prod:
      hosts:
        siyambalapitiya.desilva.takima.cloud:

Base commands:
Test server connectivity - ansible all -i inventories/setup.yml -m ping
Get Facts - ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
Remove apache2 from the server - ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become

# 3-2 Document your playbook
- hosts: all
  gather_facts: true
  become: true

  vars:
    ansible_python_interpreter: /usr/bin/python3

  roles:
    - docker      # installs Docker and requirements
    - network     # creates Docker network
    - database    # launches the database container
    - app         # launches the backend API container
    - proxy       # launches the http server (reverse proxy)


# 3-3 Document your docker_container tasks configuration.
Each role uses the docker_container module to launch a container.

App role - Deploys the Spring Boot backend application using venujadesilva/tp-devops-backend:latest. It connects to the database and exposes the internal application port.
roles/app/tasks/main.yml
- name: Launch backend container
  docker_container:
    name: app
    image: venujadesilva/tp-devops-backend:latest
    state: started
    pull: yes
    restart_policy: always
    env:
      DATABASE_HOST: database
      DATABASE_PASSWORD: pwd
    networks:
      - name: app-network

Proxy role - Launches the httpd container as a reverse proxy using venujadesilva/tp-devops-http-server:latest. It maps port 80 and forwards requests to the backend.
  roles/proxy/tasks/main.yml
  - name: Launch httpd proxy
  docker_container:
    name: http
    image: venujadesilva/tp-devops-http-server:latest
    state: started
    pull: yes
    restart_policy: always
    ports:
      - "80:80"
    networks:
      - name: app-network
    
Database role - Starts a PostgreSQL container with the necessary environment variables, including database name, user, and password. It uses the image venujadesilva/tp-devops-database:latest.
roles/database/tasks/main.yml
  - name: Launch database container
  docker_container:
    name: database
    image: venujadesilva/tp-devops-database:latest
    state: started
    pull: yes
    restart_policy: always
    env:
      POSTGRES_DB: db
      POSTGRES_USER: usr
      POSTGRES_PASSWORD: pwd

    networks:
      - name: app-network

# Is it really safe to deploy automatically every new image on the hub ? explain. What can I do to make it more secure?
Automatically deploying every new image from DockerHub is not very safe, as it can push buggy or untested code directly to production without review. This risks breaking the system if a mistake is made or if someone unintentionally pushes a faulty image. To make it more secure, deployments should only happen on official releases or protected branches, with automated tests, manual approvals, to ensure control and reliability.




