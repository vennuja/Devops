# 1-1 For which reason is it better to run the container with a flag -e to give the environment variables rather than put them directly in the Dockerfile?
  To avoid hardcoding sensitive data in version controlled files, environment variables are more secure and flexible.

# 1-2 Why do we need a volume to be attached to our postgres container?
  So that data is saved outside the container and is not lost when the container is deleted.

# 1-3 Document your database container essentials: commands and Dockerfile.
  docker build -t tp1-database .
docker run -d \
  --name database \
  --network app-network \
  -e POSTGRES_DB=db \
  -e POSTGRES_USER=usr \
  -e POSTGRES_PASSWORD=pwd \
  -v $(pwd)/database/pgdata:/var/lib/postgresql/data \
  tp1-database

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

# Pushing the images to dockerhub
  docker push venujadesilva/tp1-database:1.0
  docker push venujadesilva/tp1-backend:1.0
  docker push venujadesilva/tp1-http:1.0

# 1-10 Why do we put our images into an online repo?
  So that other people or systems can pull and run the same container anywhere, it makes it portable, shareable, and ready for deployment on another machine or server.






