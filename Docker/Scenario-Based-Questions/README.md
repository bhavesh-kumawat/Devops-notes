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

---

# 12. Explain how Docker healthchecks can be used to ensure the reliability of containerized applications

Docker **healthchecks** are a mechanism to **periodically test if a containerized application is healthy** and functioning as expected. They help maintain **reliability and availability** in production systems.

### 1️⃣ How It Works

- In the Dockerfile, you define a **HEALTHCHECK instruction** that runs a command inside the container periodically.
- Docker marks the container as **healthy** or **unhealthy** based on the command's exit status:
    - `0` → healthy
    - `1` or `>1` → unhealthy

```docker
FROM nginx:alpine
HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl -f http://localhost/ || exit 1
```

- Here, Docker checks every 30 seconds whether Nginx is responding.

### 2️⃣ Benefits for Reliability

**✅ Automated Monitoring**

- Docker continuously monitors containers without external tools.
- Unhealthy containers can trigger alerts or automated restarts.

**✅ Integration With Orchestration**

- **Docker Swarm / Kubernetes** can use health status:
    - Swarm: Restart unhealthy containers automatically.
    - Kubernetes: Liveness and readiness probes serve the same purpose.

**✅ Avoid Serving Traffic to Broken Containers**

- Load balancers or service orchestrators can **exclude unhealthy containers** from routing traffic.

### 3️⃣ Runtime Behavior

```bash
docker ps
# Shows "healthy", "unhealthy", or "starting" in the STATUS column
```

- If the container fails the healthcheck repeatedly, orchestrators can **replace it with a healthy instance**, ensuring **high availability**.

---

# 13. How would you securely manage sensitive data such as passwords and API keys in Docker containers?

Sensitive data should **never be hardcoded** in Docker images or source code. The goal is to **inject secrets at runtime** and restrict access.

**1️⃣ Docker Secrets (Best for Production – Swarm)**

- Secrets are **encrypted at rest and in transit**
- Mounted as files inside the container

```bash
echo "db_password" | docker secret create db_pass -
docker service create --name app --secret db_pass myapp
```

📍 Available inside container at:

`/run/secrets/db_pass`

**2️⃣ External Secret Managers (Best Practice)**

- Use tools like:
    - **AWS Secrets Manager / Parameter Store**
    - **HashiCorp Vault**
    - **GCP Secret Manager**
- Secrets are fetched **at runtime**, not stored in images.

👉 Most secure and scalable approach.

**3️⃣ Environment Variables (Dev / Limited Use)**

```bash
docker run -e DB_PASSWORD=secret myapp
```

⚠️ Easy but **not fully secure** (visible via `docker inspect`).

**4️⃣ CI/CD Secret Injection**

- Store secrets in **CI/CD secret stores**
- Inject during build or deployment

👉 Keeps secrets out of repos and images.

---

# 14. Describe a scenario where you would use Docker's built-in load balancer features to distribute traffic across multiple container instances

You are running a **stateless web application** (e.g., a REST API) using **Docker Swarm** and need to handle **increasing user traffic** with high availability.

### Setup

- Multiple Docker hosts form a **Swarm cluster**
- The web app is deployed as a **Docker service**
- The service is scaled to multiple container replicas

```bash
docker service create \
  --name web \
  --replicas 5 \
  -p 80:80 \
  mywebapp:v1
```

### How Docker's Built-in Load Balancer Works

- Docker Swarm provides **internal load balancing** using **IPVS**
- Traffic sent to the service's virtual IP (VIP) is **automatically distributed**
- Requests are spread across all healthy container replicas (round-robin style)

👉 Clients don't need to know individual container IPs.

### Why This Scenario Fits

- **Stateless app** → any request can go to any container
- **Horizontal scaling** → add/remove replicas easily
- **No external load balancer required** for internal traffic

### Reliability Benefit (1–2 Points)

- If one container fails, Swarm **stops routing traffic** to it
- New containers are created automatically to maintain desired state

---

# 15. Explain how you would perform rolling updates to deploy a new version of a Dockerized application without downtime

Rolling updates allow you to **deploy a new version gradually** while the old version continues serving traffic.

### Scenario

You are running a **web application** as a **Docker Swarm service** behind a load balancer.

### Step-by-Step Approach

**1️⃣ Deploy the Application as a Service**

```bash
docker service create \
  --name web \
  --replicas 4 \
  -p 80:80 \
  myapp:v1
```

**2️⃣ Configure Rolling Update Strategy**

- Update containers **one at a time**
- Ensure new containers are healthy before stopping old ones

```bash
docker service update \
  --image myapp:v2 \
  --update-parallelism 1 \
  --update-delay 10s \
  web
```

**3️⃣ How This Prevents Downtime**

- Only **one container is replaced at a time**
- Remaining containers keep serving traffic
- Built-in load balancer routes traffic only to **healthy containers**

**4️⃣ Use Health Checks**

```docker
HEALTHCHECK CMD curl -f http://localhost/ || exit 1
```

- Ensures traffic is not sent to unhealthy containers

### Key Reliability Points (1–2)

- Old version runs until new version is confirmed healthy
- Automatic rollback can be configured if update fails

---

# 16. How would you configure logging for Docker containers to capture and analyze application logs?

The goal is to **collect application logs centrally**, make them searchable, and keep containers stateless.

**1️⃣ Log to stdout / stderr (Best Practice)**

- Configure the application to log to:
    - `stdout`
    - `stderr`
- Docker automatically captures these logs.

```bash
docker logs <container_id>
```

👉 This is the foundation of Docker logging.

**2️⃣ Choose an Appropriate Logging Driver**

Docker supports multiple logging drivers depending on where you want logs stored or shipped.

**Common choices:**

- `json-file` (default, local debugging)
- `syslog`
- `journald`
- `fluentd`
- `awslogs` (AWS CloudWatch)

```bash
docker run --log-driver=awslogs myapp
```

**3️⃣ Centralize Logs (Production)**

- Send logs to centralized systems like:
    - **ELK Stack (Elasticsearch, Logstash, Kibana)**
    - **CloudWatch / Splunk / Datadog**
- Use logging drivers or sidecar collectors.

👉 Enables searching, alerting, and dashboards.

**4️⃣ Enable Log Rotation**

- Prevent disk from filling up with logs.

```bash
docker run \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  myapp
```

**5️⃣ Orchestrator Integration**

- **Docker Swarm**: Native support for logging drivers
- **Kubernetes**: Node-level log collection + centralized logging

---

# 17. Describe a scenario where you would use Docker security scanning tools to identify vulnerabilities in container images

You are building a **Dockerized microservice** that will be deployed to **production** in a regulated or security-sensitive environment (e.g., fintech or healthcare).

### Context

- Developers build Docker images using a base image like `node:18`
- The image includes OS packages and application dependencies
- Before deployment, you must ensure the image has **no critical vulnerabilities**

### How Security Scanning Is Used

**1️⃣ Scan During CI/CD Pipeline**

After building the image, run a security scan:

```bash
trivy image myapp:1.2.0
```

- The scan detects:
    - Vulnerable OS packages
    - Outdated libraries
    - Known CVEs with severity levels

**2️⃣ Act on Scan Results**

- **Fail the pipeline** if critical/high vulnerabilities are found
- Update base image or dependencies
- Rebuild and rescan until issues are resolved

👉 Prevents vulnerable images from reaching production.

**3️⃣ Ongoing Security Monitoring**

- Periodically rescan images stored in the registry
- Catch new vulnerabilities discovered **after image creation**

### Why This Scenario Matters (1–2 Points)

- Containers often inherit vulnerabilities from base images
- Early detection reduces **security breaches and compliance risks**

### Real-World Example

- A base image includes an outdated OpenSSL version
- Scanner flags a **CRITICAL CVE**
- Image is rebuilt using a patched base image before deployment

---

# 18. How would you implement backup and disaster recovery strategies for Dockerized applications?

Backup and disaster recovery (DR) focus on **data, images, and configuration**, not the containers themselves (since containers are ephemeral).

**1️⃣ Backup Persistent Data (Most Critical)**

- Store application data in **Docker volumes or external databases**
- Regularly back up volumes

```bash
docker run --rm \
  -v app_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/app_data.tar.gz /data
```

👉 Ensures data can be restored even if containers are lost.

**2️⃣ Backup Images & Configuration**

- Push Docker images to a **remote registry** (Docker Hub, ECR, GCR)
- Version-control:
    - `Dockerfile`
    - `docker-compose.yml`
    - Infrastructure-as-Code (Terraform, CloudFormation)

👉 Allows fast redeployment in a new environment.

**3️⃣ Use Orchestration for High Availability**

- Docker Swarm / Kubernetes automatically:
    - Restart failed containers
    - Reschedule workloads on healthy nodes

👉 Reduces downtime during node failures.

**4️⃣ Disaster Recovery Strategy**

- Maintain **off-site backups** (different region)
- Test restoration regularly:
    - Restore volume backups
    - Redeploy containers from images

**5️⃣ Automate & Test DR**

- Schedule backups using cron or CI/CD jobs
- Periodically simulate failures (chaos testing)

---

**Most asked docker question**
https://cloudchamp.notion.site/Docker-Interview-Questions-9317cee2392147798571c09cf27bad85
