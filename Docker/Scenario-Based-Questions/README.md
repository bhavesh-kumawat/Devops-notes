# 🔹 Scenario-Based Docker Questions

---

# 1. A container keeps restarting — how do you debug it?

**1️⃣ Check Container Status & Restart Reason**

```bash
docker ps -a
docker inspect <container_id>
```

👉 Look for **ExitCode** and **restart policy**.

**2️⃣ Check Container Logs (Most Important)**

```bash
docker logs <container_id>
```

👉 Usually shows app crashes, config errors, or missing env variables.

**3️⃣ Run Container Without Restart Policy**

```bash
docker run --rm -it myapp /bin/sh
```

👉 Helps debug interactively.

**4️⃣ Verify Configuration**

- Environment variables
- Mounted volumes
- Port bindings

👉 Misconfiguration is a very common cause.

**5️⃣ Check Resource Limits**

- Memory limit causing **OOMKilled**

```bash
docker inspect <container_id> | grep OOM
```

---

# 2. Application works locally but fails in Docker — why?

**1️⃣ Missing Dependencies**

- App uses libraries installed on your machine but **not in the Docker image**

👉 Fix: Ensure all dependencies are installed in the Dockerfile.

**2️⃣ Environment Variables Not Set**

- App relies on `.env` or local shell variables

👉 Fix: Pass env vars using `-e` or `--env-file`.

**3️⃣ Port Binding Issues**

- App listens on [`localhost`](http://localhost) instead of `0.0.0.0`

👉 Fix: Bind the app to `0.0.0.0` inside the container.

**4️⃣ File Path & Volume Issues**

- Relative paths work locally but fail in container
- Files not copied due to `.dockerignore`

**5️⃣ OS / Architecture Differences**

- Local OS ≠ container base image
- Native binaries may break

---

# 3. Container can't access the internet — what do you check?

When a container cannot reach the internet, it's usually a **network or configuration issue**. I follow a systematic approach:

**1️⃣ Check Network Mode**

- Is the container using the **default bridge network** or `host` network?
- Default bridge requires **NAT for external access**.

```bash
docker network inspect bridge
```

**2️⃣ Check DNS Resolution**

- Ensure the container can resolve domain names.

```bash
docker exec <container_id> ping google.com
docker exec <container_id> cat /etc/resolv.conf
```

**3️⃣ Check Gateway & Routing**

- Verify the container has a **default gateway**.

```bash
docker exec <container_id> ip route
```

**4️⃣ Check Firewall / Host Rules**

- Ensure **iptables or security groups** aren't blocking outbound traffic.

```bash
sudo iptables -L -n
```

**5️⃣ Test Connectivity**

- Ping external IP (e.g., `8.8.8.8`) to differentiate **DNS vs network issue**.

---

# 4. How do you roll back a Docker image in production?

Rolling back means **reverting to a previous stable version** when the current deployment fails.

**1️⃣ Use Tagged Images**

- Always **tag images** (e.g., `myapp:v1.0`, `myapp:v1.1`) instead of `latest`.
- To roll back, deploy the **previous stable tag**:

```bash
docker run -d myapp:v1.0
```

**2️⃣ Stop / Remove Failing Container**

- Stop the current container and start the previous version:

```bash
docker stop myapp
docker rm myapp
docker run -d --name myapp myapp:v1.0
```

**3️⃣ In Orchestration (Kubernetes / Swarm)**

- **Kubernetes**: `kubectl rollout undo deployment myapp`
- **Docker Swarm**: `docker service update --image myapp:v1.0 myapp`

---

# 5. How do you do zero-downtime deployments using Docker?

Zero-downtime deployments ensure **users are never affected** while updating an application.

**1️⃣ Use Orchestration Tools**

- **Docker Swarm** or **Kubernetes** handles rolling updates automatically.
- Updates containers **one by one**, keeping the old version running until the new one is ready.

**Example: Docker Swarm**

```bash
docker service update --image myapp:v1.1 --update-parallelism 1 myapp
```

**Example: Kubernetes**

```bash
kubectl set image deployment/myapp myapp=myapp:v1.1
```

**2️⃣ Reverse Proxy / Load Balancer**

- Use **Nginx, HAProxy, or ALB** to route traffic.
- Direct traffic **only to healthy containers** during the update.

**3️⃣ Blue-Green Deployment**

- Run **new version (green)** alongside the **old version (blue)**.
- Switch traffic to green once it's healthy.
- Old version can be removed after verification.

### Best Practices (1–2 Points)

- Use **health checks** to avoid routing to unhealthy containers
- Automate deployments via **CI/CD pipelines**
- Keep **image tags immutable** to avoid inconsistencies

---

# 6. How do you use Docker in CI/CD pipelines?

Docker is widely used in CI/CD pipelines to **ensure consistent builds and deployments** across environments.

**1️⃣ Build and Test in Isolated Containers**

- Each CI job runs inside a **container**, ensuring dependencies are consistent.

```bash
docker build -t myapp:${CI_COMMIT_SHA} .
docker run --rm myapp:${CI_COMMIT_SHA} npm test
```

- This prevents "works on my machine" issues.

**2️⃣ Push Images to a Registry**

- After tests pass, **push the image** to a registry for deployment.

```bash
docker push myapp:${CI_COMMIT_SHA}
```

- Can use **Docker Hub, AWS ECR, GCP Artifact Registry**.

**3️⃣ Deploy Using Containers**

- Deploy the tagged image to staging/production:

```bash
docker run -d --name myapp myapp:${CI_COMMIT_SHA}
```

- Or integrate with **Docker Swarm, Kubernetes** for rolling updates.

### Benefits in CI/CD (1–2 Points)

- **Consistency:** Same image used from build → test → production
- **Isolation:** No dependency conflicts
- **Automation:** Images can be versioned and deployed automatically

---

# 7. Docker vs Podman

| **Feature** | **Docker** | **Podman** |
| --- | --- | --- |
| Daemon | Requires a **long-running daemon** (`dockerd`) | **Daemonless**; runs containers directly |
| Rootless | Supported but optional | Fully supported & encouraged |
| CLI Compatibility | Docker CLI (`docker run`, etc.) | Docker-compatible CLI (`podman run`) |
| Kubernetes Integration | Needs Docker Desktop or CRI tools | Can generate **Kubernetes YAML** (`podman generate kube`) |
| Security | Daemon runs as root (risk if compromised) | More secure, no root daemon needed |
| Swarm Support | Native Docker Swarm orchestration | No built-in orchestration; works with Kubernetes |

---

# 8. How does Docker integrate with Kubernetes?

Kubernetes uses a **container runtime** to manage containers on nodes, and Docker has traditionally been one of the supported runtimes.

### 1️⃣ Docker as a Container Runtime

- Kubernetes nodes can use **Docker Engine** to run containers.
- Kubernetes interacts with Docker through the **Container Runtime Interface (CRI)**.
- Docker pulls images, creates containers, and manages networks/volumes on behalf of Kubernetes.

### 2️⃣ Workflow in Kubernetes

1. **Deployment / Pod YAML** specifies container image.
2. Kubernetes schedules a Pod on a node.
3. Node's container runtime (Docker) **pulls the image and runs the container**.
4. Kubernetes monitors container health, restarts if needed.

### Key Points (1–2 Points)

- Kubernetes **does not require Docker-specific commands** — it uses Docker behind the scenes.
- Since 2020, Docker runtime is deprecated in Kubernetes (Docker Shim), replaced by **containerd or CRI-O**, but Docker images still work.

---

# 9. How do you version Docker images?

Versioning Docker images ensures **predictability, rollback, and reproducibility** in deployments.

**1️⃣ Use Tags Instead of `latest`**

- Avoid using `latest` for production — always tag images.

```bash
docker build -t myapp:v1.0 .
docker push myapp:v1.0
```

- Tags can include **semantic versions** (`v1.0.0`), **commit SHA**, or build numbers.

**2️⃣ Use CI/CD Variables**

- Tag images dynamically during CI/CD pipelines:

```bash
docker build -t myapp:${CI_COMMIT_SHA} .
docker push myapp:${CI_COMMIT_SHA}
```

- Guarantees **unique, traceable images** per build.

**3️⃣ Keep Old Versions**

- Old images in the registry can be **used to rollback** if a deployment fails.

### Best Practices (1–2 Points)

- Use **semantic versioning** (MAJOR.MINOR.PATCH)
- Include **build info or git commit SHA** for traceability
- Avoid mutable tags like `latest` in production

---

# 10. How do you store secrets securely in Docker?

Docker provides several ways to **keep sensitive data (passwords, API keys) secure** without embedding them in images.

**1️⃣ Docker Secrets (Swarm Mode)**

- Used in **Docker Swarm** for production-grade secret management.
- Secrets are **encrypted at rest** and **only exposed to containers that need them**.

```bash
# Create a secret
echo "my_password" | docker secret create db_password -

# Use secret in a service
docker service create --name mydb --secret db_password mydb_image
```

- Accessible inside container at `/run/secrets/db_password`.

**2️⃣ Environment Variables (For Development Only)**

- Pass via `-e` or `--env-file`.
- **Not recommended for production** as values can be viewed via `docker inspect`.

```bash
docker run --env-file .env myapp
```

**3️⃣ External Secret Managers**

- Use tools like **HashiCorp Vault, AWS Secrets Manager, or GCP Secret Manager**.
- Docker pulls secrets at runtime instead of storing them in the image.

---

# 11. Difference between Docker Compose and Docker Swarm

| **Feature** | **Docker Compose** | **Docker Swarm** |
| --- | --- | --- |
| Purpose | Define and run **multi-container apps** on a single host | **Orchestrate containers** across multiple hosts |
| Scope | Local development / testing | Production-grade clusters |
| Deployment | `docker-compose up` | `docker stack deploy` |
| Scaling | Can scale containers on a single host (`docker-compose up --scale`) | Scale services across nodes in a cluster (`docker service scale`) |
| Networking | Creates isolated networks per project | Provides overlay networks for multi-host communication |
| Fault Tolerance | ❌ Limited | ✅ Built-in (reschedules failed containers) |
