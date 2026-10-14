# Docker Quick Notes

A concise reference guide for installing Docker and working with images, containers, volumes, networks, and Docker Compose on Ubuntu.

## Table of Contents

- [1. Install Docker on Ubuntu](#1-install-docker-on-ubuntu)
- [2. Verify Installation](#2-verify-installation)
- [3. Build and Manage Images](#3-build-and-manage-images)
- [4. Run and Manage Containers](#4-run-and-manage-containers)
- [5. Docker Volumes and Bind Mounts](#5-docker-volumes-and-bind-mounts)
- [6. Container Communication](#6-container-communication)
- [7. Docker Networking](#7-docker-networking)
- [8. Docker Compose](#8-docker-compose)
- [9. Useful Commands](#9-useful-commands)

---

## 1. Install Docker on Ubuntu

### Step 1: Update the system and install dependencies

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install ca-certificates curl
```

### Step 2: Add Docker's official GPG key

```bash
sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Step 3: Add the Docker APT repository

```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```

Update package information:

```bash
sudo apt update
```

### Step 4: Install Docker Engine

```bash
sudo apt install docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### Step 5: Check Docker status

```bash
sudo systemctl status docker
```

Start Docker if it is not running:

```bash
sudo systemctl start docker
```

---

## 2. Verify Installation

Run the test image to verify that Docker is installed correctly:

```bash
sudo docker run hello-world
```

If successful, Docker downloads and runs the `hello-world` image.

**Note:** The examples below use `docker`. Add `sudo` before commands if your user does not have permission to access the Docker daemon.

---

## 3. Build and Manage Images

A **Docker image** is a template used to create containers. A `Dockerfile` contains the instructions for building an image.

### Build an image

Run these commands from the directory containing your `Dockerfile`.

```bash
# Build an image
docker build .

# Build an image with a name and tag
docker build -t testapp:01 .

# List all images
docker images

# Alternative command
docker image ls
```

### Remove an image

```bash
docker rmi <IMAGE_ID>

# Remove an image using its repository and tag
docker rmi <REPOSITORY>:<TAG>
```

An image cannot normally be removed while a container still depends on it. Remove the relevant containers first if necessary.

### Push and pull images

```bash
# Download an image from a registry
docker pull <REPOSITORY>:<TAG>

# Upload an image to a registry
docker push <REPOSITORY>:<TAG>
```

Log in to your registry before pushing an image:

```bash
docker login
```

For Docker Hub, tag the image with your username before pushing (image name should be same as repository created at Docker Hub):

```bash
# docker tag <name1> <name2> -> changing image name from name1 to name2
docker tag testapp:01 <USERNAME>/testapp:01
docker push <USERNAME>/testapp:01
```

---

## 4. Run and Manage Containers

A **container** is a running or stopped instance of an image.

### Run a container

```bash
# Run an image
docker run <IMAGE_ID>

# Run interactively
docker run -it <IMAGE_ID>

# Run in detached mode (background)
docker run -d <IMAGE_ID>

# Run and automatically remove the container after it stops
docker run --rm <IMAGE_ID>
```

### Port mapping

Use `-p HOST_PORT:CONTAINER_PORT` to expose an application running inside a container.

```bash
docker run -p 3000:3000 <IMAGE_ID>

# Run in detached mode
docker run -d -p 3000:3000 <IMAGE_ID>

# Assign a custom container name
docker run -d --rm --name mytestapp -p 3000:3000 <IMAGE_ID>
```

To run multiple containers from the same image, map different host ports:

```bash
docker run -d -p 3000:3000 <IMAGE_ID>
docker run -d -p 3001:3000 <IMAGE_ID>
docker run -d -p 3002:3000 <IMAGE_ID>
```

All three containers listen on port `3000` internally, but are accessible through different host ports.

### List containers

```bash
# List running containers
docker ps

# List all containers, including stopped ones
docker ps -a
```

### Stop and remove containers

```bash
# Stop a container
docker stop <CONTAINER_NAME_OR_ID>

# Remove a stopped container
docker rm <CONTAINER_NAME_OR_ID>

# Remove multiple containers
docker rm <CONTAINER_ID_1> <CONTAINER_ID_2>

# Force-remove a running or stopped container
docker rm -f <CONTAINER_NAME_OR_ID>
```

**Important:** `docker rm` removes containers, not images. Use `docker rmi` to remove images.

---

## 5. Docker Volumes and Bind Mounts

Volumes and bind mounts allow data to persist beyond a container's lifecycle.

### Docker volumes

Volumes are managed by Docker and are useful for persistent application data, such as database files.

```bash
# Create a volume
docker volume create myvolume

# List volumes
docker volume ls

# Inspect a volume
docker volume inspect myvolume

# Mount a volume inside a container
docker run -it --rm -v myvolume:/testapp <IMAGE_ID>

# Display volume command options
docker volume --help
```

**Syntax:** `-v VOLUME_NAME:CONTAINER_PATH`

The volume remains available after the container is removed, unless the volume itself is deleted.

### Bind mounts

Bind mounts connect a specific file or directory on the host to a path inside the container.

```bash
docker run --rm -v <HOST_PATH>:<CONTAINER_PATH> <IMAGE_ID>
```

Example:

```bash
docker run --rm -v "$(pwd)/data:/testapp/data" <IMAGE_ID>
```

- `<HOST_PATH>` is the path on your machine.
- `<CONTAINER_PATH>` is the path inside the container.
- Changes can be shared between the host and container.
- The host path must exist or be handled appropriately by Docker.

---

## 6. Container Communication

Containers that need to communicate should generally use a shared Docker network.

### Run a MySQL container

```bash
docker run -d --rm \
  --name mysqldb \
  --env MYSQL_ROOT_PASSWORD=<YOUR_PASSWORD> \
  --env MYSQL_DATABASE=<DATABASE_NAME> \
  mysql:8
```

Replace the placeholders with your own values. Avoid committing real credentials to GitHub.

### Inspect a container

```bash
docker inspect mysqldb
```

This displays configuration details, network information, environment variables, and more.

**Important:** When applications communicate over a shared Docker network, use the destination container's name as the hostname rather than relying on its IP address.

For example, an application container can connect to MySQL using:

- **Hostname:** `mysqldb`
- **Port:** `3306`

---

## 7. Docker Networking

Docker networks allow containers to communicate with one another.

```bash
# Create a custom network
docker network create my-net

# Run a container on the network
docker run -d --rm \
  --name mysqldb \
  --network my-net \
  --env MYSQL_ROOT_PASSWORD=<YOUR_PASSWORD> \
  mysql:8

# Inspect a network
docker network inspect my-net

# List networks
docker network ls
```

To connect another container to the same network:

```bash
docker run -d --rm \
  --name myapp \
  --network my-net \
  <IMAGE_ID>
```

Containers attached to the same user-defined bridge network can generally discover each other by container name.

---

## 8. Docker Compose

Docker Compose lets you define and manage multiple services using a `compose.yaml` or `docker-compose.yml` file.

### Common commands

```bash
# Start services
docker compose up

# Start services in detached mode
docker compose up -d

# Stop and remove containers and networks
docker compose down

# Stop and remove containers, networks, and Compose-managed named volumes
docker compose down -v

# Run a one-off command in a service
docker compose run <SERVICE_NAME> <COMMAND>
```

**Note:** Use `docker compose` (the modern plugin syntax). The older `docker-compose` command may also work if the legacy standalone version is installed.

### Example workflow

1. Create a `compose.yaml` file defining your services.
2. Build and start the services:

   ```bash
   docker compose up --build -d
   ```

3. View running services:

   ```bash
   docker compose ps
   ```

4. View logs:

   ```bash
   docker compose logs -f
   ```

5. Stop and clean up:

   ```bash
   docker compose down
   ```

**Warning:** `docker compose down -v` deletes Compose-managed named volumes and can permanently remove stored database data.

---

## 9. Useful Commands

| Command | Purpose |
|---|---|
| `docker --version` | Check Docker version |
| `docker info` | View Docker system information |
| `docker images` | List images |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker logs <CONTAINER>` | View container logs |
| `docker exec -it <CONTAINER> bash` | Open a Bash shell inside a running container |
| `docker inspect <CONTAINER>` | Inspect container configuration |
| `docker system df` | Check Docker disk usage |
| `docker network ls` | List networks |
| `docker volume ls` | List volumes |
| `docker compose ps` | List Compose services |

---

## Key Concepts to Remember

- **Image:** A template used to create containers.
- **Container:** An isolated instance of an image.
- **Dockerfile:** Instructions for building an image.
- **Port mapping:** Makes a container's port accessible through a host port.
- **Volume:** Docker-managed persistent storage.
- **Bind mount:** Maps a host file or directory into a container.
- **Network:** Enables communication between containers.
- **Docker Compose:** Defines and manages multi-container applications.

## Official Documentation

- [Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Images](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/)
- [Docker Containers](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)
- [Docker Storage](https://docs.docker.com/engine/storage/)
- [Docker Networking](https://docs.docker.com/engine/network/)
- [Docker Compose](https://docs.docker.com/compose/)
