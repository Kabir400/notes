# Docker Notes

## What is Docker?

Docker is a platform that allows developers to automate the deployment of applications inside lightweight, portable containers. Containers are isolated environments that package an application and all its dependencies, ensuring consistency across different computing environments.

## What is a Docker Image?

A Docker image is a lightweight, standalone, and executable package that includes everything needed to run a piece of software, including the code, runtime, libraries, dependencies, and settings. Images serve as the blueprint for creating Docker containers.

## Important Docker Commands

- **`docker pull <image>`** - Pull an image from Docker Hub
- **`docker run <image>`** - Run a container from an image
- **`docker images`** - List available images
- **`docker ps`** - List running containers
- **`docker ps -a`** - List all containers (including stopped ones)
- **`docker start <container_id>`** - Start a stopped container
- **`docker stop <container_id>`** - Stop a running container
- **`docker rm <container_id>`** - Remove a container
- **`docker rmi <image_id>`** - Remove an image
- **`docker logs <container_id>`** - View logs of a container
- **`docker exec -it <container_id> bash`** - Access a running container’s shell
- **`docker build -t <image_name> .`** - Build a Docker image from a Dockerfile
- **`docker-compose up`** - Start services defined in a `docker-compose.yml` file
- **`docker-compose down`** - Stop and remove containers defined in `docker-compose.yml`

##### `Flags:`

- `-d`: Run the container in detached mode
- `-it`: Run the container in interactive mode
- `-p`: Map a host port to a container port
- `--name`: Assign a name to the container
- `-v`: Mount a host directory to a container directory
- `-e`: Set an environment variable in the container

##### `Example:`

```sh
docker run -d -it -p 8080:80 --name my_container -v /host/path:/container/path -e ENV_VAR1=value1 -e ENV_VAR2=value2 -e ENV_VAR3=value3 my_image
```

## Docker Modes

### Detached Mode (`-d`)

Running a container in detached mode means the container runs in the background without attaching to the terminal.

```sh
docker run -d nginx
```

### Interactive Mode (`-it`)

Interactive mode allows you to interact with a running container’s terminal.

```sh
docker run -it ubuntu bash
```

## Port Mapping

Port mapping allows external access to a container’s internal services. It maps a host machine’s port to a container’s port.

```sh
docker run -p 8080:80 nginx
```

In this example, requests to port `8080` on the host machine will be forwarded to port `80` in the container.

---

# Dockerfile & Docker Compose

### Dockerfile:

A **Dockerfile** is a script containing a series of instructions used to create a **Docker image**.

### **1. Basic Structure of a Dockerfile**

- **FROM**: Defines the base image (e.g., `FROM node:20-alpine`)
- **WORKDIR**: Sets the working directory inside the container
- **COPY**: Copies files from the host machine to the container
- **ADD**: Similar to COPY, but can also extract compressed files
- **RUN**: Executes a command inside the container (used for installing dependencies, setting up the environment, etc.)
- **CMD**: Specifies the default command that runs when the container starts
- **ENTRYPOINT**: Similar to CMD, but allows passing arguments dynamically
- **EXPOSE**: Specifies which port the container will listen on
- **ENV**: Defines environment variables
- **VOLUME**: Creates a mount point for persistent storage
- **ARG**: Defines build-time variables
- **LABEL**: Adds metadata (e.g., `LABEL version="1.0"`)
- **HEALTHCHECK**: Defines a command to check if the container is still healthy

### **2. Example Dockerfile**

```dockerfile
# Define the base image
FROM node:20-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json .
RUN npm install

# Copy rest of the application files
COPY . .

# Set an environment variable
ENV NODE_ENV=production

# Expose a port
EXPOSE 3000

# Define a volume for persistent storage
VOLUME ["/app/data"]

# Add metadata
LABEL maintainer="example@example.com"
LABEL version="1.0"

# Define a health check
HEALTHCHECK --interval=30s --timeout=10s --retries=3 CMD curl -f http://localhost:3000 || exit 1

# Set default command
CMD ["node", "server.js"]
```

### **3. Optimizing Dockerfiles**

- Use a **minimal base image** (e.g., `alpine` instead of `ubuntu`)
- **Leverage multi-stage builds** to reduce image size
- **Group similar instructions** (e.g., `COPY package.json` before copying everything)
- **Use `.dockerignore`** to exclude unnecessary files (similar to `.gitignore`)

---

## **Docker Compose: Managing Multi-Container Applications**

**Docker Compose** is a tool for defining and running multi-container applications using a `docker-compose.yml` file.

### **1. Basic Structure of docker-compose.yml**

- **version**: Defines the Docker Compose file format version
- **services**: Defines different containers
- **volumes**: Manages persistent storage
- **networks**: Connects multiple containers together

### **2. Example docker-compose.yml**

```yaml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - db
    environment:
      - NODE_ENV=production
      - MONGO_URI=mongodb://db:27017/mydb

  db:
    image: mongo:6
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: user
      MONGO_INITDB_ROOT_PASSWORD: password
      MONGO_INITDB_DATABASE: mydb
    volumes:
      - db-data:/data/db

volumes:
  db-data:
```

##### `Note:`

If you `want to use bind mounting` instead of volume mounting you have to use this syntax bellow,and `no need to create top-level volumes: section`. More details about volume mounting are given bellow.

##### `Example:`

```yaml
volumes:
  - ./data:/var/lib/postgresql/data
```

### **3. Key Docker Compose Commands**

- `docker-compose up -d` → Starts services in the background
- `docker-compose down` → Stops and removes all containers
- `docker-compose build` → Builds the images
- `docker-compose logs` → Shows logs for all containers
- `docker-compose ps` → Lists running services
- `docker-compose exec <service> bash` → Opens a terminal inside a container

---

## **Dockerfile & Docker Compose for a Node.js App with MongoDB & Redis**

### **Dockerfile**

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package.json .
RUN npm install

COPY . .

ENV NODE_ENV=production

EXPOSE 3000

CMD ["node", "server.js"]
```

### **docker-compose.yml**

```yaml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "3000:3000"
    depends_on:
      - mongo
      - redis
    environment:
      - NODE_ENV=production
      - MONGO_URI=mongodb://mongo:27017/mydb
      - REDIS_HOST=redis

  mongo:
    image: mongo:6.0
    restart: always
    volumes:
      - mongo-data:/data/db

  redis:
    image: redis:latest
    restart: always

volumes:
  mongo-data:
```

---

## **Dockerfile & Docker Compose for a Fullstack React App with Node.js & MongoDB**

### **Backend Dockerfile**

```dockerfile
FROM node:20-alpine

WORKDIR /server

COPY package.json .
RUN npm install

COPY . .

ENV NODE_ENV=production

EXPOSE 5000

CMD ["node", "server.js"]
```

### **Frontend Dockerfile**

```dockerfile
FROM node:20-alpine

WORKDIR /client

COPY package.json .
RUN npm install

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

### **docker-compose.yml**

```yaml
version: "3.8"

services:
  frontend:
    build: ./client
    ports:
      - "3000:3000"
    depends_on:
      - backend

  backend:
    build: ./server
    ports:
      - "5000:5000"
    depends_on:
      - mongo
    environment:
      - NODE_ENV=production
      - MONGO_URI=mongodb://mongo:27017/mydb

  mongo:
    image: mongo:6.0
    restart: always
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

---

### **Multi-Stage Build:**

**`Examople:`**
Build a Node.js application with dependencies in one stage and use only the necessary files in the final image.

### **Dockerfile (Multi-Stage Build)**

```dockerfile
# Stage 1: Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
CMD ["node", "dist/index.js"]
```

---

## Layer Caching in Dockerfile

Docker uses a layer caching mechanism to speed up image builds. Each instruction in a `Dockerfile` creates a new layer, and Docker caches these layers to avoid unnecessary rebuilding.

### Best Practices for Caching Efficiency:

1. **Order Matters:** Place frequently changing instructions (e.g., `COPY . .`) at the end.
2. **Separate Dependencies:** Copy only `package.json` and install dependencies before copying the rest of the files.
3. **Use Multi-Stage Builds:** Reduce image size by discarding unnecessary build dependencies.

---

# Docker Volumes and Networking

### **Volume Mounting**

Volume mounting in Docker allows data to persist beyond the lifecycle of a container. This means even if a container is stopped or removed, the data remains intact.

#### **Why Do We Need Volume Mounting?**

✅ **Data Persistence** – Ensures important data is not lost when a container is removed.  
✅ **Sharing Data** – Volumes allow data sharing between multiple containers.  
✅ **Performance** – Volumes are managed by Docker and offer better performance compared to bind mounts.  
✅ **Backup & Restore** – Easier to back up and restore container data.

#### **Creating and Using a Volume**

Create a volume:

```sh
docker volume create my_volume
```

Mounting a volume while running a container:

```sh
docker run -v my_volume:/app/data -d my_container
```

---

### **Bind Mounting**

Bind mounting allows a container to access and modify files from a specific directory on the **host machine**. Unlike volumes, bind mounts are **not managed by Docker** but instead use any existing directory on the host.

#### **Why Use Bind Mounting?**

✅ **Direct Access to Host Files** – Useful for development (e.g., mounting source code).  
✅ **Custom Storage Paths** – You control exactly where data is stored on the host.  
✅ **No Pre-Creation Required** – The directory just needs to exist before running the container.

#### **Using Bind Mounts**

Run a container with a bind mount:

```sh
docker run -v /path/on/host:/app/data -d my_container
```

📌 Example:

```sh
docker run -v /home/user/project:/app -d my_container
```

This binds **`/home/user/project`** on the host to **`/app`** inside the container, allowing the container to access and modify the host’s files directly.

---

### **Bind Mount vs Volume**

| Feature            | Bind Mounts                  | Volumes                               |
| ------------------ | ---------------------------- | ------------------------------------- |
| Storage Location   | Any directory on the host    | `/var/lib/docker/volumes/`            |
| Managed by Docker? | ❌ No                        | ✅ Yes                                |
| Persistence        | Persists if host path exists | Persists even after container removal |
| Performance        | Depends on filesystem        | Optimized for Docker                  |

---

## Networking in Docker

Docker provides different networking modes for communication between containers and external systems.

### Types of Docker Networks

Docker offers three main types of networks:

1. **Bridge Network (Default)**
2. **Host Network**
3. **None Network**

### 1. Bridge Network (Default)

By default, Docker uses the **bridge network** when a container is started without specifying a network. It allows containers to communicate with each other while keeping them isolated from the host system.

```sh
docker network ls
```

Example of running a container in the default bridge network:

```sh
docker run -d --name my_container nginx
```

---

#### `Notes`:

When you create a container, it by default starts in the `default bridge network`. In this bride network, **containers can't communicate with each other using their container names**, instead they can `communicate to each other using their ip`.

If you `create a custom network` **on bridge driver**, `it can communicate with each other using their container names`.

###### `Example`:

You have **two containers**:

1. One uses the **MongoDB** image.
2. One uses your **own_mongo_node_server** image.

### 🔹 Default Bridge Network:

- If you run them on the **default bridge network**, you must **use the MongoDB container's IP address** in your code.

```js
mongoose.connect('mongodb://<mongo_ip>:27017/mydatabase', {...});
```

### 🔹Custom Network:

- If you create and use a custom network, Docker allows you to use the container name instead of the IP.

```js
mongoose.connect("mongodb://mongo:27017/mydatabase", {...});
```

In case of `docker-compose` it automatically `creates a custom network with the bridge driver`.

---

### 2. Host Network

In the **host network** mode, the container shares the host machine’s network stack, removing network isolation. This means the container runs directly on the host network without needing port mapping.

```sh
docker run --network host nginx
```

Use case: Useful for applications requiring low-latency communication with the host system.

### 3. None Network

The **none network** disables networking entirely. The container has no access to the host or other containers.

```sh
docker run --network none nginx
```

Use case: Useful for security-sensitive applications where no external communication is needed.

### Creating a Custom Network

Creating a custom bridge network allows better control over communication between containers.

```sh
docker network create my_custom_bridge_network
```

It creates `custom netwrok useing bridge driver`.

```sh
docker network create --driver host my_custom_host_network
```

It creates `custom netwrok useing host driver`.

Running a container in a custom network:

```sh
docker run -d --name container1 --network my_custom_bridge_network nginx
```

Connecting an existing container to a network:

```sh
docker network connect my_custom_network existing_container
```

### Benefits of Creating Your Own Network

1. **Better Security:** Isolates containers from unwanted access.
2. **Custom Subnetting:** Allows defining custom IP ranges.
3. **Service Discovery:** Containers can communicate using container names instead of IPs.

---

## Pushing an Image to Docker Hub

To share a custom Docker image on Docker Hub, follow these steps:

### 1. Login to Docker Hub

```sh
docker login
```

### 2. Tag the Image

```sh
docker tag my_image username/my_image:latest
```

### 3. Push the Image

```sh
docker push username/my_image:latest
```

### 4. Verify the Image on Docker Hub

Go to `https://hub.docker.com` and check if your image is available.

---

## Conclusion

Docker simplifies application deployment by using containers that package everything needed to run an application consistently across environments. Understanding Docker images, container management, port mapping, and Docker Compose will help in efficiently using Docker for development and production environments.
