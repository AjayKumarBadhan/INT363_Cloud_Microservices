# Docker Commands Reference Guide

A comprehensive guide to Docker commands from basic to advanced levels.

---

## **LEVEL 1: BASIC COMMANDS**

### Container Management Basics

#### `docker run`
**Purpose:** Create and start a container from an image

**Basic Format:**
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**Common Options:**
- `-d, --detach` - Run container in background
- `-p, --publish` - Publish container's port to host (HOST:CONTAINER)
- `--name` - Assign a name to the container
- `-e, --env` - Set environment variables
- `-v, --volume` - Bind mount a volume
- `-it` - Interactive mode with TTY (for terminal access)
- `--rm` - Automatically remove container when it exits

**Examples:**
```bash
# Run nginx in detached mode
docker run -d nginx

# Run with port mapping and name
docker run -d -p 8080:80 --name my-nginx nginx

# Run interactively
docker run -it ubuntu bash
```

---

#### `docker ps`
**Purpose:** List running containers

**Format:**
```bash
docker ps [OPTIONS]
```

**Common Options:**
- `-a, --all` - Show all containers (including stopped)
- `-q, --quiet` - Only display container IDs
- `-s, --size` - Display total file sizes
- `--filter` - Filter output based on conditions
- `--format` - Pretty-print containers using a Go template

**Examples:**
```bash
# List running containers
docker ps

# List all containers
docker ps -a

# List only container IDs
docker ps -q
```

---

#### `docker start`
**Purpose:** Start one or more stopped containers

**Format:**
```bash
docker start [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-a, --attach` - Attach STDOUT/STDERR and forward signals
- `-i, --interactive` - Attach container's STDIN

**Examples:**
```bash
# Start a stopped container
docker start my-nginx

# Start and attach to container
docker start -a my-nginx
```

---

#### `docker stop`
**Purpose:** Stop one or more running containers

**Format:**
```bash
docker stop [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-t, --time` - Seconds to wait before killing container (default 10)

**Examples:**
```bash
# Stop a container
docker stop my-nginx

# Stop with custom timeout
docker stop -t 30 my-nginx
```

---

#### `docker restart`
**Purpose:** Restart one or more containers

**Format:**
```bash
docker restart [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-t, --time` - Seconds to wait before killing container

**Examples:**
```bash
# Restart a container
docker restart my-nginx
```

---

#### `docker rm`
**Purpose:** Remove one or more containers

**Format:**
```bash
docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `-f, --force` - Force removal of running container
- `-v, --volumes` - Remove anonymous volumes associated with container

**Examples:**
```bash
# Remove a stopped container
docker rm my-nginx

# Force remove a running container
docker rm -f my-nginx

# Remove all stopped containers
docker rm $(docker ps -aq -f status=exited)
```

---

#### `docker images`
**Purpose:** List images

**Format:**
```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```

**Common Options:**
- `-a, --all` - Show all images (including intermediates)
- `-q, --quiet` - Only show image IDs
- `--filter` - Filter output based on conditions
- `--format` - Pretty-print images using a Go template

**Examples:**
```bash
# List all images
docker images

# List only image IDs
docker images -q

# Filter images by name
docker images ubuntu
```

---

#### `docker pull`
**Purpose:** Pull an image or repository from a registry

**Format:**
```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

**Common Options:**
- `-a, --all-tags` - Download all tagged images
- `--platform` - Set platform if server is multi-platform capable

**Examples:**
```bash
# Pull latest nginx image
docker pull nginx

# Pull specific version
docker pull nginx:1.21

# Pull all tags
docker pull -a nginx
```

---

#### `docker rmi`
**Purpose:** Remove one or more images

**Format:**
```bash
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

**Common Options:**
- `-f, --force` - Force removal of image
- `--no-prune` - Do not delete untagged parents

**Examples:**
```bash
# Remove an image
docker rmi nginx

# Force remove
docker rmi -f nginx

# Remove all unused images
docker rmi $(docker images -q -f dangling=true)
```

---

#### `docker logs`
**Purpose:** Fetch logs of a container

**Format:**
```bash
docker logs [OPTIONS] CONTAINER
```

**Common Options:**
- `-f, --follow` - Follow log output
- `--tail` - Number of lines to show from end of logs
- `-t, --timestamps` - Show timestamps
- `--since` - Show logs since timestamp
- `--until` - Show logs before timestamp

**Examples:**
```bash
# View container logs
docker logs my-nginx

# Follow logs in real-time
docker logs -f my-nginx

# Show last 100 lines
docker logs --tail 100 my-nginx
```

---

## **LEVEL 2: INTERMEDIATE COMMANDS**

### Container Interaction

#### `docker exec`
**Purpose:** Run a command in a running container

**Format:**
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

**Common Options:**
- `-d, --detach` - Detached mode (run in background)
- `-i, --interactive` - Keep STDIN open
- `-t, --tty` - Allocate a pseudo-TTY
- `-e, --env` - Set environment variables
- `-u, --user` - Username or UID
- `-w, --workdir` - Working directory inside container

**Examples:**
```bash
# Execute bash in running container
docker exec -it my-nginx bash

# Run command as specific user
docker exec -u www-data my-nginx whoami

# Run command in specific directory
docker exec -w /var/log my-nginx ls -la
```

---

#### `docker attach`
**Purpose:** Attach local standard input, output, and error streams to a running container

**Format:**
```bash
docker attach [OPTIONS] CONTAINER
```

**Common Options:**
- `--detach-keys` - Override key sequence for detaching
- `--no-stdin` - Do not attach STDIN
- `--sig-proxy` - Proxy received signals to process (default true)

**Examples:**
```bash
# Attach to container
docker attach my-nginx

# Attach without STDIN
docker attach --no-stdin my-nginx
```

---

#### `docker cp`
**Purpose:** Copy files/folders between container and local filesystem

**Format:**
```bash
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH
docker cp [OPTIONS] SRC_PATH CONTAINER:DEST_PATH
```

**Common Options:**
- `-a, --archive` - Archive mode (copy all uid/gid information)
- `-L, --follow-link` - Follow symbolic links in SRC_PATH

**Examples:**
```bash
# Copy from container to host
docker cp my-nginx:/etc/nginx/nginx.conf ./nginx.conf

# Copy from host to container
docker cp ./index.html my-nginx:/usr/share/nginx/html/

# Copy directory
docker cp my-nginx:/var/log/nginx ./logs
```

---

#### `docker inspect`
**Purpose:** Return low-level information on Docker objects

**Format:**
```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
```

**Common Options:**
- `-f, --format` - Format output using Go template
- `-s, --size` - Display total file sizes (for containers)
- `--type` - Return JSON for specified type

**Examples:**
```bash
# Inspect container
docker inspect my-nginx

# Get specific field (IP address)
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-nginx

# Inspect image
docker inspect nginx:latest
```

---

#### `docker stats`
**Purpose:** Display a live stream of container resource usage statistics

**Format:**
```bash
docker stats [OPTIONS] [CONTAINER...]
```

**Common Options:**
- `-a, --all` - Show all containers (default shows running)
- `--no-stream` - Disable streaming stats (only first result)
- `--format` - Pretty-print stats using Go template

**Examples:**
```bash
# Show stats for all running containers
docker stats

# Show stats for specific container
docker stats my-nginx

# Show stats once (no streaming)
docker stats --no-stream
```

---

#### `docker top`
**Purpose:** Display running processes of a container

**Format:**
```bash
docker top CONTAINER [ps OPTIONS]
```

**Examples:**
```bash
# Show processes
docker top my-nginx

# Show processes with custom ps options
docker top my-nginx aux
```

---

#### `docker port`
**Purpose:** List port mappings or a specific mapping for container

**Format:**
```bash
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
```

**Examples:**
```bash
# Show all port mappings
docker port my-nginx

# Show specific port mapping
docker port my-nginx 80
```

---

### Image Management

#### `docker build`
**Purpose:** Build an image from a Dockerfile

**Format:**
```bash
docker build [OPTIONS] PATH | URL | -
```

**Common Options:**
- `-t, --tag` - Name and optionally tag (name:tag format)
- `-f, --file` - Name of Dockerfile (default is PATH/Dockerfile)
- `--build-arg` - Set build-time variables
- `--no-cache` - Do not use cache when building
- `--pull` - Always attempt to pull newer version of image
- `--target` - Set target build stage
- `--platform` - Set platform if server is multi-platform capable

**Examples:**
```bash
# Build image from current directory
docker build -t myapp:1.0 .

# Build with custom Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build arguments
docker build --build-arg VERSION=1.0 -t myapp .

# Build without cache
docker build --no-cache -t myapp .
```

---

#### `docker tag`
**Purpose:** Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

**Format:**
```bash
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

**Examples:**
```bash
# Tag an image
docker tag myapp:1.0 myapp:latest

# Tag for registry
docker tag myapp:1.0 myregistry.com/myapp:1.0
```

---

#### `docker push`
**Purpose:** Push an image or repository to a registry

**Format:**
```bash
docker push [OPTIONS] NAME[:TAG]
```

**Common Options:**
- `-a, --all-tags` - Push all tagged images
- `--disable-content-trust` - Skip image signing (default true)

**Examples:**
```bash
# Push to Docker Hub
docker push username/myapp:1.0

# Push to private registry
docker push myregistry.com/myapp:1.0

# Push all tags
docker push -a username/myapp
```

---

#### `docker save`
**Purpose:** Save one or more images to a tar archive

**Format:**
```bash
docker save [OPTIONS] IMAGE [IMAGE...]
```

**Common Options:**
- `-o, --output` - Write to file instead of STDOUT

**Examples:**
```bash
# Save image to tar file
docker save -o myapp.tar myapp:1.0

# Save multiple images
docker save -o images.tar myapp:1.0 nginx:latest

# Save and compress
docker save myapp:1.0 | gzip > myapp.tar.gz
```

---

#### `docker load`
**Purpose:** Load an image from a tar archive or STDIN

**Format:**
```bash
docker load [OPTIONS]
```

**Common Options:**
- `-i, --input` - Read from tar archive file instead of STDIN
- `-q, --quiet` - Suppress load output

**Examples:**
```bash
# Load image from tar file
docker load -i myapp.tar

# Load from compressed tar
docker load < myapp.tar.gz
```

---

#### `docker history`
**Purpose:** Show the history of an image

**Format:**
```bash
docker history [OPTIONS] IMAGE
```

**Common Options:**
- `-H, --human` - Print sizes in human readable format (default true)
- `--no-trunc` - Don't truncate output
- `-q, --quiet` - Only show image IDs

**Examples:**
```bash
# Show image history
docker history nginx

# Show full output without truncation
docker history --no-trunc nginx
```

---

### Volume Management

#### `docker volume create`
**Purpose:** Create a volume

**Format:**
```bash
docker volume create [OPTIONS] [VOLUME]
```

**Common Options:**
- `-d, --driver` - Specify volume driver (default "local")
- `--label` - Set metadata on volume
- `-o, --opt` - Set driver specific options

**Examples:**
```bash
# Create a volume
docker volume create my-vol

# Create with labels
docker volume create --label env=prod my-vol

# Create with driver options
docker volume create --driver local --opt type=nfs --opt device=:/path my-vol
```

---

#### `docker volume ls`
**Purpose:** List volumes

**Format:**
```bash
docker volume ls [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output based on conditions
- `--format` - Pretty-print volumes using Go template
- `-q, --quiet` - Only display volume names

**Examples:**
```bash
# List all volumes
docker volume ls

# List with filter
docker volume ls -f dangling=true

# List only names
docker volume ls -q
```

---

#### `docker volume inspect`
**Purpose:** Display detailed information on one or more volumes

**Format:**
```bash
docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Inspect volume
docker volume inspect my-vol

# Get specific field
docker volume inspect -f '{{.Mountpoint}}' my-vol
```

---

#### `docker volume rm`
**Purpose:** Remove one or more volumes

**Format:**
```bash
docker volume rm [OPTIONS] VOLUME [VOLUME...]
```

**Common Options:**
- `-f, --force` - Force removal of volume

**Examples:**
```bash
# Remove volume
docker volume rm my-vol

# Remove multiple volumes
docker volume rm vol1 vol2 vol3
```

---

#### `docker volume prune`
**Purpose:** Remove all unused local volumes

**Format:**
```bash
docker volume prune [OPTIONS]
```

**Common Options:**
- `-f, --force` - Do not prompt for confirmation
- `--filter` - Provide filter values

**Examples:**
```bash
# Remove all unused volumes
docker volume prune

# Force removal without prompt
docker volume prune -f
```

---

### Network Management

#### `docker network create`
**Purpose:** Create a network

**Format:**
```bash
docker network create [OPTIONS] NETWORK
```

**Common Options:**
- `-d, --driver` - Driver to manage network (default "bridge")
- `--subnet` - Subnet in CIDR format
- `--gateway` - IPv4 or IPv6 gateway
- `--ip-range` - Allocate container IP from sub-range
- `--label` - Set metadata on network
- `--internal` - Restrict external access to network

**Examples:**
```bash
# Create bridge network
docker network create my-network

# Create with subnet
docker network create --subnet=172.20.0.0/16 my-network

# Create isolated network
docker network create --internal my-internal-net
```

---

#### `docker network ls`
**Purpose:** List networks

**Format:**
```bash
docker network ls [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output based on conditions
- `--format` - Pretty-print networks using Go template
- `-q, --quiet` - Only display network IDs

**Examples:**
```bash
# List all networks
docker network ls

# List custom networks only
docker network ls -f type=custom
```

---

#### `docker network inspect`
**Purpose:** Display detailed information on one or more networks

**Format:**
```bash
docker network inspect [OPTIONS] NETWORK [NETWORK...]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Inspect network
docker network inspect my-network

# Get specific field
docker network inspect -f '{{.IPAM.Config}}' my-network
```

---

#### `docker network connect`
**Purpose:** Connect a container to a network

**Format:**
```bash
docker network connect [OPTIONS] NETWORK CONTAINER
```

**Common Options:**
- `--alias` - Add network-scoped alias for container
- `--ip` - IPv4 address
- `--ip6` - IPv6 address
- `--link` - Add link to another container

**Examples:**
```bash
# Connect container to network
docker network connect my-network my-container

# Connect with specific IP
docker network connect --ip 172.20.0.10 my-network my-container

# Connect with alias
docker network connect --alias db my-network postgres
```

---

#### `docker network disconnect`
**Purpose:** Disconnect a container from a network

**Format:**
```bash
docker network disconnect [OPTIONS] NETWORK CONTAINER
```

**Common Options:**
- `-f, --force` - Force disconnect

**Examples:**
```bash
# Disconnect container from network
docker network disconnect my-network my-container
```

---

#### `docker network rm`
**Purpose:** Remove one or more networks

**Format:**
```bash
docker network rm NETWORK [NETWORK...]
```

**Examples:**
```bash
# Remove network
docker network rm my-network

# Remove multiple networks
docker network rm net1 net2 net3
```

---

#### `docker network prune`
**Purpose:** Remove all unused networks

**Format:**
```bash
docker network prune [OPTIONS]
```

**Common Options:**
- `-f, --force` - Do not prompt for confirmation
- `--filter` - Provide filter values

**Examples:**
```bash
# Remove all unused networks
docker network prune

# Force without prompt
docker network prune -f
```

---

## **LEVEL 3: ADVANCED COMMANDS**

### System Management

#### `docker system df`
**Purpose:** Show docker disk usage

**Format:**
```bash
docker system df [OPTIONS]
```

**Common Options:**
- `-v, --verbose` - Show detailed information on space usage

**Examples:**
```bash
# Show disk usage
docker system df

# Show detailed usage
docker system df -v
```

---

#### `docker system prune`
**Purpose:** Remove unused data

**Format:**
```bash
docker system prune [OPTIONS]
```

**Common Options:**
- `-a, --all` - Remove all unused images, not just dangling ones
- `-f, --force` - Do not prompt for confirmation
- `--volumes` - Prune volumes
- `--filter` - Provide filter values

**Examples:**
```bash
# Remove all unused data
docker system prune

# Remove all unused images and volumes
docker system prune -a --volumes

# Prune older than 24 hours
docker system prune -a --filter "until=24h"
```

---

#### `docker system info`
**Purpose:** Display system-wide information

**Format:**
```bash
docker system info [OPTIONS]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Show system information
docker system info

# Get specific info
docker system info -f '{{.DriverStatus}}'
```

---

#### `docker system events`
**Purpose:** Get real-time events from the server

**Format:**
```bash
docker system events [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output based on conditions
- `--since` - Show events since timestamp
- `--until` - Stream events until timestamp
- `--format` - Format output using Go template

**Examples:**
```bash
# Stream all events
docker system events

# Filter container events
docker system events -f type=container

# Show events since 1 hour ago
docker system events --since 1h
```

---

### Docker Compose

#### `docker compose up`
**Purpose:** Create and start containers

**Format:**
```bash
docker compose up [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-d, --detach` - Detached mode
- `--build` - Build images before starting containers
- `--force-recreate` - Recreate containers even if config hasn't changed
- `--no-deps` - Don't start linked services
- `--scale` - Scale SERVICE to NUM instances
- `-V, --renew-anon-volumes` - Recreate anonymous volumes

**Examples:**
```bash
# Start services
docker compose up

# Start in detached mode
docker compose up -d

# Build and start
docker compose up --build

# Scale service
docker compose up --scale web=3
```

---

#### `docker compose down`
**Purpose:** Stop and remove containers, networks

**Format:**
```bash
docker compose down [OPTIONS]
```

**Common Options:**
- `-v, --volumes` - Remove named volumes
- `--rmi` - Remove images (type: 'all' or 'local')
- `--remove-orphans` - Remove containers for services not in compose file
- `-t, --timeout` - Shutdown timeout in seconds

**Examples:**
```bash
# Stop and remove containers
docker compose down

# Remove volumes too
docker compose down -v

# Remove all images
docker compose down --rmi all
```

---

#### `docker compose ps`
**Purpose:** List containers

**Format:**
```bash
docker compose ps [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-a, --all` - Show all containers (including stopped)
- `-q, --quiet` - Only display IDs
- `--services` - Display services

**Examples:**
```bash
# List running containers
docker compose ps

# List all containers
docker compose ps -a

# List only service names
docker compose ps --services
```

---

#### `docker compose logs`
**Purpose:** View output from containers

**Format:**
```bash
docker compose logs [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-f, --follow` - Follow log output
- `--tail` - Number of lines to show from end
- `-t, --timestamps` - Show timestamps
- `--no-log-prefix` - Don't print prefix in logs

**Examples:**
```bash
# View logs
docker compose logs

# Follow logs
docker compose logs -f

# Show last 100 lines
docker compose logs --tail=100

# Logs for specific service
docker compose logs web
```

---

#### `docker compose exec`
**Purpose:** Execute command in running container

**Format:**
```bash
docker compose exec [OPTIONS] SERVICE COMMAND [ARGS...]
```

**Common Options:**
- `-d, --detach` - Detached mode
- `-e, --env` - Set environment variables
- `-u, --user` - Run as specified username or uid
- `-w, --workdir` - Path to workdir directory

**Examples:**
```bash
# Execute bash in service
docker compose exec web bash

# Execute as specific user
docker compose exec -u www-data web whoami

# Execute with environment variable
docker compose exec -e DEBUG=1 web python script.py
```

---

#### `docker compose build`
**Purpose:** Build or rebuild services

**Format:**
```bash
docker compose build [OPTIONS] [SERVICE...]
```

**Common Options:**
- `--no-cache` - Do not use cache when building
- `--pull` - Always pull newer version of image
- `--parallel` - Build images in parallel
- `--build-arg` - Set build-time variables

**Examples:**
```bash
# Build all services
docker compose build

# Build without cache
docker compose build --no-cache

# Build specific service
docker compose build web

# Build with arguments
docker compose build --build-arg VERSION=1.0
```

---

#### `docker compose pull`
**Purpose:** Pull service images

**Format:**
```bash
docker compose pull [OPTIONS] [SERVICE...]
```

**Common Options:**
- `--ignore-pull-failures` - Pull what it can and ignore failures
- `--include-deps` - Also pull services declared as dependencies
- `-q, --quiet` - Pull without printing progress

**Examples:**
```bash
# Pull all images
docker compose pull

# Pull specific service
docker compose pull web

# Pull including dependencies
docker compose pull --include-deps web
```

---

#### `docker compose restart`
**Purpose:** Restart service containers

**Format:**
```bash
docker compose restart [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-t, --timeout` - Shutdown timeout in seconds

**Examples:**
```bash
# Restart all services
docker compose restart

# Restart specific service
docker compose restart web

# Restart with timeout
docker compose restart -t 30 web
```

---

#### `docker compose stop`
**Purpose:** Stop services

**Format:**
```bash
docker compose stop [OPTIONS] [SERVICE...]
```

**Common Options:**
- `-t, --timeout` - Shutdown timeout in seconds

**Examples:**
```bash
# Stop all services
docker compose stop

# Stop specific service
docker compose stop web
```

---

#### `docker compose start`
**Purpose:** Start services

**Format:**
```bash
docker compose start [SERVICE...]
```

**Examples:**
```bash
# Start all services
docker compose start

# Start specific service
docker compose start web
```

---

### Container Advanced Operations

#### `docker commit`
**Purpose:** Create a new image from container's changes

**Format:**
```bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

**Common Options:**
- `-a, --author` - Author name
- `-c, --change` - Apply Dockerfile instruction to created image
- `-m, --message` - Commit message
- `-p, --pause` - Pause container during commit (default true)

**Examples:**
```bash
# Commit container to image
docker commit my-container my-image:1.0

# Commit with message
docker commit -m "Added custom config" my-container my-image:1.0

# Commit with Dockerfile instruction
docker commit -c "ENV DEBUG=1" my-container my-image:1.0
```

---

#### `docker export`
**Purpose:** Export container's filesystem as tar archive

**Format:**
```bash
docker export [OPTIONS] CONTAINER
```

**Common Options:**
- `-o, --output` - Write to file instead of STDOUT

**Examples:**
```bash
# Export container
docker export my-container > container.tar

# Export to file
docker export -o container.tar my-container
```

---

#### `docker import`
**Purpose:** Import contents from tarball to create filesystem image

**Format:**
```bash
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

**Common Options:**
- `-c, --change` - Apply Dockerfile instruction to created image
- `-m, --message` - Set commit message

**Examples:**
```bash
# Import from tar
docker import container.tar myimage:latest

# Import from URL
docker import https://example.com/container.tar myimage:latest

# Import with changes
docker import -c "ENV PATH=/usr/bin" container.tar myimage:latest
```

---

#### `docker pause`
**Purpose:** Pause all processes within one or more containers

**Format:**
```bash
docker pause CONTAINER [CONTAINER...]
```

**Examples:**
```bash
# Pause container
docker pause my-container

# Pause multiple containers
docker pause container1 container2
```

---

#### `docker unpause`
**Purpose:** Unpause all processes within one or more containers

**Format:**
```bash
docker unpause CONTAINER [CONTAINER...]
```

**Examples:**
```bash
# Unpause container
docker unpause my-container
```

---

#### `docker rename`
**Purpose:** Rename a container

**Format:**
```bash
docker rename CONTAINER NEW_NAME
```

**Examples:**
```bash
# Rename container
docker rename old-name new-name
```

---

#### `docker update`
**Purpose:** Update configuration of one or more containers

**Format:**
```bash
docker update [OPTIONS] CONTAINER [CONTAINER...]
```

**Common Options:**
- `--cpu-shares` - CPU shares (relative weight)
- `--cpus` - Number of CPUs
- `--memory` - Memory limit
- `--memory-swap` - Total memory (memory + swap)
- `--restart` - Restart policy

**Examples:**
```bash
# Update memory limit
docker update --memory 512m my-container

# Update CPU shares
docker update --cpu-shares 512 my-container

# Update restart policy
docker update --restart unless-stopped my-container
```

---

#### `docker wait`
**Purpose:** Block until one or more containers stop, then print exit codes

**Format:**
```bash
docker wait CONTAINER [CONTAINER...]
```

**Examples:**
```bash
# Wait for container to stop
docker wait my-container

# Wait for multiple containers
docker wait container1 container2
```

---

### Image Advanced Operations

#### `docker image prune`
**Purpose:** Remove unused images

**Format:**
```bash
docker image prune [OPTIONS]
```

**Common Options:**
- `-a, --all` - Remove all unused images
- `-f, --force` - Do not prompt for confirmation
- `--filter` - Provide filter values

**Examples:**
```bash
# Remove dangling images
docker image prune

# Remove all unused images
docker image prune -a

# Remove images older than 24h
docker image prune -a --filter "until=24h"
```

---

#### `docker image inspect`
**Purpose:** Display detailed information on one or more images

**Format:**
```bash
docker image inspect [OPTIONS] IMAGE [IMAGE...]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Inspect image
docker image inspect nginx

# Get specific field
docker image inspect -f '{{.Config.ExposedPorts}}' nginx
```

---

#### `docker search`
**Purpose:** Search Docker Hub for images

**Format:**
```bash
docker search [OPTIONS] TERM
```

**Common Options:**
- `-f, --filter` - Filter output based on conditions
- `--limit` - Max number of search results (default 25)
- `--no-trunc` - Don't truncate output

**Examples:**
```bash
# Search for nginx images
docker search nginx

# Search with filter
docker search --filter stars=100 nginx

# Limit results
docker search --limit 5 nginx
```

---

### Registry & Login

#### `docker login`
**Purpose:** Log in to a Docker registry

**Format:**
```bash
docker login [OPTIONS] [SERVER]
```

**Common Options:**
- `-u, --username` - Username
- `-p, --password` - Password
- `--password-stdin` - Take password from stdin

**Examples:**
```bash
# Login to Docker Hub
docker login

# Login with credentials
docker login -u myuser -p mypassword

# Login to private registry
docker login myregistry.com

# Login with password from stdin
echo $MY_PASSWORD | docker login -u myuser --password-stdin
```

---

#### `docker logout`
**Purpose:** Log out from a Docker registry

**Format:**
```bash
docker logout [SERVER]
```

**Examples:**
```bash
# Logout from Docker Hub
docker logout

# Logout from private registry
docker logout myregistry.com
```

---

### Context Management

#### `docker context create`
**Purpose:** Create a context

**Format:**
```bash
docker context create [OPTIONS] CONTEXT
```

**Common Options:**
- `--description` - Description of context
- `--docker` - Set Docker endpoint
- `--default-stack-orchestrator` - Default orchestrator

**Examples:**
```bash
# Create context for remote Docker host
docker context create remote --docker "host=tcp://remote-host:2376"

# Create with description
docker context create remote --description "Remote Docker host" --docker "host=tcp://remote:2376"
```

---

#### `docker context ls`
**Purpose:** List contexts

**Format:**
```bash
docker context ls [OPTIONS]
```

**Common Options:**
- `--format` - Pretty-print contexts using Go template
- `-q, --quiet` - Only show context names

**Examples:**
```bash
# List all contexts
docker context ls

# Show only names
docker context ls -q
```

---

#### `docker context use`
**Purpose:** Set the current docker context

**Format:**
```bash
docker context use CONTEXT
```

**Examples:**
```bash
# Switch to context
docker context use remote

# Switch to default
docker context use default
```

---

#### `docker context inspect`
**Purpose:** Display detailed information on one or more contexts

**Format:**
```bash
docker context inspect [OPTIONS] [CONTEXT] [CONTEXT...]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Inspect context
docker context inspect remote

# Get specific field
docker context inspect -f '{{.Endpoints.docker.Host}}' remote
```

---

#### `docker context rm`
**Purpose:** Remove one or more contexts

**Format:**
```bash
docker context rm CONTEXT [CONTEXT...]
```

**Common Options:**
- `-f, --force` - Force removal

**Examples:**
```bash
# Remove context
docker context rm remote

# Force remove
docker context rm -f remote
```

---

#### `docker context update`
**Purpose:** Update a context

**Format:**
```bash
docker context update [OPTIONS] CONTEXT
```

**Common Options:**
- `--description` - Description of context
- `--docker` - Set Docker endpoint

**Examples:**
```bash
# Update context description
docker context update --description "Updated description" remote

# Update Docker endpoint
docker context update --docker "host=tcp://new-host:2376" remote
```

---

### Swarm Mode Commands

#### `docker swarm init`
**Purpose:** Initialize a swarm

**Format:**
```bash
docker swarm init [OPTIONS]
```

**Common Options:**
- `--advertise-addr` - Advertised address (format: <ip|interface>[:port])
- `--listen-addr` - Listen address (format: <ip|interface>[:port])
- `--data-path-addr` - Address for data path traffic
- `--force-new-cluster` - Force create new cluster from current state
- `--autolock` - Enable manager autolocking

**Examples:**
```bash
# Initialize swarm
docker swarm init

# Initialize with specific advertise address
docker swarm init --advertise-addr 192.168.1.100

# Initialize with autolock
docker swarm init --autolock
```

---

#### `docker swarm join`
**Purpose:** Join a swarm as a node and/or manager

**Format:**
```bash
docker swarm join [OPTIONS] HOST:PORT
```

**Common Options:**
- `--token` - Token for entry into swarm
- `--advertise-addr` - Advertised address
- `--listen-addr` - Listen address
- `--availability` - Availability of node (active|pause|drain)

**Examples:**
```bash
# Join as worker
docker swarm join --token SWMTKN-1-xxx 192.168.1.100:2377

# Join as manager
docker swarm join --token SWMTKN-1-xxx --advertise-addr 192.168.1.101 192.168.1.100:2377
```

---

#### `docker swarm leave`
**Purpose:** Leave the swarm

**Format:**
```bash
docker swarm leave [OPTIONS]
```

**Common Options:**
- `-f, --force` - Force leave ignoring warnings

**Examples:**
```bash
# Leave swarm
docker swarm leave

# Force leave (for managers)
docker swarm leave -f
```

---

#### `docker swarm join-token`
**Purpose:** Manage join tokens

**Format:**
```bash
docker swarm join-token [OPTIONS] (worker|manager)
```

**Common Options:**
- `-q, --quiet` - Only display token
- `--rotate` - Rotate join token

**Examples:**
```bash
# Get worker join token
docker swarm join-token worker

# Get manager join token
docker swarm join-token manager

# Rotate worker token
docker swarm join-token --rotate worker
```

---

#### `docker swarm update`
**Purpose:** Update the swarm

**Format:**
```bash
docker swarm update [OPTIONS]
```

**Common Options:**
- `--autolock` - Change manager autolocking setting
- `--cert-expiry` - Validity period for node certificates
- `--dispatcher-heartbeat` - Dispatcher heartbeat period
- `--task-history-limit` - Task history retention limit

**Examples:**
```bash
# Enable autolock
docker swarm update --autolock=true

# Update certificate expiry
docker swarm update --cert-expiry 720h

# Update task history limit
docker swarm update --task-history-limit 10
```

---

#### `docker swarm unlock`
**Purpose:** Unlock swarm

**Format:**
```bash
docker swarm unlock
```

**Examples:**
```bash
# Unlock swarm (will prompt for key)
docker swarm unlock
```

---

#### `docker swarm unlock-key`
**Purpose:** Manage unlock key

**Format:**
```bash
docker swarm unlock-key [OPTIONS]
```

**Common Options:**
- `-q, --quiet` - Only display token
- `--rotate` - Rotate unlock key

**Examples:**
```bash
# Get unlock key
docker swarm unlock-key

# Rotate unlock key
docker swarm unlock-key --rotate
```

---

### Service Management (Swarm)

#### `docker service create`
**Purpose:** Create a new service

**Format:**
```bash
docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]
```

**Common Options:**
- `--name` - Service name
- `--replicas` - Number of tasks
- `-p, --publish` - Publish port
- `-e, --env` - Set environment variables
- `--mount` - Attach filesystem mount
- `--network` - Network attachments
- `--constraint` - Placement constraints
- `--mode` - Service mode (replicated|global)
- `--update-delay` - Delay between updates
- `--rollback` - Rollback on update failure

**Examples:**
```bash
# Create service
docker service create --name web --replicas 3 nginx

# Create with port mapping
docker service create --name web --replicas 3 -p 80:80 nginx

# Create with constraints
docker service create --name web --constraint node.role==worker nginx

# Create global service
docker service create --name monitoring --mode global prometheus

# Create with mount
docker service create --name web --mount type=volume,source=data,target=/data nginx
```

---

#### `docker service ls`
**Purpose:** List services

**Format:**
```bash
docker service ls [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output based on conditions
- `--format` - Pretty-print services
- `-q, --quiet` - Only display IDs

**Examples:**
```bash
# List all services
docker service ls

# List with filter
docker service ls -f name=web

# List only IDs
docker service ls -q
```

---

#### `docker service ps`
**Purpose:** List tasks of one or more services

**Format:**
```bash
docker service ps [OPTIONS] SERVICE [SERVICE...]
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print tasks
- `-q, --quiet` - Only display task IDs
- `--no-trunc` - Don't truncate output

**Examples:**
```bash
# List service tasks
docker service ps web

# List with filter
docker service ps -f desired-state=running web

# Show all tasks (including stopped)
docker service ps -f desired-state=running -f desired-state=shutdown web
```

---

#### `docker service inspect`
**Purpose:** Display detailed information on one or more services

**Format:**
```bash
docker service inspect [OPTIONS] SERVICE [SERVICE...]
```

**Common Options:**
- `-f, --format` - Format output using Go template
- `--pretty` - Print information in human-friendly format

**Examples:**
```bash
# Inspect service
docker service inspect web

# Pretty print
docker service inspect --pretty web

# Get specific field
docker service inspect -f '{{.Spec.TaskTemplate.ContainerSpec.Image}}' web
```

---

#### `docker service logs`
**Purpose:** Fetch logs of a service or task

**Format:**
```bash
docker service logs [OPTIONS] SERVICE|TASK
```

**Common Options:**
- `-f, --follow` - Follow log output
- `--tail` - Number of lines to show
- `-t, --timestamps` - Show timestamps
- `--since` - Show logs since timestamp
- `--raw` - Don't neatly format logs

**Examples:**
```bash
# View service logs
docker service logs web

# Follow logs
docker service logs -f web

# Show last 100 lines
docker service logs --tail 100 web
```

---

#### `docker service scale`
**Purpose:** Scale one or multiple replicated services

**Format:**
```bash
docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]
```

**Examples:**
```bash
# Scale service to 5 replicas
docker service scale web=5

# Scale multiple services
docker service scale web=5 api=3 worker=10
```

---

#### `docker service update`
**Purpose:** Update a service

**Format:**
```bash
docker service update [OPTIONS] SERVICE
```

**Common Options:**
- `--image` - Service image tag
- `--replicas` - Number of tasks
- `--env-add` - Add environment variable
- `--env-rm` - Remove environment variable
- `--mount-add` - Add mount
- `--mount-rm` - Remove mount
- `--publish-add` - Add published port
- `--publish-rm` - Remove published port
- `--force` - Force update even if no changes
- `--rollback` - Rollback to previous specification
- `--update-parallelism` - Max number of tasks updated simultaneously

**Examples:**
```bash
# Update image
docker service update --image nginx:1.21 web

# Add environment variable
docker service update --env-add DEBUG=1 web

# Update replicas
docker service update --replicas 5 web

# Rollback service
docker service update --rollback web

# Force update (recreate tasks)
docker service update --force web
```

---

#### `docker service rm`
**Purpose:** Remove one or more services

**Format:**
```bash
docker service rm SERVICE [SERVICE...]
```

**Examples:**
```bash
# Remove service
docker service rm web

# Remove multiple services
docker service rm web api worker
```

---

#### `docker service rollback`
**Purpose:** Revert changes to service's configuration

**Format:**
```bash
docker service rollback [OPTIONS] SERVICE
```

**Common Options:**
- `-d, --detach` - Exit immediately instead of waiting
- `-q, --quiet` - Suppress progress output

**Examples:**
```bash
# Rollback service
docker service rollback web

# Rollback in detached mode
docker service rollback -d web
```

---

### Node Management (Swarm)

#### `docker node ls`
**Purpose:** List nodes in the swarm

**Format:**
```bash
docker node ls [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print nodes
- `-q, --quiet` - Only display IDs

**Examples:**
```bash
# List all nodes
docker node ls

# List manager nodes
docker node ls -f role=manager

# List available nodes
docker node ls -f availability=active
```

---

#### `docker node inspect`
**Purpose:** Display detailed information on one or more nodes

**Format:**
```bash
docker node inspect [OPTIONS] NODE [NODE...]
```

**Common Options:**
- `-f, --format` - Format output using Go template
- `--pretty` - Print in human-friendly format

**Examples:**
```bash
# Inspect node
docker node inspect node1

# Pretty print
docker node inspect --pretty node1

# Get specific field
docker node inspect -f '{{.Status.State}}' node1
```

---

#### `docker node update`
**Purpose:** Update a node

**Format:**
```bash
docker node update [OPTIONS] NODE
```

**Common Options:**
- `--availability` - Availability (active|pause|drain)
- `--label-add` - Add node label
- `--label-rm` - Remove node label
- `--role` - Node role (worker|manager)

**Examples:**
```bash
# Drain node for maintenance
docker node update --availability drain node1

# Add label
docker node update --label-add type=ssd node1

# Promote to manager
docker node update --role manager node1
```

---

#### `docker node promote`
**Purpose:** Promote one or more nodes to manager

**Format:**
```bash
docker node promote NODE [NODE...]
```

**Examples:**
```bash
# Promote node to manager
docker node promote node1

# Promote multiple nodes
docker node promote node1 node2
```

---

#### `docker node demote`
**Purpose:** Demote one or more nodes from manager

**Format:**
```bash
docker node demote NODE [NODE...]
```

**Examples:**
```bash
# Demote manager to worker
docker node demote node1
```

---

#### `docker node rm`
**Purpose:** Remove one or more nodes from the swarm

**Format:**
```bash
docker node rm [OPTIONS] NODE [NODE...]
```

**Common Options:**
- `-f, --force` - Force remove node

**Examples:**
```bash
# Remove node
docker node rm node1

# Force remove
docker node rm -f node1
```

---

#### `docker node ps`
**Purpose:** List tasks running on one or more nodes

**Format:**
```bash
docker node ps [OPTIONS] [NODE...]
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print tasks
- `-q, --quiet` - Only display task IDs

**Examples:**
```bash
# List tasks on current node
docker node ps

# List tasks on specific node
docker node ps node1

# List running tasks
docker node ps -f desired-state=running node1
```

---

### Stack Management

#### `docker stack deploy`
**Purpose:** Deploy a new stack or update an existing stack

**Format:**
```bash
docker stack deploy [OPTIONS] STACK
```

**Common Options:**
- `-c, --compose-file` - Path to Compose file(s)
- `--prune` - Prune services no longer referenced
- `--resolve-image` - Query registry to resolve image digest
- `--with-registry-auth` - Send registry authentication

**Examples:**
```bash
# Deploy stack from compose file
docker stack deploy -c docker-compose.yml myapp

# Deploy with multiple compose files
docker stack deploy -c docker-compose.yml -c docker-compose.prod.yml myapp

# Deploy and prune old services
docker stack deploy -c docker-compose.yml --prune myapp
```

---

#### `docker stack ls`
**Purpose:** List stacks

**Format:**
```bash
docker stack ls [OPTIONS]
```

**Common Options:**
- `--format` - Pretty-print stacks

**Examples:**
```bash
# List all stacks
docker stack ls

# Format output
docker stack ls --format "{{.Name}}: {{.Services}} services"
```

---

#### `docker stack ps`
**Purpose:** List tasks in the stack

**Format:**
```bash
docker stack ps [OPTIONS] STACK
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print tasks
- `-q, --quiet` - Only display task IDs
- `--no-trunc` - Don't truncate output

**Examples:**
```bash
# List stack tasks
docker stack ps myapp

# List running tasks
docker stack ps -f desired-state=running myapp

# Show task IDs only
docker stack ps -q myapp
```

---

#### `docker stack services`
**Purpose:** List services in the stack

**Format:**
```bash
docker stack services [OPTIONS] STACK
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print services
- `-q, --quiet` - Only display IDs

**Examples:**
```bash
# List stack services
docker stack services myapp

# Filter by name
docker stack services -f name=web myapp

# Show service IDs only
docker stack services -q myapp
```

---

#### `docker stack rm`
**Purpose:** Remove one or more stacks

**Format:**
```bash
docker stack rm STACK [STACK...]
```

**Examples:**
```bash
# Remove stack
docker stack rm myapp

# Remove multiple stacks
docker stack rm myapp myapp2
```

---

### Config Management (Swarm)

#### `docker config create`
**Purpose:** Create a config from a file or STDIN

**Format:**
```bash
docker config create [OPTIONS] CONFIG file|-
```

**Common Options:**
- `-l, --label` - Config labels
- `--template-driver` - Template driver (golang|none)

**Examples:**
```bash
# Create config from file
docker config create nginx-config nginx.conf

# Create from STDIN
cat nginx.conf | docker config create nginx-config -

# Create with label
docker config create --label env=prod nginx-config nginx.conf
```

---

#### `docker config ls`
**Purpose:** List configs

**Format:**
```bash
docker config ls [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print configs
- `-q, --quiet` - Only display IDs

**Examples:**
```bash
# List all configs
docker config ls

# List with filter
docker config ls -f label=env=prod

# Show IDs only
docker config ls -q
```

---

#### `docker config inspect`
**Purpose:** Display detailed information on one or more configs

**Format:**
```bash
docker config inspect [OPTIONS] CONFIG [CONFIG...]
```

**Common Options:**
- `-f, --format` - Format output using Go template
- `--pretty` - Print in human-friendly format

**Examples:**
```bash
# Inspect config
docker config inspect nginx-config

# Pretty print
docker config inspect --pretty nginx-config

# Get specific field
docker config inspect -f '{{.Spec.Name}}' nginx-config
```

---

#### `docker config rm`
**Purpose:** Remove one or more configs

**Format:**
```bash
docker config rm CONFIG [CONFIG...]
```

**Examples:**
```bash
# Remove config
docker config rm nginx-config

# Remove multiple configs
docker config rm config1 config2
```

---

### Secret Management (Swarm)

#### `docker secret create`
**Purpose:** Create a secret from a file or STDIN

**Format:**
```bash
docker secret create [OPTIONS] SECRET file|-
```

**Common Options:**
- `-l, --label` - Secret labels
- `--template-driver` - Template driver

**Examples:**
```bash
# Create secret from file
docker secret create db-password password.txt

# Create from STDIN
echo "my_secret_password" | docker secret create db-password -

# Create with label
docker secret create --label env=prod db-password password.txt
```

---

#### `docker secret ls`
**Purpose:** List secrets

**Format:**
```bash
docker secret ls [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print secrets
- `-q, --quiet` - Only display IDs

**Examples:**
```bash
# List all secrets
docker secret ls

# List with filter
docker secret ls -f label=env=prod

# Show IDs only
docker secret ls -q
```

---

#### `docker secret inspect`
**Purpose:** Display detailed information on one or more secrets

**Format:**
```bash
docker secret inspect [OPTIONS] SECRET [SECRET...]
```

**Common Options:**
- `-f, --format` - Format output using Go template
- `--pretty` - Print in human-friendly format

**Examples:**
```bash
# Inspect secret
docker secret inspect db-password

# Pretty print
docker secret inspect --pretty db-password

# Get specific field
docker secret inspect -f '{{.Spec.Name}}' db-password
```

---

#### `docker secret rm`
**Purpose:** Remove one or more secrets

**Format:**
```bash
docker secret rm SECRET [SECRET...]
```

**Examples:**
```bash
# Remove secret
docker secret rm db-password

# Remove multiple secrets
docker secret rm secret1 secret2
```

---

### Plugin Management

#### `docker plugin install`
**Purpose:** Install a plugin

**Format:**
```bash
docker plugin install [OPTIONS] PLUGIN [KEY=VALUE...]
```

**Common Options:**
- `--alias` - Local name for plugin
- `--disable` - Don't enable plugin on install
- `--grant-all-permissions` - Grant all permissions

**Examples:**
```bash
# Install plugin
docker plugin install vieux/sshfs

# Install with alias
docker plugin install --alias sshfs vieux/sshfs

# Install with grant all permissions
docker plugin install --grant-all-permissions vieux/sshfs
```

---

#### `docker plugin ls`
**Purpose:** List plugins

**Format:**
```bash
docker plugin ls [OPTIONS]
```

**Common Options:**
- `-f, --filter` - Filter output
- `--format` - Pretty-print plugins
- `-q, --quiet` - Only display plugin IDs

**Examples:**
```bash
# List all plugins
docker plugin ls

# List enabled plugins
docker plugin ls -f enabled=true
```

---

#### `docker plugin enable`
**Purpose:** Enable a plugin

**Format:**
```bash
docker plugin enable [OPTIONS] PLUGIN
```

**Common Options:**
- `--timeout` - HTTP client timeout (seconds)

**Examples:**
```bash
# Enable plugin
docker plugin enable sshfs
```

---

#### `docker plugin disable`
**Purpose:** Disable a plugin

**Format:**
```bash
docker plugin disable [OPTIONS] PLUGIN
```

**Common Options:**
- `-f, --force` - Force disable of active plugin

**Examples:**
```bash
# Disable plugin
docker plugin disable sshfs

# Force disable
docker plugin disable -f sshfs
```

---

#### `docker plugin inspect`
**Purpose:** Display detailed information on one or more plugins

**Format:**
```bash
docker plugin inspect [OPTIONS] PLUGIN [PLUGIN...]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Inspect plugin
docker plugin inspect sshfs

# Get specific field
docker plugin inspect -f '{{.Enabled}}' sshfs
```

---

#### `docker plugin rm`
**Purpose:** Remove one or more plugins

**Format:**
```bash
docker plugin rm [OPTIONS] PLUGIN [PLUGIN...]
```

**Common Options:**
- `-f, --force` - Force removal of active plugin

**Examples:**
```bash
# Remove plugin
docker plugin rm sshfs

# Force remove
docker plugin rm -f sshfs
```

---

#### `docker plugin upgrade`
**Purpose:** Upgrade an existing plugin

**Format:**
```bash
docker plugin upgrade [OPTIONS] PLUGIN [REMOTE]
```

**Common Options:**
- `--disable-content-trust` - Skip image verification
- `--grant-all-permissions` - Grant all permissions
- `--skip-remote-check` - Don't check if image is newer

**Examples:**
```bash
# Upgrade plugin
docker plugin upgrade sshfs vieux/sshfs:latest

# Upgrade with all permissions
docker plugin upgrade --grant-all-permissions sshfs vieux/sshfs:latest
```

---

### Trust & Content Management

#### `docker trust sign`
**Purpose:** Sign an image

**Format:**
```bash
docker trust sign IMAGE:TAG
```

**Examples:**
```bash
# Sign image
docker trust sign myregistry.com/myimage:1.0
```

---

#### `docker trust revoke`
**Purpose:** Remove trust for an image

**Format:**
```bash
docker trust revoke [OPTIONS] IMAGE:TAG
```

**Common Options:**
- `-y, --yes` - Don't prompt for confirmation

**Examples:**
```bash
# Revoke trust
docker trust revoke myregistry.com/myimage:1.0

# Revoke without prompt
docker trust revoke -y myregistry.com/myimage:1.0
```

---

#### `docker trust inspect`
**Purpose:** Return low-level information about keys and signatures

**Format:**
```bash
docker trust inspect IMAGE[:TAG] [IMAGE[:TAG]...]
```

**Common Options:**
- `--pretty` - Print information in human-friendly format

**Examples:**
```bash
# Inspect trust data
docker trust inspect myregistry.com/myimage:1.0

# Pretty print
docker trust inspect --pretty myregistry.com/myimage:1.0
```

---

### Manifest Commands

#### `docker manifest create`
**Purpose:** Create a local manifest list for multi-platform images

**Format:**
```bash
docker manifest create MANIFEST_LIST MANIFEST [MANIFEST...]
```

**Common Options:**
- `-a, --amend` - Amend an existing manifest list
- `--insecure` - Allow communication with insecure registry

**Examples:**
```bash
# Create manifest list
docker manifest create myapp:latest myapp:amd64 myapp:arm64

# Amend existing manifest
docker manifest create -a myapp:latest myapp:ppc64le
```

---

#### `docker manifest annotate`
**Purpose:** Add additional information to a local image manifest

**Format:**
```bash
docker manifest annotate [OPTIONS] MANIFEST_LIST MANIFEST
```

**Common Options:**
- `--os` - Set operating system
- `--arch` - Set architecture
- `--variant` - Set architecture variant
- `--os-version` - Set operating system version

**Examples:**
```bash
# Annotate manifest
docker manifest annotate myapp:latest myapp:arm64 --os linux --arch arm64

# Annotate with variant
docker manifest annotate myapp:latest myapp:armv7 --os linux --arch arm --variant v7
```

---

#### `docker manifest inspect`
**Purpose:** Display an image manifest or manifest list

**Format:**
```bash
docker manifest inspect [OPTIONS] [MANIFEST_LIST] MANIFEST
```

**Common Options:**
- `--insecure` - Allow communication with insecure registry
- `-v, --verbose` - Verbose output

**Examples:**
```bash
# Inspect manifest
docker manifest inspect myapp:latest

# Verbose inspect
docker manifest inspect -v myapp:latest
```

---

#### `docker manifest push`
**Purpose:** Push a manifest list to a repository

**Format:**
```bash
docker manifest push [OPTIONS] MANIFEST_LIST
```

**Common Options:**
- `--insecure` - Allow push to insecure registry
- `-p, --purge` - Remove local manifest after push

**Examples:**
```bash
# Push manifest list
docker manifest push myapp:latest

# Push and purge local
docker manifest push -p myapp:latest
```

---

#### `docker manifest rm`
**Purpose:** Delete one or more manifest lists from local storage

**Format:**
```bash
docker manifest rm MANIFEST_LIST [MANIFEST_LIST...]
```

**Examples:**
```bash
# Remove manifest list
docker manifest rm myapp:latest

# Remove multiple manifests
docker manifest rm myapp:latest myapp:v1.0
```

---

### Checkpoint Commands (Experimental)

#### `docker checkpoint create`
**Purpose:** Create a checkpoint from a running container

**Format:**
```bash
docker checkpoint create [OPTIONS] CONTAINER CHECKPOINT
```

**Common Options:**
- `--checkpoint-dir` - Use custom checkpoint storage directory
- `--leave-running` - Leave container running after checkpoint

**Examples:**
```bash
# Create checkpoint
docker checkpoint create my-container checkpoint1

# Create and keep running
docker checkpoint create --leave-running my-container checkpoint1
```

---

#### `docker checkpoint ls`
**Purpose:** List checkpoints for a container

**Format:**
```bash
docker checkpoint ls [OPTIONS] CONTAINER
```

**Common Options:**
- `--checkpoint-dir` - Use custom checkpoint storage directory

**Examples:**
```bash
# List checkpoints
docker checkpoint ls my-container
```

---

#### `docker checkpoint rm`
**Purpose:** Remove a checkpoint

**Format:**
```bash
docker checkpoint rm [OPTIONS] CONTAINER CHECKPOINT
```

**Common Options:**
- `--checkpoint-dir` - Use custom checkpoint storage directory

**Examples:**
```bash
# Remove checkpoint
docker checkpoint rm my-container checkpoint1
```

---

## **LEVEL 4: EXPERT & OPTIMIZATION COMMANDS**

### Advanced Build Commands

#### `docker buildx create`
**Purpose:** Create a new builder instance

**Format:**
```bash
docker buildx create [OPTIONS] [CONTEXT|ENDPOINT]
```

**Common Options:**
- `--name` - Builder instance name
- `--driver` - Driver to use (docker-container|kubernetes|remote)
- `--platform` - Fixed platforms for current node
- `--use` - Set as current builder
- `--bootstrap` - Boot builder after creation

**Examples:**
```bash
# Create builder
docker buildx create --name mybuilder

# Create and use
docker buildx create --name mybuilder --use

# Create with platforms
docker buildx create --name multibuilder --platform linux/amd64,linux/arm64

# Create and bootstrap
docker buildx create --name mybuilder --bootstrap
```

---

#### `docker buildx use`
**Purpose:** Set the current builder instance

**Format:**
```bash
docker buildx use [OPTIONS] NAME
```

**Common Options:**
- `--default` - Set as default builder
- `--global` - Builder persists across builds

**Examples:**
```bash
# Use builder
docker buildx use mybuilder

# Set as default
docker buildx use --default mybuilder
```

---

#### `docker buildx build`
**Purpose:** Build images with BuildKit

**Format:**
```bash
docker buildx build [OPTIONS] PATH | URL | -
```

**Common Options:**
- `--platform` - Set target platform for build
- `--push` - Push to registry
- `--load` - Load image to docker
- `--cache-from` - External cache sources
- `--cache-to` - Cache export destinations
- `--output` - Output destination
- `--progress` - Progress output type (auto|plain|tty)
- `--secret` - Secret file to expose to build

**Examples:**
```bash
# Build for multiple platforms
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .

# Build and push
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest --push .

# Build with cache
docker buildx build --cache-from type=registry,ref=myapp:cache --cache-to type=registry,ref=myapp:cache -t myapp:latest .

# Build with secrets
docker buildx build --secret id=mysecret,src=secret.txt -t myapp:latest .

# Build with specific output
docker buildx build --output type=tar,dest=output.tar -t myapp:latest .
```

---

#### `docker buildx inspect`
**Purpose:** Inspect current builder instance

**Format:**
```bash
docker buildx inspect [NAME]
```

**Common Options:**
- `--bootstrap` - Ensure builder has booted before inspecting

**Examples:**
```bash
# Inspect current builder
docker buildx inspect

# Inspect specific builder
docker buildx inspect mybuilder

# Inspect and bootstrap
docker buildx inspect --bootstrap mybuilder
```

---

#### `docker buildx ls`
**Purpose:** List builder instances

**Format:**
```bash
docker buildx ls
```

**Examples:**
```bash
# List all builders
docker buildx ls
```

---

#### `docker buildx rm`
**Purpose:** Remove a builder instance

**Format:**
```bash
docker buildx rm [OPTIONS] [NAME]
```

**Common Options:**
- `--all-inactive` - Remove all inactive builders
- `--force` - Don't prompt for confirmation
- `--keep-state` - Keep BuildKit state

**Examples:**
```bash
# Remove builder
docker buildx rm mybuilder

# Remove all inactive
docker buildx rm --all-inactive

# Force remove
docker buildx rm --force mybuilder
```

---

#### `docker buildx prune`
**Purpose:** Remove build cache

**Format:**
```bash
docker buildx prune [OPTIONS]
```

**Common Options:**
- `-a, --all` - Remove all cache, not just dangling
- `-f, --force` - Don't prompt for confirmation
- `--filter` - Filter values
- `--keep-storage` - Amount of disk space to keep for cache

**Examples:**
```bash
# Prune build cache
docker buildx prune

# Prune all cache
docker buildx prune -a

# Prune with filter
docker buildx prune --filter until=24h

# Keep minimum storage
docker buildx prune --keep-storage 10GB
```

---

#### `docker buildx bake`
**Purpose:** Build from a file

**Format:**
```bash
docker buildx bake [OPTIONS] [TARGET...]
```

**Common Options:**
- `-f, --file` - Build definition file
- `--print` - Print options without building
- `--push` - Push to registry
- `--load` - Load to docker
- `--set` - Override target value

**Examples:**
```bash
# Bake from default file
docker buildx bake

# Bake specific target
docker buildx bake webapp

# Bake and push
docker buildx bake --push

# Bake with custom file
docker buildx bake -f docker-bake.hcl

# Override values
docker buildx bake --set webapp.tags=myapp:dev
```

---

#### `docker buildx imagetools create`
**Purpose:** Create a new image based on source images

**Format:**
```bash
docker buildx imagetools create [OPTIONS] [SOURCE] [SOURCE...]
```

**Common Options:**
- `-t, --tag` - Set reference for new image
- `--append` - Append to existing manifest
- `--dry-run` - Show what would be done

**Examples:**
```bash
# Create multi-arch image
docker buildx imagetools create -t myapp:latest myapp:amd64 myapp:arm64

# Append to existing
docker buildx imagetools create --append -t myapp:latest myapp:ppc64le

# Dry run
docker buildx imagetools create --dry-run -t myapp:latest myapp:amd64 myapp:arm64
```

---

#### `docker buildx imagetools inspect`
**Purpose:** Show details of image in registry

**Format:**
```bash
docker buildx imagetools inspect [OPTIONS] NAME
```

**Common Options:**
- `--raw` - Show original, unformatted JSON manifest

**Examples:**
```bash
# Inspect image
docker buildx imagetools inspect myapp:latest

# Show raw manifest
docker buildx imagetools inspect --raw myapp:latest
```

---

### Advanced Container Runtime Options

#### `docker run` (Advanced Options)
**Advanced runtime configurations:**

**Resource Constraints:**
- `--cpus` - Number of CPUs (e.g., 0.5, 2)
- `--cpu-shares` - CPU shares (relative weight)
- `--cpuset-cpus` - CPUs in which to allow execution (0-3, 0,1)
- `--memory, -m` - Memory limit
- `--memory-swap` - Total memory (memory + swap), -1 to enable unlimited swap
- `--memory-reservation` - Memory soft limit
- `--kernel-memory` - Kernel memory limit
- `--oom-kill-disable` - Disable OOM Killer
- `--oom-score-adj` - Tune host's OOM preferences (-1000 to 1000)
- `--pids-limit` - Tune container pids limit

**Storage Options:**
- `--storage-opt` - Storage driver options
- `--tmpfs` - Mount tmpfs directory
- `--volume-driver` - Optional volume driver
- `--volumes-from` - Mount volumes from specified container(s)

**Network Options:**
- `--network` - Connect to network
- `--network-alias` - Add network-scoped alias
- `--ip` - IPv4 address
- `--ip6` - IPv6 address
- `--dns` - Set custom DNS servers
- `--dns-search` - Set custom DNS search domains
- `--dns-option` - Set DNS options
- `--add-host` - Add custom host-to-IP mapping
- `--mac-address` - Container MAC address

**Security Options:**
- `--cap-add` - Add Linux capabilities
- `--cap-drop` - Drop Linux capabilities
- `--privileged` - Give extended privileges
- `--security-opt` - Security options
- `--userns` - User namespace to use
- `--read-only` - Mount root filesystem as read-only
- `--user, -u` - Username or UID
- `--group-add` - Add additional groups

**Device Options:**
- `--device` - Add host device to container
- `--device-read-bps` - Limit read rate from device
- `--device-write-bps` - Limit write rate to device
- `--device-read-iops` - Limit read rate (I/O per second) from device
- `--device-write-iops` - Limit write rate (I/O per second) to device

**Restart & Health:**
- `--restart` - Restart policy (no|on-failure[:max-retries]|always|unless-stopped)
- `--health-cmd` - Command to run to check health
- `--health-interval` - Time between running check
- `--health-timeout` - Maximum time to allow check to run
- `--health-retries` - Consecutive failures needed to report unhealthy
- `--health-start-period` - Start period for container to initialize

**Logging Options:**
- `--log-driver` - Logging driver
- `--log-opt` - Log driver options

**Runtime & Init:**
- `--runtime` - Runtime to use
- `--init` - Run init inside container to forward signals

**Examples:**
```bash
# Run with CPU and memory limits
docker run -d --cpus="1.5" --memory="512m" nginx

# Run with specific CPUs
docker run -d --cpuset-cpus="0,1" nginx

# Run with tmpfs mount
docker run -d --tmpfs /tmp:rw,size=100m nginx

# Run with custom DNS
docker run -d --dns 8.8.8.8 --dns 8.8.4.4 nginx

# Run with added capabilities
docker run -d --cap-add=NET_ADMIN nginx

# Run privileged
docker run -d --privileged nginx

# Run with device
docker run -d --device=/dev/sda:/dev/xvdc nginx

# Run with restart policy
docker run -d --restart=unless-stopped nginx

# Run with health check
docker run -d --health-cmd="curl -f http://localhost/ || exit 1" --health-interval=30s nginx

# Run with custom log driver
docker run -d --log-driver=json-file --log-opt max-size=10m --log-opt max-file=3 nginx

# Run with read-only root
docker run -d --read-only --tmpfs /tmp nginx

# Run with init
docker run -d --init nginx

# Run with resource limits
docker run -d \
  --cpus="2" \
  --memory="1g" \
  --memory-swap="2g" \
  --pids-limit=100 \
  --device-read-bps /dev/sda:1mb \
  --device-write-bps /dev/sda:1mb \
  nginx
```

---

### Container Diff & Changes

#### `docker diff`
**Purpose:** Inspect changes to files or directories on a container's filesystem

**Format:**
```bash
docker diff CONTAINER
```

**Output Types:**
- `A` - File or directory was added
- `D` - File or directory was deleted
- `C` - File or directory was changed

**Examples:**
```bash
# Show filesystem changes
docker diff my-container
```

---

### Advanced Image Operations

#### `docker image import`
**Purpose:** Import contents from tarball to create filesystem image

**Format:**
```bash
docker image import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
```

**Common Options:**
- `-c, --change` - Apply Dockerfile instruction
- `-m, --message` - Set commit message
- `--platform` - Set platform if server is multi-platform

**Examples:**
```bash
# Import from file
docker image import rootfs.tar myimage:latest

# Import with changes
docker image import -c "ENV PATH=/usr/local/bin:$PATH" rootfs.tar myimage:latest
```

---

### System and Debugging Commands

#### `docker version`
**Purpose:** Show Docker version information

**Format:**
```bash
docker version [OPTIONS]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Show version
docker version

# Show specific field
docker version -f '{{.Server.Version}}'
```

---

#### `docker info`
**Purpose:** Display system-wide information (alias for docker system info)

**Format:**
```bash
docker info [OPTIONS]
```

**Common Options:**
- `-f, --format` - Format output using Go template

**Examples:**
```bash
# Show system information
docker info

# Show specific info
docker info -f '{{.ContainersRunning}}'
```

---

### Advanced Filtering & Formatting

**Filter Syntax:**
Docker commands support filtering with `-f` or `--filter` flag. Common filter types:

**Container Filters:**
- `id` - Container ID
- `name` - Container name
- `label` - Label in key=value format
- `status` - Container status (created|restarting|running|removing|paused|exited|dead)
- `ancestor` - Image or descendant used to create container
- `before` - Containers created before this container
- `since` - Containers created since this container
- `volume` - Volume mounted in container
- `network` - Network connected to container
- `health` - Health status (starting|healthy|unhealthy|none)
- `expose` - Exposed or published port
- `publish` - Published port

**Image Filters:**
- `dangling` - Boolean (true|false)
- `label` - Label in key=value format
- `before` - Images created before given image
- `since` - Images created since given image
- `reference` - Image reference pattern

**Volume Filters:**
- `dangling` - Boolean (true|false)
- `driver` - Volume driver name
- `label` - Label in key=value format
- `name` - Volume name

**Network Filters:**
- `driver` - Network driver
- `id` - Network ID
- `label` - Label in key=value format
- `name` - Network name
- `scope` - Network scope (swarm|global|local)
- `type` - Network type (custom|builtin)

**Format Templates:**
Use Go template syntax with `--format` flag:

**Examples:**
```bash
# List containers by status
docker ps -a -f status=exited

# List containers with specific label
docker ps -f label=env=prod

# List containers on network
docker ps -f network=my-network

# List dangling images
docker images -f dangling=true

# List images by reference
docker images -f reference="*:latest"

# Format container output
docker ps --format "{{.ID}}: {{.Names}} ({{.Status}})"

# Format with table
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# Format image output
docker images --format "{{.Repository}}:{{.Tag}} ({{.Size}})"

# Complex filter combination
docker ps -a -f status=exited -f ancestor=nginx -f before=container1

# List containers exposing specific port
docker ps -f expose=80

# List healthy containers
docker ps -f health=healthy
```

---

### Performance & Resource Monitoring

#### Advanced docker stats usage
**Purpose:** Monitor container resources with custom output

**Examples:**
```bash
# Monitor with custom format
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# Monitor specific containers
docker stats container1 container2 container3

# Single snapshot (no streaming)
docker stats --no-stream --format "{{.Container}}: CPU={{.CPUPerc}} MEM={{.MemPerc}}"

# Monitor all containers including stopped
docker stats -a --no-stream
```

---

### Container Orchestration Patterns

#### Multi-stage health monitoring
**Purpose:** Advanced health check configurations

**Examples:**
```bash
# HTTP health check with retries
docker run -d \
  --name web \
  --health-cmd="curl -f http://localhost/health || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=40s \
  nginx

# Database health check
docker run -d \
  --name postgres \
  --health-cmd="pg_isready -U postgres || exit 1" \
  --health-interval=10s \
  --health-timeout=5s \
  --health-retries=5 \
  postgres

# Complex health check with script
docker run -d \
  --name app \
  --health-cmd="/health-check.sh" \
  --health-interval=30s \
  myapp
```

---

### Advanced Networking Patterns

#### Custom network with subnet and gateway
**Examples:**
```bash
# Create network with specific configuration
docker network create \
  --driver bridge \
  --subnet=172.28.0.0/16 \
  --ip-range=172.28.5.0/24 \
  --gateway=172.28.5.254 \
  --opt "com.docker.network.bridge.name"="docker1" \
  my-network

# Create macvlan network
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan-network

# Create overlay network for swarm
docker network create \
  --driver overlay \
  --attachable \
  --subnet=10.0.9.0/24 \
  my-overlay
```

---

### Advanced Volume Patterns

#### Volume with specific options
**Examples:**
```bash
# Create NFS volume
docker volume create \
  --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.100,rw \
  --opt device=:/path/to/dir \
  nfs-volume

# Create volume with size limit
docker volume create \
  --driver local \
  --opt type=tmpfs \
  --opt device=tmpfs \
  --opt o=size=100m,uid=1000 \
  temp-volume

# Create CIFS/SMB volume
docker volume create \
  --driver local \
  --opt type=cifs \
  --opt o=username=user,password=pass,vers=3.0 \
  --opt device=//server/share \
  smb-volume
```

---

### Security Hardening Commands

#### AppArmor and SELinux
**Examples:**
```bash
# Run with AppArmor profile
docker run -d --security-opt apparmor=docker-default nginx

# Run with custom AppArmor profile
docker run -d --security-opt apparmor=my-profile nginx

# Run with SELinux label
docker run -d --security-opt label=level:s0:c100,c200 nginx

# Disable SELinux labeling
docker run -d --security-opt label=disable nginx
```

---

#### Seccomp profiles
**Examples:**
```bash
# Run with default seccomp profile
docker run -d --security-opt seccomp=default.json nginx

# Run without seccomp (unconfined)
docker run -d --security-opt seccomp=unconfined nginx

# Run with custom seccomp profile
docker run -d --security-opt seccomp=/path/to/profile.json nginx
```

---

#### Capability management
**Examples:**
```bash
# Drop all capabilities and add specific ones
docker run -d --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx

# Add multiple capabilities
docker run -d \
  --cap-add=SYS_ADMIN \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_TIME \
  myapp

# Drop specific capabilities
docker run -d \
  --cap-drop=CHOWN \
  --cap-drop=DAC_OVERRIDE \
  --cap-drop=SETUID \
  nginx
```

---

#### User namespace remapping
**Examples:**
```bash
# Run with user namespace
docker run -d --userns=host nginx

# Run as specific user
docker run -d --user 1000:1000 nginx

# Run as named user
docker run -d --user www-data nginx

# Add supplementary groups
docker run -d --user www-data --group-add audio --group-add video myapp
```

---

### Debugging & Troubleshooting Commands

#### Container debugging patterns
**Examples:**
```bash
# Attach to container output
docker attach --sig-proxy=false my-container

# Execute debug shell
docker exec -it my-container /bin/bash

# Execute as root for debugging
docker exec -it --user root my-container bash

# View real-time events for container
docker events --filter container=my-container

# Inspect container network
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' my-container

# View container processes
docker top my-container aux

# Copy core dumps
docker cp my-container:/var/crash/core.dump ./core.dump

# Stream container stats
docker stats --no-stream my-container

# Check container exit code
docker inspect -f '{{.State.ExitCode}}' my-container
```

---

#### Image debugging
**Examples:**
```bash
# Inspect image layers
docker history --no-trunc myimage:latest

# Check image for vulnerabilities (if using Docker scan)
docker scan myimage:latest

# Export image filesystem
docker save myimage:latest | tar -x

# Inspect image configuration
docker inspect myimage:latest

# Check image size breakdown
docker history --human=false --format "{{.Size}}\t{{.CreatedBy}}" myimage:latest
```

---

### Build Optimization Techniques

#### Multi-stage build optimization
**Examples of advanced Dockerfile patterns:**

```dockerfile
# Multi-stage build for smaller images
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]

# Build with specific target
# docker build --target builder -t myapp:builder .
```

```dockerfile
# Build with cache mounts (BuildKit)
# syntax=docker/dockerfile:1
FROM golang:1.19 AS builder
WORKDIR /app
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    go mod download

# Build with secrets
# docker build --secret id=npm,src=$HOME/.npmrc .
FROM node:16
RUN --mount=type=secret,id=npm,target=/root/.npmrc \
    npm install -g some-private-package
```

---

### Advanced Docker Compose Patterns

#### Override and environment-specific configs
**Examples:**
```bash
# Use multiple compose files
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Override with environment variables
docker compose --env-file .env.prod up -d

# Specify project name
docker compose -p myproject up -d

# Run specific profiles
docker compose --profile debug up -d

# Validate compose file
docker compose config

# View configuration with variables resolved
docker compose config --resolve-image-digests

# Convert compose file to JSON
docker compose convert --format json

# Pull and build all services
docker compose pull && docker compose build
```

---

### Registry Operations

#### Advanced registry interactions
**Examples:**
```bash
# Push to specific registry
docker tag myapp:latest myregistry.com:5000/myapp:latest
docker push myregistry.com:5000/myapp:latest

# Pull from insecure registry (requires daemon config)
docker pull --platform linux/amd64 insecure-registry.local/myapp

# Search with filters
docker search --filter stars=50 --filter is-official=true nginx

# Login to multiple registries
docker login docker.io
docker login ghcr.io
docker login myregistry.com
```

---

### Backup & Migration Strategies

#### Container backup patterns
**Examples:**
```bash
# Backup container volumes
docker run --rm \
  --volumes-from my-container \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/volumes-backup.tar.gz /data

# Restore container volumes
docker run --rm \
  --volumes-from new-container \
  -v $(pwd):/backup \
  ubuntu tar xzf /backup/volumes-backup.tar.gz -C /

# Backup database container
docker exec postgres pg_dump -U user dbname > backup.sql

# Export all containers and images
docker save $(docker images -q) -o all-images.tar
docker export $(docker ps -aq) -o all-containers.tar

# Migration: save and load
docker save myapp:latest | gzip > myapp.tar.gz
# On new host:
gunzip -c myapp.tar.gz | docker load
```

---

### Performance Tuning

#### Container resource limits
**Examples:**
```bash
# CPU constraints
docker run -d \
  --cpus="2.5" \
  --cpu-shares=1024 \
  --cpuset-cpus="0-3" \
  --cpu-quota=50000 \
  --cpu-period=100000 \
  nginx

# Memory constraints
docker run -d \
  --memory="1g" \
  --memory-reservation="750m" \
  --memory-swap="2g" \
  --memory-swappiness=60 \
  --oom-kill-disable \
  nginx

# I/O constraints
docker run -d \
  --device=/dev/sda \
  --device-read-bps /dev/sda:10mb \
  --device-write-bps /dev/sda:5mb \
  --device-read-iops /dev/sda:1000 \
  --device-write-iops /dev/sda:500 \
  --blkio-weight=500 \
  nginx

# Update running container limits
docker update \
  --cpus="1.5" \
  --memory="768m" \
  --memory-swap="1.5g" \
  my-container
```

---

### Logging Strategies

#### Advanced logging configuration
**Examples:**
```bash
# JSON file logging with rotation
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=5 \
  --log-opt compress=true \
  nginx

# Syslog driver
docker run -d \
  --log-driver syslog \
  --log-opt syslog-address=tcp://192.168.1.100:514 \
  --log-opt tag="{{.Name}}/{{.ID}}" \
  nginx

# Fluentd driver
docker run -d \
  --log-driver fluentd \
  --log-opt fluentd-address=localhost:24224 \
  --log-opt tag="docker.{{.Name}}" \
  nginx

# GELF driver (Graylog)
docker run -d \
  --log-driver gelf \
  --log-opt gelf-address=udp://graylog.example.com:12201 \
  --log-opt tag="{{.Name}}" \
  nginx

# Disable logging
docker run -d --log-driver none nginx

# View logs from specific log driver
docker logs --since 1h my-container
```

---

## **ADVANCED TIPS & BEST PRACTICES**

### Command Chaining & Cleanup

**Useful one-liners:**
```bash
# Remove all stopped containers
docker rm $(docker ps -aq -f status=exited)

# Remove all dangling images
docker rmi $(docker images -qf dangling=true)

# Stop and remove all containers
docker stop $(docker ps -aq) && docker rm $(docker ps -aq)

# Remove all containers and volumes
docker rm -vf $(docker ps -aq)

# Remove all custom networks
docker network rm $(docker network ls -q -f type=custom)

# Complete cleanup
docker system prune -a --volumes -f

# Kill all running containers
docker kill $(docker ps -q)

# Remove images by pattern
docker rmi $(docker images | grep "pattern" | awk '{print $3}')

# Export all volumes
for v in $(docker volume ls -q); do
  docker run --rm -v $v:/data -v $(pwd):/backup alpine tar czf /backup/$v.tar.gz /data
done

# Get container IPs
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)
```

---

### Environment Variable Management

**Examples:**
```bash
# Load from env file
docker run -d --env-file production.env myapp

# Multiple environment variables
docker run -d \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgres://... \
  -e REDIS_URL=redis://... \
  myapp

# Environment from labels
docker run -d --label env=prod --label tier=backend myapp

# Pass through specific env vars
docker run -d -e HOME -e USER myapp
```

---

### Signal Handling

**Examples:**
```bash
# Stop with custom timeout
docker stop -t 60 my-container

# Send custom signal
docker kill --signal=SIGHUP my-container

# Graceful restart
docker kill --signal=SIGUSR1 my-container

# Force kill
docker kill --signal=SIGKILL my-container
```

---

### Development Workflows

**Local development patterns:**
```bash
# Development with live reload
docker run -d \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  --name dev-server \
  node:16 \
  npm run dev

# Run tests in container
docker run --rm \
  -v $(pwd):/app \
  -w /app \
  node:16 \
  npm test

# Interactive debugging
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -p 9229:9229 \
  node:16 \
  node --inspect=0.0.0.0:9229 index.js

# Build and test pipeline
docker build -t myapp:test --target test . && \
docker run --rm myapp:test && \
docker build -t myapp:latest .
```

---

## **CONCLUSION**

This comprehensive guide covers Docker commands from basic operations to advanced enterprise patterns. Key takeaways:

1. **Basic Level:** Master container lifecycle, images, and simple networking
2. **Intermediate Level:** Learn volume management, compose, and container interaction
3. **Advanced Level:** Utilize swarm, buildx, security hardening, and orchestration
4. **Expert Level:** Optimize performance, implement advanced patterns, and master debugging

**Best Practices Summary:**
- Always use specific image tags, not `latest`
- Implement health checks for production containers
- Use multi-stage builds to minimize image size
- Leverage BuildKit cache for faster builds
- Apply principle of least privilege with user namespaces and capabilities
- Monitor resource usage with `docker stats`
- Implement proper logging strategies
- Regular cleanup with `docker system prune`
- Use secrets management for sensitive data
- Document your configurations with compose files

**Continuous Learning:**
- Official documentation: https://docs.docker.com
- Docker Hub: https://hub.docker.com
- Docker Samples: https://github.com/docker/awesome-compose
- Security best practices: https://docs.docker.com/engine/security/

---

**Document Version:** 1.0  
**Last Updated:** 2025  
**Compatibility:** Docker Engine 20.10+ and Docker Compose v2
