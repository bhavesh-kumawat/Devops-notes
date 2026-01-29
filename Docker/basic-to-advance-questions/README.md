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

---

# 15. How do you reduce Docker image size?

**1️⃣ Use a smaller base image**

- Prefer `alpine` or slim images.

👉 Reduces unnecessary OS packages.

**2️⃣ Use multi-stage builds**

- Keep build tools out of the final image.

👉 Results in much smaller production images.

**3️⃣ Leverage Docker cache properly**

- Copy only required files early (`package.json` before source).

👉 Avoids rebuilding heavy layers.

**4️⃣ Remove unnecessary files**

- Use `.dockerignore` to exclude logs, `.git`, tests.

---

# 16. What is `.dockerignore` and why is it important?

**`.dockerignore`** is a file that tells Docker **which files and folders to exclude** when building an image.

### How It Works (1–2 Points)

- Docker ignores listed files during `docker build`.
- These files are **not sent to the Docker daemon**.

### Why It's Important

- **Reduces image size**
- **Speeds up build time**
- Prevents **sensitive or unnecessary files** from entering the image

### Simple Example

```
node_modules
.git
.env
logs/
```

---

# 17. Difference between volumes and bind mounts

| **Feature** | **Volumes** | **Bind Mounts** |
| --- | --- | --- |
| Managed by | Docker | Host filesystem |
| Location | Docker-controlled directory | Any path on host |
| Portability | High | Low |
| Security | Safer (isolated) | Less safe (full host access) |
| Common use | Production data | Local development |

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
docker run -v /host/data:/app/data myapp+

---

# 18. How do containers communicate with each other?

Containers communicate **over networks**, just like normal machines.

### 1️⃣ Same Docker Network (Most Common)

- Containers attached to the **same network** can talk using **container names**.
- Docker provides **built-in DNS**.

👉 Example:

`app` → `db` using `db:5432`

### 2️⃣ Using Ports (Across Networks or Host)

- One container **exposes a port**.
- Another container or host accesses it via **IP:port**.

👉 Example:

[`localhost:8080`](http://localhost:8080)

### 3️⃣ Overlay Network (Multi-Host)

- Used in **Docker Swarm / Kubernetes**.
- Containers communicate across **different hosts**.

### Simple Example

```bash
docker network create mynet
docker run --name db --network mynet postgres
docker run --name app --network mynet myapp
```

`app` connects to `db` using hostname `db`.

---

# 19. How do you pass environment variables to a container?

### 1️⃣ Using `-e` Flag

- Pass variables directly at runtime.

```bash
docker run -e ENV=prod -e DEBUG=false myapp
```

### 2️⃣ Using `.env` File

- Store variables in a file and load them together.

```bash
docker run --env-file .env myapp
```

### 3️⃣ Using Dockerfile (`ENV`)

- Sets default environment variables inside the image.

```docker
ENV APP_ENV=prod
```

---

# 🔹 Advanced Docker Questions

---

# 20. How does Docker use Linux namespaces and cgroups?

Docker relies on **Linux kernel features** to provide **isolation and resource control** for containers.

### 1️⃣ Linux Namespaces (Isolation)

Namespaces isolate what a container can **see**.

- **PID namespace** → separate process tree
- **Network namespace** → own IP, ports
- **Mount namespace** → own filesystem view
- **UTS namespace** → own hostname

👉 **Result:** Each container thinks it has its **own system**.

### 2️⃣ cgroups (Control Groups) (Resource Control)

cgroups limit **how much resources** a container can use.

- CPU usage
- Memory limits
- Disk I/O

👉 **Result:** One container cannot **starve others** of resources.

### Why Both Are Needed (1–2 Points)

- **Namespaces** provide isolation
- **cgroups** provide resource limits

Together, they make containers **lightweight and safe**.

---

# 21. What is the difference between `EXPOSE` and `-p`?

| **Feature** | **EXPOSE** | **-p (port mapping)** |
| --- | --- | --- |
| Defined where | Dockerfile | `docker run` command |
| Purpose | Documentation / metadata | Actually publishes the port |
| Opens port? | ❌ No | ✅ Yes |
| Required for access | No | Yes (for external access) |

### Key Explanation (1–2 Points)

**EXPOSE**

- Tells Docker **which port the container listens on**
- Used by tools like Docker Compose

**-p**

- Maps **container port to host port**
- Enables access from outside the container

### Example

```docker
EXPOSE 8080
```

```bash
docker run -p 8080:8080 myapp
```

---

# 22. How do you debug a failing container?

**1️⃣ Check Container Status**

- See if it's running, exited, or restarting.

```bash
docker ps -a
```

**2️⃣ Check Container Logs**

- Most failures are visible in logs.

```bash
docker logs <container_id>
```

**3️⃣ Inspect Exit Code**

- Helps identify crash or misconfiguration.

```bash
docker inspect <container_id> | grep ExitCode
```

**4️⃣ Run Container Interactively**

- Useful to debug startup issues.

```bash
docker run -it --entrypoint sh <image>
```

**5️⃣ Check Ports, Env, Volumes**

- Verify port mapping, environment variables, and mounted volumes.

---

# 23. What is Docker Swarm? How is it different from Kubernetes?

**Docker Swarm** is Docker's **native container orchestration tool** used to **manage and scale containers** across multiple hosts.

### Key Features (1–2 Points)

- Simple setup using Docker CLI
- Built-in **service discovery, load balancing, and scaling**

### Docker Swarm vs Kubernetes

| **Feature** | **Docker Swarm** | **Kubernetes** |
| --- | --- | --- |
| Complexity | Simple | Complex |
| Setup | Easy | Steep learning curve |
| Scalability | Good | Excellent (enterprise-grade) |
| Ecosystem | Smaller | Very large |
| Use case | Small to medium setups | Large, production systems |

### Simple Example

- Swarm: `docker service scale web=5`
- Kubernetes: `kubectl scale deployment web --replicas=5`

---

# 24. Explain Docker overlay networks

A **Docker overlay network** allows **containers running on different Docker hosts** to **communicate as if they are on the same network**.

### How It Works (1–2 Points)

- Uses **VXLAN tunneling** to encapsulate container traffic.
- Docker handles **service discovery and routing** automatically.

### When It's Used

- In **Docker Swarm** or multi-host container setups
- For **distributed microservices**

### Simple Example

```bash
docker network create -d overlay my-overlay
```

Containers on different nodes can now communicate using **service names**.

### Why It's Important (1 Point)

- Enables **scalable, multi-host container networking** without manual configuration.

---

# 25. How do you secure Docker containers?

I focus on **image security, runtime isolation, and access control**.

**1️⃣ Use Secure & Minimal Images**

- Use **official or trusted images**
- Prefer **alpine/slim** images to reduce attack surface

👉 Fewer packages = fewer vulnerabilities.

**2️⃣ Run Containers with Least Privilege**

- Avoid running containers as **root**
- Use `USER` in Dockerfile

👉 Limits damage if a container is compromised.

**3️⃣ Scan Images for Vulnerabilities**

- Scan images during CI/CD (e.g., Trivy, Docker scan)

👉 Catches known CVEs early.

**4️⃣ Limit Resources & Capabilities**

- Use CPU/memory limits
- Drop unnecessary Linux capabilities

👉 Prevents resource abuse and privilege escalation.

**5️⃣ Secure Secrets & Networking**

- Don't hardcode secrets in images
- Restrict container-to-container communication using networks

---

# 26. What are rootless containers?

**Rootless containers** are containers that **run without root (admin) privileges** on the host system.

### How They Work (1–2 Points)

- The container engine runs as a **normal user**, not root.
- Uses **user namespaces** to map container root → non-root host user.

👉 Inside the container it *looks* like root, but on the host it's not.

### Why Rootless Containers Are Important

- **Improves security** by reducing privilege escalation risk
- Even if a container is compromised, the attacker **cannot gain root access** on the host

### Example

Docker can be run in rootless mode:

```bash
dockerd-rootless-setuptool.sh install
```

### Limitation (1 Point)

- Some features (low ports, certain storage drivers) may be **restricted**.

---

# 27. How do you scan Docker images for vulnerabilities?

Docker images are scanned to **detect known security vulnerabilities (CVEs)** in OS packages and dependencies.

### 1️⃣ Using Image Scanning Tools (Most Common)

**Trivy (Popular & Simple)**

```bash
trivy image myapp:latest
```

- Scans OS packages and application dependencies
- Shows **severity levels** (LOW, MEDIUM, HIGH, CRITICAL)

**Docker Scan (Built-in)**

```bash
docker scan myapp:latest
```

- Uses Snyk under the hood
- Good for quick checks

### 2️⃣ Integrate Scanning into CI/CD

- Scan images **during build or before deployment**
- Fail the pipeline if **critical vulnerabilities** are found

👉 Prevents vulnerable images from reaching production.

### Why Image Scanning Is Important (1–2 Points)

- Containers often include **outdated libraries**
- Early detection reduces **security risk and compliance issues**

### Simple Example Flow

Docker build → **Image scan** → Push to registry → Deploy

---

# 28. What is the difference between `ONBUILD` and normal instructions?

| **Feature** | **Normal Instructions** | **ONBUILD Instructions** |
| --- | --- | --- |
| Execution time | During image build | During **child image** build |
| Scope | Applies to current image | Applies to images built **FROM it** |
| Visibility | Runs immediately | Deferred execution |
| Typical use | Standard image creation | Base images for developers |

### Key Explanation (1–2 Points)

**Normal Instructions**

- Executed **when the image is built**
- Directly affect the current image

**ONBUILD**

- Acts like a **trigger**
- Runs only when another Dockerfile uses this image as a base

### Simple Example

```docker
# Base image
FROM node:18
ONBUILD COPY . /app
ONBUILD RUN npm install
```

```docker
# Child image
FROM my-node-base
```

👉 `COPY` and `RUN` execute **here**, not earlier.

---

# 29. How does Docker handle logs?

Docker handles logs by **capturing stdout and stderr** of containers and sending them to a **logging driver**.

### Default Logging Behavior

- Docker captures application logs written to:
    - `stdout`
    - `stderr`
- Stored as **JSON log files** on the host

```bash
docker logs <container_id>
```

### Logging Drivers (How Logs Are Handled)

Docker supports multiple **logging drivers**, such as:

- `json-file` (default)
- `syslog`
- `journald`
- `awslogs`
- `fluentd`

👉 Drivers define **where and how logs are stored**.

### Example

```bash
docker run --log-driver=awslogs myapp
```

### Why This Matters (1–2 Points)

- Keeps containers **stateless**
- Enables **centralized logging** in production
