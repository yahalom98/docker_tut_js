
---

# Docker + Node.js + MongoDB Tutorial

A complete and detailed guide for containerizing a Node.js application using Docker, Docker Compose, MongoDB, and Mongo Express.

---

## Table of Contents

1. [Introduction](#introduction)
2. [What Are Containers?](#what-are-containers)
3. [Why Use Docker for Nodejs?](#why-use-docker-for-nodejs)
4. [Containers vs Virtual Machines](#containers-vs-virtual-machines)
5. [Installing Docker](#installing-docker)
6. [Docker Architecture Overview](#docker-architecture-overview)
7. [Key Docker Concepts](#key-docker-concepts)
8. [Core Docker Commands](#core-docker-commands)
9. [Project Overview](#project-overview)
10. [Preparing the Nodejs Application](#preparing-the-nodejs-application)
11. [Creating the Dockerfile](#creating-the-dockerfile)
12. [Building and Running the Docker Image](#building-and-running-the-docker-image)
13. [Running MongoDB in Docker](#running-mongodb-in-docker)
14. [Running Mongo Express](#running-mongo-express)
15. [Connecting Nodejs to MongoDB](#connecting-nodejs-to-mongodb)
16. [Using Docker Networks](#using-docker-networks)
17. [Introduction to Docker Compose](#introduction-to-docker-compose)
18. [Full Docker Compose Configuration](#full-docker-compose-configuration)
19. [Persisting MongoDB Data Using Volumes](#persisting-mongodb-data-using-volumes)
20. [Using Environment Variables](#using-environment-variables)
21. [Debugging Common Docker Issues](#debugging-common-docker-issues)
22. [Pushing Docker Images to AWS ECR](#pushing-docker-images-to-aws-ecr)
23. [Cleaning Up Docker Resources](#cleaning-up-docker-resources)
24. [Full Project Workflow](#full-project-workflow)
25. [Summary](#summary)

---

# Introduction

This guide walks through the process of containerizing a full Node.js + MongoDB project using Docker.
You will learn how to:

* Build Docker images
* Run MongoDB inside a container
* Build and run a Node.js backend container
* Use Docker networks for communication
* Manage multi-service containers with Docker Compose
* Persist MongoDB data using volumes
* Push images to AWS Elastic Container Registry (ECR)

The tutorial is crafted to simulate a real production-ready environment.

---

# What Are Containers?

Containers are isolated environments that include:

* Application code
* Dependencies
* Runtime environment
* Libraries and system tools
* Configurations

They ensure applications run **consistently** across different machines.

Containers solve problems such as:

* “Works on my machine”
* Library and dependency mismatches
* Node version inconsistencies
* Environment drift

---

# Why Use Docker for Node.js?

Node.js apps depend on:

* Node version
* npm package versions
* OS-level dependencies

Docker allows you to:

* Package everything together
* Guarantee predictable builds
* Scale services more easily
* Deploy anywhere (AWS, GCP, Azure, on-prem, etc.)
* Ensure team members use the same environment

---

# Containers vs Virtual Machines

| Virtual Machines      | Containers              |
| --------------------- | ----------------------- |
| Full OS per VM        | Share host OS           |
| Heavy (GBs)           | Lightweight (MBs)       |
| Slow startup          | Fast startup            |
| Resource heavy        | Efficient               |
| Hard to scale quickly | Ideal for microservices |

---

# Installing Docker

Download Docker Desktop from the official site:

* Windows
* macOS
* Linux (via apt, yum, pacman)

Verify installation:

```sh
docker --version
docker compose version
```

---

# Docker Architecture Overview

Docker uses a **client-server model**:

* Docker Client → CLI you interact with
* Docker Daemon → Runs containers, builds images
* Docker Registry → Stores images
* Docker Objects → Images, containers, volumes, networks

---

# Key Docker Concepts

### Image

A packaged snapshot of your application.

### Container

A running instance of an image.

### Dockerfile

A script to build Docker images.

### Volume

Persistent storage on the host.

### Network

Allows containers to communicate.

### Docker Compose

A tool to run multi-container apps.

---

# Core Docker Commands

### Pull an image

```sh
docker pull mongo
```

### Build image

```sh
docker build -t my-app:1.0 .
```

### Run container

```sh
docker run my-app:1.0
```

### Run with ports

```sh
docker run -p 3000:3000 my-app:1.0
```

### List containers

```sh
docker ps      # running only
docker ps -a   # all containers
```

### Logs

```sh
docker logs <container>
```

### Remove resources

```sh
docker rm <container>
docker rmi <image>
```

---

# Project Overview

This project includes:

* Node.js backend
* MongoDB database
* Mongo Express admin interface
* Dockerfile for Node
* Docker Compose file
* Persistent storage for DB

You will run the environment both manually and through Compose.

---

# Preparing the Nodejs Application

Basic file structure:

```
project/
  server.js
  package.json
  public/
  Dockerfile
  docker-compose.yml
```

`server.js` contains:

* Express server
* MongoDB connection
* CRUD endpoints

---

# Creating the Dockerfile

Create `Dockerfile`:

```dockerfile
FROM node:18

WORKDIR /home/app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

Explanation:

* Uses Node 18 official image
* Sets working directory
* Installs dependencies
* Copies code
* Exposes port 3000
* Runs the server

---

# Building and Running the Docker Image

### Build the image:

```sh
docker build -t todo-app:1.0 .
```

### Run the container:

```sh
docker run -p 3000:3000 todo-app:1.0
```

Now your Node app is containerized.

---

# Running MongoDB in Docker

```sh
docker run -p 27017:27017 -d \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  --name mongo \
  mongo
```

This:

* Runs MongoDB
* Creates root credentials
* Exposes port 27017

---

# Running Mongo Express

```sh
docker run -d \
  -p 8081:8081 \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=password \
  -e ME_CONFIG_MONGODB_SERVER=mongo \
  --name mongo-express \
  mongo-express
```

Open browser:

```
http://localhost:8081
```

---

# Connecting Nodejs to MongoDB

### Outside Docker:

```js
const mongoUrl = "mongodb://admin:password@localhost:27017";
```

### Inside Docker Compose:

```js
const mongoUrl = "mongodb://admin:password@mongodb:27017";
```

Notes:

* Hostname becomes Compose service name
* Docker DNS resolves automatically

---

# Using Docker Networks

Create network:

```sh
docker network create mongo-network
```

Attach containers:

```sh
docker run --network mongo-network ...
```

This allows internal communication without exposing ports.

---

# Introduction to Docker Compose

Docker Compose allows you to:

* Start multiple services
* Configure them in one file
* Automatically create networks
* Manage environment variables
* Define persistent volumes
* Reproduce environments instantly

Run everything with:

```sh
docker compose up --build
```

---

# Full Docker Compose Configuration

Create `docker-compose.yml`:

```yaml
version: "3.8"

services:
  mongodb:
    image: mongo
    container_name: mongo
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongo-data:/data/db

  mongo-express:
    image: mongo-express
    container_name: mongo-express
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: password
      ME_CONFIG_MONGODB_SERVER: mongodb
    depends_on:
      - mongodb

volumes:
  mongo-data:
```

Run it:

```sh
docker compose up --build
```

---

# Persisting MongoDB Data Using Volumes

Volumes prevent data loss when a container is removed.

```yaml
volumes:
  mongo-data:
```

Data is stored locally on the host machine.

---

# Using Environment Variables

Create `.env` file:

```
MONGO_USER=admin
MONGO_PASS=password
```

Reference in Compose:

```yaml
environment:
  MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
  MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASS}
```

---

# Debugging Common Docker Issues

### Check container logs:

```sh
docker logs <container>
```

### Remove port conflicts:

```sh
lsof -i :3000
```

### Enter container shell:

```sh
docker exec -it <container> sh
```

### Rebuild without cache:

```sh
docker build --no-cache -t app:latest .
```

---

# Pushing Docker Images to AWS ECR

### Authenticate:

```sh
aws ecr get-login-password --region <region> | \
docker login --username AWS --password-stdin <repo-url>
```

### Tag:

```sh
docker tag todo-app:1.0 <repo-url>:1.0
```

### Push:

```sh
docker push <repo-url>:1.0
```

---

# Cleaning Up Docker Resources

Remove containers:

```sh
docker rm -f $(docker ps -aq)
```

Remove images:

```sh
docker rmi $(docker images -q)
```

Remove volumes:

```sh
docker volume prune
```

Remove networks:

```sh
docker network prune
```

---

# Full Project Workflow

1. Build Node.js image
2. Run MongoDB
3. Run Mongo Express
4. Test API manually
5. Create Docker Compose file
6. Add volumes
7. Add environment variables
8. Push final image to AWS ECR
9. Deploy to production

---

# Summary

You now understand how to:

* Containerize Node.js applications
* Run MongoDB inside Docker
* Use Docker networks for communication
* Build multi-container environments with Compose
* Persist data with volumes
* Push images to AWS ECR
* Debug and maintain Docker-based systems
