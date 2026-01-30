🔹 **Basic Kubernetes Questions**

## 1. What is Kubernetes and why is it used?

**Kubernetes** is an **open-source container orchestration platform** used to **deploy, manage, and scale containerized applications automatically**.

**Why it is used:**

It solves problems like **manual deployment, scaling, and failure handling** by automating container management across multiple servers.

**How it works (brief):**

Kubernetes groups containers into **pods**, runs them on a **cluster of nodes**, and continuously ensures the **desired state** (correct number of replicas, healthy containers).

**Example:**

If one container crashes, Kubernetes **automatically restarts it**, and if traffic increases, it **scales the application** by creating more pods.

---

## 2. What problem does Kubernetes solve?

**Kubernetes solves the problem of managing containerized applications at scale.**

**Before Kubernetes:**

Teams had to **manually deploy, scale, and monitor containers**, which became difficult as the number of services and servers increased.

**Problems it solves:**

- **Manual scaling** → Kubernetes auto-scales applications
- **Container failures** → Kubernetes self-heals (restart, reschedule)
- **Load balancing** → Automatically distributes traffic
- **Environment inconsistency** → Same behavior across dev, test, prod

**Example:**

If a pod crashes or a node goes down, Kubernetes **automatically recreates the pod on another node** without manual intervention.

---

## 3. What are the main components of Kubernetes architecture?

The **main components of Kubernetes architecture** are divided into **Control Plane components** and **Node (Worker) components**. This is a classic DevOps interview question.

### 1. Control Plane Components (Master manages the cluster)

| **Component** | **Purpose** |
| --- | --- |
| **kube-apiserver** | Exposes the Kubernetes API. All communication (kubectl, controllers, etc.) goes through it. |
| **etcd** | Distributed key-value store. Stores cluster state, configuration, and secrets. |
| **kube-scheduler** | Assigns pods to nodes based on resource availability and constraints. |
| **kube-controller-manager** | Runs controllers that maintain desired state (replica count, node health, endpoints). |
| **cloud-controller-manager** | Integrates Kubernetes with cloud providers (load balancers, VMs, networking). |

### 2. Node (Worker) Components

| **Component** | **Purpose** |
| --- | --- |
| **kubelet** | Agent running on each node. Ensures containers in pods are running as defined. |
| **kube-proxy** | Handles networking and load balancing for services on the node. |
| **Container Runtime** | Runs containers (Docker, containerd, CRI-O). |

### 3. Optional / Higher-Level Concepts

- **Pods:** Smallest deployable unit, can contain one or more containers.
- **Services:** Abstracts pods and provides stable networking.
- **Namespaces:** Logical isolation for resources.
- **ConfigMaps & Secrets:** Store configuration and sensitive data.

---

## 4. What is a cluster in Kubernetes?

In Kubernetes, a **cluster** is a set of **nodes (machines)** that work together to run containerized applications. It represents the **entire Kubernetes environment** where workloads are deployed, managed, and scaled.

**How it works:**

- A cluster has **two main types of nodes**:
    1. **Control Plane (Master) Node(s):**
        - Manages the cluster state
        - Runs components like **kube-apiserver, etcd, kube-scheduler, controller-manager**
    2. **Worker Node(s):**
        - Runs **pods** (containers)
        - Has **kubelet, kube-proxy, and container runtime**
- The **control plane** ensures that the **desired state** (number of pods, configuration, networking) is maintained across all worker nodes.

**Why it is used:**

- Provides a **single logical environment** for deploying applications.
- Enables **high availability**: if one node fails, pods are rescheduled on other nodes.
- Supports **scaling**: add or remove nodes to match workload demand.

**Example:**

- A company has a Kubernetes cluster with 3 worker nodes.
- You deploy a web service with 6 replicas. Kubernetes automatically **distributes the pods across the nodes**.
- If one node crashes, the pods on that node are **rescheduled on remaining nodes**, ensuring the service stays online.

---

## 5. What is a node?

In Kubernetes, a **node** is a physical or virtual machine that runs the workloads (containers) in the cluster. Nodes are the workers of a Kubernetes cluster and execute the pods scheduled by the control plane.

**How it works:**

Every node has these components:

- **kubelet** – Agent that ensures containers in pods are running and healthy.
- **kube-proxy** – Handles networking and load balancing for services.
- **Container Runtime** – Software that runs containers (Docker, containerd, CRI-O).

Nodes register with the control plane, which schedules pods onto them.

**Types of Nodes:**

- **Master/Control Plane Node** – Runs cluster management components (API server, scheduler, controller manager).
- **Worker Node** – Runs the actual application workloads (pods).

**Why nodes are important:**

- They provide compute, storage, and network resources for pods.
- Kubernetes can distribute workloads across nodes for load balancing and fault tolerance.
- Adding more nodes increases cluster capacity and scalability.

**Example:**

- You have a cluster with 3 worker nodes.
- You deploy a web application with 6 pods. Kubernetes distributes 2 pods per node.
- If one node fails, the pods on that node are rescheduled to other nodes, keeping the application running.

---

## 6. What is a pod?

In Kubernetes, a **pod** is the **smallest deployable unit**. It represents **one or more containers** that share the **same network namespace, storage, and configuration**. Pods are the building blocks of Kubernetes applications.

**How it works:**

- A pod can contain **a single container** or **multiple tightly coupled containers** (called a **multi-container pod**).
- Containers in the same pod share:
    - **IP address and port space** → can communicate via [`localhost`](http://localhost)
    - **Volumes** → shared storage
    - **Namespace and lifecycle** → they start, stop, and restart together
- Pods are **ephemeral**. If a pod dies, Kubernetes can **create a new pod** based on the Deployment or ReplicaSet definition.

**Why pods are used:**

- They provide a **logical host** for containers in Kubernetes.
- They allow **tight coupling of containers** (sidecars, logging agents, proxies).
- Kubernetes schedules pods onto nodes based on **resource requirements and constraints**.

**Example:**

- A web application pod might contain:
    - **Main container:** Nginx server
    - **Sidecar container:** Log collector or monitoring agent
- Both containers share the **same network and storage**, but run separate processes.
- If the node crashes, Kubernetes **reschedules the pod** to another node, ensuring the application remains available.

---

## 7. Difference between container and pod?

The **difference between a container and a pod** is a common Kubernetes interview question, and it's important to answer **clearly with both technical and practical perspective**.

### 1. Container

- **Definition:** A container is a **single, isolated runtime environment** for an application and its dependencies.
- **Scope:** Runs a single process, has its own filesystem, network namespace (unless shared).
- **Lifecycle:** Managed by the container runtime (Docker, containerd).
- **Standalone:** Can run independently outside Kubernetes.

**Example:**

- A Docker container running an Nginx web server.

### 2. Pod

- **Definition:** A pod is the **smallest deployable unit in Kubernetes**, which can contain **one or more containers**.
- **Scope:** Containers in a pod **share network, storage, and lifecycle**.
- **Lifecycle:** Managed by Kubernetes (via Deployment, ReplicaSet, etc.).
- **Not standalone:** A pod cannot exist outside Kubernetes.

**Example:**

- A pod with:
    - Nginx container (main app)
    - Sidecar container (logs collector)
    - Both share the same IP, port space, and volumes

### Key Differences (Interview-Ready Table)

| **Feature** | **Container** | **Pod** |
| --- | --- | --- |
| Unit of Deployment | Single application | One or more containers |
| Isolation | Own filesystem, network | Shared IP, port, storage between containers |
| Lifecycle Management | Container runtime | Kubernetes scheduler & controllers |
| Scalability | Independent scaling possible | Pod scaled as a unit |
| Standalone | Yes | No, needs Kubernetes |

---

## 8. What is kube-apiserver?

**kube-apiserver** is the **central component of the Kubernetes control plane** and serves as the **main entry point for all administrative tasks and API requests** in the cluster.

**How it works:**

- All interactions with the Kubernetes cluster—whether from **kubectl, other control plane components, or external tools**—go through the **API server**.
- It **validates and processes requests**, then updates the cluster state in **etcd** (the key-value store).
- Exposes **RESTful APIs** for all Kubernetes resources like pods, deployments, services, secrets, etc.

**Why it is used:**

- Acts as the **gatekeeper and brain** of the cluster.
- Ensures **desired state consistency**: e.g., when you create a deployment, kube-apiserver stores it in etcd and the controllers act to maintain it.
- Provides a **secure and consistent interface** for managing the cluster.

**Example scenario:**

1. You run:

```
kubectl get pods
```

1. `kubectl` sends the request to **kube-apiserver**.
2. kube-apiserver **authenticates and authorizes** the request.
3. It queries **etcd** for pod information and returns the result.

---

## 9. What is etcd?

**etcd** is a **distributed key-value store** that Kubernetes uses to **persist the cluster state and configuration**. It is a **critical component of the control plane**.

**How it works:**

- Stores the **entire state of the cluster**, including:
    - Pods, Deployments, Services
    - ConfigMaps, Secrets
    - Node information, labels, and annotations
- **Highly available and consistent**: etcd uses the **Raft consensus algorithm** to ensure all nodes have the same data.
- kube-apiserver interacts with etcd to **read/write cluster state**.

**Why it is used:**

- **Single source of truth** for Kubernetes cluster state.
- Ensures **consistency and reliability** across control plane components.
- Enables **cluster recovery** if control plane components crash: etcd contains all definitions to restore pods and resources.

**Example scenario:**

1. You create a Deployment:

```
kubectl apply -f deployment.yaml
```

1. kube-apiserver validates the request and writes the Deployment object to **etcd**.
2. Controller-manager reads the desired state from etcd and creates pods accordingly.
- If the master node restarts, **etcd data ensures the cluster comes back to the desired state**.

---

## 10. What is kube-scheduler?

**kube-scheduler** is a **control plane component** in Kubernetes that is responsible for **assigning pods to nodes** in the cluster based on resource availability and scheduling policies.

**How it works:**

1. When a new pod is created (or a pod needs rescheduling), it **remains unscheduled** until kube-scheduler processes it.
2. kube-scheduler **evaluates all available nodes** and selects the most suitable one based on:
    - CPU, memory, and other resource requirements
    - Node labels and taints/tolerations
    - Affinity/anti-affinity rules
    - Custom policies or priorities
3. Once a node is selected, kube-scheduler **binds the pod** to that node.
4. kubelet on the assigned node then starts the container(s) in the pod.

**Why it is used:**

- Ensures **optimal utilization of cluster resources**.
- Enforces **scheduling policies**, such as spreading pods across nodes for high availability.
- Automatically handles **rescheduling** when nodes go down or resources change.

**Example scenario:**

- A pod requires 2 CPU cores and 4GB memory.
- The cluster has 3 nodes:
    - Node A: 1 CPU, 2GB → not enough resources
    - Node B: 4 CPU, 8GB → sufficient resources
    - Node C: 2 CPU, 4GB → exact match
- kube-scheduler selects **Node B or Node C** depending on policies and schedules the pod there.

---

## 11. What is kubelet?

**kubelet** is a **node-level agent** in Kubernetes that runs on every worker node. Its main responsibility is to **ensure that containers in pods are running and healthy** according to the pod specifications.

**How it works:**

1. kubelet receives **pod specifications** (PodSpecs) from the **kube-apiserver**.
2. It interacts with the **container runtime** (Docker, containerd, or CRI-O) to **start, stop, or restart containers**.
3. Continuously monitors the **health of containers** using **liveness and readiness probes**.
4. Reports the **status of pods and node resources** back to the kube-apiserver.

**Why it is used:**

- Acts as the **"executor" on each node**, ensuring Kubernetes desired state is applied locally.
- Performs **self-healing** by restarting failed containers.
- Enables **node-level monitoring and reporting** to the control plane.

**Example scenario:**

- You deploy a pod with an Nginx container.
- kube-scheduler assigns the pod to Node B.
- kubelet on Node B instructs Docker to start the Nginx container.
- If the container crashes, kubelet **automatically restarts it**.
- Node status and pod health are reported to kube-apiserver.

---

## 12. What is kubectl?

**kubectl** is the **command-line interface (CLI) tool** used to interact with a Kubernetes cluster. It allows developers and administrators to **create, manage, and troubleshoot Kubernetes resources**.

**How it works:**

1. You run a **kubectl command** on your machine.
2. kubectl sends an **API request to the kube-apiserver**.
3. kube-apiserver validates the request and updates **etcd** or returns the requested cluster state.
4. Output (e.g., list of pods, logs) is displayed in your terminal.

**Why it is used:**

- Provides **control over all Kubernetes resources** (pods, deployments, services, etc.).
- Enables **cluster management** from anywhere, as long as you have access credentials.
- Useful for **deployment, scaling, troubleshooting, and debugging**.

**Example commands:**

- List pods:

```
kubectl get pods
```

- Get pod details:

```
kubectl describe pod <pod-name>
```

- Apply a manifest file:

```
kubectl apply -f deployment.yaml
```

- View pod logs:

```
kubectl logs <pod-name>
```

---

## 13. What is a namespace and why is it used?

In Kubernetes, a **namespace** is a **logical partition within a cluster** that allows you to **divide cluster resources between multiple users, teams, or environments**. It provides **scope for names** and **resource isolation**.

**How it works:**

- Kubernetes objects (pods, services, deployments, etc.) exist **within a namespace**.
- Names of resources only need to be **unique within the same namespace**, not across the cluster.
- Default namespaces include:
    - `default` → used if no namespace is specified
    - `kube-system` → for control plane components
    - `kube-public` → publicly readable resources

**Why it is used:**

1. **Resource isolation:** Different teams can share a cluster without interfering with each other.
2. **Organization:** Separate environments (dev, staging, prod) within the same cluster.
3. **Access control:** RBAC can be applied per namespace to limit permissions.

**Example scenario:**

- Team A deploys pods in `dev` namespace.
- Team B deploys pods in `staging` namespace.
- Both can use a pod named `web-app` without conflicts because the **namespace separates their scope**.
- RBAC rules can allow Team A to only manage `dev` namespace, and Team B only `staging`.

---

## 14. What is a label and selector?

In Kubernetes, **labels** and **selectors** are used to **organize, identify, and group resources** in a cluster. They are a **core mechanism for management and scheduling**.

### 1. Label

- **Definition:** A **key-value pair** attached to Kubernetes objects (pods, services, deployments, etc.).
- **Purpose:** Adds **metadata to resources** to categorize or identify them.
- **Syntax example:**

```
labels:
  app: web-app
  environment: dev
```

**Why it's used:**

- Helps to **filter, query, and manage resources** dynamically.
- Works for grouping pods, selecting workloads, and applying policies.

### 2. Selector

- **Definition:** A **query that matches objects based on their labels**.
- **Purpose:** Allows **controllers, services, and deployments to target specific pods**.
- **Example:**

```
selector:
  matchLabels:
    app: web-app
```

- This selector matches **all pods labeled with `app=web-app`**.

**Example scenario:**

- You have 6 pods in your cluster:
    - 3 labeled `app=web-app, environment=dev`
    - 3 labeled `app=web-app, environment=prod`
- A service uses the selector `environment=prod` → **only routes traffic to prod pods**.
- A Deployment with the same selector ensures **only the intended pods are managed**.

---

## 15. What is a manifest file?

In Kubernetes, a **manifest file** is a **YAML (or JSON) file that defines the desired state of a Kubernetes resource**. It tells Kubernetes **what to create, how it should behave, and its configuration**.

**How it works:**

- Manifest files describe **Kubernetes objects** such as:
    - Pods
    - Deployments
    - Services
    - ConfigMaps, Secrets, etc.
- When you run:

```
kubectl apply -f deployment.yaml
```

- Kubernetes reads the manifest and **creates or updates the resource** to match the desired state.
- The **control plane components** (API server, controllers) ensure the cluster **matches the specifications** in the manifest.

**Key parts of a manifest file:**

1. **apiVersion** → Kubernetes API version for the object
2. **kind** → Type of resource (Pod, Deployment, Service)
3. **metadata** → Name, labels, annotations
4. **spec** → Configuration of the resource (containers, replicas, ports, etc.)

**Example manifest (Deployment):**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

- This manifest tells Kubernetes to **create 3 replicas of a pod** running Nginx.
- Kubernetes ensures that the **actual state matches this desired state** at all times.

**Why it's used:**

- **Declarative configuration** → "Tell Kubernetes what you want, it ensures it happens."
- **Version control friendly** → YAML files can be stored in Git.
- Enables **reproducibility and automation**.

---

🔹 **Workloads & Controllers (Core Concepts)**

## 16. What is a Deployment?

**In Kubernetes, a** Deployment **is a** higher-level controller that manages pods and ReplicaSets**. It ensures that the** desired number of pod replicas are running, up-to-date, and healthy.

**How it works:**

1. You define a Deployment in a **manifest file**, specifying:
    - Number of replicas
    - Pod template (container image, ports, environment variables, etc.)
    - Update strategy (rolling update or recreate)
2. When applied via `kubectl`, Kubernetes:
    - Creates a **ReplicaSet** automatically
    - Ensures the **specified number of pods are running**
    - Monitors pods and **recreates them if they fail**
    - Handles **updates to pod templates** via rolling updates

**Why it is used:**

- **Self-healing:** Automatically restarts failed pods
- **Scaling:** Easy to scale up or down by changing replicas
- **Rolling updates:** Deploy new versions without downtime
- **Rollback:** Can revert to a previous version if the update fails

**Example manifest:**

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

- This Deployment ensures **3 replicas of an Nginx pod** are running.
- If a pod crashes, Kubernetes **creates a new one automatically**.

---

## 17. Difference between Deployment and StatefulSet?

The **difference between a Deployment and a StatefulSet** is a classic Kubernetes interview question. Both are controllers that manage pods, but they serve **different use cases**.

### 1. Deployment

- **Purpose:** Manages **stateless applications** (web servers, APIs, microservices).
- **Pod identity:** Pods are **interchangeable**, have no stable network identity.
- **Storage:** Pods typically use **ephemeral storage**; not tied to persistent volumes by default.
- **Scaling:** Easy to scale up or down; new pods are identical and can be replaced arbitrarily.
- **Update strategy:** Supports **rolling updates** with automatic rollback.

**Example use case:** Web servers, stateless microservices, API servers.

### 2. StatefulSet

- **Purpose:** Manages **stateful applications** that require **stable identity, network, and storage** (databases, caches).
- **Pod identity:** Each pod gets a **stable hostname and ordinal index** (pod-0, pod-1, pod-2).
- **Storage:** Each pod can have its **own PersistentVolume** that persists across restarts.
- **Scaling:** Pods are created/destroyed **in order** (sequentially).
- **Update strategy:** Rolling updates are **ordered and controlled**, preserving pod identity.

**Example use case:** MySQL, MongoDB, Kafka, Redis clusters.

### Key Differences (Interview-Ready Table)

| **Feature** | **Deployment** | **StatefulSet** |
| --- | --- | --- |
| Use case | Stateless apps | Stateful apps |
| Pod identity | Interchangeable | Stable, unique |
| Storage | Ephemeral by default | Persistent per pod |
| Pod creation | Any order | Sequential, ordered |
| Network identity | Dynamic | Stable DNS & hostname |
| Scaling | Easy | Ordered scaling & updates |
| Update/rollback | Rolling update | Ordered rolling update |

---

## 18. What is a ReplicaSet?

**A** ReplicaSet **is a** Kubernetes controller **that ensures a** specified number of identical pods **are running at all times. It is the** underlying mechanism used by Deployments to manage pods.

**How it works:**

1. You define a **ReplicaSet** with:
    - A **pod template**
    - The **desired number of replicas**
2. Kubernetes constantly monitors the cluster:
    - If a pod dies or a node fails, the ReplicaSet **creates a new pod** to maintain the desired count.
    - If extra pods exist, it **terminates excess pods**.

**Why it is used:**

- **High availability:** Ensures that the application always has the required number of pods running.
- **Self-healing:** Automatically replaces failed or deleted pods.
- **Foundation for Deployments:** Deployments use ReplicaSets for rolling updates and scaling.

---

## 19. What is a DaemonSet?

A **DaemonSet** is a Kubernetes controller that ensures **one (or more) pod runs on every (or selected) node** in a cluster. It's mainly used for **running cluster-level or node-level services**.

**How it works:**

1. You define a **DaemonSet** with a pod template.
2. Kubernetes ensures:
    - **One pod per node** (or per node matching a label selector).
    - When a **new node is added**, a pod is automatically created on it.
    - When a **node is removed**, the pod on that node is deleted.

**Why it is used:**

- **Node-level services:** DaemonSets are ideal for services that need to run on every node.
- **Examples:**
    - Log collection (Fluentd, Logstash)
    - Monitoring agents (Prometheus Node Exporter, Datadog agent)
    - Networking plugins (Calico, Weave)
- Ensures **uniform coverage** across the cluster.

---

## 20. What is a Job and CronJob?

In Kubernetes, **Job** and **CronJob** are **controllers used for running batch or scheduled tasks**, unlike Deployments or StatefulSets which manage long-running applications.

### 1. Job

**Definition:**

A **Job** ensures that **one or more pods run to completion** successfully. It is used for **finite, short-lived tasks** rather than long-running services.

**How it works:**

- You define a pod template and **the number of completions** (usually 1).
- Kubernetes creates pods to run the task.
- Once the task finishes successfully, the **Job is marked complete**.
- If a pod fails, Kubernetes **retries until completion**.

**Example use case:**

- Data processing job
- Database migration
- One-time script execution

**Example manifest:**

```
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: migration-tool:latest
      restartPolicy: OnFailure
```

### 2. CronJob

**Definition:**

A **CronJob** is like a **Job with a schedule**, similar to a Linux `cron` job. It runs tasks **periodically at defined intervals**.

**How it works:**

- You specify a **cron schedule** (e.g., `"0 0 * * *"`).
- Kubernetes creates a **Job for each scheduled execution**.
- Supports retry on failure and concurrency policies.

**Example use case:**

- Daily database backup
- Periodic report generation
- Cleanup jobs

**Example manifest:**

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *" # 2 AM daily
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool:latest
          restartPolicy: OnFailure
```

### Key Differences (Job vs CronJob)

| **Feature** | **Job** | **CronJob** |
| --- | --- | --- |
| Purpose | Run task to completion | Run tasks on a schedule |
| Execution | Once (or specified completions) | Repeated automatically |
| Use case | Database migration, script execution | Daily backups, periodic reporting |
| Scheduling | Manual | Cron-like schedule |

---

## 21. What is rolling update in Kubernetes?

A **rolling update** in Kubernetes is a **deployment strategy** that allows you to **update an application's pods gradually without downtime**, ensuring the application remains available while new versions are rolled out.

**How it works:**

1. You update a **Deployment** with a new container image or configuration.
2. Kubernetes **creates new pods** with the updated version while **gradually terminating old pods**.
3. The number of pods updated at a time is controlled by the **maxUnavailable** and **maxSurge** settings in the Deployment.
4. Kubernetes continuously monitors **health checks** (liveness/readiness probes) to ensure the new pods are running correctly.
5. If any new pod fails, the update **pauses or rolls back automatically** depending on the policy.

**Why it is used:**

- **Zero downtime deployments:** Users can continue using the application while updates are applied.
- **Safe updates:** Only a subset of pods is updated at a time, reducing risk.
- **Automated rollback:** Failed updates can be reverted automatically.

**Example scenario:**

- Deployment has 3 replicas running Nginx `1.21`.
- You update the Deployment to Nginx `1.22`.
- Kubernetes creates **one new pod** with `1.22`, waits until it's ready, then deletes one old pod with `1.21`.
- Process repeats until all pods are updated.

---

## 22. What is rollback in Kubernetes?

In Kubernetes, a **rollback** is the process of **reverting an application Deployment to a previous stable version** if the current update fails or behaves unexpectedly. It ensures **high availability and reliability** of applications.

**How it works:**

1. Kubernetes **keeps a revision history** of Deployment updates.
2. If a rolling update introduces issues (failed pods, errors, unhealthy readiness checks), you can **rollback to a previous revision**.
3. Rollback restores the **old pod template**, **replica count**, and configuration.
4. Kubernetes automatically **creates pods with the previous version** and removes the problematic pods.

**Why it is used:**

- **Recover from failed deployments** quickly.
- **Maintain uptime** while ensuring application stability.
- Works with **rolling updates** to make deployments safer.

**Example scenario:**

- Deployment has 3 replicas running Nginx `1.21`.
- You update to Nginx `1.22`.
- After update, pods crash due to configuration issues.
- Run rollback command:

```
kubectl rollout undo deployment web-app
```

- Kubernetes restores **Nginx `1.21` pods** automatically, and the service remains available.

**Commands:**

- **Check rollout status:**

```
kubectl rollout status deployment web-app
```

- **Rollback to previous version:**

```
kubectl rollout undo deployment web-app
```

- **View rollout history:**

```
kubectl rollout history deployment web-app
```

---

## 23. What are init containers?

**Init containers** in Kubernetes are **special containers that run before the main containers in a pod start**. They perform **initialization tasks** and ensure the pod environment is ready for the application containers.

**How it works:**

1. A pod can define **one or more init containers** in its manifest.
2. Init containers **run sequentially**, one after another.
3. Each init container must **complete successfully** before the main application containers start.
4. If an init container fails, Kubernetes **retries it until it succeeds**.

**Why they are used:**

- **Pre-initialization:** Set up environment, fetch secrets, or configure settings before main app starts.
- **Dependency handling:** Wait for external services or databases to be available.
- **Security isolation:** Init containers can run with **different privileges** from main containers.

**Example scenario:**

- A web app pod needs a **configuration file from a remote storage**.
- You create an **init container** that downloads the file into a shared volume.
- Only after the init container completes does the main container (web app) start.

---

## 24. What is a sidecar container?

A **sidecar container** is a **secondary container in a pod** that runs alongside the main application container to provide **supporting functionality**. Unlike init containers, **sidecars run concurrently** with the main container for the lifetime of the pod.

**How it works:**

1. A pod can have **one main container** and **one or more sidecar containers**.
2. Sidecars share:
    - **Network namespace** → can communicate with main container via [`localhost`](http://localhost)
    - **Volumes** → share files, logs, or configuration
3. Sidecars **enhance or extend the main application** without changing its code.

**Why it is used:**

- **Logging / monitoring:** Collect and forward logs (e.g., Fluentd, Logstash).
- **Proxies:** Add service mesh functionality (e.g., Envoy in Istio).
- **Data synchronization:** Sync files, configs, or secrets from external sources.
- **Security / authentication:** Handle certificates, tokens, or encryption for main app.

**Example scenario:**

- Main container: Nginx web server
- Sidecar container: Log collector that streams logs to a central server
- Both containers write logs to a **shared volume** so the main container doesn't need to manage logging itself.

**🔹 Networking**

## 25. How does Kubernetes networking work?

**Kubernetes networking defines** how pods, services, and external users communicate with each other **in a cluster. It follows a** simple but powerful model to enable seamless communication.

**Core networking principles:**

1. **Every pod gets its own IP address**
    - Pods can communicate with each other **directly using pod IPs**, without NAT.
2. **Pods can talk to nodes and other pods**
    - No port conflicts because each pod has its own network namespace.
3. **Networking is implemented by a CNI plugin**
    - Examples: **Calico, Flannel, Weave, Cilium**

**How it works step by step:**

1. When a pod is created, the **CNI plugin assigns an IP** to the pod.
2. The CNI sets up routing so that **pods across different nodes can reach each other**.
3. **kube-proxy** runs on each node and manages traffic routing for Services.
4. **Services provide stable IPs and DNS names**, even if pods are recreated.

**Key components involved:**

- **Pod-to-Pod networking:**
    - Uses the cluster network (flat network model).
- **Service networking:**
    - Services expose pods using **ClusterIP, NodePort, or LoadBalancer**.
- **DNS:**
    - CoreDNS enables service discovery using names like `service-name.namespace`.

**Example flow:**

- Pod A sends a request to `backend-service`.
- DNS resolves it to a **Service IP**.
- kube-proxy forwards traffic to one of the **healthy backend pods**.
- If a pod dies, traffic is automatically routed to another pod.

---

## 26. What is a Service?

In Kubernetes, a **Service** is an abstraction that provides a **stable network endpoint** to access a group of pods, even though pods are **dynamic and can change IPs**.

**Why a Service is needed:**

- Pods are **created and destroyed frequently**, so their IPs change.
- A Service gives a **fixed IP and DNS name** to access pods.
- It also provides **load balancing** across multiple pod replicas.

**How it works:**

1. A Service uses **labels and selectors** to identify target pods.
2. kube-proxy routes traffic sent to the Service IP to one of the matching pods.
3. If pods are replaced, the Service automatically updates its endpoints.

**Example scenario:**

- You have 3 backend pods with label `app=backend`.
- A Service selects these pods and exposes them via a single endpoint.
- Clients use the Service name instead of pod IPs.

---

## 27. Types of Services in Kubernetes?

Kubernetes provides **different Service types** to expose applications **inside or outside the cluster**, depending on the access requirement.

### 1. ClusterIP (Default)

**What it is:**

- Exposes the Service **only inside the cluster**.
- Assigned an **internal virtual IP**.

**Use case:**

- Backend or internal microservices.

**Example:**

```
type: ClusterIP
```

### 2. NodePort

**What it is:**

- Exposes the Service on a **static port** on each node (range: `30000–32767`).
- Accessible using:

```
NodeIP:NodePort
```

**Use case:**

- Simple external access (mainly for testing).

**Example:**

```
type: NodePort
```

### 3. LoadBalancer

**What it is:**

- Creates an **external cloud load balancer**.
- Routes external traffic to nodes → pods.

**Use case:**

- Production-grade external access in cloud environments (AWS, Azure, GCP).

**Example:**

```
type: LoadBalancer
```

### 4. ExternalName

**What it is:**

- Maps a Service to an **external DNS name**.
- No pod selector or proxying.

**Use case:**

- Accessing external services (e.g., external database).

**Example:**

```
type: ExternalName
externalName: db.example.com
```

### Service Type Comparison

| **Service Type** | **Access Scope** | **Common Use** |
| --- | --- | --- |
| ClusterIP | Internal only | Internal microservices |
| NodePort | External via node IP | Testing, demos |
| LoadBalancer | External via cloud LB | Production apps |
| ExternalName | External DNS | External dependencies |

---

## 28. What is an Ingress?

**An** Ingress **in Kubernetes is an** API object that manages external HTTP/HTTPS access **to services inside a cluster, usually using** URL paths or domain names**. It acts as a** smart entry point for your applications.

**Why Ingress is used:**

- Avoids creating **multiple LoadBalancer services** (cost-effective).
- Provides **path-based and host-based routing**.
- Supports **TLS/SSL termination**.
- Centralizes traffic management.

**How it works:**

1. You define **Ingress rules** (host/path → service).
2. An **Ingress Controller** (like NGINX, Traefik, HAProxy) watches these rules.
3. The controller configures a reverse proxy/load balancer.
4. External traffic is routed to the correct service based on rules.

**Example scenario:**

- [`example.com/app1`](http://example.com/app1) → frontend service
- [`example.com/api`](http://example.com/api) → backend service
- All traffic enters through **one load balancer**.

---

## 29. Difference between Ingress and LoadBalancer?

| **Feature** | **LoadBalancer** | **Ingress** |
| --- | --- | --- |
| Kubernetes object | Service | Ingress |
| Exposes | Single service | Multiple services |
| Routing | Basic | Advanced (path/host) |
| TLS termination | Limited | Built-in |
| Cost | Higher | Lower |
| Traffic types | Any (TCP/UDP) | HTTP/HTTPS only |

---

## 30. What is CoreDNS?

**CoreDNS** is the **DNS server used by Kubernetes** for **service discovery**. It translates **service and pod names into IP addresses**, allowing applications to communicate using names instead of hard-coded IPs.

**Why CoreDNS is used:**

- Pods and services are **dynamic** and IPs change frequently.
- CoreDNS provides **name-based discovery** inside the cluster.
- It replaces the older **kube-dns** with a faster and more flexible solution.

**How it works:**

1. When a pod tries to access a service like `backend-service`, it sends a DNS query.
2. CoreDNS resolves:

```
backend-service.namespace.svc.cluster.local
```

1. CoreDNS returns the **Service IP** (ClusterIP).
2. Traffic is routed to one of the backend pods via the Service.

**What CoreDNS resolves:**

- **Services:** `service-name.namespace.svc.cluster.local`
- **Pods (optional):** `pod-ip.namespace.pod.cluster.local`
- **External domains:** Forwards queries to upstream DNS (e.g., Google DNS)

**Example scenario:**

- Frontend pod calls [`http://api-service`](http://api-service)
- CoreDNS resolves the name to the API Service IP
- Request reaches a healthy API pod automatically

---

## 31. How do pods communicate with each other?

Pods in Kubernetes communicate with each other using the **cluster's flat networking model**, where **every pod has its own IP address** and can reach other pods directly.

**Ways pods communicate:**

### 1. Pod-to-Pod (direct)

- Each pod gets a **unique IP**.
- Pods can talk to each other using **pod IP + port**, even across nodes.
- No NAT is required.

**Example:**

```
http://10.244.1.5:8080
```

⚠️ Pod IPs are not stable, so this method is **not recommended for production**.

### 2. Pod-to-Pod via Service (recommended)

- Pods communicate using a **Service name** instead of pod IP.
- CoreDNS resolves the service name to a **ClusterIP**.
- kube-proxy forwards traffic to one of the healthy pods.

**Example:**

```
http://backend-service:80
```

### 3. Pods within the same Pod

- Containers in the **same pod**:
    - Share the **same IP**
    - Communicate using [`localhost`](http://localhost)
- Common for **sidecar patterns**.

**Example:**

```
http://localhost:9000
```

**Networking components involved:**

- **CNI plugin** (Calico, Flannel): Enables pod networking across nodes
- **CoreDNS:** Resolves service names
- **kube-proxy:** Handles service traffic routing

**Example flow:**

1. Frontend pod calls `backend-service`.
2. CoreDNS resolves it to a Service IP.
3. kube-proxy routes traffic to a backend pod.
4. If a pod fails, traffic is redirected automatically.

---

## 32. What is a Network Policy?

A **NetworkPolicy** in Kubernetes is a **security feature** that controls **how pods are allowed to communicate with each other and with external traffic**. It acts like a **firewall for pods**.

**Why NetworkPolicy is used:**

- By default, **all pods can talk to each other** in a cluster.
- NetworkPolicy allows you to **restrict traffic** for better security.
- Helps implement **zero-trust networking** in Kubernetes.

**How it works:**

1. NetworkPolicy uses **labels and selectors** to choose target pods.
2. It defines **ingress (incoming)** and/or **egress (outgoing)** rules.
3. Once a policy is applied, **only allowed traffic is permitted**; everything else is denied.
4. Requires a **CNI plugin that supports NetworkPolicy** (e.g., Calico, Cilium).

**Example scenario:**

- Only frontend pods should access backend pods.
- Database pods should not be accessible from other services.

---

🔹 **Storage & Configuration**

## 33. What is a Volume?

In Kubernetes, a **Volume** is a **storage mechanism** that allows data to **persist beyond a container's lifecycle** inside a pod. It solves the problem of **ephemeral container storage**.

**Why Volume is needed:**

- Containers can be **restarted or recreated**, and their filesystem is lost.
- Volumes provide **shared and persistent storage** for containers in a pod.

---

## 34. What is StorageClass?

A **StorageClass** in Kubernetes is a way to **define different types of storage** and enable **dynamic provisioning of Persistent Volumes (PVs)** without manual intervention.

**Why StorageClass is used:**

- Avoids **manual creation of PersistentVolumes**.
- Allows teams to request storage **on demand**.
- Supports different storage needs like **SSD, HDD, encrypted, high-IOPS**.

---

🔹 **Security**

## 35. How do you secure a Kubernetes cluster?

Securing a Kubernetes cluster means protecting **access, workloads, network, and data**. Interviewers usually expect a **layered (defense-in-depth) answer**, not just one control.

### 1. Secure cluster access

- Use **RBAC** to give users and services **least-privilege access**.
- Disable anonymous access to the **kube-apiserver**.
- Use **strong authentication** (certs, OIDC, IAM in cloud).

**Example:**

Only CI/CD service account can deploy to `prod` namespace.

### 2. Secure workloads (pods & containers)

- Run containers as **non-root**.
- Use **Pod Security Standards** (restricted/baseline).
- Scan images for vulnerabilities before deployment.
- Use **read-only root filesystem** where possible.

### 3. Network security

- Apply **NetworkPolicies** to control pod-to-pod traffic.
- Expose services using **Ingress** instead of public NodePorts.
- Use **TLS** for service-to-service communication.

### 4. Protect secrets & data

- Store sensitive data in **Secrets**, not ConfigMaps.
- Enable **encryption at rest** for etcd.
- Restrict access to secrets via RBAC.

### 5. Secure the control plane

- Restrict access to **etcd** (no public access).
- Enable **audit logging** on kube-apiserver.
- Keep Kubernetes and components **up to date**.

### 6. Node & runtime security

- Harden nodes (OS patches, minimal packages).
- Use **container runtime security tools** (Falco, AppArmor, SELinux).
- Disable SSH access where possible.

### 7. Monitoring & auditing

- Enable **audit logs**.
- Monitor cluster activity and alerts.
- Detect unusual behavior early.

---

## 36. What is RBAC in Kubernetes?

**RBAC (Role-Based Access Control)** in Kubernetes is a **security mechanism** that controls **who can do what on which resources** in a cluster. It enforces the **principle of least privilege**.

**Why RBAC is used:**

- Prevents unauthorized access to cluster resources
- Limits user and service permissions
- Improves cluster security and auditability

**How RBAC works:**

RBAC is based on **four main objects**:

1. **Role**
    - Defines permissions **within a namespace**
    - Example: allow `get`, `list`, `create` pods
2. **ClusterRole**
    - Defines permissions **cluster-wide**
    - Used for nodes, namespaces, or global access
3. **RoleBinding**
    - Assigns a **Role** to a user, group, or service account
4. **ClusterRoleBinding**
    - Assigns a **ClusterRole** cluster-wide

**Example scenario:**

- Developer can **view pods** but cannot delete them.
- CI/CD service account can **deploy applications** only in `prod` namespace.

---

## 37. What is a ServiceAccount?

A **ServiceAccount** in Kubernetes is an **identity used by applications (pods) to authenticate with the Kubernetes API**. It allows pods to securely access cluster resources without using a human user's credentials.

**Why ServiceAccount is used:**

- Pods often need to **interact with the Kubernetes API** (e.g., CI/CD tools, controllers).
- ServiceAccounts provide **machine-level authentication**.
- Works together with **RBAC** to control permissions.

**How it works:**

1. A ServiceAccount is created in a **namespace**.
2. Kubernetes automatically creates a **token (JWT)** for it.
3. When a pod uses that ServiceAccount, the token is **mounted inside the pod**.
4. The pod uses this token to authenticate to the API server.

**Example scenario:**

- A Jenkins pod needs to deploy applications to Kubernetes.
- Jenkins uses a ServiceAccount with permissions to create Deployments and Services.

---

## 38. What is Pod Security (PSP / PSA)?

**Pod Security** in Kubernetes refers to mechanisms that **control what a pod is allowed to do**, mainly from a **security and privilege perspective**. Kubernetes has evolved from **PSP (PodSecurityPolicy)** to **PSA (Pod Security Admission)**.

### 1. PodSecurityPolicy (PSP) – Deprecated

**What it was:**

- A **cluster-level security policy** that defined **allowed pod behaviors**.
- Enforced by the API server at pod creation time.

**What it controlled:**

- Running as **root or non-root**
- Use of **privileged containers**
- Access to **host network, PID, IPC**
- Volume types allowed

**Why it was removed:**

- Complex to configure
- Hard to manage with RBAC
- Deprecated in **Kubernetes 1.21** and removed in **1.25**

### 2. Pod Security Admission (PSA) – Current Standard

**What it is:**

- A **built-in admission controller** that enforces pod security using **predefined profiles**.
- Easier and standardized replacement for PSP.

**PSA security levels:**

1. **Privileged**
    - No restrictions
    - For system-level workloads
2. **Baseline**
    - Prevents **known privilege escalations**
    - Allows common app workloads
3. **Restricted**
    - Strongest security
    - Enforces **non-root**, seccomp, minimal privileges

**How PSA is applied:**

- Applied using **namespace labels**.

```
pod-security.kubernetes.io/enforce: restricted
pod-security.kubernetes.io/enforce-version: latest
```

**Example scenario:**

- `dev` namespace → **baseline**
- `prod` namespace → **restricted**
- `kube-system` → **privileged**

---

## 39. What is a namespace-level security boundary?

**A** namespace-level security boundary **in Kubernetes means using a** namespace to isolate resources, access, and policies **so that workloads in one namespace** cannot interfere with another.

**How it works:**

Security controls are **applied per namespace**, such as:

- **RBAC Roles & RoleBindings**
- **Pod Security Admission (PSA)**
- **NetworkPolicies**
- **ResourceQuota & LimitRange**

These controls apply **only inside that namespace**.

**Example scenario:**

- `dev` namespace:
    - Developers can deploy pods
    - PSA = baseline
- `prod` namespace:
    - Limited access
    - PSA = restricted
    - NetworkPolicy blocks all except required traffic

---

## 40. How do you restrict container privileges?

Restricting container privileges in Kubernetes is crucial to **minimize the risk of compromise**. You limit what a container can do at runtime so even if it's compromised, it **cannot harm the node or other pods**.

### 1. Run as non-root

- Ensure containers **do not run as root** unless necessary.
- Use **Pod Security Admission (PSA)** or **SecurityContext**.

```
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  runAsNonRoot: true
```

### 2. Drop unnecessary capabilities

- Linux capabilities allow containers to perform privileged operations.
- Drop all capabilities and add only required ones.

```
securityContext:
  capabilities:
    drop: ["ALL"]
    add: ["NET_BIND_SERVICE"]
```

### 3. Disable privileged mode

- Avoid `privileged: true` unless absolutely required.
- Privileged containers have **full host access**, which is risky.

```
securityContext:
  privileged: false
```

### 4. Read-only root filesystem

- Prevent modifications to the container filesystem.

```
securityContext:
  readOnlyRootFilesystem: true
```

### 5. Use Pod Security Admission (PSA)

- Apply **baseline** or **restricted** profiles to enforce:
    - Non-root containers
    - No privileged escalation
    - Read-only root filesystem

```
pod-security.kubernetes.io/enforce: restricted
```

### 6. Limit host access

- Avoid mounting **hostPath**, **hostNetwork**, or **hostPID** unless necessary.
- Prevent containers from accessing sensitive host resources.

---

## 41. What is image scanning?

**Image scanning** in Kubernetes and DevOps is the process of **analyzing container images for security vulnerabilities, misconfigurations, or compliance issues** before they are deployed.

**Why it's important:**

- Containers are often built from **public images**, which may contain **known vulnerabilities**.
- Running unscanned images can lead to **security breaches, malware, or compliance failures**.
- Helps enforce **DevSecOps practices** by catching issues early.

**How it works:**

1. The **container image** (Docker image, OCI image) is scanned.
2. The scanner checks for:
    - Known **CVEs (Common Vulnerabilities and Exposures)**
    - Outdated packages
    - Unsafe configurations (e.g., running as root)
3. Reports are generated, sometimes **blocking deployment** if critical issues are found.

**Tools for image scanning:**

- **Trivy** → Simple, fast, open-source
- **Clair** → Static analysis of container images
- **Anchore** → Policy-based scanning
- **Aqua Security, Prisma Cloud** → Enterprise solutions

**Example workflow:**

1. CI/CD pipeline builds Docker image.
2. Image is scanned with Trivy:

```
trivy image my-app:latest
```

1. Vulnerabilities are reported.
2. If critical, the pipeline **fails**, preventing deployment to Kubernetes.

---

🔹 **Scaling & Reliability**

## 42. What is Cluster Autoscaler?

The **Cluster Autoscaler** in Kubernetes is a component that **automatically adjusts the size of a cluster** based on the **resource demands of pods**. It **adds nodes when resources are insufficient** and **removes underutilized nodes** to optimize cost and efficiency.

**Why it is used:**

- Ensure the cluster can **accommodate pending pods**.
- **Reduce cost** by removing idle nodes.
- Improves **scalability and reliability** of workloads.

**How it works:**

1. The autoscaler watches **unschedulable pods**.
2. If pods cannot be scheduled due to insufficient CPU/memory:
    - It requests the cloud provider to **add nodes**.
3. It also monitors **underutilized nodes**:
    - If a node is mostly empty and pods can move elsewhere, it **removes the node**.
4. Works only with **cloud-managed clusters** (AWS, GCP, Azure) or environments supporting dynamic node provisioning.

**Key features:**

- **Scale up:** Automatically adds nodes when pods cannot be scheduled.
- **Scale down:** Removes nodes when they are underutilized.
- **Works with multiple node groups:** Can scale different types of nodes based on pod requirements.
- **Supports taints and tolerations:** Ensures workloads are scheduled correctly.

**Example scenario:**

- Cluster has 3 nodes, all fully used.
- CI/CD pipeline launches a heavy job, pods remain pending.
- Cluster Autoscaler adds **one more node**.
- Once the job finishes and the node is underutilized, it is **removed** to save cost.

---

## 43. How does Kubernetes handle self-healing?

Kubernetes provides **self-healing** capabilities to ensure applications remain **available and resilient**, even when pods or nodes fail. This is one of its core features for **reliable container orchestration**.

**How it works:**

1. **Pod health monitoring**
    - Kubernetes uses **readiness** and **liveness probes** to check if a pod is healthy.
    - **Liveness probe failure** → pod is **killed and restarted** automatically.
    - **Readiness probe failure** → pod is **temporarily removed** from Service endpoints until it becomes healthy.
2. **Replica management**
    - Deployments, StatefulSets, and ReplicaSets ensure the **desired number of replicas** are always running.
    - If a pod crashes, Kubernetes automatically **creates a replacement pod**.
3. **Node failure recovery**
    - kubelet continuously reports node health to the **control plane**.
    - If a node fails, pods running on it are marked unschedulable.
    - Kubernetes **reschedules pods** onto healthy nodes in the cluster.
4. **Crash looping detection**
    - Kubernetes can **restart crashing pods repeatedly**.
    - Uses **backoff policies** to prevent rapid restarts from overwhelming the cluster.

**Example scenario:**

- A backend pod crashes due to memory exhaustion.
- **Deployment detects only 2 replicas running**, desired replicas = 3.
- Kubernetes automatically **creates a new pod** to maintain availability.
- Users experience **no downtime** as the Service routes traffic to healthy pods.

---

## 44. What happens when a pod crashes?

When a **pod crashes** in Kubernetes, the system automatically tries to **recover it** based on the pod's **controller and restart policy**. Kubernetes is designed to **ensure application availability** even in the event of failures.

**Step-by-step behavior:**

1. **Crash detection**
    - kubelet monitors pod containers.
    - Liveness probes detect **unhealthy containers**.
    - kubelet reports the pod as **failed** if it stops unexpectedly.
2. **Restart policy**
    - Each pod has a **restartPolicy** (`Always`, `OnFailure`, `Never`).
    - Common default in Deployments: `Always`.
    - Kubernetes restarts the container according to the policy.
3. **Replica management**
    - If the pod is part of a **Deployment, ReplicaSet, StatefulSet**, Kubernetes ensures the **desired number of replicas** is maintained.
    - If a pod crashes, a **new pod is automatically scheduled** on a healthy node.
4. **Service continuity**
    - Pods failing readiness checks are removed from **Service endpoints**.
    - Traffic is routed to **healthy pods**, so users usually do **not notice downtime**.

---

## 45. What happens when a node goes down?

When a **node goes down** in Kubernetes, the system automatically detects the failure and **reschedules workloads** to maintain application availability. This is part of Kubernetes' **self-healing and high-availability mechanism**.

**Step-by-step behavior:**

1. **Node monitoring**
    - kubelet on each node sends **heartbeats** to the **API server**.
    - The control plane checks the node status periodically.
    - If heartbeats stop, the node is marked **NotReady**.
2. **Pod eviction**
    - Pods running on the failed node are **unschedulable**.
    - The **node controller** waits for a configurable period (default 5 minutes) before evicting pods.
3. **Rescheduling pods**
    - Controller managers (ReplicaSet, Deployment, StatefulSet) detect missing replicas.
    - New pods are **scheduled on healthy nodes** automatically.
    - This ensures the **desired replica count** is maintained.
4. **Service continuity**
    - Services stop routing traffic to pods on the downed node.
    - Traffic is routed to **healthy pods** on other nodes.
5. **Optional node recovery**
    - When the node comes back online, pods may be **rescheduled back**, depending on the scheduler rules.

**Example scenario:**

- Node1 runs 3 pods of a backend Deployment (replicas: 6).
- Node1 crashes. API server marks Node1 as NotReady.
- Controller notices only 3 replicas are running on other nodes.
- Kubernetes **creates 3 new pods** on healthy nodes.
- Traffic continues via Service → no downtime for users.

---

🔹 **Monitoring & Troubleshooting**

## 46. How do you troubleshoot a failing pod?

Troubleshooting a **failing pod** in Kubernetes involves **checking the pod status, logs, events, and configuration** to identify the root cause. Here's a structured interview-level approach:

### 1. Check pod status

```
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

- **Fields to check:**
    - `STATUS` → `CrashLoopBackOff`, `Pending`, `Error`
    - `Events` → shows reasons for failures (e.g., `ImagePullBackOff`, `FailedScheduling`)
    - `ContainerStatuses` → last state, restart count, exit code

**Example:**

- `CrashLoopBackOff` → container is repeatedly crashing
- `ImagePullBackOff` → Kubernetes cannot pull the image

### 2. Check pod logs

```
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -c <container-name> -n <namespace>
```

- **Look for errors** in application startup or runtime
- If logs are empty, it may be a **startup crash**, missing configuration, or failing liveness probe

### 3. Check container exit code

- `kubectl describe pod` shows **container exit codes**
- Common codes:
    - `0` → normal exit
    - `1` → application error
    - `137` → out-of-memory (OOMKilled)

### 4. Check events and scheduling issues

```
kubectl get events -n <namespace> --sort-by='.metadata.creationTimestamp'
```

- Look for:
    - `FailedScheduling` → no nodes have enough resources
    - `ErrImagePull` → image not found or authentication issue
    - `BackOff` → pod keeps failing

### 5. Verify configuration

- **Check ConfigMaps and Secrets** mounted in the pod
- Ensure **environment variables** are correct
- Validate **resource requests/limits** (CPU/Memory)

### 6. Advanced debugging

- **Execute into a pod** for live troubleshooting:

```
kubectl exec -it <pod-name> -- /bin/sh
```

- **Run a debug pod** on the same node with hostPath or network access

### 7. Health probes

- Check **liveness/readiness probes**:
    - Misconfigured probes can cause **CrashLoopBackOff**
- Adjust **timeout, period, and initialDelay** if needed

---

## 47. How do you check pod logs?

In Kubernetes, checking pod logs is **one of the first steps** to troubleshoot applications. Logs show **what the container is doing or why it's failing**.

### 1. Check logs of a single container pod

```
kubectl logs <pod-name> -n <namespace>
```

- **`-n <namespace>`** is optional if in the default namespace.
- Shows **stdout/stderr** of the container.

### 2. Check logs of a specific container in a pod

```
kubectl logs <pod-name> -c <container-name> -n <namespace>
```

- Useful when a pod has **multiple containers**, e.g., **sidecars**.

### 3. Check logs of previous container instance

```
kubectl logs -p <pod-name> -n <namespace>
```

- Shows logs from the **last terminated container**
- Helps debug **CrashLoopBackOff** or container crashes.

### 4. Stream live logs (like `tail -f`)

```
kubectl logs -f <pod-name> -n <namespace>
kubectl logs -f <pod-name> -c <container-name> -n <namespace>
```

- Useful to **watch logs in real-time** during debugging or deployment.

### 5. View logs across all pods in a Deployment

```
kubectl logs -l app=<label> -n <namespace>
```

- Combines logs of multiple pods with the same label
- Helpful for **replicated workloads**.

**Best practices:**

- Use **namespace and container name** to avoid ambiguity
- For persistent logging in production, integrate with **EFK stack (Elasticsearch, Fluentd, Kibana)** or **Prometheus + Grafana**
- Avoid relying solely on `kubectl logs` for long-term storage

---

## 48. What is `kubectl describe` used for?

`kubectl describe` is a **diagnostic command** in Kubernetes that gives **detailed information about a resource**, including its **status, configuration, events, and recent errors**. It's often used in **troubleshooting pods, nodes, deployments, and services**.

**Purpose of `kubectl describe`:**

- Inspect the **current state** of a resource.
- View **events** generated by the Kubernetes control plane.
- Understand **why a resource is failing or pending**.

---

## 49. How do you debug CrashLoopBackOff?

A **CrashLoopBackOff** happens when a container in a pod **keeps crashing repeatedly**, and Kubernetes backs off restarting it. Debugging it requires a **structured approach** to find the root cause.

### 1. Check pod status

```
kubectl get pods -n <namespace>
kubectl describe pod <pod-name> -n <namespace>
```

- Look for:
    - `State: Waiting` and `Reason: CrashLoopBackOff`
    - `Last State` → exit code and message
    - Events → image pull errors, scheduling issues

**Tip:** The exit code gives clues:

- `0` → normal exit
- `1` → app error
- `137` → OOMKilled (out-of-memory)
- `139` → segmentation fault

### 2. Check logs

```
kubectl logs <pod-name> -n <namespace>
kubectl logs -p <pod-name> -n <namespace>
```

- `-p` shows logs from the **previous container instance**
- Look for:
    - Application crashes
    - Missing environment variables
    - Configuration errors

### 3. Check probes

- **Liveness/Readiness probes** misconfiguration can cause CrashLoopBackOff.

```
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

- Increase `initialDelaySeconds` if the app takes time to start.

### 4. Check resources

- Pod may crash due to **CPU/memory limits**:

```
kubectl describe pod <pod-name>
```

- Look for `OOMKilled` or `CPU throttling`.
- Adjust **requests and limits** in the pod spec.

### 5. Run a debug pod

- Launch an interactive pod on the same node or with the same image:

```
kubectl run -it --rm debug --image=my-app:latest -- /bin/sh
```

- Test commands and check for missing files, configs, or permissions.

### 6. Common causes

1. Application crashes due to code bug
2. Missing or misconfigured environment variables
3. Container runs as root, violates PSA/PSP policies
4. Misconfigured probes
5. Insufficient CPU/memory
6. Image pull errors

---

## 50. What are liveness and readiness probes?

**Liveness and readiness probes** in Kubernetes are **health checks for containers** that help ensure pods are running properly and serving traffic. They are **key to Kubernetes self-healing and load balancing**.

### 1. Liveness Probe

- **Purpose:** Detects if a container is **alive or stuck**.
- **Action:** If the probe fails, Kubernetes **restarts the container**.
- **Use case:** Fix containers that hang, deadlock, or crash silently.

**Example:**

```
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

- Checks `/healthz` endpoint every 10 seconds after an initial delay of 5 seconds.
- If failing repeatedly, kubelet restarts the container.

### 2. Readiness Probe

- **Purpose:** Detects if a container is **ready to serve traffic**.
- **Action:** If the probe fails, Kubernetes **removes the pod from Service endpoints**.
- **Use case:** Prevent traffic being sent to a container that is **starting up or temporarily unhealthy**.

**Example:**

```
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

- Pod is **excluded from the load balancer** if `/ready` fails.

---

## 51. How do you monitor Kubernetes?

Monitoring Kubernetes is crucial for **observability, reliability, and troubleshooting** in production clusters. It involves tracking **cluster health, node/pod metrics, resource usage, and events**.

### 1. Metrics Monitoring

- **Tools:** Prometheus + Grafana (most common)
- **What to monitor:**
    - CPU, memory, disk usage of nodes and pods
    - Pod counts and replica status
    - Node availability and health
- Prometheus scrapes metrics from:
    - **kubelet** → node/pod metrics
    - **cAdvisor** → container-level metrics
    - **API server, scheduler, controller-manager** → control plane metrics

**Example:** Grafana dashboards visualize cluster utilization and alerts.

### 2. Log Monitoring

- **Tools:** EFK stack (Elasticsearch, Fluentd, Kibana) or Loki
- **What to monitor:**
    - Application logs from pods
    - Kubernetes system logs (`kube-apiserver`, `kubelet`, etc.)
    - Audit logs for security

**Benefit:** Quickly debug pod failures, deployments, and network issues.

### 3. Events Monitoring

- Kubernetes emits **events** for resource changes:

```
kubectl get events -n <namespace> --sort-by='.metadata.creationTimestamp'
```

- Example events: `PodFailed`, `FailedScheduling`, `BackOff`
- Useful for **real-time alerting** in CI/CD or operations.

### 4. Health Probes

- **Liveness and readiness probes** are a form of **automatic monitoring**
- Alerts operators if containers are unhealthy or not ready.

### 5. Cluster Monitoring Services

- Cloud providers offer managed monitoring:
    - **GKE:** Stackdriver / Cloud Monitoring
    - **EKS:** CloudWatch Container Insights
    - **AKS:** Azure Monitor
- These integrate metrics, logs, and events in one console.

### 6. Alerting

- Set up **alerts for CPU/memory thresholds, pod failures, or node down events**.
- Tools like **Prometheus Alertmanager** can notify via Slack, email, or PagerDuty.

---

## 52. What metrics does Kubernetes expose?

Kubernetes exposes a wide range of metrics about nodes, pods, containers, and the control plane. These metrics are essential for monitoring, scaling, and troubleshooting. Most of these are collected via kubelet, cAdvisor, and the API server, often scraped by Prometheus.

### 1. Node Metrics

- CPU usage → current, request, and limit utilization
- Memory usage → total, used, available
- Disk usage → ephemeral storage used
- Network usage → packets sent/received

**Example:**

- `node_cpu_seconds_total` → CPU usage per core
- `node_memory_MemAvailable_bytes` → available memory

### 2. Pod & Container Metrics

- CPU and memory usage for each pod/container
- Restart count → container crash frequency
- Filesystem usage → disk consumed by containers
- Network usage → bytes in/out per container

**Example:**

- `container_cpu_usage_seconds_total` → cumulative CPU time
- `container_memory_working_set_bytes` → memory used by a container

### 3. Deployment & ReplicaSet Metrics

- Replicas desired vs. actual → helps detect missing pods
- Available replicas → shows healthy pods
- Unavailable replicas → indicates pending or crashing pods

**Example:**

- `kube_deployment_status_replicas_available`
- `kube_replicaset_status_ready_replicas`

### 4. Control Plane Metrics

- API server metrics:
    - Request rates (`apiserver_request_total`)
    - Latency (`apiserver_request_duration_seconds`)
    - Error counts (`apiserver_request_total{code="500"}`)
- Scheduler metrics → scheduling latency, queue length
- Controller manager metrics → reconciliation loops, errors

### 5. Cluster Autoscaler Metrics

- Number of nodes added/removed
- Pending pods that triggered scaling
- Node utilization statistics

### 6. Custom Application Metrics

- Kubernetes supports custom metrics for Horizontal Pod Autoscaler (HPA) using:
    - Prometheus Adapter
    - Custom metrics API
- Example: scale a pod based on queue length in RabbitMQ or request rate per second.

---

🔹 **CI/CD & DevOps Integration**

## 53. How do you use Jenkins with Kubernetes?

**Using** Jenkins with Kubernetes **is a common DevOps pattern to** dynamically provision build agents (pods)**, scale pipelines, and deploy applications. Kubernetes acts as the** executor environment for Jenkins jobs.

### 1. Jenkins-Kubernetes integration

- **Jenkins master** runs the UI, scheduling, and orchestration.
- **Jenkins agents (slaves)** run inside Kubernetes pods.
- The **Kubernetes plugin** allows Jenkins to:
    - Launch pods dynamically per job
    - Reuse pods for different builds
    - Automatically clean up after builds

### 2. Jenkins Kubernetes Plugin

- Install via Jenkins → Manage Jenkins → Manage Plugins → **Kubernetes**.
- Configuration:
    - **Kubernetes cluster URL** (in-cluster or external)
    - **Credentials** (ServiceAccount token)
    - **Pod template** defining container images and volumes for agents

### 3. Dynamic agent pods

**Example Pod Template:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins-agent: "true"
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:latest
    args: ["$(JENKINS_SECRET)", "$(JENKINS_NAME)"]
  - name: maven
    image: maven:3.8.5-jdk11
    command: ["cat"]
    tty: true
```

- `jnlp` container → connects back to Jenkins master
- Additional containers → build tools like Maven, Node, Docker

### 4. Using a Jenkins Pipeline with Kubernetes agents

```groovy
pipeline {
    agent {
        kubernetes {
            label 'k8s-agent'
            yaml """
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: maven
                image: maven:3.8.5-jdk11
                command:
                - cat
                tty: true
            """
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
            }
        }
    }
}
```

- `agent kubernetes {}` → dynamically provisions a pod per build
- `container('maven') {}` → runs steps inside the specified container

### 5. Benefits of Jenkins + Kubernetes

1. **Dynamic scaling:** No permanent agents needed
2. **Isolation:** Each job runs in a clean pod
3. **Consistency:** Use the same container image across builds
4. **Resource efficiency:** Pods are destroyed after jobs complete
5. **Integration with CI/CD:** Build, test, and deploy to Kubernetes clusters

---

## 54. What is Helm?

**Helm** is a **package manager for Kubernetes**, similar to `apt` or `yum` for Linux, that simplifies **deploying, managing, and versioning applications** on a Kubernetes cluster.

**Why Helm is used:**

- Kubernetes manifests (YAML) can get **complex** for multi-component applications.
- Helm **packages resources into charts**, making deployment **repeatable, consistent, and manageable**.
- Supports **versioning, templating, and easy upgrades/rollbacks**.

**Key concepts:**

1. **Chart**
    - A **Helm package** containing:
        - Kubernetes manifests (`Deployment`, `Service`, etc.)
        - Default configuration values
        - Optional templates for dynamic values
    - Example: `nginx` chart deploys an Nginx deployment + service.
2. **Release**
    - An **instance of a chart deployed to the cluster**
    - Can have its own configuration values and can be **upgraded or rolled back**.
3. **Values**
    - Default variables for a chart
    - Can be overridden at install/upgrade time using a **`values.yaml`** file.

**Basic Helm commands:**

```bash
# Add a chart repository
helm repo add stable https://charts.helm.sh/stable

# Update repo index
helm repo update

# Install a chart
helm install my-release stable/nginx

# Upgrade a release
helm upgrade my-release stable/nginx --values custom-values.yaml

# Rollback a release
helm rollback my-release 1

# List releases
helm list

# Uninstall a release
helm uninstall my-release
```

**Example scenario:**

- Deploy a microservices app:
    - Each service (frontend, backend, database) packaged as a Helm chart
    - Use `values.yaml` to configure environment-specific variables
    - Deploy to dev/staging/prod with one command per environment
    - Upgrade or rollback easily without manually editing YAML

---

## 55. Why use Helm over kubectl?

Using **Helm over plain `kubectl`** offers several advantages for **deploying and managing applications** on Kubernetes, especially as complexity grows. Here's a detailed explanation:

### 1. Templating and Parameterization

- **Helm charts** allow dynamic templates in manifests (`Deployment`, `Service`, etc.).
- Use **`values.yaml`** or CLI overrides to customize deployments per environment (dev, staging, prod).

**Example:**

```yaml
replicas:  .Values.replicas 
image:  .Values.image.repository : .Values.image.tag 
```

- With `kubectl`, you'd need separate YAML files or manual edits for each environment.

### 2. Versioning

- Helm **tracks releases** with versions.
- Enables **rollback** to a previous version if an upgrade fails:

```bash
helm rollback my-app 2
```

- With `kubectl`, rollbacks require manual deletion and re-apply of old manifests, which is error-prone.

### 3. Dependency Management

- Helm charts can **depend on other charts** (e.g., a web app depends on a database).
- Helm automatically installs/upgrades dependencies.
- With `kubectl`, you'd manually manage each component.

### 4. Reproducibility

- Helm charts package **all resources needed** for an app (Deployment, Service, ConfigMaps, Secrets).
- This ensures **consistent deployments** across clusters and teams.

### 5. Easier Upgrades

- Helm **computes differences** between chart versions and applies only necessary changes.
- `kubectl apply` can be used with YAML, but managing multi-component apps becomes cumbersome.

### 6. Release Management

- Helm keeps a **history of all releases**, including failed deployments.
- Provides **auditability and traceability**.

### 7. Community Charts

- There's a rich ecosystem of **prebuilt charts** (nginx, MySQL, Prometheus).
- Saves time compared to writing raw YAML manifests.

---

## 56. What is a Helm chart?

A **Helm chart** is a **package of pre-configured Kubernetes resources** that defines an application, including its deployments, services, configuration, and dependencies. Think of it as a **"containerized app recipe"** for Kubernetes.

**Key points about Helm charts:**

1. **Chart = Package**
    - Similar to a `.deb` or `.rpm` package in Linux.
    - Contains **all the Kubernetes manifests** required to run an app.
2. **Templated Manifests**
    - Charts use **Go templates** to make manifests configurable.
    - Values can be set using `values.yaml` or overridden at install/upgrade time.
3. **Reusable**
    - Same chart can be deployed multiple times with **different configurations** (dev, staging, prod).
4. **Versioned**
    - Charts have **version numbers**, enabling upgrades and rollbacks.

---

## 57. What is GitOps?

**GitOps** is a modern DevOps practice where **Git is the single source of truth for declarative infrastructure and application deployment**. Changes to the system are **driven automatically by updates to Git repositories**.

**Key Concepts:**

1. **Declarative Infrastructure**
    - Kubernetes manifests, Helm charts, or Terraform code describe the **desired state** of the system.
    - The Git repository stores this **declarative configuration**.
2. **Automatic Reconciliation**
    - A GitOps operator (like **Argo CD** or **Flux**) continuously monitors the Git repo and the live cluster.
    - If the cluster drifts from the Git-defined state, the operator **automatically applies changes** to match Git.
3. **Pull-based Deployment**
    - GitOps uses a **pull model**, where the cluster pulls changes from Git rather than CI/CD pushing changes.
    - Improves **security** because the cluster controls what is applied.

**Benefits of GitOps:**

| **Benefit** | **Description** |
| --- | --- |
| Version control | Every change is in Git, with history and rollback capabilities |
| Auditability | You can see who changed what and when |
| Consistency | Cluster state always matches Git |
| Self-healing | Operators detect drift and restore the desired state |
| Improved CI/CD | Git triggers deployments automatically via reconciliation |

**How GitOps works (example flow):**

1. Developer pushes a commit with updated manifests or Helm values to Git.
2. GitOps operator detects the change.
3. Operator applies the manifests to the Kubernetes cluster.
4. Cluster state converges with the Git-defined desired state.
5. Rollbacks can be done by reverting the commit in Git.

---

## 58. How does ArgoCD work?

**Argo CD** is a **GitOps continuous delivery tool for Kubernetes**. It continuously monitors a Git repository for declarative Kubernetes manifests or Helm charts and ensures the **cluster state matches what's defined in Git**.

**Workflow Example:**

1. Developer pushes updated Helm values or manifests to Git.
2. Argo CD detects the commit in the repository.
3. Application controller calculates the **difference** between live cluster and Git.
4. If automatic sync is enabled, it applies changes to the cluster; otherwise, it waits for manual approval.
5. Cluster state now matches Git. Any drift in the future is automatically corrected.

---

## 59. Difference between Jenkins and ArgoCD?

### 1. Purpose

| **Feature** | **Jenkins** | **Argo CD** |
| --- | --- | --- |
| Primary function | CI/CD automation server | GitOps continuous delivery for Kubernetes |
| Focus | Build, test, package, deploy | Continuous deployment and cluster reconciliation |
| Workflow style | Push-based (Jenkins pushes changes) | Pull-based (cluster pulls desired state from Git) |

### 2. Approach

**Jenkins:**

- Declarative or scripted pipelines
- Can build, test, and deploy applications to any environment
- Triggers via commits, webhooks, or schedules
- Imperative style: Jenkins executes commands in sequence

**Argo CD:**

- Declarative GitOps approach
- Only deploys applications to Kubernetes
- Continuously monitors Git for the desired state and reconciles the cluster
- Pull-based: cluster automatically updates itself from Git
