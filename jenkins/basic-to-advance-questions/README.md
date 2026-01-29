# Jenkins Interview Questions

## 🔹 Jenkins Basics

### 1. What is Jenkins and why is it used in DevOps?

**Jenkins** is an **open-source automation tool** used to **build, test, and deploy applications automatically**.

**Why it is used in DevOps:**

Jenkins helps implement **CI/CD (Continuous Integration and Continuous Delivery)** by automatically running jobs whenever developers push code. This reduces manual work and helps teams detect errors early.

**Example:**

When a developer commits code to Git, Jenkins can automatically compile the code, run tests, and deploy it to a server — all without manual intervention.

---

### 2. Difference between **Freestyle job** and **Pipeline job**

**Freestyle Job vs Pipeline Job in Jenkins**

**Freestyle Job:**

- It is a **simple, GUI-based job** where you configure build steps by clicking options.
- Best for **small or simple projects**, but it becomes hard to manage for complex workflows.

**Pipeline Job:**

- It is **code-based**, written using a **Jenkinsfile**, which defines the entire CI/CD process.
- Best for **complex projects** because it supports version control, automation, and advanced logic.

**Key Difference (in short):**

Freestyle is **easy to set up but limited**, while Pipeline is **powerful, flexible, and suitable for real DevOps workflows**.

---

### 3. What is **CI/CD**, and how does Jenkins fit into it?

**CI/CD** stands for **Continuous Integration and Continuous Delivery (or Deployment)**.

**CI (Continuous Integration):**

Developers frequently merge code into a shared repository, and each change is **automatically built and tested** to catch issues early.

**CD (Continuous Delivery/Deployment):**

After successful tests, the application is **automatically delivered or deployed** to staging or production environments.

**How Jenkins fits in:**

Jenkins acts as the **automation engine** for CI/CD. It monitors code changes, triggers builds, runs tests, and deploys applications automatically through pipelines.

**Example:**

When code is pushed to Git, Jenkins triggers a pipeline that builds the app, runs tests, and deploys it to a server without manual effort.

---

### 4. How does Jenkins work internally (master/agent architecture)?

Jenkins works using a **Master–Agent architecture** to distribute and manage build tasks efficiently.

**Master (Controller):**

- Manages jobs, schedules builds, and provides the **web UI**.
- Decides **which agent** should run a job and monitors the build results.

**Agent (Node):**

- Executes the actual **build, test, or deployment tasks**.
- Agents connect to the master and run jobs as instructed.

**How it works (flow):**

When a job is triggered, the master assigns it to an available agent. The agent runs the job and sends the results back to the master.

**Why this architecture is used:**

It improves **scalability and performance** by allowing multiple builds to run in parallel on different machines.

---

### 5. What are Jenkins agents (nodes)?

**Jenkins agents (also called nodes)** are **machines that execute Jenkins jobs** assigned by the Jenkins master (controller).

**How they work:**

The master schedules a job and sends it to an agent. The agent runs the build steps (compile, test, deploy) and reports the results back to the master.

**Why they are used:**

Agents help Jenkins **scale and perform faster** by running multiple jobs in parallel and by using different environments (Linux, Windows, Docker, etc.) for different builds.

**Example:**

One agent can run Java builds on Linux, while another agent runs .NET builds on Windows.

---

### 6. Default Jenkins port and home directory?

**Default Jenkins Port:**

- Jenkins runs on **port 8080** by default.
- This means you can access it via:

```
http://<server-ip>:8080
```

**Default Jenkins Home Directory:**

- The home directory is:

```
/var/lib/jenkins
```

- It stores **all configurations, jobs, plugins, and build history**.

**Why this matters:**

Knowing the port helps in **accessing Jenkins**, and the home directory is important for **backup and troubleshooting**.

---

### 7. What is a Jenkins workspace?

**Jenkins Workspace** is the **directory on an agent or master where Jenkins runs a specific job**.

**How it works:**

- When a job is triggered, Jenkins **checks out the source code** into the workspace.
- All **build, test, and deployment operations** for that job happen inside this folder.

**Why it is important:**

- It **isolates jobs** so that each job has its own environment.
- It stores **temporary files, logs, and build artifacts** for that job.

**Example:**

For a job named `MyApp`, the workspace could be:

```
/var/lib/jenkins/workspace/MyApp
```

Jenkins will compile the code and run tests inside this folder before archiving or deploying the results.

---

## 🔹 Jenkins Pipeline

### 8. What is a **Jenkins Pipeline**?

**Jenkins Pipeline** is a **code-defined workflow** that automates the process of building, testing, and deploying applications. Unlike freestyle jobs, a pipeline is written as code in a **Jenkinsfile**, allowing version control and complex automation.

**How it works:**

- A pipeline defines **stages** (like Build, Test, Deploy) and **steps** (specific commands) that Jenkins executes in order.
- It can run on **multiple agents**, handle failures, and include conditional logic.

**Why it is used:**

- Ensures **repeatable, consistent CI/CD processes**.
- Makes **automation transparent**, since the workflow is stored in code alongside the project.

**Example:**

A simple pipeline might have:

1. **Build stage:** compile the code.
2. **Test stage:** run unit tests.
3. **Deploy stage:** push the application to a staging server.

---

### 9. Difference between **Declarative** and **Scripted Pipeline**

Both are ways to define **Jenkins Pipelines**, but they differ in **syntax, structure, and use cases**.

**1. Declarative Pipeline:**

- **Structured and simpler**; uses a predefined `pipeline {}` syntax.
- Designed for **readability and ease of use**, ideal for teams starting with Jenkins.
- Supports **built-in error handling, stages, and post actions** easily.

**Example snippet:**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') { steps { sh 'make' } }
        stage('Test') { steps { sh 'make test' } }
    }
}
```

**2. Scripted Pipeline:**

- **More flexible and programmatic**, written in Groovy code.
- Suitable for **complex workflows**, loops, and conditional logic not easily handled in declarative syntax.
- Requires more **expertise** to maintain and debug.

**Example snippet:**

```groovy
node {
    stage('Build') { sh 'make' }
    stage('Test') { sh 'make test' }
}
```

---

### 10. What is a `post` block in declarative pipeline?

In a **Declarative Jenkins Pipeline**, a **`post` block** is used to **define actions that run after the pipeline or a stage completes**, regardless of whether it **succeeded, failed, or was unstable**.

**How it works:**

- It allows you to **handle cleanup, notifications, or other final steps** after the main pipeline execution.
- The `post` block supports **conditions** like `always`, `success`, `failure`, `unstable`, and `changed`.

**Example:**

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make'
            }
        }
    }
    post {
        always {
            echo 'This runs regardless of success or failure'
        }
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

**Why it is used:**

- Ensures **critical tasks like cleanup, archiving artifacts, or sending notifications** always happen.
- Makes the pipeline **more robust and maintainable**, especially in automated CI/CD workflows.

**Example scenario:**

Even if a test fails, the `post` block can **archive logs and send a Slack notification** to the team.

---

### 11. How do you handle pipeline failures?

**Handling Pipeline Failures in Jenkins** involves proactive and reactive steps to ensure **CI/CD reliability**.

**1. Using `post` blocks in Declarative Pipelines:**

- You can define actions that run on **failure** using the `failure` condition.
- Typical actions include **sending notifications, archiving logs, or cleaning up resources**.

**Example:**

```groovy
post {
    failure {
        echo 'Build failed!'
        archiveArtifacts artifacts: '**/logs/*.log', allowEmptyArchive: true
        mail to: 'dev-team@example.com', subject: 'Pipeline Failed', body: 'Check Jenkins logs'
    }
}
```

**2. Catching errors in Scripted Pipelines:**

- Use `try-catch` blocks to **handle errors gracefully** and take corrective steps.

**Example:**

```groovy
node {
    try {
        stage('Build') { sh 'make' }
        stage('Test') { sh 'make test' }
    } catch (err) {
        echo "Pipeline failed: ${err}"
        currentBuild.result = 'FAILURE'
    } finally {
        echo 'Cleanup actions can go here'
    }
}
```

**Why it's important:**

- Ensures the team **knows what failed** immediately.
- Prevents **half-deployed or broken builds** from affecting production.
- Helps maintain **reliability in automated CI/CD pipelines**.

**Example scenario:**

If unit tests fail, you can automatically **archive logs, notify developers, and stop deployment**, preventing faulty code from reaching production.

---

### 12. How do you add approval steps in a pipeline?

In Jenkins, **approval steps** are used to **pause a pipeline until a human reviews or approves it** before continuing, often in **production or sensitive deployments**.

**How it works:**

- Jenkins provides the **`input` step** in both Declarative and Scripted pipelines.
- The pipeline **waits at that point**, and a user must manually approve or reject to continue.

**Example in Declarative Pipeline:**

```groovy
pipeline {
    agent any
    stages {
        stage('Deploy to Staging') {
            steps {
                sh 'deploy-staging.sh'
            }
        }
        stage('Approval') {
            steps {
                input message: 'Approve deployment to Production?', ok: 'Deploy'
            }
        }
        stage('Deploy to Production') {
            steps {
                sh 'deploy-production.sh'
            }
        }
    }
}
```

**How it works internally:**

- Jenkins pauses at the `input` step.
- Only a user with the required permissions can **click "Proceed"** on the Jenkins UI.
- After approval, the next stage executes.

**Why it is used:**

- Ensures **controlled deployments** and prevents accidental changes in production.
- Adds a **human checkpoint** in an otherwise fully automated CI/CD pipeline.

**Example scenario:**

After successful testing, the pipeline pauses for **manager approval** before deploying a critical update to production servers.

---

## 🔹 Jenkins Plugins

### 13. What are Jenkins plugins?

**Jenkins Plugins** are **add-ons that extend Jenkins' functionality**, allowing it to integrate with tools, services, and custom workflows.

**How they work:**

- Jenkins has a **core system**, and plugins add extra features like **SCM integration, build notifications, pipeline enhancements, or deployment tools**.
- Plugins can be **installed via the Jenkins UI** or updated as needed to add new capabilities.

**Why they are used:**

- Make Jenkins **flexible and scalable** for different DevOps needs.
- Allow integration with tools like **Git, Docker, Slack, Maven, Kubernetes**, etc., without writing custom scripts.

**Example:**

- **Git Plugin:** Allows Jenkins to pull code from Git repositories.
- **Pipeline Plugin:** Enables writing declarative or scripted pipelines.

---

### 14. What happens if a plugin fails?

If a **Jenkins plugin fails**, it can affect the jobs or features that depend on it, but Jenkins itself usually continues running.

**How it works internally:**

- Jenkins loads plugins during startup. If a plugin has an **error or incompatibility**, it may **not load**, and features relying on it may fail.
- During a job, if a plugin step fails (like Git checkout via Git plugin), Jenkins will **mark the build as failed** and log the error.

**Why this matters:**

- A failed plugin can **break CI/CD pipelines**, prevent deployments, or stop integrations (like Slack notifications).
- Proper plugin management—**updating, checking compatibility, and backups**—is critical in production environments.

**Example scenario:**

- If the **Docker plugin fails**, jobs that build or push Docker images will fail, but other unrelated jobs (like Maven builds) will continue to work.
- Logs will show the **error**, helping administrators identify and fix the issue.

---

### 15. How do you upgrade Jenkins plugins safely?

**Upgrading Jenkins plugins safely** is crucial to avoid **breaking pipelines or Jenkins itself**. Here's an **interview-level explanation**:

**1. Check plugin compatibility:**

- Go to **Manage Jenkins → Manage Plugins → Updates**.
- Review the **plugin release notes** and ensure they are **compatible with your Jenkins version**.
- Avoid upgrading **core or critical plugins** immediately on production; test first.

**2. Backup Jenkins before upgrading:**

- Backup **Jenkins home directory (`/var/lib/jenkins`)**, which contains jobs, configs, and plugins.
- This ensures you can **restore Jenkins** if a plugin upgrade fails.

**3. Upgrade plugins in a controlled way:**

- Prefer upgrading **one plugin at a time** in production.
- For multiple plugins, consider **testing in a staging Jenkins instance** first.

**4. Restart Jenkins if needed:**

- Some plugin upgrades require a **Jenkins restart**.
- Use **safe restart**:

```bash
java -jar jenkins-cli.jar safe-restart
```

This allows running jobs to finish before restarting.

**Why this approach is used:**

- Prevents **pipeline failures or downtime**.
- Ensures **system stability** while keeping Jenkins up-to-date.

**Example scenario:**

Before upgrading the **Git plugin**, you back up Jenkins, test the upgrade on staging, then upgrade in production to avoid breaking any jobs that pull code from Git repositories.

---

## 🔹 Jenkins & Git

### 16. What is a webhook?

A **webhook** is a **mechanism for one system to notify another system automatically** when an event occurs. Instead of constantly checking (polling) for changes, the system **"pushes" information in real-time**.

**How it works:**

- A system (like GitHub) is configured with a **URL endpoint** (the webhook).
- When an event happens (e.g., a code push), GitHub **sends an HTTP POST request** to the URL.
- Jenkins or another service receives the request and triggers an action, like a **build or deployment**.

**Why it is used:**

- Enables **real-time automation** in CI/CD pipelines.
- Reduces **manual intervention and resource-heavy polling**.

**Example scenario:**

- A developer pushes code to GitHub.
- GitHub sends a webhook to Jenkins.
- Jenkins automatically starts the pipeline to **build, test, and deploy the application**.

---

### 17. Difference between **poll SCM** and **webhooks**

Both are ways to **trigger Jenkins jobs when code changes**, but they work differently:

**1. Poll SCM:**

- Jenkins **checks the source code repository periodically** (e.g., every 5 minutes) for changes.
- If changes are detected, it triggers a build.
- **Example:** `H/5 * * * *` polls every 5 minutes.

**Pros:** Simple to set up.

**Cons:** **Consumes resources** even if no changes occur and introduces slight **delay** in builds.

**2. Webhooks:**

- The repository (like GitHub or GitLab) **notifies Jenkins immediately** when code is pushed, via an HTTP POST request.
- Jenkins triggers the job **instantly**, without periodic polling.

**Pros:** **Real-time triggering** and more efficient.

**Cons:** Requires **repository support and correct configuration** of webhook URL.

**Key Difference:**

- **Poll SCM = pull-based** (Jenkins asks for changes).
- **Webhook = push-based** (repository tells Jenkins about changes).

**Example scenario:**

- Poll SCM: Jenkins checks Git every 5 minutes, possibly delaying build.
- Webhook: As soon as a developer pushes code, Jenkins starts the pipeline immediately.

---

### 18. How do you trigger Jenkins job on Git commit?

To **trigger a Jenkins job on a Git commit**, you can use either **webhooks** (recommended) or **polling SCM**.

---

### 19. How do you build only a specific branch?

To **build only a specific branch** in Jenkins, you configure the job or pipeline to **target that branch** rather than building all branches in the repository.

**1. In a Freestyle Job:**

- Go to **Job Configuration → Source Code Management → Git**.
- Set the **Branch Specifier** to the desired branch, e.g.:

```
*/main
```

- Only commits pushed to the `main` branch will trigger a build (if combined with webhooks or polling).

**2. In a Declarative Pipeline (Jenkinsfile):**

You can define the branch dynamically or statically:

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/user/repo.git'
            }
        }
        stage('Build') {
            steps {
                sh 'make'
            }
        }
    }
}
```

- The `branch` parameter ensures Jenkins **checks out only the `main` branch**.

**Why this is used:**

- Prevents unnecessary builds on feature branches or other development branches.
- Focuses CI/CD resources on **production or release branches**, improving efficiency.

**Example scenario:**

- You have `develop` and `main` branches.
- Only `main` triggers production deployment; `develop` triggers staging builds.

---

## 🔹 Jenkins Security

### 20. How do you secure Jenkins?
