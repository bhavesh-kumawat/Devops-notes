# DevOps Engineering Interview Guide

# 🔧 CI/CD & Automation

## 1. Your production deployment failed halfway through. What do you do?

### 🔹 Step 1: Stop the Bleeding (Stabilize First)

- Pause or abort the deployment pipeline immediately
- Prevent further changes from being rolled out
- Communicate in the incident channel: "Deployment paused, investigating"

**First goal is limit blast radius, not fix everything instantly.**

### 🔹 Step 2: Assess Impact

**Ask:**

- Is the app partially serving traffic?
- Are users affected or just some instances?
- Which components are broken? (API, DB, background jobs)

**Check:**

- Load balancer health checks
- Error rates (4xx/5xx)
- Key business metrics

### 🔹 Step 3: Rollback or Fail Forward

**Option A: Rollback (Most Common)**

- Roll back to last known good version
- **Examples:**
    - Blue-Green → switch traffic back to blue
    - Kubernetes → `kubectl rollout undo`
    - ECS → revert task definition
    - Lambda → rollback to previous version/alias

**Rollback should be fast, automated, and boring**

**Option B: Fix Forward (If Safe & Small)**

- Only if:
    - Issue is minor
    - No data corruption
    - Fix is quick and isolated

### 🔹 Step 4: Verify System Health

**After rollback:**

- Confirm:
    - App is stable
    - Error rates return to normal
    - Latency and traffic are healthy
- Keep monitoring for at least 10–30 minutes

### 🔹 Step 5: Root Cause Analysis (RCA)

**Once stable:**

- Review:
    - CI/CD logs
    - Deployment diffs
    - Config changes
    - Infra vs app failures
- Identify:
    - Why it failed halfway
    - Why it wasn't caught earlier

### 🔹 Step 6: Prevent It from Happening Again

**Examples:**

- Add:
    - Pre-deployment checks
    - Smoke tests
    - Canary or blue-green deployments
- Improve:
    - Rollback automation
    - Alerting during deploys
    - Feature flags for risky changes

---

## 2. A build works locally but fails in the CI pipeline. How do you debug it?

### 🔹 Step 1: Read the CI Error Carefully (Don't Guess)

- Look at:
    - Exact error message
    - Which stage failed (build / test / package)
- Compare:
    - Local success logs vs CI failure logs

**Many candidates jump to conclusions — good engineers read first.**

### 🔹 Step 2: Compare Local vs CI Environments

Most "works locally" issues come from environment mismatch.

**Check:**

- OS (Linux CI vs macOS/Windows local)
- Language/runtime versions (Java, Node, Python, Go)
- Package manager versions
- Environment variables

💡 **Fix:** Pin versions using:

- `.nvmrc`, `.python-version`
- Docker
- Version managers

### 🔹 Step 3: Dependency Issues

**Common problems:**

- Missing dependencies in CI
- `package-lock.json` / `poetry.lock` / `go.sum` not committed
- Cached dependencies causing conflicts

**Actions:**

- Clear CI cache
- Reinstall dependencies from scratch
- Run:

```bash
npm ci
pip install -r requirements.txt
```

### 🔹 Step 4: Environment Variables & Secrets

- Local machine has env vars that CI doesn't
- Secrets missing or misconfigured in CI

**Check:**

- CI secret manager (GitHub Actions, GitLab CI, Jenkins, etc.)
- Variable names and scopes
- Masked secrets vs plaintext values

### 🔹 Step 5: File Paths & Permissions

CI runs in a clean filesystem.

**Common failures:**

- Case-sensitive paths (`Config.yaml` vs `config.yaml`)
- Files not committed to repo
- Permission issues (`chmod +x` missing)

### 🔹 Step 6: Network / External Services

CI environment may:

- Block outbound traffic
- Not have access to:
    - Databases
    - Private repos
    - Internal APIs

**Fixes:**

- Mock external services
- Use test containers
- Whitelist CI IPs if needed

### 🔹 Step 7: Reproduce CI Locally

**Best debugging move 🚀**

- Run CI steps locally:

```bash
docker run <ci-image>
```

- Or:

```bash
act   # GitHub Actions
```

If it fails locally → easier to debug.

### 🔹 Step 8: Add Debugging Output (Temporarily)

- Print:
    - Environment variables
    - Versions
    - Working directory
- Enable verbose logging:

```bash
set -x
```

---

## 3. How would you design a CI/CD pipeline for a microservices application?

### 🔹 High-Level Goals

For a microservices CI/CD pipeline, the goals are:

- Independent deployments per service
- Fast feedback (fail early)
- Safe releases (no downtime)
- Easy rollback
- Consistent environments

### 🔹 High-Level Architecture

```
Git → CI → Artifact Registry → CD → Kubernetes / ECS → Monitoring
```

Each microservice has its own pipeline, but follows a standard template.

### 🔹 Step 1: Source Control Strategy

- One repo per microservice (preferred) or mono-repo with paths
- Git branching:
    - `main` → production
    - `feature/*` → feature branches
- Every commit triggers CI automatically

### 🔹 Step 2: Continuous Integration (CI)

**CI Stages (per microservice)**

1. Code Checkout
2. Static Checks
    - Linting
    - Formatting
    - Security scanning (SAST)
3. Unit Tests
4. Build Artifact
    - Docker image
    - Tag with commit-sha + version
5. Push to Registry
    - ECR / Docker Hub / Artifact Registry

💡 **Fail fast — don't build if tests fail.**

### 🔹 Step 3: Artifact Management

- Store immutable artifacts:
    - Docker images
    - Helm charts
- Never rebuild the same artifact in CD

**"Build once, deploy many times"**

### 🔹 Step 4: Continuous Deployment (CD)

**Deployment Flow**

1. Deploy to Dev
2. Automated integration tests
3. Deploy to Staging
4. Manual approval (optional)
5. Deploy to Production

**Deployment Strategies**

- Blue-Green for critical services
- Canary for high-traffic APIs
- Rolling updates for low-risk services

### 🔹 Step 5: Configuration & Secrets

- **Config:**
    - ConfigMaps / Parameter Store
- **Secrets:**
    - Secrets Manager / Vault / Kubernetes Secrets
- Environment-specific configs, same image

### 🔹 Step 6: Infrastructure as Code

- Provision infra using:
    - Terraform / CloudFormation
- Kubernetes:
    - Helm or Kustomize
- Version-controlled infrastructure

### 🔹 Step 7: Observability & Feedback

- **Monitoring:** Prometheus + Grafana / CloudWatch
- **Logging:** ELK / CloudWatch Logs
- **Tracing:** X-Ray / Jaeger
- Alerts tied to deployments

### 🔹 Step 8: Rollback Strategy

- Rollback by:
    - Reverting Helm release
    - Switching traffic back (blue-green)
    - Redeploying previous image tag

**Rollback must be:**

- Fast
- Automated
- Safe

### 🔹 Security Built-In (Shift Left)

- Dependency scanning
- Image scanning
- Secrets scanning
- Least-privilege IAM roles

---

## 4. Your pipeline is too slow and blocking releases. How do you optimize it?

### 🔹 Step 1: Find Where Time Is Actually Spent

First rule: don't optimize blindly.

- **Measure:**
    - Build time
    - Test time
    - Image build & push
    - Deployment time
- Use pipeline timing metrics or logs

**"I start by identifying the slowest stages before making changes."**

### 🔹 Step 2: Fail Fast (Shift Left)

- Run linting & unit tests first
- Stop the pipeline early if something fails
- Don't build Docker images if tests fail

💡 **Saves time and money.**

### 🔹 Step 3: Parallelize Aggressively

- Run:
    - Unit tests in parallel
    - Integration tests in parallel
    - Independent microservice pipelines independently
- **Examples:**
    - Test sharding
    - Matrix builds

**Serial pipelines are the biggest CI killer.**

### 🔹 Step 4: Use Caching Correctly (But Carefully)

**Cache:**

- Dependencies (`node_modules`, Maven, pip)
- Docker layers
- Build artifacts

**Avoid:**

- Over-caching → flaky builds
- Cache invalidation bugs

### 🔹 Step 5: Optimize Docker Builds

- Use multi-stage Docker builds
- Minimize layers
- Use smaller base images (Alpine, distroless)
- Reorder Dockerfile:
    - Copy dependency files first
    - Install deps once

### 🔹 Step 6: Reduce Test Scope Smartly

- Run:
    - Full test suite on main
    - Targeted tests on feature branches
- Use:
    - Contract tests instead of heavy E2E tests
    - Nightly full regression tests

### 🔹 Step 7: Optimize Deployment Strategy

- Use:
    - Rolling updates
    - Canary instead of full redeploys
- Avoid redeploying unchanged services
- Use immutable artifacts

### 🔹 Step 8: Scale CI Infrastructure

- Increase:
    - CI runners
    - CPU / memory for build agents
- Use autoscaling runners

**A slow pipeline is often an underpowered pipeline.**

### 🔹 Step 9: Introduce Async & Decoupling

- Make non-critical checks async:
    - Security scans
    - Performance tests
- Don't block releases unnecessarily

---

# ☁️ Cloud & Infrastructure

## 5. Traffic suddenly spikes 10×. Your app becomes slow. What steps do you take?

### 🔴 Phase 1: Stabilize the System (First 5–10 Minutes)

**1️⃣ Confirm the Spike Is Real**

- Check:
    - Load balancer request count
    - Traffic sources (organic vs bots)
- Validate it's not a monitoring glitch

**Don't fight ghosts.**

**2️⃣ Protect the User Experience**

- Enable / tighten rate limiting
- Turn on:
    - API Gateway throttling
    - WAF rules (bot / abuse protection)
- Return graceful degradation:
    - Feature flags off
    - Disable non-critical functionality

### ⚡ Phase 2: Scale Fast (Next 10–20 Minutes)

**3️⃣ Scale the App Layer**

- Trigger or increase:
    - Auto Scaling Groups
    - Kubernetes HPA
    - ECS service desired count
- Scale horizontally first

**Vertical scaling is slower and riskier during incidents.**

**4️⃣ Check the Load Balancer**

- Ensure:
    - Healthy targets
    - No connection saturation
- Enable connection draining if needed

### 🧠 Phase 3: Identify Bottlenecks

**5️⃣ Identify the Limiting Resource**

Check dashboards:

- CPU / memory
- Thread pools
- DB connections
- Queue depth
- Cache hit ratio

**Ask:**

- App layer?
- Database?
- Cache?
- External API?

**6️⃣ Database & Cache Protection**

- Enable / expand:
    - Read replicas
    - Query caching
    - Connection pooling
- Apply timeouts and circuit breakers

**Databases are usually the first to melt.**

### 🔍 Phase 4: Optimize Traffic Flow

**7️⃣ Use Caching Aggressively**

- Enable:
    - CDN (CloudFront)
    - Application cache (Redis / ElastiCache)
- Cache:
    - Static assets
    - Read-heavy API responses

**8️⃣ Queue Where Possible**

- Offload:
    - Emails
    - Reports
    - Background jobs
- Use:
    - SQS / Kafka
    - Async processing

### 📈 Phase 5: Observe & Communicate

**9️⃣ Monitor in Real Time**

- Error rate
- Latency
- Saturation metrics
- Watch for cascading failures

**🔟 Communicate Clearly**

- Update:
    - Incident channel
    - Stakeholders
- Set expectations

**Silence causes more damage than outages.**

### 🧪 Phase 6: After the Incident

**11️⃣ Post-Incident Review**

- Root cause
- Why scaling wasn't fast enough
- What broke first

**12️⃣ Prevent Recurrence**

- Load testing
- Autoscaling tuning
- Capacity planning
- Better caching strategy

---

## 6. One Availability Zone goes down. How do you design for high availability?

### 🔹 1. Multi-AZ Architecture (Baseline)

**Compute**

- Run workloads across multiple Availability Zones
- Use:
    - Auto Scaling Groups spanning ≥2 AZs
    - ECS / EKS with multi-AZ scheduling

**Load Balancing**

- Use ALB / NLB across AZs
- Health checks automatically route traffic away from failed AZs

### 🔹 2. Stateless Application Design

- Keep app instances stateless
- Store state in:
    - RDS / DynamoDB
    - S3
    - ElastiCache (with replication)

**Stateless apps recover instantly when instances are replaced.**

### 🔹 3. Highly Available Data Layer

**Databases**

- RDS / Aurora Multi-AZ
    - Automatic failover
- DynamoDB
    - Multi-AZ by default

**Caching**

- ElastiCache with replication groups
- Primary + replicas across AZs

### 🔹 4. Storage Resilience

- **S3** → multi-AZ by default
- **EBS**
    - Use snapshots
    - Attach only within AZ, but recover quickly
- **EFS**
    - Multi-AZ file system

### 🔹 5. Health Checks & Auto Recovery

- Load balancer health checks
- Auto Scaling replaces unhealthy instances
- **Kubernetes:**
    - Liveness & readiness probes
    - Pod rescheduling to healthy AZs

### 🔹 6. Traffic & DNS Strategy

- Route 53 health checks
- **Optional:**
    - Failover routing
    - Weighted routing (for multi-region)

### 🔹 7. Deployment Strategy Matters

- Rolling, Blue-Green, or Canary
- Never deploy all AZs at once
- Always keep at least one AZ serving traffic

### 🔹 8. Test AZ Failures (Very Important)

- Simulate AZ failures:
    - Disable subnets
    - Kill nodes
- Validate:
    - Traffic shifts
    - Recovery time (RTO)
    - Data integrity (RPO)

**HA you don't test is just hope.**

---

## 7. You need to reduce cloud costs without affecting performance. What do you do?

### 🧠 Core Principle

**Reduce waste first. Optimize later. Never degrade performance.**

### 🔹 Step 1: Get Visibility (You Can't Fix What You Can't See)

- Use:
    - AWS Cost Explorer
    - Cost & Usage Reports (CUR)
    - Per-service and per-team tagging
- Identify:
    - Top 5 cost drivers
    - Underutilized resources

### 🔹 Step 2: Eliminate Obvious Waste (Fast Wins)

**Compute**

- Stop or downsize:
    - Idle EC2 instances
    - Unused ASGs
- Right-size instances using:
    - CPU / memory metrics
- Remove:
    - Unattached EBS volumes
    - Old snapshots

### 🔹 Step 3: Optimize Scaling (Biggest Impact)

- Use Auto Scaling aggressively
- Scale based on:
    - Request count
    - Queue depth
    - Latency (not just CPU)
- Reduce baseline capacity off-peak

**Paying for idle capacity is the #1 cost leak.**

### 🔹 Step 4: Use the Right Pricing Models

- **Reserved Instances / Savings Plans** for steady workloads
- **Spot Instances** for:
    - Batch jobs
    - CI runners
    - Non-critical services
- Keep critical workloads on **On-Demand**

### 🔹 Step 5: Improve Performance to Reduce Cost

**Counterintuitive but true 👇**

- Faster apps = fewer resources

**Actions:**

- Add caching (Redis, CloudFront)
- Optimize DB queries
- Reduce chatty APIs

### 🔹 Step 6: Storage Optimization

- Move infrequently accessed data to:
    - S3 IA / Glacier
- Enable:
    - S3 lifecycle policies
    - Log retention policies
- Compress logs

### 🔹 Step 7: Network & Data Transfer Costs

- Use:
    - VPC endpoints (avoid NAT data charges)
    - CloudFront to reduce egress
- Keep traffic inside the same region/AZ

### 🔹 Step 8: CI/CD & Dev/Test Cost Control

- Auto-shutdown non-prod environments
- Use smaller instances for test workloads
- Limit parallel CI runners

### 🔹 Step 9: Guardrails & Automation

- Budgets + alerts
- Policy-as-code:
    - Block oversized instances
    - Enforce tagging
- Regular cost reviews (FinOps culture)

---

# 🐳 Containers & Kubernetes

## 8. A Kubernetes pod keeps restarting. How do you troubleshoot it?

### 🔹 Step 1: Confirm Why the Pod Is Restarting

```bash
kubectl get pod <pod-name>
```

**Check:**

- STATUS → `CrashLoopBackOff`, `Error`, `OOMKilled`

**Then:**

```bash
kubectl describe pod <pod-name>
```

**Look at:**

- Last State
- Exit codes
- Events (often the biggest clue)

### 🔹 Step 2: Check Container Logs

```bash
kubectl logs <pod-name>
```

**If it already restarted:**

```bash
kubectl logs <pod-name> --previous
```

**Look for:**

- App crashes
- Stack traces
- Config or env variable errors

### 🔹 Step 3: Identify Common Root Causes

**1️⃣ Application Crash**

- Unhandled exception
- Misconfigured startup command
- Missing config or secrets

➡️ Fix app config or code.

**2️⃣ Liveness / Readiness Probe Failures**

- Probe too aggressive
- App needs more startup time

**Check:**

```yaml
livenessProbe:
readinessProbe:
```

**Fix:**

- Increase `initialDelaySeconds`
- Adjust `timeoutSeconds` / `failureThreshold`

**3️⃣ OOMKilled (Out of Memory)**

- Pod exceeds memory limits

**Check:**

```bash
kubectl describe pod
```

**Fix:**

- Increase memory limits
- Optimize memory usage
- Add HPA if needed

**4️⃣ Image Pull Issues**

- Wrong image tag
- Registry auth problems

**Check events:**

```
ImagePullBackOff
```

**Fix:**

- Correct image
- Fix `imagePullSecrets`

**5️⃣ Dependency Failures**

- DB / API not reachable
- App exits when dependency is down

**Fix:**

- Add retries
- Use readiness probes instead of crashing
- Implement graceful startup logic

### 🔹 Step 4: Validate Configuration

- ConfigMaps mounted?
- Secrets present?
- Correct env vars?

```bash
kubectl describe pod
```

### 🔹 Step 5: Check Node & Resource Pressure

- Node out of memory or disk
- Pod evicted

```bash
kubectl describe node <node-name>
```

### 🔹 Step 6: Debug Interactively (If Needed)

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

**Check:**

- Files
- Env vars
- Network access

### 🔹 Step 7: Fix & Roll Forward

- Apply fix
- Restart deployment:

```bash
kubectl rollout restart deployment <name>
```

**Monitor:**

- Pod stability
- Restart count
- Error rate

---

## 9. One microservice is consuming excessive CPU in the cluster. What's your approach?

### 🔹 Phase 1: Confirm & Scope the Problem

**1️⃣ Identify the Offending Service**

- Check metrics:
    - Prometheus / Grafana
    - `kubectl top pods`
- Verify:
    - Is CPU high for one pod or all replicas?
    - Is it sustained or spiky?

**Always confirm before acting.**

### 🔹 Phase 2: Contain the Impact

**2️⃣ Protect the Cluster**

- Apply or verify:
    - CPU requests and limits
    - Pod disruption budgets
- Prevent noisy neighbor problems

**One bad service should never starve the cluster.**

### 🔹 Phase 3: Identify the Root Cause

**3️⃣ Check Traffic Patterns**

- Has request volume increased?
- Any new consumers?
- Is it being hammered by retries or bots?

**4️⃣ Analyze Application Behavior**

- Check:
    - Logs for tight loops
    - Inefficient algorithms
    - Serialization or parsing hot paths
- Profile if possible (pprof, JVM tools)

**5️⃣ Look for Dependency Issues**

- Slow downstream service causing retries
- Missing timeouts or circuit breakers

**High CPU is often symptom, not cause.**

### 🔹 Phase 4: Fix or Mitigate

**6️⃣ Short-Term Mitigation**

- Scale horizontally (HPA)
- Rate-limit incoming requests
- Enable caching
- Reduce log verbosity

**7️⃣ Long-Term Fix**

- Optimize code paths
- Improve caching
- Add async processing
- Tune CPU requests/limits
- Add autoscaling based on:
    - CPU
    - RPS
    - Latency

### 🔹 Phase 5: Prevent Recurrence

**8️⃣ Improve Observability**

- Service-level dashboards
- CPU per request metrics
- Alerts on abnormal CPU growth

**9️⃣ Add Guardrails**

- Resource quotas
- Admission controllers
- Cost and usage alerts

---

## 10. How would you deploy a zero-downtime update in Kubernetes?

### 🧠 Core Principle

**Never take down all replicas at once, and never send traffic to an unready pod.**

### 🔹 Step 1: Use a Deployment (Not Naked Pods)

- Deploy apps using Deployment or StatefulSet
- Ensure:
    - At least 2 replicas
    - Pod is stateless

### 🔹 Step 2: Configure Rolling Update Strategy

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

- `maxUnavailable: 0` → no downtime
- `maxSurge: 1` → bring up new pod before killing old one

### 🔹 Step 3: Proper Readiness Probes (Critical)

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
```

- Traffic is sent only when the pod is ready
- Unready pods never receive traffic

**Readiness probes are the backbone of zero downtime.**

### 🔹 Step 4: Graceful Shutdown Handling

- Handle `SIGTERM` properly
- Add:

```yaml
terminationGracePeriodSeconds: 30
```

- Let:
    - In-flight requests finish
    - Load balancer drain connections

### 🔹 Step 5: Use Pod Disruption Budgets (PDBs)

```yaml
minAvailable: 1
```

- Prevents voluntary disruptions from killing all pods
- Protects during node drains or cluster upgrades

### 🔹 Step 6: Validate During Rollout

```bash
kubectl rollout status deployment <name>
```

**Monitor:**

- Error rates
- Latency
- Pod restarts

### 🔹 Step 7: Rollback If Needed

```bash
kubectl rollout undo deployment <name>
```

**Rollback should be:**

- Fast
- Automatic
- Safe

---

# 🔐 Security & Compliance

## 11. Secrets are accidentally committed to Git. What immediate and long-term steps do you take?

### 🚨 Immediate Actions (First Minutes)

**1️⃣ Assume the Secret Is Compromised**

- Treat it as already leaked
- Do not wait to see if it was abused

**Deleting the commit alone is never enough.**

**2️⃣ Revoke & Rotate the Secret Immediately**

- Rotate:
    - API keys
    - DB passwords
    - Tokens
- Invalidate:
    - Sessions
    - Certificates (if applicable)
- Use:
    - AWS Secrets Manager / Vault / IAM

**3️⃣ Stop Further Exposure**

- Remove secrets from:
    - Repo
    - CI logs
    - Artifacts
- Make repo private if needed
- Notify security team

### 🔍 Short-Term Cleanup

**4️⃣ Remove Secrets From Git History**

- Use:
    - `git filter-repo`
    - BFG Repo-Cleaner
- Force-push cleaned history
- Invalidate forks if possible

⚠️ **Still assume the secret was copied.**

**5️⃣ Audit for Abuse**

- Check:
    - CloudTrail
    - Access logs
    - DB logs
- Look for:
    - Unauthorized access
    - Suspicious API calls

### 🛡️ Long-Term Prevention

**6️⃣ Move Secrets Out of Code**

- Use:
    - AWS Secrets Manager / Parameter Store
    - HashiCorp Vault
    - Kubernetes Secrets
- Inject secrets at runtime

**7️⃣ Add Guardrails**

- Pre-commit hooks:
    - `git-secrets`
    - `detect-secrets`
- CI secret scanning:
    - GitHub secret scanning
    - TruffleHog
- Block commits with secrets automatically

**8️⃣ Improve Access Control**

- Use:
    - IAM roles instead of static credentials
- Enforce:
    - Least privilege
    - Short-lived credentials

**9️⃣ Educate & Document**

- Update:
    - Secure coding guidelines
    - Incident playbooks
- Run a blameless postmortem

---

## 12. How do you secure a CI/CD pipeline?

### 🧠 Core Principle

**The CI/CD pipeline is production's front door — compromise it and everything downstream is compromised.**

### 🔹 1. Secure Source Control

- Protect branches:
    - Require PR reviews
    - Enforce status checks
- Restrict who can:
    - Push to main
    - Modify pipeline configs
- Enable:
    - Signed commits
    - MFA for all users

### 🔹 2. Secrets Management (Critical)

- Never hardcode secrets
- Store secrets in:
    - AWS Secrets Manager / Vault
    - CI secret stores
- Use:
    - Short-lived credentials
    - IAM roles / OIDC (no static keys)
- Mask secrets in logs

### 🔹 3. Secure the CI Runners

- Use:
    - Ephemeral runners (preferred)
    - Isolated build environments
- Restrict:
    - Network access
    - Privileged containers
- Patch runners regularly

### 🔹 4. Dependency & Code Security (Shift Left)

- SAST (static analysis)
- Dependency scanning (CVEs)
- Secrets scanning
- Fail builds on high-severity issues

### 🔹 5. Secure Build Artifacts

- Build once, deploy many times
- Sign artifacts:
    - Container image signing (cosign)
- Store in private registries
- Prevent artifact overwrites

### 🔹 6. Deployment Security

- Use:
    - Least-privilege service accounts
    - Environment-specific roles
- Require:
    - Manual approvals for prod
- Separate:
    - Build vs deploy permissions

### 🔹 7. Infrastructure as Code Security

- Scan IaC:
    - Terraform / CloudFormation checks
- Enforce policies:
    - OPA / Sentinel
- Prevent insecure infra from being deployed

### 🔹 8. Observability & Auditing

- Log:
    - Pipeline executions
    - Access changes
- Enable:
    - CloudTrail / audit logs
- Monitor:
    - Unusual pipeline behavior

### 🔹 9. Protect the Supply Chain

- Pin dependencies
- Verify checksums
- Use trusted base images
- Prevent dependency confusion attacks

---

## 13. A vulnerability is found in a production container image. What do you do?

### 🔹 Step 1: Assess the Risk Immediately

- Check:
    - Severity of the vulnerability (CVSS score, exploitability)
    - Whether it affects your application functionality
- Identify:
    - Which environments and containers are affected
    - Whether the container is exposed to the public

**Assume the worst-case scenario until confirmed otherwise.**

### 🔹 Step 2: Mitigate Exposure Quickly

- **If critical:**
    - Remove the vulnerable container from public access or routing
    - Disable affected services temporarily if needed
- **For less critical:**
    - Monitor closely
    - Apply temporary firewall, WAF, or network isolation

### 🔹 Step 3: Patch & Rebuild

- Pull the latest base image with the security fix
- Rebuild the container from scratch
- Run all CI/CD tests to ensure the patch does not break functionality
- Push the new image to your private registry

### 🔹 Step 4: Redeploy Safely

- Use zero-downtime deployment strategies:
    - Rolling update
    - Blue-Green
    - Canary
- Ensure:
    - Health checks pass
    - Monitoring dashboards show stable performance

### 🔹 Step 5: Remove Vulnerable Images

- Delete the old image from:
    - Registries
    - Build caches
- Prevent accidental redeployments

### 🔹 Step 6: Verify & Monitor

- Monitor:
    - Error rates
    - Security alerts
    - Traffic anomalies
- Run vulnerability scanning on deployed images to confirm the fix

### 🔹 Step 7: Implement Long-Term Prevention

- Enable image scanning in CI/CD (Snyk, Trivy, Clair, Anchore)
- Pin base images to known versions
- Automate rebuilds on base image updates
- Adopt a supply chain security approach (SLSA / SBOM)

---

# 📊 Monitoring, Logging & Incident Response

## 14. Users report intermittent latency, but monitoring shows everything "green." What next?

### 🔹 Step 1: Confirm the Issue

- Collect as much detail from users as possible:
    - Time of latency spikes
    - Which endpoints / services are affected
    - Geography / device patterns
- Try to reproduce the latency yourself

**Sometimes users' perception differs from system dashboards.**

### 🔹 Step 2: Check Detailed Metrics

- "Green" in high-level dashboards may hide spikes. Check:
    - Percentiles (p95/p99), not just averages
    - Latency by endpoint / service / pod / node
    - Traffic volume / request bursts
    - Database query times and queue lengths

**High tail latency often hides in averages.**

### 🔹 Step 3: Inspect Logs & Traces

- Enable distributed tracing (X-Ray, Jaeger, OpenTelemetry)
- Look for:
    - Slow requests
    - Timeouts waiting on dependencies
    - Retries or failed connections
- Check application logs for errors or warnings not captured in metrics

### 🔹 Step 4: Check External Dependencies

- DB, cache, third-party APIs, or message queues
- Look for intermittent slow queries or throttling
- Sometimes the app itself is fine; latency is upstream

### 🔹 Step 5: Network & Infrastructure

- Inspect:
    - VPC / subnet / firewall issues
    - Load balancer spikes / connection limits
    - DNS resolution latency
    - Pod-to-pod network latency in Kubernetes

**Network blips often cause intermittent issues while metrics look green on average.**

### 🔹 Step 6: Resource Contention

- Check node-level CPU/memory spikes or throttling
- Check JVM / GC pauses if running Java apps
- Container CPU/memory limits causing throttling

### 🔹 Step 7: Deploy Instrumentation (If Needed)

- Add more granular metrics:
    - Endpoint-level timing
    - DB query duration histograms
    - Request queue depth
- This helps capture tail latency spikes

### 🔹 Step 8: Communicate & Mitigate

- Inform users / stakeholders about investigation
- If possible, implement temporary mitigation:
    - Enable caching
    - Reduce non-critical features
    - Retry/backoff optimization

---

## 15. A critical service goes down at 2 AM. Walk me through your response.

### 🧰 Phase 1: Immediate Response (First 0–5 minutes)

**1️⃣ Acknowledge & Triage**

- Respond immediately to the alert (PagerDuty / Opsgenie / Slack)
- Identify:
    - Which service is down
    - Impact scope (number of users, criticality)
- Communicate:

**"Incident acknowledged — investigating."**

**2️⃣ Mitigate Customer Impact**

- Route traffic away if possible:
    - Use load balancer health checks
    - Disable affected endpoints
- Enable fallbacks or feature flags
- Goal: minimize user impact while investigation proceeds

### 🧪 Phase 2: Investigate the Root Cause (5–20 minutes)

**3️⃣ Check System & Service Health**

- `kubectl get pods` / `kubectl describe` for K8s
- Check auto-scaling, node availability
- Verify load balancers, DNS, network connectivity

**4️⃣ Examine Logs & Metrics**

- CloudWatch / Prometheus / Grafana dashboards
- Application logs for errors
- Metrics for CPU, memory, disk, request latency, queue depth

**5️⃣ Identify Recent Changes**

- Deployment? Config change? Infra update?
- Rollback candidate if new release caused failure

### ⚡ Phase 3: Fix or Mitigate (20–60 minutes)

**6️⃣ Quick Mitigation**

- Redeploy last stable version
- Restart failing pods / services
- Remove misbehaving nodes or containers
- Activate backup / failover systems if available

**7️⃣ Confirm Recovery**

- Ensure the service is healthy (`kubectl rollout status`, metrics normalized)
- Verify users can access functionality

### 📝 Phase 4: Post-Incident Actions

**8️⃣ Root Cause Analysis (RCA)**

- Determine underlying cause:
    - Bug, configuration error, infrastructure failure, external dependency
- Document timeline and decisions
- Include metrics and logs for reference

**9️⃣ Prevent Recurrence**

- Adjust monitoring & alerts (p95 latency, error spikes)
- Fix deployment pipeline or IaC issues
- Update runbooks

---

## 16. How do you decide what metrics to monitor for a new service?

### 🧠 Core Principle

**Monitor what matters to reliability, performance, and business outcomes.**

### 🔹 Step 1: Understand the Service

- Ask:
    - What does the service do? (business context)
    - What are critical paths / endpoints?
    - Which dependencies matter (DB, cache, external APIs)?

**Example:** A payment service → success rate, latency, transaction volume.

### 🔹 Step 2: Monitor Infrastructure Health

- CPU / memory / disk / network of nodes or containers
- Pod / instance status (Kubernetes pod restarts, node health)
- Autoscaling metrics (desired vs actual replicas)

**Ensures the service has enough resources.**

### 🔹 Step 3: Monitor Application Health

- Request rate / throughput
- Latency (p50, p95, p99)
- Error rates (4xx / 5xx, exceptions, failed jobs)
- Queue depth (for async processing)

**Captures internal performance and reliability.**

### 🔹 Step 4: Monitor External Dependencies

- **Database:**
    - Connection pool usage, query latency, replication lag
- **Caches:**
    - Hit ratio, eviction rate
- **External APIs:**
    - Success rate, latency, rate limits

**Slow dependencies often cause service degradation.**

### 🔹 Step 5: Monitor Business Metrics (Optional but Valuable)

- Number of transactions per minute
- Orders processed
- Signups / logins / feature usage

**Detect issues invisible in system metrics.**

### 🔹 Step 6: Define Alerts Carefully

- Avoid alert fatigue
- Alert thresholds for:
    - High error rate
    - Latency spikes
    - Resource exhaustion
- Use multi-metric correlation for smarter alerts

### 🔹 Step 7: Review and Iterate

- After initial deployment:
    - Add/remove metrics based on real-world behavior
    - Monitor tail latency, rare errors, saturation points
- Continuous improvement → dashboards evolve with service

---

# 📦 Infrastructure as Code (IaC)

## 17. A Terraform apply fails in production. What's your rollback plan?

### 🧠 Core Principle

**Always assume Terraform can fail. Have a rollback plan and minimize blast radius.**

### 🔹 Step 1: Pause & Assess

- Do not panic — stop further changes.
- Check:
    - Error message (`terraform apply` output)
    - Which resources failed / partially applied
    - State file status (local or remote backend)

**Understanding what went wrong before acting is critical.**

### 🔹 Step 2: Identify the Impacted Resources

- Use:

```bash
terraform plan
terraform state list
```

- Determine:
    - Resources fully created
    - Resources partially modified
    - Dependencies that could break

**Partial applies are the biggest risk in production.**

### 🔹 Step 3: Rollback Options

**Option A: Terraform Rollback**

- Terraform doesn't have a native "undo", but you can:
    1. Restore previous state file from remote backend (S3 / Terraform Cloud)
    2. Run:

```bash
terraform apply
```

→ This will revert infrastructure to the last known good state

**Option B: Manual Rollback**

- For critical resources (databases, networking):
    - Revert configuration manually
    - Use snapshots/backups if needed
    - Apply changes incrementally

**Option C: Blue-Green / Immutable Infrastructure**

- If using:
    - EC2 / ECS / Kubernetes / Terraform modules
- Switch traffic back to previous environment
- Destroy failed resources after verification

### 🔹 Step 4: Verify Recovery

- Validate:
    - Services are up and healthy
    - Configurations match desired state
    - No downtime for users (if applicable)

### 🔹 Step 5: Post-Incident

- Analyze:
    - Why `terraform apply` failed (syntax, permissions, resource limits, API errors)
    - Why rollback was needed
- Prevent recurrence:
    - Use Terraform plan + approval in production
    - Enable state versioning / locking
    - Use smoke tests before applying critical changes

---

## 18. Multiple teams modify the same infrastructure. How do you prevent conflicts?

### 🧠 Core Principle

**Infrastructure should be versioned, collaborative, and conflict-free — like code.**

### 🔹 Step 1: Use Remote State with Locking

- Store state in remote backends:
    - S3 + DynamoDB (locking)
    - Terraform Cloud / Enterprise
- Enable state locking to prevent multiple applies at the same time

**Prevents concurrent modifications that break the state.**

### 🔹 Step 2: Modularize Infrastructure

- Split infra into independent modules:
    - Networking, DB, compute, monitoring
- Assign ownership per module to different teams
- Avoid multiple teams modifying the same module simultaneously

**Smaller modules = smaller blast radius.**

### 🔹 Step 3: Version Control & Branching

- All changes go through Git:
    - Feature branches
    - Pull requests / merge requests
- Require:
    - Code review
    - Terraform plan review before merge
- CI validates plan against remote state

**Code + plan review catches conflicts before apply.**

### 🔹 Step 4: Apply Approval Workflows

- Use manual approvals for production:
    - Terraform Cloud / Enterprise workspaces
    - GitHub Actions + protected branches
- Prevent accidental direct `terraform apply`

### 🔹 Step 5: Use Workspaces or Environments

- Separate workspaces for:
    - Dev / QA / Staging / Prod
- Teams can test changes without affecting others

### 🔹 Step 6: Naming Conventions & Scoping

- Consistent resource naming
- Avoid hardcoded values
- Scope resources by team or project to prevent accidental overlap

### 🔹 Step 7: Monitor & Audit Changes

- Enable logging and audit trails:
    - Who applied what and when
    - State changes over time
- Helps quickly resolve conflicts if they occur

---

# 🤝 Collaboration & Process

## 19. Developers want faster releases, ops wants stability. How do you balance both?

### 🧠 Core Principle

**Balance speed and stability by automating safe releases and building confidence in the system.**

### 🔹 Step 1: Implement CI/CD Pipelines

- Developers push code frequently
- Pipelines automatically:
    - Build
    - Run unit tests
    - Run integration tests
- Artifacts are immutable and ready for deployment

**Fast feedback catches issues before reaching production.**

### 🔹 Step 2: Use Progressive Deployment Strategies

- **Canary deployments:** release to a small subset of users first
- **Blue-Green deployments:** switch traffic only when the new version is ready
- **Feature flags:** release features safely without full rollout

**Reduces risk while allowing faster releases.**

### 🔹 Step 3: Automate Testing and Validation

- Automated tests:
    - Unit, integration, contract, performance
- Pre-prod environments:
    - Mirror production for realistic testing
- Static analysis / security scanning
- Ensures stability doesn't block speed

### 🔹 Step 4: Monitor & Rollback Quickly

- Observability in production:
    - Prometheus/Grafana metrics
    - Logs / distributed tracing
- Set up alerts and automated rollback triggers
- Confidence that if something goes wrong, it can be reverted immediately

### 🔹 Step 5: Collaboration & Culture

- Shared metrics for both teams:
    - Deployment frequency
    - Change failure rate
    - Mean time to recovery (MTTR)
- Ops provides guardrails; developers innovate within safe boundaries

**Transparency and feedback loops align goals.**

### 🔹 Step 6: Small, Incremental Changes

- Smaller, frequent releases are safer than large, infrequent ones
- Easier to debug, easier to rollback
- Keeps velocity high without compromising stability

---

## 20. How do you introduce DevOps practices in a legacy organization?

### 🧠 Core Principle

**DevOps is as much cultural as it is technical — you can't just install tools.**

### 🔹 Step 1: Assess the Current State

- Understand:
    - How teams build, test, deploy software
    - Pain points in development, operations, and QA
    - Existing tooling and workflows
- Identify low-hanging fruit for early wins

**Baseline knowledge helps prioritize initiatives and get buy-in.**

### 🔹 Step 2: Start with Culture & Collaboration

- Break down silos:
    - Devs, Ops, QA working together
- Introduce shared responsibility for:
    - Deployment success
    - Application reliability
- Encourage blameless postmortems

**Culture is the hardest but most important part.**

### 🔹 Step 3: Automate the Basics

- Implement CI pipelines first:
    - Version control, automated builds, unit tests
- Gradually add:
    - Integration tests
    - Artifact repositories
    - Automated deployments to staging

**Early automation builds confidence and reduces friction.**

### 🔹 Step 4: Introduce Incremental Deployment Practices

- Start small:
    - Feature flags
    - Canary or blue-green deployments
- Target non-critical services first
- Measure impact and reliability before scaling to production

### 🔹 Step 5: Monitoring & Observability

- Implement metrics, logging, and alerts for early wins:
    - Prometheus, Grafana, ELK stack, CloudWatch, etc.
- Teams can see the impact of changes and identify bottlenecks

**Visibility drives trust in DevOps processes.**

### 🔹 Step 6: Measure Success & Iterate

- Track KPIs:
    - Deployment frequency
    - Change failure rate
    - MTTR (mean time to recovery)
- Celebrate wins publicly
- Iterate on pipelines, automation, and processes

### 🔹 Step 7: Expand Gradually

- After early wins:
    - Introduce Infrastructure as Code (Terraform, Ansible)
    - Implement secrets management
    - Apply security scanning / DevSecOps practices

**Don't try to change everything at once — DevOps is evolutionary, not revolutionary.**
