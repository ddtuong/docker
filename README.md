Sure. Below is the same README rewritten in **clear, concise English**, while keeping the important technical details for long-term reference.

# Docker Fundamentals

> Learning path: Docker Concepts → Docker CLI → Dockerfile → Images → Containers → Layers & Cache

---

## 1. What is Docker?

**Docker** is a platform for packaging applications together with their dependencies and required runtime environment into **containers**.

The goal is to make applications behave consistently across different environments.

```text
Application
     +
Dependencies
     +
Runtime
     +
Configuration
     │
     ▼
   Docker
     │
     ▼
 Container
```

### What problem does Docker solve?

It reduces the classic:

> "It works on my machine."

problem caused by differences in:

* Operating systems
* Runtime versions
* Libraries and dependencies
* Environment variables
* System configuration

---

# 2. Container vs Virtual Machine

|                   | Virtual Machine               | Container                        |
| ----------------- | ----------------------------- | -------------------------------- |
| Virtualization    | Hardware-level                | OS-level                         |
| OS                | Each VM has its own OS/kernel | Containers share the host kernel |
| Size              | Usually GBs                   | Usually much smaller             |
| Startup           | Relatively slow               | Very fast                        |
| Isolation         | Stronger                      | Process-level isolation          |
| Resource overhead | Higher                        | Lower                            |

Linux containers rely on Linux kernel mechanisms:

```text
Namespaces → Process/resource isolation
cgroups    → Resource control
```

> On Windows/macOS, Linux containers generally run through a Linux VM/backend underneath.

---

# 3. Docker Architecture

A simplified Docker architecture:

```text
Docker CLI
    │
    ▼
Docker Engine / dockerd
    │
    ▼
containerd
    │
    ▼
runc
    │
    ▼
Linux Kernel
    │
    ▼
Container
```

### Docker CLI

The command-line interface used to interact with Docker:

```bash
docker run
docker build
docker ps
docker images
```

### Docker Daemon — `dockerd`

A background service responsible for managing:

* Images
* Containers
* Networks
* Volumes

### containerd

A container runtime manager used by Docker Engine.

### runc

A low-level runtime responsible for creating and running containers according to OCI specifications.

---

# 4. Core Docker Concepts

## Image

An **image** is a read-only template used to create containers.

Examples:

```text
nginx image
python image
node image
postgres image
```

One image can create multiple containers:

```text
          nginx:latest
          /     |     \
         /      |      \
        ▼       ▼       ▼
     web-1    web-2    web-3
```

---

## Container

A **container** is a runnable instance created from an image.

```text
Image
  │
  │ docker run
  ▼
Container
```

A container has a lifecycle:

```text
Created
   │
   ▼
Running
   │
   │ docker stop
   ▼
Stopped
   │
   │ docker start
   ▼
Running
```

---

## Dockerfile

A **Dockerfile** is a text file containing instructions for building a Docker image.

```text
Dockerfile
     │
     │ docker build
     ▼
   Image
     │
     │ docker run
     ▼
 Container
```

---

## Registry

A **registry** stores and distributes Docker images.

Examples:

* Docker Hub
* AWS ECR
* GitHub Container Registry
* Private registries

Basic workflow:

```text
Local Machine
     │
     │ docker push
     ▼
  Registry
     │
     │ docker pull
     ▼
Another Machine
```

---

# 5. Basic Docker CLI

## Check Docker

```bash
docker --version
docker info
```

---

## Pull an Image

```bash
docker pull nginx
```

`docker pull` downloads an image to the local machine.

It does **not** create a running container.

```text
Registry
   │
   │ docker pull
   ▼
Local Image
```

---

## List Images

```bash
docker images
```

or:

```bash
docker image ls
```

---

# 6. Run a Container

```bash
docker run nginx
```

Docker will:

```text
1. Look for the image locally
2. Pull it if it does not exist
3. Create a container
4. Start the container
```

A more practical example:

```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

### Options

```text
-d
```

Run in **detached mode** (background).

```text
-p 8080:80
```

Map:

```text
HOST : CONTAINER
8080 : 80
```

```text
--name my-nginx
```

Give the container a custom name.

```text
nginx
```

The image used to create the container.

---

# 7. Port Mapping

Example:

```bash
docker run -d -p 8080:80 nginx
```

Meaning:

```text
-p HOST_PORT:CONTAINER_PORT

8080:80
  │    │
  │    └── Port inside container
  └─────── Port on host
```

Flow:

```text
Browser
   │
   │ localhost:8080
   ▼
Host :8080
   │
   │ Docker port mapping
   ▼
Container :80
   │
   ▼
Nginx
```

### `EXPOSE` vs `-p`

Dockerfile:

```dockerfile
EXPOSE 80
```

`EXPOSE` mainly documents the port the application uses.

It does **not** publish the port to the host.

You still need:

```bash
docker run -p 8080:80 nginx
```

---

# 8. Container Management

### List running containers

```bash
docker ps
```

### List all containers

```bash
docker ps -a
```

### Stop

```bash
docker stop my-nginx
```

### Start

```bash
docker start my-nginx
```

### Restart

```bash
docker restart my-nginx
```

### Remove

```bash
docker rm my-nginx
```

### View logs

```bash
docker logs my-nginx
```

### Follow logs in real time

```bash
docker logs -f my-nginx
```

---

# 9. Execute Commands Inside a Container

A very common debugging command:

```bash
docker exec -it my-nginx bash
```

Meaning:

```text
exec → Execute a command
-i   → Interactive
-t   → Allocate a terminal
bash → Start Bash shell
```

You may see:

```text
root@cbf79dd3ce67:/#
```

This means you are now inside the container.

Exit with:

```bash
exit
```

Some minimal images do not contain Bash. Use:

```bash
docker exec -it <container> sh
```

---

# 10. Dockerfile Example — Python

Example project:

```text
my-app/
├── Dockerfile
├── requirements.txt
├── app.py
└── .dockerignore
```

Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

---

# 11. Dockerfile Instructions

| Instruction  | Purpose                                   |
| ------------ | ----------------------------------------- |
| `FROM`       | Select the base image                     |
| `WORKDIR`    | Set the working directory                 |
| `COPY`       | Copy files into the image                 |
| `RUN`        | Execute commands during image build       |
| `CMD`        | Default command when the container starts |
| `ENTRYPOINT` | Define the main executable                |
| `EXPOSE`     | Document the container port               |
| `ENV`        | Define environment variables              |
| `ARG`        | Define build-time variables               |

---

# 12. `RUN` vs `CMD`

This distinction is extremely important.

## `RUN`

Executed during:

```bash
docker build
```

Example:

```dockerfile
RUN pip install -r requirements.txt
```

Flow:

```text
docker build
     │
     ▼
RUN executes
     │
     ▼
Image
```

The result becomes part of the image.

---

## `CMD`

Executed when the container starts:

```dockerfile
CMD ["python", "app.py"]
```

Flow:

```text
docker run
     │
     ▼
CMD executes
     │
     ▼
Application
```

---

# 13. Docker Layers & Build Cache

Docker builds an image as a series of layers.

Example:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

Conceptually:

```text
Layer 1 → FROM
Layer 2 → WORKDIR
Layer 3 → COPY requirements.txt
Layer 4 → RUN pip install
Layer 5 → COPY .
Layer 6 → CMD
```

Docker caches build results so that unchanged steps can be reused.

---

# 14. Why Copy Dependencies First?

Recommended:

```dockerfile
COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .
```

Instead of:

```dockerfile
COPY . .

RUN pip install -r requirements.txt
```

Why?

Application source code changes frequently, while dependencies usually change less frequently.

If only:

```text
app.py
```

changes, Docker can reuse previous layers:

```text
FROM                  → CACHE
WORKDIR               → CACHE
COPY requirements.txt → CACHE
pip install           → CACHE
COPY .                → REBUILD
```

Therefore, `pip install` does not need to run again.

If `requirements.txt` changes:

```text
COPY requirements.txt → REBUILD
pip install           → REBUILD
COPY .                → REBUILD
```

### General rule

> Put stable operations earlier and frequently changing operations later.

---

# 15. `--no-cache-dir`

For Python:

```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

`--no-cache-dir` tells `pip` not to keep its package download cache.

It helps avoid unnecessary files in the Docker image.

Important:

```text
pip cache
     ≠
Docker build cache
```

They are two different caching mechanisms.

```text
pip --no-cache-dir
        │
        └── Controls pip's package cache

Docker layer cache
        │
        └── Controls Docker build layers
```

---

# 16. `.dockerignore`

Similar to `.gitignore`.

Python example:

```text
.venv
venv
__pycache__
*.pyc
.git
.env
.pytest_cache
```

Purpose:

* Reduce build context
* Make builds faster
* Reduce unnecessary files
* Avoid copying secrets
* Keep images cleaner

---

# 17. Build an Image

```bash
docker build -t my-python-app .
```

Breakdown:

```text
docker build
    │
    ├── -t my-python-app
    │       └── Image name/tag
    │
    └── .
        └── Build context
```

Flow:

```text
Dockerfile + Source Code
          │
          │ docker build
          ▼
       Docker Image
```

---

# 18. Run the Image

```bash
docker run -d \
  -p 8000:8000 \
  --name my-python-app \
  my-python-app
```

Flow:

```text
Image
  │
  │ docker run
  ▼
Container
  │
  ▼
python app.py
```

---

# 19. Image vs Container

Remember this relationship:

```text
                IMAGE
          Template / Read-only
                  │
        ┌─────────┼─────────┐
        │         │         │
        ▼         ▼         ▼
   Container  Container  Container
       #1         #2         #3
```

An **image is not a container**.

> Image → Template used to create containers.

---

# 20. Cleanup

Remove a container:

```bash
docker stop my-nginx
docker rm my-nginx
```

Remove an image:

```bash
docker rmi nginx
```

Clean unused Docker resources:

```bash
docker system prune
```

Be careful with:

```bash
docker system prune --volumes
```

because unused volumes may also be removed.

---

# 21. Complete Docker Workflow

The most important workflow to remember:

```text
                 Dockerfile
                     │
                     │ docker build
                     ▼
                   IMAGE
                     │
                     │ docker run
                     ▼
                 CONTAINER
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
      logs          exec         ports
        │            │            │
        └────────────┴────────────┘
                     │
                     ▼
                  stop/rm
```

When sharing an image:

```text
Developer
    │
    │ docker build
    ▼
  Image
    │
    │ docker push
    ▼
 Registry
    │
    │ docker pull
    ▼
Server
    │
    │ docker run
    ▼
Container
```

---

# 22. Docker Mental Model

If you remember only one model, remember this:

```text
Dockerfile
    │
    │ docker build
    ▼
  IMAGE
    │
    │ docker run
    ▼
CONTAINER
    │
    ├── Network
    ├── Port
    ├── Volume
    └── Process
```

And:

```text
Dockerfile  = Recipe
Image       = Template
Container   = Running Instance
Registry    = Image Storage
Docker CLI  = Command Interface
Docker Engine = Docker runtime/management engine
```

---

# 23. Learning Roadmap

## Module 1 — Docker Fundamentals

* What is Docker?
* Container vs VM
* Image
* Container
* Docker Engine
* Registry
* Docker architecture
* Namespaces
* cgroups

## Module 2 — Docker CLI

* `docker run`
* `docker pull`
* `docker images`
* `docker ps`
* `docker start`
* `docker stop`
* `docker restart`
* `docker rm`
* `docker logs`
* `docker exec`
* `docker inspect`
* `docker prune`

## Module 3 — Dockerfile

* `FROM`
* `WORKDIR`
* `COPY`
* `RUN`
* `CMD`
* `ENTRYPOINT`
* `ENV`
* `ARG`
* `EXPOSE`
* Layers
* Build cache
* `.dockerignore`
* Multi-stage builds

## Module 4 — Storage & Networking

* Container filesystem
* Volumes
* Bind mounts
* tmpfs
* Bridge networks
* Port mapping
* Container-to-container communication
* Docker DNS

## Module 5 — Docker Compose

* `compose.yaml`
* Services
* Networks
* Volumes
* Environment variables
* Service dependencies
* Healthchecks
* Multi-container applications

## Module 6 — Images & Registries

* Docker Hub
* Private registries
* `docker login`
* `docker tag`
* `docker push`
* `docker pull`
* Image tags
* Image layers
* Image optimization

## Module 7 — Production & Security

* Non-root containers
* Secrets
* Resource limits
* Healthchecks
* Read-only filesystems
* Multi-stage builds
* Minimal base images
* Image scanning

## Module 8 — CI/CD & Orchestration

* Docker in CI/CD
* Build → Test → Push → Deploy
* Docker Swarm
* Kubernetes introduction
* Pods
* Deployments
* Services
* Docker → Kubernetes
