# Docker Notes

A beginner-friendly guide to installing Docker Desktop on Windows and practicing Docker images, containers, ports, volumes, networks, and Dockerfiles.

## 1. Install Docker Desktop

- Install **Docker Desktop for Windows**: https://www.docker.com/products/docker-desktop/
- Follow the installation instructions and start Docker Desktop.
- If prompted, enable the required WSL 2 features and restart your computer.
- Confirm Docker Desktop is running before using Docker commands.

### Verify the installation

Open Command Prompt, PowerShell, or the VS Code terminal:

```bash
docker --version
docker info
```

Example version output:

```text
Docker version 29.8.2, build 7fc2dff
```

Your installed version may be different. If `docker info` cannot connect to the daemon, make sure Docker Desktop is running.

## 2. Docker Hub

Docker Hub is a registry where you can find and share container images.

- Docker Hub: https://hub.docker.com/
- Official Nginx image: https://hub.docker.com/_/nginx

## 3. Clone the Docker Practice Project

Repository: https://github.com/axtionable-saas/cloudtrain-docker-projects

Clone the repository:

```bash
git clone https://github.com/axtionable-saas/cloudtrain-docker-projects
```

Change to the project directory:

```bash
cd cloudtrain-docker-projects
```

Open the folder in Visual Studio Code:

```bash
code .
```

If `code .` is not recognized, open the folder using **VS Code → File → Open Folder**.

## 4. Download and List Docker Images

### Pull the Nginx image

```bash
docker pull nginx:latest
```

- `nginx` is the image name.
- `latest` is the image tag. For reproducible environments, consider using a specific version tag instead of `latest`.

### List local images

```bash
docker images
```

Or:

```bash
docker image ls
```

## 5. Run a Container

Start an Nginx container in detached mode:

```bash
docker run -d nginx:latest
```

- `docker run` creates and starts a container from an image.
- `-d` runs it in the background (detached mode).
- Docker assigns a container ID and usually a generated name if no name is provided.

List containers:

```bash
docker ps
```

List all containers, including stopped containers:

```bash
docker ps -a
```

## 6. Container Lifecycle

Replace `<container-id>` with the actual container ID or container name. Do not type the angle brackets.

### Stop a running container

```bash
docker stop <container-id>
```

### Start a stopped container

```bash
docker start <container-id>
```

### Restart a container

```bash
docker restart <container-id>
```

### Remove a container

Stop it first if it is running, then remove it:

```bash
docker stop <container-id>
docker rm <container-id>
```

A stopped container can be removed directly.

### Inspect container details

```bash
docker inspect <container-id>
```

### View container logs

```bash
docker logs <container-id>
```

Follow logs continuously:

```bash
docker logs -f <container-id>
```

Press `Ctrl+C` to stop following logs; this does not stop the container.

### Open a shell inside a running container

```bash
docker exec -it <container-id> bash
```

Some minimal images do not include Bash. If Bash is unavailable, try:

```bash
docker exec -it <container-id> sh
```

Exit the container shell with:

```bash
exit
```

## 7. Port Forwarding

A container's ports are not automatically exposed to your host computer. Use `-p` to publish a container port.

Run Nginx with host port `8080` mapped to container port `80`:

```bash
docker run -d --name nginx-web -p 8080:80 nginx:latest
```

Open this URL in your browser:

http://localhost:8080

Port mapping format:

```text
-p HOST_PORT:CONTAINER_PORT
```

For example, `-p 8080:80` maps port `8080` on your computer to port `80` inside the container.

To remove this test container:

```bash
docker stop nginx-web
docker rm nginx-web
```

If host port `8080` is already in use, choose another host port, such as `8082:80`.

## 8. Docker Volumes and Host Bind Mounts

Volumes and bind mounts let containers access data that persists outside the container's writable layer.

### A. Named volume

Docker manages the volume's storage location.

Create a named volume:

```bash
docker volume create nginx-vol
```

List volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect nginx-vol
```

Run Nginx with the named volume mounted at `/nginx-container-vol`:

```bash
docker run -d --name nginx-volume-demo -p 8080:80 -v nginx-vol:/nginx-container-vol nginx:latest
```

The path `/nginx-container-vol` is an example mount path. The standard Nginx image serves its default website from `/usr/share/nginx/html`; mounting the volume at that path would affect the content Nginx serves.

For a website-content example using a named volume:

```bash
docker run -d --name nginx-volume-site -p 8082:80 -v nginx-vol:/usr/share/nginx/html nginx:latest
```

Note: a newly created empty volume mounted over `/usr/share/nginx/html` hides the image's default website files. Add your own `index.html` to the volume if you want to serve a page from it.

### B. Host bind mount

A bind mount maps a specific directory or file on your computer into the container. Make sure the host folder exists.

Example for Windows:

```bash
docker run -d --name nginx-bind-demo -p 8081:80 -v "C:\Users\Hp\cloudtrain-docker-projects\nginx-container-folder:/usr/share/nginx/html" nginx:latest
```

Create the `nginx-container-folder` directory and add an `index.html` file to see your own page at:

http://localhost:8081

In Docker Desktop, bind mounts use a Windows host path and a Linux container path separated by a colon. Quoting the path helps when it contains spaces.

### C. Remove a volume

A volume cannot be removed while it is in use by a container. Stop and remove any container using it, then run:

```bash
docker volume rm nginx-vol
```

To remove unused volumes, Docker also provides:

```bash
docker volume prune
```

Use prune carefully: it removes unused volumes that may contain data you still want.

## 9. Docker Networks

Docker networks allow containers to communicate with each other.

### List networks

```bash
docker network ls
```

Show help:

```bash
docker network --help
```

### Create a network

```bash
docker network create app-network
```

### Connect a container to a network

```bash
docker network connect app-network <container-id>
```

### Disconnect a container from a network

```bash
docker network disconnect app-network <container-id>
```

### Inspect a network

```bash
docker network inspect app-network
```

### Remove a network

```bash
docker network rm app-network
```

The network must not have connected containers when you remove it. For a simple test, run two containers on the same user-defined bridge network and use the other container's name as its hostname.

## 10. Build a Custom Docker Image Using a Dockerfile

A **Dockerfile** contains instructions Docker uses to build an image.

Example project structure:

```text
my-nginx-project/
├── Dockerfile
└── index.html
```

Create a file named `Dockerfile`:

```dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

Create `index.html` in the same directory:

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>My Docker Website</title>
</head>
<body>
  <h1>Hello from my custom Docker image!</h1>
</body>
</html>
```

Build the image from the directory containing the Dockerfile:

```bash
docker build -t my-nginx-app:v1 .
```

- `-t` assigns a name and tag to the image.
- `my-nginx-app` is the image name.
- `v1` is the tag.
- `.` is the build context (the current directory).

List images:

```bash
docker images
```

Run the custom image:

```bash
docker run -d --name my-nginx-container -p 8083:80 my-nginx-app:v1
```

Open:

http://localhost:8083

When finished:

```bash
docker stop my-nginx-container
docker rm my-nginx-container
```

### General Docker build command

```bash
docker build -t <project-name>:<tag-name> .
```

Replace the placeholders with your desired image name and tag.

## 11. Quick Reference

| Command | Purpose |
|---|---|
| `docker --version` | Show Docker CLI version |
| `docker pull nginx:latest` | Download an image |
| `docker images` | List local images |
| `docker run -d nginx:latest` | Start a container in the background |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker stop <container-id>` | Stop a container |
| `docker start <container-id>` | Start a stopped container |
| `docker restart <container-id>` | Restart a container |
| `docker rm <container-id>` | Remove a container |
| `docker logs -f <container-id>` | Follow container logs |
| `docker exec -it <container-id> sh` | Open a shell in a container |
| `docker volume ls` | List volumes |
| `docker network ls` | List networks |
| `docker build -t app:v1 .` | Build an image from a Dockerfile |

## 12. Practice Checklist

- [ ] Install Docker Desktop and verify `docker --version`.
- [ ] Clone the Docker practice repository.
- [ ] Pull the Nginx image and list local images.
- [ ] Run Nginx and view the running container.
- [ ] Publish a port and open Nginx in a browser.
- [ ] Stop, start, inspect, and remove a container.
- [ ] Create a named volume and a bind mount.
- [ ] Create a Docker network and connect a container.
- [ ] Write a Dockerfile, build a custom image, and run it.
