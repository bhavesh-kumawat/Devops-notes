**🔹 Scenario-Based Questions**

## 60. How do you design Kubernetes for production?

Designing Kubernetes for production requires careful planning to ensure **high availability, scalability, security, and observability**. Here's a structured approach based on best practices:

### 1. Cluster Architecture

**Control Plane:**

- Use **high availability (HA)**: at least 3 master nodes.
- Deploy across **multiple availability zones** to survive failures.
- Components:
    - **kube-apiserver** → exposed via load balancer
    - **etcd** → distributed key-value store, backed up regularly
    - **kube-scheduler, controller-manager** → HA configuration

**Worker Nodes:**

- Separate compute nodes for workloads.
- Use **node pools** for different workloads (e.g., GPU nodes, general compute).
- **Label nodes** for workload-specific scheduling.

### 2. Networking

- Use **CNI plugins** like Calico, Cilium, or Flannel for pod networking.
- Ensure **network policies** for pod-to-pod traffic restrictions.
- Use **Ingress controllers** (NGINX, Traefik) for external traffic.
- Configure **service meshes** (Istio, Linkerd) if required for observability and security.

### 3. Storage

- Use **persistent volumes** for stateful applications.
- Use **StorageClasses** to manage dynamic provisioning.
- Separate fast storage for databases, slower for logs or backups.

### 4. Security

- Enable **RBAC** to control access.
- Use **Pod Security Admission (PSA)** or Pod Security Policies (deprecated in v1.25).
- **Secrets management**: store secrets in Kubernetes Secrets, optionally integrate with Vault.
- Limit container privileges (`runAsNonRoot`, read-only root filesystem).
- Enable **network policies** to restrict traffic.

### 5. High Availability and Reliability

- Use **Deployments, StatefulSets, ReplicaSets** to ensure desired replicas.
- Use **Horizontal Pod Autoscaler (HPA)** for scaling workloads automatically.
- Use **Cluster Autoscaler** to scale nodes dynamically.
- Configure **liveness and readiness probes** for self-healing.

### 6. Observability

- **Metrics:** Prometheus + Grafana
- **Logs:** EFK (Elasticsearch, Fluentd, Kibana) or Loki
- **Events:** Monitor `kubectl get events` or use Alertmanager
- **Tracing:** Jaeger or OpenTelemetry for microservices

### 7. CI/CD Integration

- Use **GitOps tools** (Argo CD, Flux) for deployment.
- Optionally integrate Jenkins or other CI tools for building artifacts.
- **Helm charts** or **Kustomize templates** for repeatable deployments.

### 8. Backup and Disaster Recovery

- Backup **etcd** regularly.
- Backup **persistent volumes** (Velero or cloud-native snapshots).
- Plan for **multi-region recovery** if needed.

### 9. Multi-Tenancy & Namespaces

- Use **namespaces** to isolate workloads.
- Apply **ResourceQuotas** and **LimitRanges** per namespace.
- Combine with **RBAC** for team isolation.

### 10. Monitoring & Alerting

- Alerts for:
    - Node/Pod failures
    - High CPU/memory usage
    - Pod crash loops
- Integrate with **Slack, PagerDuty, or email** for operational awareness.

---

## 61. How do you perform blue-green or canary deployments in Kubernetes?

Performing **blue-green** or **canary deployments** in Kubernetes ensures **zero-downtime releases** and allows you to **safely test new versions** of applications. Here's a detailed explanation with approaches and examples:

### 1. Blue-Green Deployment

**Concept:**

- Two identical environments: **Blue (current)** and **Green (new version)**.
- Switch traffic from Blue to Green after verification.

**Steps:**

1. Deploy the **new version** in a separate deployment or namespace (Green).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
      version: green
  template:
    metadata:
      labels:
        app: my-app
        version: green
    spec:
      containers:
      - name: my-app
        image: my-app:2.0
        ports:
        - containerPort: 8080
```

1. Configure **Service selector** to point to the Green version after testing:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
    version: green
  ports:
  - port: 80
    targetPort: 8080
```

1. Once verified, **decommission Blue deployment** or keep it as a backup.

**Pros:** Simple, clean cutover

**Cons:** Requires double resources during deployment

### 2. Canary Deployment

**Concept:**

- Gradually route a small portion of traffic to the new version to **test in production**.
- Gradually increase traffic as confidence grows.

**Steps:**

1. Deploy **canary pods** with a new version:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: canary
  template:
    metadata:
      labels:
        app: my-app
        version: canary
    spec:
      containers:
      - name: my-app
        image: my-app:2.0
        ports:
        - containerPort: 8080
```

1. Use a **Service or Ingress** to split traffic between stable and canary:
    - **Option 1: Service selectors**
        - Weighted traffic via custom controllers or service meshes
    - **Option 2: Istio/Linkerd**
        - Traffic routing rules, e.g., 10% to canary, 90% to stable
2. Monitor **metrics and logs**:
    - CPU/memory
    - Error rates
    - User experience
3. Gradually increase traffic to canary, then **promote to stable**:

```bash
helm upgrade my-app ./helm --set canaryWeight=50
```

### 3. CI/CD Pipeline Integration

**Jenkins + Helm example (Canary):**

```groovy
stage('Deploy Canary') {
    steps {
        sh 'helm upgrade --install my-app ./helm --set canary=true --set canaryWeight=10'
    }
}
stage('Monitor Canary') {
    steps {
        // Integrate monitoring checks or automated tests
        sh './monitor_canary.sh'
    }
}
stage('Promote Canary') {
    steps {
        sh 'helm upgrade my-app ./helm --set canary=false'
    }
}
```

---

## 62. How do you upgrade a Kubernetes cluster?

Upgrading a Kubernetes cluster in **production** requires careful planning to avoid downtime and ensure compatibility. The process depends on whether you are using a **managed service** (like GKE, EKS, AKS) or **self-managed cluster** (kubeadm, kops, or bare-metal). Here's a structured guide:

### 1. Pre-Upgrade Planning

1. **Check current cluster version and components:**

```bash
kubectl version
kubectl get nodes
kubectl get pods --all-namespaces
```

- Ensure all nodes are **Ready**
- Make sure workloads are healthy
1. **Review Kubernetes release notes:**
- Check **deprecated APIs** or breaking changes
- Verify **CNI plugin compatibility** (Calico, Flannel, etc.)
- Check **storage driver** support
1. **Backup cluster state:**
- Backup **etcd** (critical for control plane data)

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

- Backup persistent volumes if needed

### 2. Upgrade Control Plane

**Managed Kubernetes:**

- Upgrade using cloud provider console/CLI:
    - **GKE:** `gcloud container clusters upgrade`
    - **EKS:** `eksctl upgrade cluster`
    - **AKS:** `az aks upgrade`
- Control plane is upgraded first; worker nodes can remain on the old version temporarily.

**Self-Managed (kubeadm):**

```bash
sudo apt-get update && sudo apt-get upgrade kubeadm
sudo kubeadm upgrade plan
sudo kubeadm upgrade apply v1.x.y
```

- Upgrade **masters first**, one at a time for HA clusters

### 3. Upgrade Worker Nodes

- **Managed Kubernetes:** Cloud provider handles draining and upgrading nodes:

```bash
kubectl drain <node> --ignore-daemonsets
kubectl uncordon <node>
```

- **Self-managed:**
    1. Drain node (`kubectl drain <node>`)
    2. Upgrade kubelet and kubectl
    3. Restart kubelet
    4. Uncordon node
- Upgrade nodes **one at a time** to ensure high availability.

### 4. Upgrade Add-ons

- Upgrade **CNI plugin** (Calico, Cilium, Flannel) if required
- Upgrade **Ingress controllers** (NGINX, Traefik)
- Upgrade **metrics server**, **CoreDNS**, and other system components

```bash
kubectl get deployment -n kube-system
kubectl rollout status deployment coredns -n kube-system
```

### 5. Verify Cluster Post-Upgrade

- Check **node versions:**

```bash
kubectl get nodes
```

- Check **pods and deployments** are healthy:

```bash
kubectl get pods --all-namespaces
kubectl describe pod <pod-name>
```

- Run **smoke tests** on critical applications

### 6. Rollback Plan

- Keep **etcd snapshot** before upgrading
- If something fails:
    1. Restore etcd snapshot
    2. Revert node upgrades
- Cloud-managed services often provide **one-click rollback**

---

## 63. How do you back up etcd?

Backing up **etcd** is critical in Kubernetes because it stores **all cluster state**: resources, secrets, and configuration. A proper backup ensures you can **recover from cluster failures**. Here's a complete guide:

### 1. Check etcd Version

```bash
etcdctl version
```

- Backup commands vary slightly depending on **etcd version**.
- Ensure **etcdctl** is installed on the master node.

### 2. Set Environment Variables

- If Kubernetes uses **TLS for etcd**, set the following:

```bash
export ETCDCTL_API=3
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/etcd/server.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/etcd/server.key
```

### 3. Create a Snapshot (Backup)

```bash
etcdctl snapshot save /path/to/backup/etcd-snapshot.db
```

- Example:

```bash
etcdctl snapshot save /var/backups/etcd-snapshot-$(date +%F).db
```

- This creates a **full backup of the etcd database**.

### 4. Verify the Snapshot

```bash
etcdctl snapshot status /path/to/backup/etcd-snapshot.db
```

- Checks **size, revision, and consistency** of the snapshot.

### 5. Restore from Snapshot (if needed)

1. Stop the etcd service:

```bash
systemctl stop etcd
```

1. Restore the snapshot:

```bash
etcdctl snapshot restore /path/to/backup/etcd-snapshot.db \
  --data-dir /var/lib/etcd-restored
```

1. Update **etcd systemd service** to point to the restored data directory.
2. Start etcd again:

```bash
systemctl start etcd
```

1. Verify cluster health:

```bash
etcdctl member list
kubectl get nodes
```

### 6. Best Practices for etcd Backups

1. **Automate backups** using cronjobs or scripts.
2. **Store backups off-cluster** (S3, GCS, or NFS).
3. **Encrypt backups** if they contain secrets.
4. **Test restores regularly** to ensure disaster recovery works.
5. **Keep multiple backup versions** to handle corruption or accidental deletion.

---

## 64. How do you handle multi-tenant Kubernetes clusters?

Handling **multi-tenant Kubernetes clusters** means running workloads for multiple teams, applications, or customers on the **same cluster** while ensuring **isolation, security, and fair resource usage**. Here's a structured approach:

### 1. Namespace-Based Isolation

- Use **namespaces** to logically separate tenants:

```bash
kubectl create namespace team-a
kubectl create namespace team-b
```

- Each team gets a dedicated namespace for:
    - Deployments
    - Services
    - ConfigMaps and Secrets

### 2. Role-Based Access Control (RBAC)

- Limit access per namespace using **Roles** and **RoleBindings**:

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: team-a
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps"]
  verbs: ["get", "list", "create", "update", "delete"]
```

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: bind-devs
  namespace: team-a
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer-role
  apiGroup: rbac.authorization.k8s.io
```

- Ensures **teams cannot access each other's resources**.

### 3. Resource Quotas & Limits

- Prevent one tenant from consuming all resources:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    cpu: "10"
    memory: 20Gi
    pods: "50"
```

- Use **LimitRanges** to enforce **per-pod limits**:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-mem-limit
  namespace: team-a
spec:
  limits:
  - type: Container
    default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 250m
      memory: 256Mi
```

### 4. Network Isolation

- Use **NetworkPolicies** to control traffic between tenants:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-other-namespaces
  namespace: team-a
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
      namespaceSelector:
        matchLabels:
          name: team-a
```

- Ensures **pods in one namespace cannot talk to pods in another**.

### 5. Pod Security and Admission

- Enforce **Pod Security Standards** per tenant:
    - `runAsNonRoot: true`
    - `readOnlyRootFilesystem: true`
    - Drop unnecessary capabilities (`capabilities.drop`)
- Use **PodSecurityAdmission (PSA)** for namespace-level enforcement.

### 6. Secrets Management

- Store **tenant secrets** separately using:
    - Namespace-scoped **Kubernetes Secrets**
    - External tools (HashiCorp Vault, AWS Secrets Manager)
- Avoid sharing secrets across tenants.

### 7. Monitoring and Billing

- Tag namespaces or labels for **tenant-specific monitoring**:
    - Prometheus labels: `namespace="team-a"`
- Track **resource usage** for fair allocation and chargeback if needed.

### 8. CI/CD Considerations

- Use **namespace-scoped pipelines** in Jenkins, Argo CD, or Flux.
- Each team can deploy independently without affecting others.

---

## 65. What causes pods to stay in Pending state?

A Kubernetes pod stays in the **`Pending` state** when it has been accepted by the API server but **cannot be scheduled to a node**. There are several common causes, usually related to **resources, scheduling, or cluster configuration**.

Here's a detailed breakdown:

### 1. Insufficient Resources

- The node(s) do not have enough **CPU, memory, or ephemeral storage** to run the pod.

```bash
kubectl describe pod <pod-name>
```

- Look for events like:

```
0/3 nodes are available: 3 Insufficient cpu.
```

- **Fix:**
    - Add more nodes or increase node resources.
    - Reduce pod resource requests (`requests` in the container spec).

### 2. Node Selectors, Taints, and Tolerations

- **NodeSelector / NodeAffinity** constraints prevent scheduling if no node matches:

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

- **Taints** prevent pods from being scheduled unless they have **matching tolerations**:

```bash
kubectl describe node <node-name> | grep Taints
```

- **Fix:**
    - Add tolerations to the pod.
    - Remove or adjust node taints if appropriate.

### 3. PersistentVolume Claims (PVCs) Not Bound

- If the pod uses a **PVC** that is **unbound**, the pod cannot start.

```bash
kubectl get pvc
```

- **Fix:**
    - Ensure the **StorageClass** exists and dynamic provisioning works.
    - Manually bind PV to PVC if needed.

### 4. Pod Limits Exceed Node Capacity

- Pod `requests` exceed any node's available capacity:

```yaml
resources:
  requests:
    cpu: 2
    memory: 8Gi
```

- **Fix:**
    - Adjust resource requests/limits.
    - Scale the cluster to add larger nodes.

### 5. Scheduler Constraints

- **Affinity/Anti-affinity** rules may prevent pod placement:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - my-app
      topologyKey: "kubernetes.io/hostname"
```

- If no node satisfies these rules, pod stays `Pending`.

### 6. Node Conditions

- Node may be **NotReady, Unschedulable, or Cordoned**:

```bash
kubectl get nodes
kubectl describe node <node-name>
```

- **Fix:**
    - Uncordon node: `kubectl uncordon <node-name>`
    - Ensure node is healthy and network/storage is operational.

### 7. Image Pull Failures (sometimes misreported as Pending)

- Kubernetes tries to schedule pods but fails to pull the image due to authentication or non-existent image.
- Usually shows up as `ContainerCreating` with `ErrImagePull`.

**Debugging Pending Pods:**

```bash
kubectl describe pod <pod-name>
kubectl get events --sort-by='.lastTimestamp'
kubectl get nodes
kubectl get pvc
```

- Look for **events** and **unschedulable reasons**.

---

## 66. How do you reduce Kubernetes costs?

Reducing **Kubernetes costs** in production involves **optimizing resource usage, scaling efficiently, and leveraging cloud-native pricing features**. Here's a structured approach:

### 1. Right-size Resources

- Set **CPU and memory requests and limits** for all pods:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

- Avoid over-provisioning resources; unused requests consume node capacity and drive costs.
- Use tools like **Goldilocks** (checks recommended requests/limits) or **Kubernetes Metrics Server + VPA** (Vertical Pod Autoscaler) to optimize.

### 2. Use Autoscaling

1. **Horizontal Pod Autoscaler (HPA)** – scale pods based on CPU, memory, or custom metrics:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

1. **Cluster Autoscaler** – scale nodes automatically to match pod requirements.
- Reduces costs by **removing idle nodes**.

### 3. Use Spot/Preemptible Instances

- Cloud providers offer **cheaper spot instances** for non-critical workloads:
    - GCP: Preemptible VMs
    - AWS: EC2 Spot Instances
    - Azure: Spot VMs
- Combine with **PodDisruptionBudgets (PDBs)** and node taints to handle sudden termination.

### 4. Optimize Storage Costs

- Use appropriate **StorageClasses**:
    - High-performance storage only where needed (databases)
    - Standard or cheaper volumes for logs or backups
- Use **dynamic volume provisioning** to avoid over-provisioning persistent volumes.

### 5. Consolidate Node Pools

- Avoid **many underutilized small nodes**.
- Combine workloads into **fewer nodes with higher utilization**:
    - Use **resource quotas** to control per-team usage
    - Schedule workloads with **bin packing** (affinity/anti-affinity)

### 6. Reduce Idle Resources

- Delete **unused namespaces, deployments, and jobs**.
- Use **CronJobs and ephemeral jobs** efficiently.
- Evict **unused services or LoadBalancer IPs** in cloud providers.

### 7. Optimize Container Images

- Use **smaller base images** to reduce startup time and resource usage.
- Multi-arch and Alpine-based images are cheaper and faster to pull.

### 8. Use Cost Monitoring Tools

- Native tools: **GCP Cloud Billing, AWS Cost Explorer, Azure Cost Management**
- Kubernetes-specific:
    - **Kubecost** → shows cost per namespace, deployment, and node
    - **Karpenter metrics** → optimize autoscaling decisions

### 9. Batch Processing and Scheduling

- Schedule **non-critical workloads** during off-peak hours.
- Use **Jobs and CronJobs** on spot or cheaper nodes.

### 10. Governance & Policies

- Enforce **resource quotas and limit ranges** per namespace.
- Prevent teams from requesting excessive CPU/memory.
- Automate **idle resource cleanup** using Kubernetes operators.

---

## 67. How do you check events in a namespace?

In Kubernetes, **events** provide insight into what's happening in your cluster, including pod scheduling, errors, or resource issues. You can check events for a **specific namespace** using the `kubectl` command.

### 1. Basic Command

```bash
kubectl get events -n <namespace>
```

**Example:**

```bash
kubectl get events -n dev
```

- Lists events in the `dev` namespace with columns like **TYPE, REASON, OBJECT, MESSAGE, AGE**.
- Default sorting may be by **timestamp of creation**, sometimes not chronological.

---

## 68. How do you roll back a deployment?

Rolling back a deployment in Kubernetes is straightforward if you are using a **Deployment resource**, because Kubernetes **keeps a history of revisions**. Here's a structured guide:

### 1. Check Deployment History

```bash
kubectl rollout history deployment/<deployment-name>
```

**Example:**

```bash
kubectl rollout history deployment/my-app
```

Output shows revisions:

```
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Updated image to v2.0
3         Updated replicas to 5
```

### 2. Roll Back to a Previous Revision

- To roll back to the **previous revision**:

```bash
kubectl rollout undo deployment/<deployment-name>
```

- To roll back to a **specific revision**:

```bash
kubectl rollout undo deployment/<deployment-name> --to-revision=2
```

- The deployment will revert to the **Pod template and configuration** of that revision.

### 3. Check Rollout Status

- After initiating rollback, monitor the deployment:

```bash
kubectl rollout status deployment/<deployment-name>
```

- This ensures the **new pods are running and ready**.

### 4. Verify Pods

- Check the current pods and their images:

```bash
kubectl get pods -l app=my-app
kubectl describe pod <pod-name>
```

- Confirm that the rollback **reverted the desired version**.

### 5. Rollback in Helm (if using Helm charts)

- Helm stores **release history**, so rollback is similar:

```bash
helm history my-app
helm rollback my-app 2
```

- Useful if your deployments are managed via **GitOps or CI/CD pipelines**.

### 6. Automated Rollback with CI/CD

- Many pipelines can automatically rollback on **failed health checks or metrics**.
- Example in Jenkins:

```groovy
post {
    failure {
        sh 'kubectl rollout undo deployment/my-app'
    }
}
```

- Ensures **self-healing deployments**.

**Common Causes for Rollback:**

- Canary deployment shows high error rates
- Pod crashes (`CrashLoopBackOff`)
- Performance degradation
- Failed health or readiness probes
