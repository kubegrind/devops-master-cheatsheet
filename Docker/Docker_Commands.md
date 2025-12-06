# Docker & Docker Compose Commands Reference Guide

**Version:** 1.0  
**Last Updated:** October 2025  
**Audience:** DevOps Engineers & DevOps Teams

---

## Table of Contents

1. [Docker Basics](#docker-basics)
2. [Image Management](#image-management)
3. [Container Lifecycle](#container-lifecycle)
4. [Container Inspection & Monitoring](#container-inspection--monitoring)
5. [Networking](#networking)
6. [Volumes & Storage](#volumes--storage)
7. [Logging & Debugging](#logging--debugging)
8. [Docker Compose Basics](#docker-compose-basics)
9. [Docker Compose Configuration](#docker-compose-configuration)
10. [Docker Compose Operations](#docker-compose-operations)
11. [Registry & Image Distribution](#registry--image-distribution)
12. [Performance & Optimization](#performance--optimization)
13. [Security](#security)
14. [Common Workflows](#common-workflows)
15. [Troubleshooting](#troubleshooting)

---

## Docker Basics

### Check Docker Installation

```bash
docker version
```

Displays Docker client and server version information. Confirms Docker daemon is running.

### Check Docker Status

```bash
docker info
```

Shows detailed Docker system information including storage driver, container/image count, and memory limits.

### Docker System Prune

```bash
docker system prune
```

Removes dangling images, containers, networks, and volumes. Use `-a` to remove all unused images.

### Display Docker Disk Usage

```bash
docker system df
```

Shows Docker resource usage including images, containers, and volumes with space consumption.

### Check Docker Configuration

```bash
docker config ls
```

Lists available Docker configurations for swarm services.

### Get Docker CLI Help

```bash
docker --help
docker container --help
docker image --help
```

Displays help for Docker commands. Use for any command to see available options.

---

## Image Management

### List Local Images

```bash
docker images
docker image ls
```

Shows all local images with repository, tag, image ID, size, and creation date.

### List Images with Format

```bash
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

Displays images in custom format for easier parsing and scripting.

### Search Docker Hub

```bash
docker search ubuntu
```

Searches Docker Hub for images matching the query. Shows official and community images.

### Search with Filters

```bash
docker search ubuntu --filter "is-official=true"
```

Searches with filters like `is-official`, `stars`, and `is-automated`.

### Pull Image from Registry

```bash
docker pull ubuntu:22.04
docker pull myregistry.azurecr.io/myapp:1.0
```

Downloads image from Docker Hub or private registry. Tags specify version.

### Pull Latest Image Tag

```bash
docker pull ubuntu
```

Without tag specification, pulls the `latest` tag by default.

### Push Image to Registry

```bash
docker push myregistry.azurecr.io/myapp:1.0
```

Uploads image to registry. Requires login and proper image tagging.

### Tag Image

```bash
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0
docker tag myapp:latest myapp:1.0
```

Creates an alias or version tag for an image. Useful for versioning and registry distribution.

### Build Image from Dockerfile

```bash
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f ./Dockerfile .
```

Builds an image from Dockerfile. `-t` specifies tag, `-f` specifies Dockerfile path.

### Build with Build Arguments

```bash
docker build -t myapp:1.0 --build-arg NODE_ENV=production .
```

Passes build-time variables to Dockerfile for conditional builds.

### Build with Multiple Tags

```bash
docker build -t myapp:1.0 -t myapp:latest .
```

Tags image with multiple names in one build operation.

### Build from Git Repository

```bash
docker build https://github.com/user/repo.git
docker build https://github.com/user/repo.git#main
```

Builds directly from a remote Git repository URL with optional branch.

### View Image History

```bash
docker history myapp:1.0
```

Shows layers and commands used to build the image with sizes.

### Inspect Image

```bash
docker inspect myapp:1.0
```

Displays detailed JSON metadata about the image including environment variables, exposed ports, and volumes.

### Remove Image

```bash
docker rmi myapp:1.0
docker image rm myapp:1.0
```

Deletes an image locally. Use `-f` to force delete if container is running.

### Remove Multiple Images

```bash
docker rmi myapp:1.0 ubuntu:22.04 nginx:latest
```

Removes multiple images in one command.

### Remove Dangling Images

```bash
docker image prune
```

Removes unused images (dangling images with no tags or containers).

### Remove All Unused Images

```bash
docker image prune -a
```

Removes all images not used by any container.

### Export Image

```bash
docker save myapp:1.0 -o myapp.tar
```

Exports image to a tar archive for transport or backup.

### Load Image from Archive

```bash
docker load -i myapp.tar
```

Imports image from a tar archive created with `docker save`.

### Get Image Digest

```bash
docker inspect --format='{{index .RepoDigests 0}}' myapp:1.0
```

Retrieves the image digest (SHA256 hash) for verification and reproducibility.

---

## Container Lifecycle

### Run Container

```bash
docker run -d -p 8080:80 --name mywebapp nginx:latest
```

Creates and starts a container. `-d` runs detached, `-p` maps ports, `--name` sets container name.

### Run Container with Environment Variables

```bash
docker run -d --name myapp -e NODE_ENV=production -e DATABASE_URL=postgres://localhost/db myapp:1.0
```

Sets environment variables accessible inside the container.

### Run Container with Volume Mount

```bash
docker run -d -v /host/path:/container/path --name myapp myapp:1.0
docker run -d -v myvolume:/data --name myapp myapp:1.0
```

Mounts host directory or named volume into container. First is bind mount, second is named volume.

### Run Container with Resource Limits

```bash
docker run -d --memory 512m --cpus 0.5 --name myapp myapp:1.0
```

Sets memory limit (512MB) and CPU limit (0.5 cores). Prevents resource exhaustion.

### Run Container in Interactive Mode

```bash
docker run -it ubuntu:22.04 /bin/bash
```

Runs container with interactive terminal. `-i` for stdin, `-t` for pseudo-terminal.

### Run Container as Specific User

```bash
docker run -d --user nobody --name myapp myapp:1.0
```

Runs container with specific user ID instead of root. Improves security.

### Run Container with Custom Hostname

```bash
docker run -d --hostname myhost --name myapp myapp:1.0
```

Sets container hostname for network communication.

### Run Container with Health Check

```bash
docker run -d --health-cmd='curl -f http://localhost/ || exit 1' --health-interval=30s --name myapp nginx:latest
```

Defines health check command, interval, and other parameters for container monitoring.

### Run Container with Restart Policy

```bash
docker run -d --restart unless-stopped --name myapp myapp:1.0
```

Sets restart policy: `no`, `always`, `unless-stopped`, `on-failure`. Ensures container resilience.

### Run Container with Network

```bash
docker run -d --network custom-network --name myapp myapp:1.0
```

Connects container to specific Docker network for service discovery.

### Run Container with Logging Driver

```bash
docker run -d --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 --name myapp myapp:1.0
```

Configures logging driver and options like log rotation and size limits.

### Start Stopped Container

```bash
docker start myapp
docker container start myapp
```

Starts a previously stopped container with the same configuration.

### Stop Running Container

```bash
docker stop myapp
```

Gracefully stops a running container with SIGTERM signal (default 10 second timeout).

### Stop Container with Timeout

```bash
docker stop -t 30 myapp
```

Waits up to 30 seconds before force killing the container.

### Kill Container

```bash
docker kill myapp
```

Immediately terminates container with SIGKILL signal. Use only if `stop` fails.

### Pause Container

```bash
docker pause myapp
```

Suspends all processes in container without stopping it.

### Unpause Container

```bash
docker unpause myapp
```

Resumes suspended container processes.

### Restart Container

```bash
docker restart myapp
docker container restart myapp
```

Stops and then starts a container. Useful for applying configuration changes.

### Remove Container

```bash
docker rm myapp
```

Deletes a stopped container. Use `-f` to force remove running container.

### Remove Multiple Containers

```bash
docker rm container1 container2 container3
```

Removes multiple containers in one command.

### Remove All Stopped Containers

```bash
docker container prune
```

Removes all stopped containers. Add `-f` to skip confirmation.

### Rename Container

```bash
docker rename old-name new-name
```

Changes container name while running or stopped.

### Commit Container Changes

```bash
docker commit myapp myapp:2.0
```

Creates a new image from container's current state. Useful for capturing manual changes.

### Export Container

```bash
docker export myapp -o myapp.tar
```

Exports container filesystem as tar archive.

---

## Container Inspection & Monitoring

### List Running Containers

```bash
docker ps
docker container ls
```

Shows all running containers with ID, image, status, ports, and names.

### List All Containers

```bash
docker ps -a
docker container ls -a
```

Shows all containers including stopped ones.

### List Containers with Custom Format

```bash
docker ps --format "table {{.ID}}\t{{.Image}}\t{{.Status}}\t{{.Names}}"
```

Displays containers in custom format for scripting.

### Get Container ID

```bash
docker ps -q
```

Returns only container IDs, useful for scripting and bulk operations.

### Inspect Container

```bash
docker inspect myapp
```

Displays detailed JSON metadata including environment, volumes, network, and resource limits.

### Inspect Multiple Containers

```bash
docker inspect container1 container2 container3
```

Inspects multiple containers in one command.

### Get Container IP Address

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' myapp
```

Retrieves container's IP address for network communication.

### Get Container Logs

```bash
docker logs myapp
```

Displays all container output (stdout and stderr). Useful for debugging.

### Follow Container Logs

```bash
docker logs -f myapp
```

Streams logs in real-time as container writes them. Similar to `tail -f`.

### Get Last N Log Lines

```bash
docker logs --tail 100 myapp
```

Shows last 100 lines of logs without entire history.

### Get Logs Since Timestamp

```bash
docker logs --since 2024-01-01 myapp
```

Shows logs after specified timestamp (RFC3339 format).

### Get Logs with Timestamps

```bash
docker logs -t myapp
```

Includes timestamp with each log line for precise timing.

### View Container Statistics

```bash
docker stats myapp
```

Shows real-time CPU, memory, and network usage for container.

### View Stats for All Containers

```bash
docker stats
```

Displays statistics for all running containers continuously.

### View Stats Without Stream

```bash
docker stats --no-stream myapp
```

Shows one snapshot of statistics instead of continuous update.

### Get Container Processes

```bash
docker top myapp
```

Shows running processes inside the container similar to `ps` command.

### Get Container Events

```bash
docker events --filter type=container
```

Shows real-time Docker events like container start, stop, create, destroy.

### Filter Events by Container

```bash
docker events --filter container=myapp
```

Shows events only for specific container.

### Get Image Events

```bash
docker events --filter type=image
```

Shows events related to images like pull, build, delete.

### Check Container Health Status

```bash
docker inspect --format='{{.State.Health.Status}}' myapp
```

Returns health status: `starting`, `healthy`, `unhealthy`.

### Get Container Uptime

```bash
docker inspect -f '{{.State.StartedAt}}' myapp
```

Shows when container was started for uptime calculation.

---

## Networking

### List Networks

```bash
docker network ls
```

Shows all Docker networks with driver and scope information.

### Inspect Network

```bash
docker network inspect bridge
```

Displays detailed network information including connected containers.

### Create Network

```bash
docker network create mynetwork
```

Creates a new bridge network for container communication.

### Create Network with Options

```bash
docker network create --driver bridge --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 mynetwork
```

Creates network with specific driver, subnet, and IP range.

### Connect Container to Network

```bash
docker network connect mynetwork myapp
```

Attaches running container to an additional network.

### Disconnect Container from Network

```bash
docker network disconnect mynetwork myapp
```

Removes container from network.

### Remove Network

```bash
docker network rm mynetwork
```

Deletes a Docker network. Requires disconnecting all containers first.

### Prune Unused Networks

```bash
docker network prune
```

Removes all unused networks not connected to any container.

### Create Overlay Network (Swarm)

```bash
docker network create --driver overlay mynetwork
```

Creates overlay network for multi-host communication in Docker Swarm.

### Map Container Port to Host

```bash
docker run -p 8080:80 nginx
```

Maps port 80 in container to port 8080 on host. Format: `host:container`.

### Map All Exposed Ports

```bash
docker run -P nginx
```

Automatically maps all exposed ports to random host ports.

### Expose Port Range

```bash
docker run -p 8000-9000:8000-9000 myapp
```

Maps range of ports from container to host.

### Bind to Specific Interface

```bash
docker run -p 127.0.0.1:8080:80 nginx
```

Maps port only to specific host interface instead of all interfaces.

### Link Containers (Legacy)

```bash
docker run --link mydb:db myapp
```

Legacy linking (deprecated). Use networks instead for service discovery.

### DNS Configuration

```bash
docker run --dns 8.8.8.8 myapp
```

Sets custom DNS server for container name resolution.

### Add Host Entry

```bash
docker run --add-host myhost:192.168.1.1 myapp
```

Adds custom hostname to `/etc/hosts` inside container.

---

## Volumes & Storage

### List Volumes

```bash
docker volume ls
```

Shows all Docker volumes with driver and mount point.

### Create Volume

```bash
docker volume create myvolume
```

Creates a new named volume for persistent data storage.

### Create Volume with Options

```bash
docker volume create --driver local --opt type=tmpfs --opt device=tmpfs myvolume
```

Creates volume with specific driver and options.

### Inspect Volume

```bash
docker volume inspect myvolume
```

Shows volume metadata including mount point and driver information.

### Mount Volume in Container

```bash
docker run -d -v myvolume:/data myapp
```

Mounts named volume at `/data` inside container.

### Mount Host Directory

```bash
docker run -d -v /host/data:/container/data myapp
```

Mounts host directory into container (bind mount).

### Mount Read-Only Volume

```bash
docker run -d -v myvolume:/data:ro myapp
```

Mounts volume as read-only (`:ro` flag). Container cannot modify data.

### Mount Multiple Volumes

```bash
docker run -d -v vol1:/data1 -v vol2:/data2 -v /host:/host myapp
```

Attaches multiple volumes and bind mounts to container.

### Remove Volume

```bash
docker volume rm myvolume
```

Deletes a volume. Requires removing containers using it first.

### Remove All Unused Volumes

```bash
docker volume prune
```

Removes all volumes not currently used by any container.

### Copy File to Container

```bash
docker cp localfile.txt myapp:/path/in/container/
```

Copies file from host to running container.

### Copy File from Container

```bash
docker cp myapp:/path/in/container/file.txt localfile.txt
```

Copies file from running container to host.

### Copy Directory

```bash
docker cp -r /host/directory myapp:/container/path/
```

Recursively copies directories between host and container.

### Create Volume from Container

```bash
docker create -v /data --name data-container myapp
docker run --volumes-from data-container myapp
```

Creates volume container and mounts it in other containers (data-only pattern).

### Backup Volume

```bash
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine tar czf /backup/volume-backup.tar.gz -C /data .
```

Backs up volume contents to host filesystem.

### Restore Volume

```bash
docker run --rm -v myvolume:/data -v $(pwd):/backup alpine tar xzf /backup/volume-backup.tar.gz -C /data
```

Restores volume from backup archive.

---

## Logging & Debugging

### View Container Logs

```bash
docker logs myapp
```

Shows all container logs from stdout and stderr.

### Stream Logs

```bash
docker logs -f myapp
```

Continuously streams logs in real-time like `tail -f`.

### Show Last N Lines

```bash
docker logs --tail 50 myapp
```

Shows only the last 50 lines of logs.

### Show Logs with Timestamps

```bash
docker logs -t myapp
```

Includes timestamp with each log line.

### Logs Since Specific Time

```bash
docker logs --since 10m myapp
```

Shows logs from last 10 minutes. Supports formats like `10s`, `10m`, `1h`.

### Logs Until Specific Time

```bash
docker logs --until 5m myapp
```

Shows logs up to 5 minutes ago.

### Execute Command in Container

```bash
docker exec myapp ls -la /
```

Runs command inside running container without accessing shell.

### Execute Command with Interaction

```bash
docker exec -it myapp /bin/bash
```

Opens interactive shell inside running container. `-i` for stdin, `-t` for terminal.

### Execute as Specific User

```bash
docker exec -u root myapp apt-get update
```

Runs command as specified user instead of default container user.

### Execute with Environment Variable

```bash
docker exec -e NODE_ENV=production myapp npm start
```

Sets environment variables for the executed command.

### Attach to Container

```bash
docker attach myapp
```

Connects to running container's stdout and stderr streams.

### Detach from Container

```bash
Press Ctrl+P then Ctrl+Q
```

Detaches from attached container without stopping it.

### Get Docker Event Stream

```bash
docker events
```

Shows all Docker events (container, image, network, volume) in real-time.

### Filter Events

```bash
docker events --filter type=container --filter status=start
```

Shows only specific event types and statuses.

### Check Container Diff

```bash
docker diff myapp
```

Shows changes made to container filesystem since creation.

### Get Container Changes

```bash
docker container diff myapp
```

Lists modified files and directories in container (A=added, C=changed, D=deleted).

### Analyze Image Layers

```bash
docker history myapp:1.0
```

Shows each layer of the image with size and creation command.

### View Image Details

```bash
docker image inspect myapp:1.0
```

Displays comprehensive image metadata as JSON.

### Check Disk Usage Details

```bash
docker system df -v
```

Shows verbose disk usage information for images, containers, and volumes.

---

## Docker Compose Basics

### Check Docker Compose Version

```bash
docker-compose version
docker compose version
```

Displays Docker Compose version. Newer installations use `docker compose` (plugin format).

### Validate Compose File

```bash
docker-compose config
docker compose config
```

Validates docker-compose.yml syntax and displays resolved configuration.

### View Compose Services

```bash
docker-compose config --services
docker compose config --services
```

Lists all services defined in docker-compose.yml.

### Create Compose File Template

```bash
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
EOF
```

Creates basic multi-container compose file with web and database services.

---

## Docker Compose Configuration

### Basic Service Configuration

```yaml
version: '3.8'

services:
  myapp:
    image: myapp:1.0
    container_name: myapp-container
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://db:5432/mydb
    ports:
      - "8080:3000"
    volumes:
      - ./data:/app/data
      - myvolume:/app/cache
    networks:
      - mynetwork
    depends_on:
      - db
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

Complete service definition with all common configuration options.

### Multi-Service Configuration

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./html:/usr/share/nginx/html
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - api
    networks:
      - frontend

  api:
    build: ./api
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    networks:
      - backend
      - frontend

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  pgdata:

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

Complex configuration with multiple services, networks, and dependencies.

### Build Configuration

```yaml
services:
  myapp:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
        BUILD_VERSION: 1.0.0
      cache_from:
        - myapp:latest
    image: myapp:1.0
```

Build options for creating images from Dockerfiles with arguments and caching.

### Volume Configuration

```yaml
volumes:
  pgdata:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs

services:
  myapp:
    volumes:
      - ./local:/app/local              # Bind mount
      - pgdata:/var/lib/postgresql/data # Named volume
      - /tmp:/tmp:ro                    # Read-only mount
```

Various volume mounting strategies including named volumes and bind mounts.

### Network Configuration

```yaml
networks:
  frontend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br0
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1

  backend:
    driver: bridge
    internal: true

services:
  web:
    networks:
      - frontend
      - backend
```

Network definitions with custom drivers, IP ranges, and service connectivity.

### Logging Configuration

```yaml
services:
  myapp:
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
        labels: "app=myapp"
```

Logging driver configuration with rotation and label options.

---

## Docker Compose Operations

### Start Services

```bash
docker-compose up
docker compose up
```

Creates and starts all services defined in docker-compose.yml. Runs in foreground.

### Start Services in Background

```bash
docker-compose up -d
docker compose up -d
```

Starts services in detached mode (background). Useful for production.

### Start Specific Service

```bash
docker-compose up -d web
docker compose up -d web
```

Starts only the specified service and its dependencies.

### Build Before Starting

```bash
docker-compose up -d --build
docker compose up -d --build
```

Rebuilds images before starting containers.

### Stop Services

```bash
docker-compose stop
docker compose stop
```

Gracefully stops all running services without removing containers.

### Stop Specific Service

```bash
docker-compose stop web
docker compose stop web
```

Stops only the specified service.

### Start Stopped Services

```bash
docker-compose start
docker compose start
```

Starts previously stopped services.

### Restart Services

```bash
docker-compose restart
docker compose restart
```

Stops and then starts all services.

### Remove Services

```bash
docker-compose down
docker compose down
```

Stops and removes all containers, networks. Volumes persist by default.

### Remove Services and Volumes

```bash
docker-compose down -v
docker compose down -v
```

Removes containers, networks, and volumes. Data is deleted.

### Remove Services and Images

```bash
docker-compose down --rmi all
docker compose down --rmi all
```

Removes containers, networks, and all images. Options: `local`, `all`.

### View Running Services

```bash
docker-compose ps
docker compose ps
```

Shows status of all services defined in docker-compose.yml.

### View All Services

```bash
docker-compose ps -a
docker compose ps -a
```

Shows all services including stopped ones.

### Execute Command in Service

```bash
docker-compose exec web /bin/bash
docker compose exec web /bin/bash
```

Opens interactive shell in running service container.

### Execute Non-Interactive Command

```bash
docker-compose exec web npm test
docker compose exec web npm test
```

Runs command inside service container without interaction.

### Execute as Specific User

```bash
docker-compose exec -u root web apt-get update
docker compose exec -u root web apt-get update
```

Runs command as specified user in service container.

### View Service Logs

```bash
docker-compose logs
docker compose logs
```

Shows logs from all services.

### View Specific Service Logs

```bash
docker-compose logs web
docker compose logs web
```

Shows logs only for specified service.

### Stream Logs

```bash
docker-compose logs -f
docker compose logs -f
```

Continuously streams logs from all services.

### Stream Specific Service Logs

```bash
docker-compose logs -f web
docker compose logs -f web
```

Streams logs from specific service in real-time.

### Show Last N Lines

```bash
docker-compose logs --tail 100
docker compose logs --tail 100
```

Shows last 100 lines from all services.

### Pause Services

```bash
docker-compose pause
docker compose pause
```

Pauses all running service processes.

### Unpause Services

```bash
docker-compose unpause
docker compose unpause
```

Resumes paused service processes.

### View Service Statistics

```bash
docker-compose stats
docker compose stats
```

Shows real-time CPU, memory, and network usage for all services.

### Pull Images

```bash
docker-compose pull
docker compose pull
```

Pulls latest images for all services from registry.

### Build Images

```bash
docker-compose build
docker compose build
```

Builds or rebuilds images for services with build configuration.

### Build Specific Service

```bash
docker-compose build web
docker compose build web
```

Builds only the specified service image.

### Push Images

```bash
docker-compose push
docker compose push
```

Pushes built images to registry (requires Docker login).

### Remove Stopped Containers

```bash
docker-compose rm
docker compose rm
```

Removes stopped containers for services.

### Remove Stopped Containers Without Prompt

```bash
docker-compose rm -f
docker compose rm -f
```

Removes stopped containers without confirmation prompt.

### Scale Service

```bash
docker-compose up -d --scale web=3
docker compose up -d --scale web=3
```

Runs 3 instances of the web service for load balancing.

### Update Service Configuration

```bash
docker-compose up -d web
docker compose up -d web
```

Applies configuration changes from docker-compose.yml to running service.

### Environment File

```bash
docker-compose --env-file .env.prod up -d
docker compose --env-file .env.prod up -d
```

Uses custom environment file for variable substitution.

### Docker Compose File Override

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

Merges multiple compose files for environment-specific configuration.

### Set Service Project Name

```bash
docker-compose -p myproject up -d
docker compose -p myproject up -d
```

Sets project name for container naming and isolation.

---

## Registry & Image Distribution

### Login to Docker Hub

```bash
docker login
```

Authenticates with Docker Hub. Stores credentials in `~/.docker/config.json`.

### Login to Private Registry

```bash
docker login myregistry.azurecr.io
docker login gcr.io
```

Logs into private container registries with credentials.

### Logout from Registry

```bash
docker logout
docker logout myregistry.azurecr.io
```

Removes stored credentials for registry.

### Push Image to Registry

```bash
docker push myregistry.azurecr.io/myapp:1.0
```

Uploads tagged image to registry. Requires proper tagging and authentication.

### Pull Image from Private Registry

```bash
docker pull myregistry.azurecr.io/myapp:1.0
```

Downloads image from private registry after authentication.

### Create Private Registry

```bash
docker run -d -p 5000:5000 --name registry registry:2
```

Runs local Docker registry for private image storage.

### Push to Local Registry

```bash
docker tag myapp:1.0 localhost:5000/myapp:1.0
docker push localhost:5000/myapp:1.0
```

Tags and pushes image to local registry running on port 5000.

### View Registry Catalog

```bash
curl http://localhost:5000/v2/_catalog
```

Lists all images in local registry.

### Retag Image

```bash
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0
docker tag myapp:1.0 myregistry.azurecr.io/myapp:latest
```

Creates additional tags for an image for versioning and distribution.

### Create Multi-Architecture Image

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:1.0 .
```

Builds image for multiple architectures using buildx.

### Push Multi-Architecture Image

```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myregistry.azurecr.io/myapp:1.0 --push .
```

Builds and pushes image manifest for multiple architectures.

---

## Performance & Optimization

### Optimize Image Size

```dockerfile
FROM alpine:3.18
RUN apk add --no-cache python3 pip
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python3", "app.py"]
```

Uses minimal base image, clean up package manager cache, minimize layers.

### Multi-Stage Build

```dockerfile
FROM golang:1.20 as builder
WORKDIR /build
COPY . .
RUN go build -o app .

FROM alpine:3.18
COPY --from=builder /build/app /app
CMD ["/app"]
```

Separates build stage from runtime, reducing final image size.

### Limit Container Resources

```bash
docker run -d \
  --memory 512m \
  --memory-swap 1g \
  --cpus 0.5 \
  --cpus-shares 1024 \
  --blkio-weight 300 \
  myapp:1.0
```

Sets CPU, memory, and I/O limits to prevent resource exhaustion.

### Enable BuildKit

```bash
DOCKER_BUILDKIT=1 docker build -t myapp:1.0 .
```

Uses BuildKit for faster, more efficient builds with better caching.

### View Image Layers

```bash
docker inspect myapp:1.0 | grep -A 100 '"Layers"'
```

Shows image layer composition and sizes for optimization analysis.

### Scan Image for Vulnerabilities

```bash
docker scan myapp:1.0
```

Scans image for known vulnerabilities in dependencies.

### Check Image History

```bash
docker history myapp:1.0
```

Displays layer-by-layer history for identification of large or unnecessary layers.

### Cache From Option

```bash
docker build --cache-from myapp:latest -t myapp:1.0 .
```

Uses existing image as cache source for faster builds.

### Prune All Unused Resources

```bash
docker system prune -a --volumes
```

Removes all unused images, containers, networks, and volumes to free space.

---

## Security

### Run Container as Non-Root

```bash
docker run -d --user 1000:1000 myapp:1.0
```

Runs container with non-root user for improved security.

### Create Unprivileged User in Dockerfile

```dockerfile
RUN useradd -m -u 1000 appuser
USER appuser
COPY --chown=appuser:appuser . /app
```

Creates non-root user and sets file ownership in image.

### Drop Capabilities

```bash
docker run -d --cap-drop ALL myapp:1.0
```

Removes all Linux capabilities from container.

### Add Specific Capabilities

```bash
docker run -d --cap-drop ALL --cap-add NET_BIND_SERVICE myapp:1.0
```

Drops all capabilities then adds only required ones.

### Read-Only Root Filesystem

```bash
docker run -d --read-only myapp:1.0
```

Makes container root filesystem read-only, preventing unauthorized writes.

### Read-Only with Temporary Filesystem

```bash
docker run -d --read-only -v /tmp:/tmp myapp:1.0
```

Read-only filesystem with writable temporary mount point.

### Use Security Options

```bash
docker run -d --security-opt no-new-privileges:true myapp:1.0
```

Prevents processes from gaining additional privileges.

### Use AppArmor Profile

```bash
docker run -d --security-opt apparmor=docker-default myapp:1.0
```

Applies AppArmor profile for mandatory access control.

### Use SELinux Context

```bash
docker run -d --security-opt label=type:svirt_sandbox_file_t myapp:1.0
```

Sets SELinux context for container processes.

### Use Seccomp Profile

```bash
docker run -d --security-opt seccomp=unconfined myapp:1.0
```

Configures seccomp system call filtering. Default restricts dangerous calls.

### Scan Image for Vulnerabilities

```bash
docker scout cves myapp:1.0
```

Uses Docker Scout to identify security vulnerabilities in image.

### Sign Images with Notation

```bash
notation sign myregistry.azurecr.io/myapp:1.0
```

Cryptographically signs image for authenticity verification.

### Enable Docker Content Trust

```bash
export DOCKER_CONTENT_TRUST=1
docker push myregistry.azurecr.io/myapp:1.0
```

Requires images to be signed before push/pull operations.

### Use Network Isolation

```bash
docker network create --internal isolated
docker run -d --network isolated myapp:1.0
```

Creates isolated network preventing external internet access.

### Limit Resource Usage

```bash
docker run -d \
  --memory 256m \
  --cpus 0.25 \
  --pids-limit 100 \
  myapp:1.0
```

Restricts memory, CPU, and process count to prevent DoS.

### Configure Logging

```bash
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp:1.0
```

Configures log rotation and size limits for security auditing.

---

## Common Workflows

### Development Environment Setup

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DEBUG=true
    depends_on:
      - db

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: dev
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  pgdata:
```

Typical development compose file with live code reloading and database.

### Production Deployment

```bash
# Build image for production
docker build --build-arg NODE_ENV=production -t myapp:1.0 .

# Tag for registry
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0

# Login and push
docker login myregistry.azurecr.io
docker push myregistry.azurecr.io/myapp:1.0

# Deploy with compose
docker compose -f docker-compose.prod.yml up -d
```

Production workflow including build optimization and registry push.

### Database Migration with Compose

```bash
# Start database service
docker-compose up -d db

# Wait for database to be ready
docker-compose exec db pg_isready -U postgres

# Run migration
docker-compose exec app npm run migrate

# Start remaining services
docker-compose up -d
```

Workflow ensuring database is ready before running migrations.

### Blue-Green Deployment

```bash
# Current production (blue)
docker-compose -p myapp-blue up -d

# New version (green)
docker-compose -p myapp-green -f docker-compose.yml up -d

# Switch load balancer/proxy to green
# (via DNS update or reverse proxy config change)

# Keep blue running for quick rollback
```

Deployment strategy allowing zero-downtime updates with quick rollback.

### Backup and Restore Database

```bash
# Backup
docker-compose exec -T db pg_dump -U postgres mydb > backup.sql

# Restore
docker-compose exec -T db psql -U postgres < backup.sql
```

Database backup and restore procedures with compose.

### Scale Service with Compose

```bash
# Start with 1 instance
docker-compose up -d api

# Scale to 3 instances
docker-compose up -d --scale api=3

# Use load balancer for distribution
docker-compose up -d nginx
```

Horizontal scaling of services with load balancing.

### CI/CD Integration

```bash
# Test image build
docker build -t myapp:test .

# Run tests
docker run --rm myapp:test npm test

# Build final image
docker build -t myapp:1.0 .

# Push to registry on success
docker tag myapp:1.0 myregistry.azurecr.io/myapp:1.0
docker push myregistry.azurecr.io/myapp:1.0
```

Typical CI/CD pipeline for image building and pushing.

### Local Development with Docker

```bash
# Create network for services
docker network create dev-network

# Run database
docker run -d --name postgres \
  -e POSTGRES_PASSWORD=dev \
  --network dev-network \
  -p 5432:5432 \
  postgres:15

# Run application
docker run -d --name myapp \
  -e DATABASE_URL=postgresql://postgres:dev@postgres:5432/mydb \
  -v $(pwd):/app \
  -p 3000:3000 \
  --network dev-network \
  myapp:latest
```

Manual container orchestration for local development.

### Troubleshooting with Container Shell

```bash
# Access running container
docker exec -it myapp /bin/bash

# Or create temporary container
docker run -it --rm --entrypoint /bin/bash myapp:1.0

# Install debugging tools
apt-get update && apt-get install -y curl wget netcat

# Test connectivity
curl http://db:5432
netstat -tuln
```

Common debugging techniques using container shell access.

---

## Troubleshooting

### Container Fails to Start

```bash
# Check logs
docker logs myapp

# Inspect container
docker inspect myapp | grep -A 5 "State"

# Check health status
docker inspect --format='{{.State.Health.Status}}' myapp

# Try running with entrypoint override
docker run -it --entrypoint /bin/bash myapp:1.0
```

Diagnosis of container startup failures.

### Out of Disk Space

```bash
# Check disk usage
docker system df

# Check specific images
docker image du -a

# Remove dangling images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove all unused resources
docker system prune -a --volumes
```

Managing Docker disk usage and cleanup.

### Network Connectivity Issues

```bash
# Check networks
docker network ls

# Inspect network
docker network inspect mynetwork

# Test DNS resolution
docker run --rm --network mynetwork alpine nslookup myservice

# Ping container
docker run --rm --network mynetwork alpine ping myservice

# Check container IP
docker inspect myapp | grep -A 2 NetworkSettings
```

Debugging network and service discovery issues.

### Port Already in Use

```bash
# Find process using port
lsof -i :8080
netstat -tuln | grep 8080

# Use different port
docker run -p 8081:80 nginx

# Check port mapping
docker ps --format "table {{.ID}}\t{{.Ports}}"
```

Resolving port conflict issues.

### Memory Issues

```bash
# Check memory limits
docker inspect myapp | grep -i memory

# View actual usage
docker stats myapp

# Increase memory limit
docker update --memory 1g myapp

# Restart container
docker restart myapp
```

Managing and troubleshooting memory constraints.

### Slow Build Performance

```bash
# Check build cache
docker system df

# Use BuildKit
DOCKER_BUILDKIT=1 docker build -t myapp:1.0 .

# Check Dockerfile optimization
docker history myapp:1.0

# Clear build cache
docker builder prune
```

Optimizing Docker build performance.

### Volume Permission Issues

```bash
# Check volume ownership
ls -la /var/lib/docker/volumes/myvolume/_data

# Fix permissions
docker run --rm -v myvolume:/data alpine chown -R 1000:1000 /data

# Use bind mount with proper ownership
docker run -v $(pwd):/app:z myapp
```

Resolving file permission problems with volumes.

### Image Pull Failures

```bash
# Verify credentials
docker logout
docker login myregistry.azurecr.io

# Check image exists
docker search myimage

# Try explicit tag
docker pull myregistry.azurecr.io/myapp:1.0

# Check network connectivity
docker run --rm alpine ping registry.hub.docker.com
```

Troubleshooting image pull issues.

### Compose Service Won't Start

```bash
# Validate compose file
docker-compose config

# Check service logs
docker-compose logs myservice

# Start with verbose output
docker-compose -f docker-compose.yml up

# Check dependencies
docker-compose exec db pg_isready
```

Debugging Docker Compose service startup failures.

### Health Check Failures

```bash
# Check health status
docker inspect --format='{{.State.Health}}' myapp

# View health check logs
docker logs myapp

# Run health check command manually
docker exec myapp curl -f http://localhost/health

# Update health check
docker update --health-interval=10s myapp
```

Resolving container health check failures.

### Environment Variable Not Working

```bash
# Verify variables were set
docker exec myapp env

# Check compose file syntax
docker-compose config

# Verify order of precedence
# .env file → docker-compose.yml → docker run -e flag

# Test with explicit variable
docker run -e VAR=value myapp
```

Debugging environment variable issues.

### DNS Resolution Failures

```bash
# Test DNS
docker run --rm alpine nslookup google.com

# Check container DNS settings
docker inspect myapp | grep -i "dns\|127.0"

# Add custom DNS
docker run --dns 8.8.8.8 myapp

# Check hosts file
docker exec myapp cat /etc/hosts
```

Troubleshooting DNS resolution in containers.

### High CPU Usage

```bash
# Monitor CPU
docker stats myapp

# Identify problematic process
docker exec myapp top

# Check container limits
docker inspect myapp | grep -i cpus

# Limit CPU
docker update --cpus 0.5 myapp
```

Managing high CPU usage in containers.

### Orphaned Containers and Images

```bash
# Find dangling images
docker images -f dangling=true

# Find stopped containers
docker ps -a --filter status=exited

# Clean up
docker image prune -a
docker container prune
docker volume prune
```

Identifying and removing orphaned Docker resources.

---

## Best Practices

Use specific image tags instead of `latest`. Always specify version numbers for reproducibility and explicit version control.

Keep images small using multi-stage builds, minimal base images like Alpine, and removing unnecessary files and dependencies.

Run containers as non-root users for security. Create unprivileged users in Dockerfile and use `--user` flag.

Use health checks to automatically restart unhealthy containers. Define meaningful health check commands for application-specific verification.

Implement proper logging with configured log drivers and retention policies. Centralize logs for monitoring and debugging.

Define resource limits for CPU and memory to prevent resource exhaustion. Set appropriate limits based on application requirements.

Use named volumes for persistent data instead of relying on container state. Backup important volumes regularly.

Organize services with Docker networks for secure inter-service communication and automatic DNS resolution.

Version control docker-compose files and Dockerfiles. Use environment-specific compose files for different deployment environments.

Scan images for vulnerabilities regularly using Docker Scout or similar tools. Keep base images updated.

Use environment variables for configuration instead of hardcoding values in images.

Document Dockerfile decisions and service requirements in comments and READMEs.

Implement proper monitoring and alerting for container performance and health.

Use BuildKit for faster, more efficient builds with better caching.

Rotate credentials and secrets regularly. Never commit them to version control.

---

## Resources

- Official Docker Documentation: https://docs.docker.com/
- Docker Hub Registry: https://hub.docker.com/
- Docker Compose Documentation: https://docs.docker.com/compose/
- Docker Best Practices: https://docs.docker.com/develop/
- Docker Security: https://docs.docker.com/engine/security/
- Docker Scout: https://docs.docker.com/scout/
- BuildKit Documentation: https://docs.docker.com/build/buildkit/

