# Docker - সম্পূর্ণ গাইড

## ১. কেন দরকার? (Why?)

### সমস্যা:

```
Developer: "আমার computer এ তো কাজ করছে! 🤷"

Production Server: "আমার server এ crash করছে! 😱"
```

**কেন হয়?**
- Python version আলাদা (3.8 vs 3.11)
- Library version আলাদা
- Operating system আলাদা (Ubuntu vs macOS)
- Configuration আলাদা

### Solution: Docker

**"আমার পুরো environment টাই একসাথে pack করে দাও!"**

```
Code + Dependencies + Configuration = Container
  ↓
যেখানে run করো না কেন, একই behavior!
```

---

## ২. কী? (What?)

### Docker = Application packaging এবং running এর platform

```
Traditional:
App → Installed on OS → সমস্যা!

Docker:
App → Container → Isolated → ✅ Works everywhere!
```

### Key Concepts:

#### 1. **Image** (Recipe/Template):

```
Image = একটা blueprint যেখানে আছে:
  - Operating system (minimal)
  - Code
  - Dependencies
  - Configuration
```

উদাহরণ: `nginx:latest` হলো একটা image

#### 2. **Container** (Running Instance):

```
Container = Image থেকে চালানো running instance

Image                  Container
(Recipe)              (Food from recipe)
   ↓                         ↓
nginx:latest    →    Running nginx server
```

একটা image থেকে অনেক container:

```
nginx:latest
    ├── container1 (running)
    ├── container2 (running)
    └── container3 (stopped)
```

#### 3. **Dockerfile** (Build Instructions):

```dockerfile
FROM ubuntu:22.04        # Base image
RUN apt-get update       # Commands to run
COPY app.py /app/        # Copy files
CMD ["python3", "app.py"] # Default command
```

---

## ৩. কীভাবে কাজ করে? (How?)

### Docker Architecture:

```
Docker Client (docker command)
       ↓
Docker Daemon (dockerd)
       ↓
├── Images (stored)
├── Containers (running)
├── Networks
└── Volumes
```

### Container vs Virtual Machine:

```
Virtual Machine:
┌─────────────────────┐
│   App A   │  App B  │
├───────────┼─────────┤
│  OS A     │  OS B   │  ← Full OS (heavy)
├─────────────────────┤
│   Hypervisor        │
├─────────────────────┤
│   Host OS           │
└─────────────────────┘

Docker Container:
┌─────────────────────┐
│   App A   │  App B  │  ← Only app
├───────────┼─────────┤
│  Docker Engine      │
├─────────────────────┤
│   Host OS           │
└─────────────────────┘
```

**Container:**
- ✅ Lightweight (MBs)
- ✅ Fast startup (seconds)
- ✅ Shares host kernel
- ❌ Less isolation than VM

**Virtual Machine:**
- ❌ Heavy (GBs)
- ❌ Slow startup (minutes)
- ✅ Full isolation
- ✅ Run different OS

---

## ৪. Hands-On: Basic Commands

### Install Docker:

```bash
# Ubuntu
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# macOS
# Download Docker Desktop from docker.com

# Verify
docker --version
# Output: Docker version 24.0.0
```

### Hello World:

```bash
# Run প্রথম container
docker run hello-world

# কী হয়:
# 1. Local এ hello-world image খোঁজে
# 2. না থাকলে Docker Hub থেকে download
# 3. Container run করে
# 4. Message print করে
# 5. Container stop হয়
```

### Pull একটা Image:

```bash
# Image download করো
docker pull nginx

# Image list দেখো
docker images

# Output:
REPOSITORY   TAG       IMAGE ID       SIZE
nginx        latest    605c77e624dd   142MB
```

### Run একটা Container:

```bash
# Background এ run করো
docker run -d --name mynginx nginx

# -d: detached (background)
# --name: container name

# Running containers দেখো
docker ps

# Output:
CONTAINER ID   IMAGE     STATUS       PORTS      NAMES
abc123         nginx     Up 2 mins    80/tcp     mynginx

# All containers (stopped + running)
docker ps -a
```

### Container এ Access:

```bash
# Logs দেখো
docker logs mynginx

# Real-time logs
docker logs -f mynginx

# Container এর ভিতরে ঢুকো
docker exec -it mynginx bash

# এখন তুমি container এর ভিতরে:
root@abc123:/# ls
root@abc123:/# pwd
root@abc123:/# exit
```

### Container Stop/Remove:

```bash
# Stop
docker stop mynginx

# Start again
docker start mynginx

# Restart
docker restart mynginx

# Remove (must be stopped first)
docker stop mynginx
docker rm mynginx

# Force remove (even if running)
docker rm -f mynginx
```

---

## ৫. Dockerfile - Image তৈরি করো

### Basic Dockerfile:

```dockerfile
# Base image
FROM python:3.9

# Working directory set করো
WORKDIR /app

# Dependencies copy করো
COPY requirements.txt .

# Install dependencies
RUN pip install -r requirements.txt

# Code copy করো
COPY . .

# Port expose
EXPOSE 8000

# Run command
CMD ["python", "app.py"]
```

### Build করো:

```bash
# Dockerfile আছে এমন directory তে:
docker build -t myapp:v1.0 .

# -t: tag (name:version)
# .: context (current directory)

# Build হচ্ছে...
# Sending build context to Docker daemon
# Step 1/7 : FROM python:3.9
# Step 2/7 : WORKDIR /app
# ...
# Successfully built abc123def456
# Successfully tagged myapp:v1.0

# Image দেখো
docker images
```

### Run করো:

```bash
docker run -d -p 8000:8000 --name myapp myapp:v1.0

# -p 8000:8000: host:container port mapping
# Access: http://localhost:8000
```

---

## ৬. Dockerfile Instructions

### FROM:

```dockerfile
# Official image
FROM ubuntu:22.04
FROM python:3.9
FROM node:18

# Multi-stage build
FROM node:18 AS builder
FROM nginx:alpine
```

### RUN:

```dockerfile
# Execute command during build
RUN apt-get update && apt-get install -y curl
RUN pip install flask
RUN npm install

# Multiple commands
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean
```

### COPY vs ADD:

```dockerfile
# COPY: Simple file copy
COPY app.py /app/
COPY requirements.txt /app/

# ADD: Copy + extract tar + URL support
ADD https://example.com/file.tar.gz /tmp/
ADD archive.tar.gz /app/

# ✅ Best practice: Use COPY unless you need ADD features
```

### WORKDIR:

```dockerfile
# Set working directory
WORKDIR /app

# Now all commands run in /app
COPY . .         # Copies to /app
RUN ls           # Lists /app
```

### ENV:

```dockerfile
# Set environment variables
ENV APP_ENV=production
ENV PORT=8000
ENV DATABASE_URL=postgres://localhost/db

# Use in commands
RUN echo $APP_ENV
```

### EXPOSE:

```dockerfile
# Document which port app uses
EXPOSE 8000

# Note: এটা শুধু documentation
# Actual port publish করতে: -p flag দরকার
```

### CMD vs ENTRYPOINT:

```dockerfile
# CMD: Default command (can be overridden)
CMD ["python", "app.py"]

# Run:
docker run myimage              # Runs: python app.py
docker run myimage python test.py  # Runs: python test.py (overridden)

# ENTRYPOINT: Fixed command
ENTRYPOINT ["python", "app.py"]

# Run:
docker run myimage              # Runs: python app.py
docker run myimage --debug      # Runs: python app.py --debug

# Both together:
ENTRYPOINT ["python"]
CMD ["app.py"]
# Default: python app.py
# Override: docker run myimage test.py → python test.py
```

---

## ৭. Docker Volumes - Data Persistence

### সমস্যা:

```bash
# Container এ file লিখো
docker exec mycontainer touch /data/myfile.txt

# Container delete করো
docker rm -f mycontainer

# নতুন container run করো
docker run -d --name mycontainer myimage

# File নেই! ❌
# কারণ: Container filesystem ephemeral
```

### Solution: Volumes

```bash
# Named volume তৈরি করো
docker volume create mydata

# Volume mount করে container run করো
docker run -d \
  --name mycontainer \
  -v mydata:/data \
  myimage

# এখন /data তে লেখা data volume এ save হবে
docker exec mycontainer touch /data/myfile.txt

# Container delete করো
docker rm -f mycontainer

# নতুন container, same volume
docker run -d \
  --name newcontainer \
  -v mydata:/data \
  myimage

# File আছে! ✅
docker exec newcontainer ls /data
# myfile.txt
```

### Volume Types:

#### 1. Named Volume:

```bash
docker volume create myvolume
docker run -v myvolume:/data myimage
```

#### 2. Bind Mount (Host directory):

```bash
# Host directory কে mount করো
docker run -v /host/path:/container/path myimage

# Example:
docker run -v $(pwd):/app myimage
# Current directory → /app (container)
```

#### 3. tmpfs Mount (RAM):

```bash
# Memory তে (temporary, fast)
docker run --tmpfs /tmp myimage
```

### Volume Commands:

```bash
# Create
docker volume create myvolume

# List
docker volume ls

# Inspect
docker volume inspect myvolume

# Remove
docker volume rm myvolume

# Remove unused volumes
docker volume prune
```

---

## ৮. Docker Networks

### Network Types:

#### 1. Bridge (Default):

```bash
# Create network
docker network create mynetwork

# Run containers
docker run -d --name db --network mynetwork postgres
docker run -d --name app --network mynetwork myapp

# app container থেকে:
# postgres://db:5432  ← container name = hostname!
```

#### 2. Host:

```bash
# Host network use করো
docker run --network host myapp

# Container host এর network share করে
# localhost:8000 → directly host এ
```

#### 3. None:

```bash
# No network
docker run --network none myapp

# Fully isolated, no internet
```

### Network Commands:

```bash
# List networks
docker network ls

# Create
docker network create mynet

# Connect container to network
docker network connect mynet mycontainer

# Disconnect
docker network disconnect mynet mycontainer

# Inspect
docker network inspect mynet

# Remove
docker network rm mynet
```

---

## ৯. Docker Compose

### কেন দরকার?

```bash
# ❌ Without Compose: অনেক commands
docker network create mynetwork
docker run -d --name db --network mynetwork postgres
docker run -d --name redis --network mynetwork redis
docker run -d --name app --network mynetwork -p 8000:8000 myapp

# ✅ With Compose: একটা file
docker-compose up
```

### docker-compose.yml:

```yaml
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: mydb
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

  redis:
    image: redis:7
    networks:
      - backend

  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgres://db:5432/mydb
      REDIS_URL: redis://redis:6379
    depends_on:
      - db
      - redis
    networks:
      - backend
      - frontend
    volumes:
      - ./code:/app

volumes:
  db-data:

networks:
  backend:
  frontend:
```

### Compose Commands:

```bash
# Start all services
docker-compose up

# Background
docker-compose up -d

# Build এবং start
docker-compose up --build

# Stop
docker-compose down

# Stop এবং volumes remove
docker-compose down -v

# Logs
docker-compose logs
docker-compose logs -f app

# Specific service
docker-compose up db
docker-compose restart app
docker-compose exec app bash
```

---

## ১০. Container Lifecycle

### States:

```
      created
         ↓
      running  ←→  paused
         ↓
      stopped
         ↓
      removed
```

### Commands:

```bash
# Create (don't start)
docker create nginx

# Start
docker start <container>

# Stop (SIGTERM)
docker stop <container>

# Kill (SIGKILL)
docker kill <container>

# Pause (suspend)
docker pause <container>

# Unpause
docker unpause <container>

# Remove
docker rm <container>

# Remove all stopped containers
docker container prune
```

---

## ১১. Docker Image Layers

### Image = Stack of Layers:

```dockerfile
FROM ubuntu:22.04        # Layer 1 (base)
RUN apt-get update       # Layer 2
RUN apt-get install py3  # Layer 3
COPY app.py /app/        # Layer 4
CMD ["python3", "app.py"] # Layer 5 (metadata)
```

```
Image:
┌─────────────────┐
│ CMD ["python3"] │ ← Layer 5 (metadata)
├─────────────────┤
│ COPY app.py     │ ← Layer 4
├─────────────────┤
│ RUN apt install │ ← Layer 3
├─────────────────┤
│ RUN apt update  │ ← Layer 2
├─────────────────┤
│ FROM ubuntu     │ ← Layer 1 (base)
└─────────────────┘
```

### Layer Caching:

```dockerfile
# ❌ Bad: Code change হলে dependencies rebuild
FROM python:3.9
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt

# ✅ Good: Cache leverage করো
FROM python:3.9
WORKDIR /app
COPY requirements.txt .       # প্রথমে dependencies
RUN pip install -r requirements.txt  # Cache!
COPY . .                      # তারপর code
```

### Image Commands:

```bash
# History দেখো (layers)
docker history myimage

# Remove unused images
docker image prune

# Remove all images
docker image prune -a

# Tag image
docker tag myimage:v1.0 myimage:latest

# Save image to file
docker save myimage > myimage.tar

# Load from file
docker load < myimage.tar
```

---

## ১২. Docker Registry

### Docker Hub:

```bash
# Login
docker login

# Tag for push
docker tag myapp:v1.0 username/myapp:v1.0

# Push
docker push username/myapp:v1.0

# Pull
docker pull username/myapp:v1.0

# Logout
docker logout
```

### Private Registry:

```bash
# Run local registry
docker run -d -p 5000:5000 registry:2

# Tag for local registry
docker tag myapp localhost:5000/myapp

# Push
docker push localhost:5000/myapp

# Pull
docker pull localhost:5000/myapp
```

---

## ১৩. Practical Exercise

### Task 1: Simple Web App

```bash
# 1. Create app
mkdir myapp && cd myapp

cat > app.py << 'EOF'
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Docker!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
EOF

cat > requirements.txt << 'EOF'
flask==2.3.0
EOF

# 2. Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
EOF

# 3. Build
docker build -t myapp:v1 .

# 4. Run
docker run -d -p 8000:8000 --name myapp myapp:v1

# 5. Test
curl http://localhost:8000
# Output: Hello from Docker!

# 6. Logs
docker logs myapp

# 7. Inside container
docker exec -it myapp bash
> ls
> exit

# 8. Cleanup
docker stop myapp
docker rm myapp
```

### Task 2: Multi-container with Compose

```bash
# docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: mysecret
      POSTGRES_DB: testdb
    ports:
      - "5432:5432"

  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgres://postgres:mysecret@db:5432/testdb
    depends_on:
      - db
EOF

# Start
docker-compose up -d

# Check
docker-compose ps

# Logs
docker-compose logs app

# Stop
docker-compose down
```

---

## ১৪. Best Practices

### ✅ Dockerfile Optimization:

```dockerfile
# 1. Use specific tags
FROM python:3.9-slim  # Not: python:latest

# 2. Minimize layers
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && apt-get clean

# 3. Order by change frequency
COPY requirements.txt .   # Less frequently changed
RUN pip install -r requirements.txt
COPY . .                  # More frequently changed

# 4. Use .dockerignore
# Create .dockerignore:
# .git
# __pycache__
# *.pyc
# node_modules

# 5. Don't run as root
RUN useradd -m myuser
USER myuser
```

### ✅ Security:

```dockerfile
# Use official images
FROM python:3.9-slim

# Run as non-root
USER nobody

# Scan for vulnerabilities
# docker scan myimage
```

---

## ১৫. Kubernetes Relationship

Docker থেকে Kubernetes:

```
Docker:
- Single host
- Manual scaling
- No auto-healing

Kubernetes:
- Multiple hosts (cluster)
- Auto-scaling
- Self-healing
- Load balancing
- Rolling updates
```

```yaml
# Kubernetes Pod = Docker container(s)
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mycontainer
    image: myapp:v1.0      # ← Docker image!
    ports:
    - containerPort: 8000
```

---

## ১৬. Summary

✅ **Docker Basics:**
- Image: Template/Blueprint
- Container: Running instance
- Dockerfile: Build instructions

✅ **Key Commands:**
```bash
docker build -t myapp .
docker run -d -p 8080:80 nginx
docker ps
docker logs <container>
docker exec -it <container> bash
docker stop/rm <container>
```

✅ **Volumes:**
```bash
docker run -v myvolume:/data myimage
```

✅ **Networks:**
```bash
docker network create mynet
docker run --network mynet myimage
```

✅ **Compose:**
```yaml
docker-compose up
```

**পরবর্তী পাঠ:** Kubernetes fundamentals - Docker থেকে orchestration! 🚀
