# 🔹 Jenkins Scenario-Based

## 1. How would you design a Jenkins CI/CD pipeline for a microservices app?

To design a **Jenkins CI/CD pipeline for a microservices application**, the goal is to make it **scalable, fast, and independent per service**, while still supporting end-to-end deployment.

### High-level design approach

**1. Pipeline per Microservice**

- Each microservice has its **own Git repo and Jenkins pipeline**.
- This allows **independent builds, tests, and deployments**.
- A failure in one service does **not block others**.

**Why:** Microservices must be deployed and scaled independently.

### 2. Typical Pipeline Stages

**Stage 1: Checkout**

- Pull code for the specific microservice from Git (triggered via webhook).

**Stage 2: Build**

- Compile or package the service (Maven/Gradle/npm).
- Run fast validations (lint, unit tests).

**Stage 3: Test**

- Unit tests
- Optional: contract tests (important for microservices).

**Stage 4: Docker Build & Push**

- Build a Docker image for the service.
- Tag it with **version + commit ID**.
- Push to a container registry (Docker Hub / ECR / GCR).

**Stage 5: Deploy**

- Deploy to **Kubernetes** using Helm or kubectl.
- Environment-based deployments:
    - `develop` → dev
    - `main` → staging / production (with approval)

### 3. Example Jenkinsfile (simplified)

```groovy
pipeline {
    agent any
    environment {
        IMAGE = "myrepo/user-service:${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/org/user-service.git'
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }
        stage('Docker Build & Push') {
            steps {
                sh '''
                  docker build -t $IMAGE .
                  docker push $IMAGE
                '''
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'helm upgrade --install user-service ./helm --set image.tag=${BUILD_NUMBER}'
            }
        }
    }
}
```

### 4. Key Design Considerations

- **Parallel pipelines**: Different services build simultaneously.
- **Kubernetes agents**: Jenkins spins up agents dynamically per build.
- **Centralized logging & monitoring** (ELK, Prometheus).
- **Secrets via Jenkins credentials + Kubernetes secrets**.
- **Approval step** before production deployment.

### Example scenario

- A change is pushed to the **payment service** repo.
- Jenkins pipeline runs only for that service.
- Docker image is built and deployed to Kubernetes.
- Other microservices remain untouched.

## 2. How do you implement blue-green or canary deployment using Jenkins?

To implement **blue-green or canary deployments using Jenkins**, Jenkins acts as the **orchestrator**, while tools like **Kubernetes, Helm, or load balancers** handle traffic switching. Here's an **interview-level explanation**.

### 1. Blue-Green Deployment using Jenkins

**How it works:**

- Maintain **two environments**:
    - **Blue** → current production
    - **Green** → new version
- Jenkins deploys the new version to **Green**.
- After validation, traffic is **switched from Blue to Green**.
- Blue stays as a **rollback option**.

**Pipeline flow:**

1. Build & test application
2. Deploy to **Green environment**
3. Run smoke tests
4. Manual approval
5. Switch traffic (Ingress / Load Balancer)

**Example Jenkins stage (Kubernetes + Ingress):**

```groovy
stage('Deploy to Green') {
    steps {
        sh 'helm upgrade --install app-green ./helm --set version=green'
    }
}
stage('Switch Traffic') {
    steps {
        sh 'kubectl apply -f ingress-green.yaml'
    }
}
```

### 3. Jenkins Role in Both

- Controls **pipeline logic and approvals**
- Integrates with **Kubernetes, Helm, Istio, or Nginx**
- Automates **promotion, rollback, and notifications**

### Example scenario

- New version of `payment-service` deployed.
- Canary receives 10% traffic.
- No errors → Jenkins promotes to 100%.
- Errors detected → Jenkins rolls back automatically.

## 3. How do you handle rollback in Jenkins?

**Rollback in Jenkins** means **reverting an application to a previously stable version** when a deployment fails or causes issues. Jenkins usually **coordinates the rollback**, while tools like **Docker, Kubernetes, or Helm** perform the actual revert.

### How rollback is handled

**1. Rollback using previous build/artifact**

- Jenkins keeps **build history and artifacts**.
- If the current deployment fails, Jenkins redeploys the **last successful build**.

**Example:**

- Build #25 fails in production
- Jenkins redeploys Build #24 automatically

**2. Rollback in Kubernetes using Helm (most common)**

Helm tracks **release versions**, making rollback easy.

**How it works:**

- Every `helm upgrade` creates a new release revision.
- On failure, Jenkins triggers:

```bash
helm rollback my-app <previous-revision>
```

**Why it's effective:**

- Fast rollback
- No need to rebuild images

**3. Automatic rollback using pipeline logic**

Jenkins can rollback **automatically** if a stage fails.

**Example logic:**

```groovy
stage('Deploy') {
    steps {
        sh 'helm upgrade my-app ./helm'
    }
}
post {
    failure {
        sh 'helm rollback my-app'
    }
}
```

**4. Rollback in Blue-Green / Canary deployments**

- **Blue-Green:** Switch traffic back to the **Blue** environment.
- **Canary:** Stop canary and route traffic back to **stable version**.

**Example:**

- Canary shows high error rate
- Jenkins stops canary and restores 100% traffic to stable pods

### Why rollback strategy is important

- Prevents **production downtime**
- Reduces **business impact**
- Enables **safe and confident deployments**

## 4. Jenkins master goes down — what's your recovery plan?

If the **Jenkins master (controller) goes down**, the recovery plan focuses on **restoring Jenkins quickly with minimal CI/CD downtime**, using backups and redundancy. Here's an **interview-ready recovery approach**:

### 1. Identify the Failure

- Check whether it's a **service crash, VM/container failure, disk issue, or plugin problem**.
- Review system logs (`jenkins.log`, OS logs) to understand the root cause.

**Why:** Helps decide whether to **restart, restore, or rebuild** Jenkins.

### 2. Immediate Recovery (Quick Win)

- If Jenkins is installed as a service:

```bash
sudo systemctl restart jenkins
```

- If Jenkins runs in Docker/Kubernetes, **restart the pod/container**.

**Why:** Many outages are temporary (memory spike, stuck process).

### 3. Restore from Backup (If Restart Fails)

- Provision a **new Jenkins server** (VM, container, or pod).
- Install the **same Jenkins version**.
- Restore the **Jenkins Home directory** backup:

```bash
/var/lib/jenkins
```

This restores:

- Jobs & pipelines
- Plugins
- Credentials
- Build history

**Result:** Jenkins comes back **exactly as before**.

### 4. Reconnect Agents

- Agents automatically reconnect if configured correctly.
- If not, re-register agents using:
    - SSH keys
    - JNLP tokens

**Why:** Builds can resume without job reconfiguration.

### 5. Validate & Resume Pipelines

- Verify:
    - Webhooks
    - Credentials
    - Plugin health
- Trigger a **test pipeline** to confirm everything works.

### 6. Long-Term Prevention Strategy

- **Regular automated backups** (ThinBackup / snapshots).
- Run Jenkins:
    - In **Docker or Kubernetes**
    - With **persistent volumes**
- Store Jenkins Home on **shared storage (EBS / NFS)**.
- Use **Pipeline-as-Code (Jenkinsfile in Git)** so pipelines aren't lost.

### Example real-world scenario

- Jenkins master VM crashes.
- New VM is created in 10 minutes.
- `/var/lib/jenkins` restored from backup.
- Jenkins is live with all jobs, credentials, and history intact.

## 5. How do you scale Jenkins?

**Scaling Jenkins** means handling **more builds, more teams, and higher workload** without slowing down or failing. Jenkins scales mainly by **distributing work**, not by making the master bigger.

### 1. Scale with Jenkins Agents (Primary Method)

- Keep the **master lightweight** (UI, scheduling only).
- Add **multiple agents** to run jobs in parallel.
- Agents can be:
    - Static VMs
    - Docker containers
    - Kubernetes pods (dynamic)

**Why:** Builds are the heavy part; agents do the heavy work.

### 2. Dynamic Scaling with Kubernetes

- Use the **Jenkins Kubernetes Plugin**.
- Agents (pods) are created **on demand** and destroyed after jobs finish.
- No idle infrastructure.

**Example:**

- 5 jobs → 5 pods
- 50 jobs → 50 pods

This is the **most scalable and cloud-native approach**.

### 3. Use Parallel Pipelines

- Split tests and builds into **parallel stages**.

```groovy
stage('Tests') {
    parallel {
        stage('Unit') { steps { sh 'run-unit-tests' } }
        stage('Integration') { steps { sh 'run-integration-tests' } }
    }
}
```

**Why:** Reduces overall pipeline time.

### 4. Separate Jenkins Masters (Logical Scaling)

- Use **multiple Jenkins masters/controllers** for:
    - Different teams
    - Different environments (prod vs non-prod)
- Prevents one team from affecting others.

### 5. Optimize Jenkins Performance

- Clean old builds and workspaces.
- Increase JVM heap size.
- Limit concurrent builds per job.
- Move heavy tasks off the master.

### 6. High Availability (Not True Horizontal Scaling)

- Jenkins master is **not easily horizontally scalable**.
- Instead:
    - Run Jenkins in **Docker/Kubernetes**
    - Use **persistent volumes**
    - Fast recovery instead of active-active HA

### Interview-ready summary

- Jenkins scales **vertically for control**, **horizontally for execution**
- Best practice: **Jenkins master + dynamic agents (Kubernetes)**
- Do **not run builds on the master**

### Example scenario:

A company grows from 10 to 200 pipelines → Kubernetes-based Jenkins agents handle spikes automatically without performance issues.

## 6. Difference between Jenkins and GitHub Actions?

**Jenkins vs GitHub Actions** — both are **CI/CD tools**, but they differ in **architecture, control, and use cases**.

### 1. Jenkins

- **Self-hosted CI/CD tool** (you manage servers, scaling, security).
- Extremely **flexible and customizable** using plugins.
- Supports **complex pipelines**, multiple SCMs, and hybrid environments.
- Requires **maintenance** (upgrades, backups, plugin compatibility).

**Best for:** Large enterprises, complex workflows, multi-repo or multi-cloud setups.

### 2. GitHub Actions

- **Fully managed CI/CD service** built into GitHub.
- Uses **YAML workflows** stored in the repo.
- **Auto-scales** with no infrastructure management.
- Limited customization compared to Jenkins, but very easy to start.

**Best for:** GitHub-centric projects, small to medium teams, fast setup.

| **Feature** | **Jenkins** | **GitHub Actions** |
| --- | --- | --- |
| Hosting | Self-hosted | GitHub-managed |
| Setup | Complex | Very simple |
| Scalability | Manual / Kubernetes | Automatic |
| Flexibility | Very high | Moderate |
| Maintenance | Required | None |
| Ecosystem | Plugins | Actions marketplace |

## 7. Can Jenkins be used without pipelines?

Yes, **Jenkins can be used without pipelines**.

### How:

- Jenkins can run jobs using **Freestyle jobs**.
- These are **GUI-based** jobs where you configure steps like build, test, and deploy through the Jenkins UI, without writing a Jenkinsfile.

### Why people still use it:

- Simple to set up for **small or legacy projects**.
- Useful when the workflow is **straightforward** and doesn't need complex logic.

### Limitation (important for interviews):

- Freestyle jobs are **harder to scale, version, and maintain** compared to pipelines.
- No "pipeline as code", limited reuse, and less flexibility.

### Example:

- A basic job that pulls code from Git and runs `mvn clean install` can be done entirely as a **Freestyle job**, with no pipeline involved.

## 8. How do you migrate Jenkins to cloud?

Migrating **Jenkins to the cloud** means moving the Jenkins controller, data, and execution environment from on-prem to a **cloud-based, scalable setup**. Here's an **interview-level, step-by-step approach**.

### 1. Assess the Existing Jenkins

- Identify:
    - Jenkins version
    - Installed plugins
    - Number of jobs & pipelines
    - Agents and workloads
- Clean up **unused jobs, plugins, and old builds**.

**Why:** Reduces migration risk and cloud costs.

### 2. Choose Cloud Architecture

Common cloud setups:

- **VM-based Jenkins** (EC2 / Compute Engine)
- **Docker-based Jenkins**
- **Kubernetes-based Jenkins** (most scalable)

**Best practice:** Jenkins controller in **Docker/Kubernetes** + **cloud storage**.

### 3. Back Up Jenkins

- Back up the **Jenkins Home directory**:

```bash
/var/lib/jenkins
```

Includes jobs, plugins, credentials, and configs.

**Critical:** This backup = your Jenkins state.

### 4. Provision Jenkins in the Cloud

- Create cloud resources (VM / EKS / GKE / AKS).
- Install Jenkins (same or compatible version).
- Mount **persistent storage** (EBS / PersistentVolume).

### 5. Restore Jenkins Data

- Copy the backup into the new Jenkins home directory.
- Start Jenkins and verify:
    - Jobs
    - Pipelines
    - Credentials
    - Plugins

**Result:** Jenkins looks exactly like on-prem.

### 6. Migrate & Optimize Agents

- Replace static on-prem agents with:
    - Cloud VMs
    - **Kubernetes dynamic agents**
- Configure auto-scaling.

**Why:** Cloud elasticity = faster builds, lower cost.

### 7. Update Integrations

- Update:
    - Webhooks (GitHub/GitLab URLs)
    - DNS and firewall rules
    - Cloud IAM roles for access

### 8. Test & Cut Over

- Run test pipelines.
- Validate deployments.
- Switch traffic and disable on-prem Jenkins.

### Example scenario

- Jenkins moved from on-prem VM to **EKS**.
- Jenkins controller runs as a pod.
- Agents are created dynamically per job.
- Builds scale automatically during peak hours.

## 9. Jenkins vs GitLab CI/CD?

**Jenkins vs GitLab CI/CD** — both support **CI/CD**, but they differ in **architecture, setup, and control**. Here's a **clear interview-level comparison**.

### 1. Jenkins

- **Self-hosted CI/CD tool** (you manage infra, scaling, and security).
- Uses **Jenkinsfile (Groovy)** for pipelines.
- Huge **plugin ecosystem** → very flexible.
- Works with **any Git provider** (GitHub, GitLab, Bitbucket).

**Best for:** Large enterprises, complex workflows, multi-tool and multi-cloud environments.

### 2. GitLab CI/CD

- **Built-in CI/CD** inside GitLab.
- Uses **.gitlab-ci.yml (YAML)**.
- **Auto-scaling runners** supported.
- Less flexible than Jenkins but **simpler to manage**.

**Best for:** Teams already using GitLab and wanting an **all-in-one DevOps platform**.

| **Feature** | **Jenkins** | **GitLab CI/CD** |
| --- | --- | --- |
| Hosting | Self-hosted | GitLab-managed or self-hosted |
| Setup | Complex | Easy |
| Configuration | Groovy (Jenkinsfile) | YAML |
| Flexibility | Very high | Moderate |
| Maintenance | Required | Minimal |
| SCM Support | Any | GitLab only |
| Plugins / Integrations | Massive plugin ecosystem | Built-in features |
