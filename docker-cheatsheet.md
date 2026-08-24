# 🐳 Docker & Docker Compose Cheatsheet

A practical, production-ready reference guide for Docker commands, Dockerfile syntax, container management, and Docker Compose workflows.

---

## 🚀 Container Operations

```bash
# Run container in background (detached)
docker run -d --name my-app -p 8080:80 nginx:alpine

# Run interactively with terminal attached (removed on exit)
docker run --rm -it ubuntu:22.04 bash

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# View container logs (follow live)
docker logs -f my-app

# Execute command inside running container
docker exec -it my-app /bin/sh

# Inspect container details (JSON configuration & IP address)
docker inspect my-app

# Monitor resource usage (CPU, Memory, Network I/O)
docker stats
```

---

## 📦 Image Operations

```bash
# Build image from local Dockerfile
docker build -t my-username/my-app:v1.0 .

# Build with build arguments
docker build --build-arg NODE_ENV=production -t my-app .

# Tag an existing image for remote registry
docker tag my-app:latest my-registry.com/my-app:v1.0

# Push image to Docker Hub / Registry
docker push my-registry.com/my-app:v1.0

# Pull image from Docker Hub
docker pull python:3.11-slim

# List local images
docker images

# Remove an image
docker rmi my-app:v1.0

# View image layer history
docker history my-app:latest
```

---

## 📜 Dockerfile Syntax & Directives

```dockerfile
# Base image specification
FROM node:18-alpine AS builder

# Set working directory inside container
WORKDIR /app

# Copy dependency files first (optimizes layer caching)
COPY package*.json ./

# Run build time commands
RUN npm ci --only=production

# Copy application source code
COPY . .

# Environment variables
ENV NODE_ENV=production \
    PORT=3000

# Document exposed container ports
EXPOSE 3000

# Define non-root execution user for security
USER node

# Main command executed on container startup
CMD ["node", "server.js"]
```

---

## ⚡ Multi-Stage Build Pattern

```dockerfile
# Stage 1: Build Application
FROM golang:1.21-alpine AS build-stage
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /my-service

# Stage 2: Minimal Runtime Image
FROM alpine:3.18 AS production-stage
WORKDIR /root/
COPY --from=build-stage /my-service .
EXPOSE 8080
CMD ["./my-service"]
```

---

## 💾 Volumes & Persistence

```bash
# Create a named volume
docker volume create app_data

# List volumes
docker volume ls

# Inspect volume directory on host
docker volume inspect app_data

# Mount volume into container
docker run -d -v app_data:/var/lib/data postgres:15

# Bind mount local host folder into container (useful for dev)
docker run -d -v $(pwd):/app -p 3000:3000 node:18-alpine
```

---

## 🌐 Networking

```bash
# Create custom bridge network
docker network create my-network

# List networks
docker network ls

# Connect running container to network
docker network connect my-network my-container

# Run container attached to custom network
docker run -d --name db --network my-network postgres:15
docker run -d --name api --network my-network -p 8080:8080 my-api-image
```

---

## 🐙 Docker Compose Reference

### `docker-compose.yml` Example:

```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app-net
    restart: always

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - app-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 5s
      timeout: 5s
      retries: 5

networks:
  app-net:
    driver: bridge

volumes:
  pgdata:
```

### Essential Compose Commands:

```bash
# Start all services defined in docker-compose.yml
docker compose up -d

# Rebuild containers before starting
docker compose up -d --build

# View running services status
docker compose ps

# Follow logs of all services
docker compose logs -f

# Stop and remove containers, networks, and volumes
docker compose down -v

# Execute command inside a compose service container
docker compose exec web npm test
```

---

## 🧹 Cleanup & Maintenance

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all stopped containers
docker container prune -f

# Remove dangling (un-tagged) images
docker image prune -f

# Remove unused volumes
docker volume prune -f

# System deep clean (removes unused containers, networks, images, volumes)
docker system prune -a --volumes -f
```
