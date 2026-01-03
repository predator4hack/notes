
A comprehensive guide to essential Docker commands with practical examples and explanations.

---

## Container Management

### Running Containers

```bash
# Run a container (creates and starts)
docker run <image>
docker run nginx

# Run with a name
docker run --name my-nginx nginx

# Run in detached mode (background)
docker run -d nginx

# Run with port mapping (host:container)
docker run -p 8080:80 nginx

# Run with environment variables
docker run -e "DATABASE_URL=postgres://db" myapp

# Run with volume mount
docker run -v /host/path:/container/path nginx

# Run interactively with terminal
docker run -it ubuntu bash

# Run and remove container after exit
docker run --rm ubuntu echo "Hello"

# Run with resource limits
docker run --memory="512m" --cpus="1.5" myapp
```

**Intuitive Example:** Think of `docker run` like renting a fully furnished apartment. The `-d` flag means you rent it but don't live there yourself (it runs in the background), `-p` is like giving your friends a key to visit (port access), and `-v` is like bringing your own furniture (mounting your data).

### Container Lifecycle

```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Start a stopped container
docker start <container-id or name>

# Stop a running container
docker stop <container-id or name>

# Restart a container
docker restart <container-id or name>

# Pause a container
docker pause <container-id or name>

# Unpause a container
docker unpause <container-id or name>

# Kill a container (force stop)
docker kill <container-id or name>

# Remove a container
docker rm <container-id or name>

# Remove a running container (force)
docker rm -f <container-id or name>

# Remove all stopped containers
docker container prune
```

**Intuitive Example:** Container lifecycle is like a car: `start` turns it on, `stop` turns it off properly, `kill` is like pulling the ignition out, `pause` is like putting it in park with the engine running, and `rm` is selling the car.

### Interacting with Containers

```bash
# Execute command in running container
docker exec <container-id> <command>
docker exec my-container ls /app

# Execute interactive shell
docker exec -it <container-id> bash

# View container logs
docker logs <container-id>

# Follow logs (like tail -f)
docker logs -f <container-id>

# View last 100 lines of logs
docker logs --tail 100 <container-id>

# Attach to a running container
docker attach <container-id>

# Copy files from container to host
docker cp <container-id>:/path/to/file /host/path

# Copy files from host to container
docker cp /host/path <container-id>:/path/to/file

# View container resource usage
docker stats

# View container processes
docker top <container-id>

# Inspect container details
docker inspect <container-id>
```

**Intuitive Example:** `docker exec` is like calling someone in the apartment and asking them to do something, while `docker attach` is like actually going into the apartment yourself. `docker logs` is checking the security camera footage.

---

## Image Management

### Building Images

```bash
# Build image from Dockerfile
docker build -t myapp:latest .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp .

# Build without cache
docker build --no-cache -t myapp .

# Build for specific platform
docker build --platform linux/amd64 -t myapp .
```

**Intuitive Example:** Building a Docker image is like creating a blueprint for houses. Once you have the blueprint (`Dockerfile`), you can build identical houses (`containers`) from it anywhere.

### Managing Images

```bash
# List images
docker images

# List all images (including intermediate)
docker images -a

# Pull image from registry
docker pull nginx:latest

# Push image to registry
docker push myregistry/myapp:latest

# Tag an image
docker tag myapp:latest myapp:v1.0

# Remove an image
docker rmi <image-id or name>

# Remove all unused images
docker image prune

# Remove all images
docker image prune -a

# Search Docker Hub for images
docker search nginx

# View image history/layers
docker history <image-id>

# Inspect image details
docker inspect <image-id>

# Save image to tar file
docker save -o myapp.tar myapp:latest

# Load image from tar file
docker load -i myapp.tar
```

**Intuitive Example:** Docker images are like frozen meals - they're templates that don't change. When you run them (create containers), you're cooking the meal. You can share the recipe (push/pull), make copies (tag), or freeze new versions (build).

---

## Volume Management

```bash
# Create a volume
docker volume create my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Remove a volume
docker volume rm my-volume

# Remove all unused volumes
docker volume prune

# Run container with named volume
docker run -v my-volume:/app/data nginx

# Run container with bind mount
docker run -v /host/path:/container/path nginx

# Run with read-only volume
docker run -v my-volume:/app/data:ro nginx
```

**Intuitive Example:** Volumes are like external hard drives. Named volumes (`my-volume`) are managed by Docker like cloud storage, while bind mounts (`/host/path`) are like plugging in your own USB drive directly.

---

## Network Management

```bash
# List networks
docker network ls

# Create a network
docker network create my-network

# Create bridge network (default)
docker network create --driver bridge my-bridge

# Create with specific subnet
docker network create --subnet=172.18.0.0/16 my-network

# Connect container to network
docker network connect my-network <container-id>

# Disconnect container from network
docker network disconnect my-network <container-id>

# Run container on specific network
docker run --network my-network nginx

# Inspect network
docker network inspect my-network

# Remove network
docker network rm my-network

# Remove all unused networks
docker network prune
```

**Intuitive Example:** Docker networks are like apartment building hallways. Containers on the same network can talk to each other by name (like neighbors knocking on doors), while different networks are like separate buildings (isolated).

---

## Docker Compose

### Basic Commands

```bash
# Start services
docker-compose up

# Start in detached mode
docker-compose up -d

# Start specific service
docker-compose up <service-name>

# Build and start
docker-compose up --build

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# View running services
docker-compose ps

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Execute command in service
docker-compose exec <service> <command>

# Run one-off command
docker-compose run <service> <command>

# Scale services
docker-compose up --scale web=3

# Restart services
docker-compose restart

# Pull images for services
docker-compose pull
```

**Intuitive Example:** Docker Compose is like a building manager who knows how to set up an entire apartment complex from blueprints. Instead of manually starting each service, you describe everything in `docker-compose.yml` and let Compose handle the coordination.

---

## System Management

### Cleanup Commands

```bash
# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune

# Remove all unused volumes
docker volume prune

# Remove all unused networks
docker network prune

# Remove everything unused (containers, networks, images, build cache)
docker system prune

# Remove everything including volumes
docker system prune -a --volumes

# View disk usage
docker system df
```

**Intuitive Example:** `docker system prune` is like spring cleaning - it removes all the clutter you're not using anymore, freeing up disk space.

### System Information

```bash
# Show Docker version
docker version

# Show system-wide information
docker info

# View real-time events
docker events

# View events from last 10 minutes
docker events --since 10m
```

---

## Registry & Repository

```bash
# Login to Docker Hub
docker login

# Login to private registry
docker login myregistry.com

# Logout
docker logout

# Tag image for registry
docker tag myapp:latest myregistry.com/myapp:latest

# Push to registry
docker push myregistry.com/myapp:latest

# Pull from registry
docker pull myregistry.com/myapp:latest
```

**Intuitive Example:** Docker registries are like app stores. Docker Hub is the public app store, while private registries are like your company's internal app store. `push` uploads your app, `pull` downloads it.

---

## Useful Combinations & Tips

### Python Development Workflow

```bash
# Run Python app with live code reload
docker run -v $(pwd):/app -w /app -p 5000:5000 python:3.11 python app.py

# Run Python shell in container
docker run -it --rm python:3.11 python

# Install requirements and run
docker run -v $(pwd):/app -w /app python:3.11 sh -c "pip install -r requirements.txt && python app.py"

# Run pytest in container
docker run -v $(pwd):/app -w /app python:3.11 pytest
```

### Java Development Workflow

```bash
# Compile and run Java app
docker run -v $(pwd):/app -w /app openjdk:17 sh -c "javac Main.java && java Main"

# Run Maven build
docker run -v $(pwd):/app -w /app maven:3.9-openjdk-17 mvn clean package

# Run Spring Boot app
docker run -p 8080:8080 -v $(pwd):/app -w /app maven:3.9-openjdk-17 mvn spring-boot:run
```

### Debugging Tips

```bash
# Keep container running for debugging
docker run -d <image> tail -f /dev/null

# View container filesystem
docker export <container-id> | tar -t

# Check why container exited
docker logs <container-id>
docker inspect <container-id> | grep -A 10 State

# Debug networking
docker exec <container-id> ping <other-container-name>
docker exec <container-id> curl http://other-container:8080
```

### Common Patterns

```bash
# Remove all containers (running and stopped)
docker rm -f $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Stop all running containers
docker stop $(docker ps -q)

# Follow logs of multiple containers
docker-compose logs -f service1 service2

# Shell into database container
docker exec -it postgres-container psql -U postgres

# Backup database from container
docker exec postgres-container pg_dump -U postgres dbname > backup.sql

# Restore database to container
docker exec -i postgres-container psql -U postgres dbname < backup.sql
```

---

## Quick Reference Table

|Category|Command|Description|
|---|---|---|
|**Run**|`docker run -d -p 8080:80 nginx`|Run container in background with port mapping|
|**Shell**|`docker exec -it <container> bash`|Open shell in running container|
|**Logs**|`docker logs -f <container>`|Follow container logs|
|**Stop**|`docker stop <container>`|Gracefully stop container|
|**Remove**|`docker rm -f <container>`|Force remove container|
|**Build**|`docker build -t myapp .`|Build image from Dockerfile|
|**List**|`docker ps -a`|List all containers|
|**Clean**|`docker system prune -a`|Remove all unused resources|
|**Compose Up**|`docker-compose up -d`|Start all services in background|
|**Compose Down**|`docker-compose down`|Stop and remove all services|

---

## Flags Worth Remembering

- `-d` = detached (run in background)
- `-it` = interactive terminal
- `-p` = port mapping (host:container)
- `-v` = volume mount
- `-e` = environment variable
- `--rm` = remove after exit
- `-f` = force
- `-a` = all
- `-q` = quiet (only IDs)
- `--name` = assign name to container

---

**Pro Tip:** Use `docker run --help` or `docker <command> --help` to see all options for any command. Docker's help is excellent and provides examples!