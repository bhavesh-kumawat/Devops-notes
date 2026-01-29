# 1. What is Docker?

**Docker** is a **containerization platform** that packages an application and all its dependencies into a **container**, so it runs **consistently across different environments**.

### How Docker Works

- Applications run inside **containers**, which share the host OS kernel.
- Each container includes the app, libraries, and configs—but not a full OS.

### Why Docker Is Used

- **Consistency:** "Works on my machine" problem is eliminated
- **Lightweight & fast:** Containers start faster than virtual machines

### Simple Example

- App works in **developer laptop → test → production** without changes.

---

# 2. Difference between Docker and Virtual Machines

| **Feature** | **Docker (Containers)** | **Virtual Machines (VMs)** |
| --- | --- | --- |
| Architecture | Shares host OS kernel | Runs its own full OS |
| Resource usage | Lightweight | Heavy |
| Startup time | Seconds | Minutes |
| Performance | Near native | Slight overhead |
| Isolation | Process-level isolation | Full OS-level isolation |
| Portability | Very high | Lower than containers |

---

# 3. What are Docker images and containers?

### Docker Image

- A **read-only blueprint** that contains the application, libraries, and dependencies.
- Used to **create containers**.

**Example:**

`nginx:latest` image

### Docker Container

- A **running instance of a Docker image**.
- It's where the application actually **executes**.

**Example:**

Running an Nginx web server from the `nginx` image.

### Key Difference (1–2 Points)

- **Image** = Template / Snapshot
- **Container** = Running application

### Simple Analogy

- Image → **Class**
- Container → **Object**

---

# 4. What is a Dockerfile?

**A** Dockerfile **is a** text file **that contains a set of** instructions **used to** build a Docker image**.**

### How It Works (1–2 Points)

- Docker reads the Dockerfile **top to bottom**.
- Each instruction creates a **layer** in the image.

### Simple Example

```docker
FROM nginx
COPY index.html /usr/share/nginx/html
```

- Uses Nginx base image
- Copies app file into the image

### Why Dockerfile Is Important

- Ensures **repeatable and consistent image builds**
- Used in **CI/CD pipelines**

---

# 5. Explain the Docker architecture

**Docker uses a** client–server architecture **to build, run, and manage containers.**

- Docker Client
- Docker Daemon
- Docker Registry

### 1️⃣ Docker Client

- The **user interface** for Docker.
- Commands like `docker build`, `docker run`, `docker pull` are issued here.
- Sends requests to the **Docker Daemon** via REST API.

👉 Example: Your terminal.

### 2️⃣ Docker Daemon (dockerd)

- The **core Docker engine**.
- Responsible for **building images, running containers, managing networks and volumes**.
- Listens for requests from the Docker Client.

👉 Runs on the host machine.

### 3️⃣ Docker Registry

- A **storage repository** for Docker images.
- Used to **pull and push images**.
- Can be public or private.

👉 Example: **Docker Hub**, AWS ECR.

### Simple Flow

Docker Client → **Docker Daemon** → Docker Registry

(build / pull / run)

<aside>
💡

**One-Line Interview Answer:** Docker uses a client–server architecture where the client sends commands to the Docker daemon, which pulls images from a registry and runs containers.

</aside>

---

# 6. What is Docker Hub?

**Docker Hub** is a **cloud-based Docker image registry** used to **store, share, and manage Docker images**.

### How It's Used (1–2 Points)

- Developers **push images** to Docker Hub.
- Other systems **pull images** to run containers.

### Why Docker Hub Is Important

- Provides **official and community images** (nginx, mysql, redis)
- Simplifies **image distribution** in teams and CI/CD

### Example

```bash
docker pull nginx
```

Pulls the Nginx image from Docker Hub.

---

# 7. What is the difference between `COPY` and `ADD`?

| **Feature** | **COPY** | **ADD** |
| --- | --- | --- |
| Purpose | Copies files/directories | Copies files + extra features |
| Source | Local files only | Local files, URLs, tar files |
| Auto-extract | ❌ No | ✅ Yes (for `.tar` archives) |
| Best practice | ✅ Preferred | ⚠️ Use only when needed |

---

# 8. What is a Docker volume?

A **Docker volume** is a **persistent storage mechanism** used to store data **outside the container's filesystem**, so data is **not lost when a container stops or is removed**.

### Why Volumes Are Needed (1–2 Points)

- Containers are **ephemeral** (data inside them is temporary)
- Volumes keep data **safe and reusable**

### How It Works

- Volume is managed by **Docker**
- Can be **shared between multiple containers**

### Example

```bash
docker run -v mydata:/app/data nginx
```

Here, `mydata` is a Docker volume.

---

# 9. What is the difference between image and container?

| **Feature** | **Docker Image** | **Docker Container** |
| --- | --- | --- |
| Nature | Static | Running |
| State | Read-only | Read-write |
| Purpose | Blueprint / template | Actual execution |
| Lifecycle | Created once | Can be started, stopped, deleted |
| Analogy | Class | Object |

---

# 10. How do you list running and stopped containers?

**List running containers:**

```bash
docker ps
```

**List all containers (running and stopped):**

```bash
docker ps -a
```

---

# 🔹 Intermediate Docker Questions

---

# 11. Explain Docker networking types

**Docker Networking Types (Interview Level)**

Docker provides different network drivers to control **how containers communicate**.

### 1️⃣ Bridge (Default)

- Containers get a **private IP** on a virtual bridge network.
- Communication happens **within the same host**.

**Use case:**

Single-host applications

### 2️⃣ Host

- Container **shares the host's network stack**.
- No port mapping needed.

**Use case:**

High-performance or low-latency applications

### 3️⃣ None

- Container has **no network access**.
- Completely isolated.

**Use case:**

Batch jobs or security-sensitive tasks

### 4️⃣ Overlay

- Enables container communication **across multiple Docker hosts**.
- Used with **Docker Swarm / Kubernetes**.

**Use case:**

Multi-host, distributed applications

### Summary Table

| **Network** | **Scope** | **Key Point** |
| --- | --- | --- |
| Bridge | Single host | Default & isolated |
| Host | Single host | Fast but less secure |
| None | Single container | Full isolation |
| Overlay | Multi-host | Cluster networking |

---

# 12. What is the difference between CMD and ENTRYPOINT?

| **Feature** | **CMD** | **ENTRYPOINT** |
| --- | --- | --- |
| Purpose | Default command | Main executable |
| Overridable | Easily overridden | Not overridden by default |
| Use case | Provide defaults | Enforce execution |
| Flexibility | More flexible | More strict |

---

# 13. How does Docker caching work during image build?

Docker uses **layer-based caching** to speed up image builds.

### How It Works (Step-by-Step)

**1️⃣ Each Dockerfile instruction creates a layer**

- Instructions like `FROM`, `RUN`, `COPY`, `ADD` each form a layer.

**2️⃣ Docker checks the cache per layer**

- If the instruction **and its inputs** haven't changed, Docker **reuses the cached layer**.

**3️⃣ Cache breaks on change**

- If one layer changes, **that layer and all layers after it are rebuilt**.

### Why It's Important (1–2 Points)

- **Faster builds** by reusing unchanged layers
- Saves **CPU, time, and CI/CD cost**

### Simple Example

```docker
FROM node:18
COPY package.json .
RUN npm install   # cached if package.json unchanged
COPY . .
```

- Changing source code won't rerun `npm install` if `package.json` is unchanged.

---

# 14. What is a multi-stage Docker build? Why use it?

A **multi-stage Docker build** uses **multiple `FROM` stages** in a single Dockerfile to **separate build-time and runtime environments**.

### How It Works (1–2 Points)

- First stage builds the application (with compilers, tools).
- Final stage copies **only the required output** into a lightweight image.

### Why Use Multi-Stage Builds

- **Smaller image size**
- **More secure** (no build tools in production image)

Key Explanation (1–2 Points)

Volumes

Fully managed by Docker

Preferred for production and persistent data

Bind Mounts

Directly map a host directory into a container

Useful for development and debugging

Simple Example
# Volume
docker run -v mydata:/app/data myapp


# Bind mount
docker run -v /host/data:/app/data myapp
One-Line Interview Answer

Volumes are Docker-managed and portable, while bind mounts directly map host directories and are mainly used in development.
