# AWS DevOps Interview Questions & Answers

---

## EC2 (Elastic Compute Cloud)

## What is EC2?
EC2 is a virtual server in the cloud. Instead of buying a physical computer, you rent a server from AWS and use it to run your applications.

**1. Purchasing options — On-Demand, Reserved, Spot, Savings Plans, Dedicated Hosts?**
- On-Demand: pay by the hour/second, no commitment — good for unpredictable workloads.
- Reserved/Savings Plans: commit for 1-3 years, get big discounts — good for steady, predictable workloads.
- Spot: use AWS's spare capacity at up to 90% discount, but AWS can take it back with short notice — good for batch jobs, CI runners, anything that can restart.
- Dedicated Hosts: a physical server just for you — used when licensing or compliance requires it.

## 2. Spot vs On-Demand, and how do you handle interruptions?
Spot is cheap leftover capacity; On-Demand is guaranteed but full price. Spot instances can be reclaimed by AWS with a 2-minute warning. You handle this by designing stateless/restartable jobs, saving progress often, and using Spot Fleets or ASGs that mix Spot with On-Demand as a fallback.

## 3. Instance store vs EBS — what happens on stop/terminate?
Instance store is temporary disk space physically attached to the server — data is lost if you stop or terminate the instance. EBS is a separate network drive that stays even if you stop the instance (only lost on terminate, unless you set "delete on termination" to false).

## 4. EBS volume types — gp3, gp2, io1/io2, st1, sc1?
- gp3/gp2: general-purpose SSD, good for most workloads (boot volumes, small-medium databases).
- io1/io2: high-performance SSD for heavy databases needing very fast, consistent speed.
- st1: cheap spinning disk for big sequential data like logs.
- sc1: cheapest, slowest, for rarely-accessed data (cold storage).

## 5. AMI vs Snapshot?
A Snapshot is a backup of a single EBS volume (just the disk). An AMI (Amazon Machine Image) is a full template to launch a new EC2 instance — it includes the OS, settings, and often references a snapshot.

## 6. User data vs Instance metadata?
User data is a script you give an instance to run once when it first boots (e.g., install software). Instance metadata is information the instance can look up about itself while running (like its IP, instance ID, IAM role) by calling a special internal address.

## 7. IMDSv1 vs IMDSv2?
IMDSv1 lets anything on the instance simply ask for metadata with a GET request — risky if there's a web app bug that lets attackers make requests on your behalf (SSRF attack), because they could steal IAM credentials. IMDSv2 requires you to first get a secret session token before you can read metadata, closing that hole. Always prefer IMDSv2.

## 8. Troubleshooting a failed status check?
EC2 has two status checks: "system status" (AWS's hardware/network problem — usually fixed by stopping/starting the instance to move it to new hardware) and "instance status" (something wrong inside your instance, like OS/kernel/config issues — checked via console logs or by attaching the volume to another instance to inspect it).

## 9. Security Groups vs NACLs?
Security Groups work at the instance level and are stateful — if you allow traffic in, the reply is automatically allowed out. NACLs work at the subnet level and are stateless — you must explicitly allow both inbound and outbound traffic.

## 10. Connecting to a private EC2 without exposing SSH?
Use a Bastion Host (a small public "jump" server you SSH into first) or, better, AWS Systems Manager Session Manager, which lets you get a shell on the instance through the AWS Console/CLI with no open SSH port and no need for a public IP at all.

## 11. Automating patching across a fleet?
Use AWS Systems Manager Patch Manager — it scans instances, applies OS patches on a schedule, and reports compliance, without you logging into each machine.

## 12. Placement groups?
- Cluster: packs instances physically close together for the fastest network speed (good for HPC).
- Spread: keeps instances on separate hardware to reduce the chance they fail together.
- Partition: groups instances into partitions on different hardware, used for big distributed systems like Kafka/Hadoop.

---

## IAM (Identity and Access Management)

## What is IAM?
IAM controls who (or what) can do what inside your AWS account. It's how you manage logins and permissions.

## 1. User vs Group vs Role vs Policy?
- User: an identity for a real person or app, with permanent credentials.
- Group: a collection of users who share the same permissions.
- Role: a set of temporary permissions that anything (a user, an AWS service, or an external identity) can "put on" temporarily — no long-term password/keys.
- Policy: a JSON document that actually lists what's allowed or denied — attached to users, groups, or roles.

## 2. Identity-based vs Resource-based policy?
An identity-based policy is attached to a user/group/role and says "this identity can do X." A resource-based policy is attached directly to a resource (like an S3 bucket) and says "these identities can do X to me." S3 bucket policies are a common example of resource-based.

## 3. Policy evaluation — how does Deny beat Allow?
AWS checks all applicable policies. If even one policy says explicit Deny, that wins no matter how many Allows exist elsewhere. If there's no explicit Deny and at least one Allow, the action is allowed. By default, everything is denied unless something explicitly allows it.

## 4. Principle of least privilege?
Only give people/services the exact permissions they need to do their job — nothing extra "just in case." In practice: start with a very narrow policy, look at what actions actually get used (via CloudTrail/Access Analyzer), and expand only when needed.

## 5. How do EC2/ECS/Lambda actually get IAM permissions?
The service internally calls AWS's Security Token Service (STS) to "assume" the role you attached (via an instance profile for EC2, a task role for ECS, an execution role for Lambda), which hands it short-lived, auto-rotating temporary credentials — you never store real keys on the machine.

## 6. Permissions boundary vs SCP?
A permissions boundary limits the maximum permissions one specific IAM user/role can ever have, set by an account admin, even if their attached policy tries to grant more. An SCP (Service Control Policy) does something similar but at the AWS Organizations level, applying a ceiling across whole accounts or OUs (organizational units).

## 7. Cross-account access setup?
In Account A, create a role that trusts Account B (a trust policy naming Account B's ID). Give that role the permissions needed. Then users/roles in Account B can call `AssumeRole` to get temporary credentials into Account A — no need to create duplicate users.

## 8. IAM Identity Center vs managing users per account?
IAM Identity Center (formerly AWS SSO) is a central place to manage user logins once and give them access across many AWS accounts, instead of creating separate IAM users in every single account — much easier at scale.

## 9. Rotating access keys / why avoid long-lived keys?
Long-lived keys can leak (in code, logs, GitHub) and stay valid forever if not caught. Best practice: avoid access keys altogether by using IAM roles wherever possible; if keys must exist, rotate them regularly and use tools like AWS Secrets Manager or short-lived STS tokens instead.

## 10. Debugging "Access Denied"?
Use the IAM Policy Simulator to test what a policy allows before running it for real, and check CloudTrail logs to see exactly which action was denied and which policy caused it.

## 11. Trust policy vs permissions policy?
The trust policy on a role defines *who is allowed to assume* the role (e.g., "EC2 service" or "Account B"). The permissions policy defines *what that role can do* once assumed. They answer two different questions: "who can wear this hat" vs "what can you do while wearing it."

---

## VPC (Virtual Private Cloud)

## What is a VPC?
A VPC is your own private, isolated network inside AWS — like your own mini data center — where you control IP ranges, subnets, and routing.

## 1. VPC components — subnets, route tables, IGW, NAT, ENIs?
- Subnet: a smaller slice of the VPC's IP range, tied to one Availability Zone.
- Route table: rules that decide where network traffic is sent.
- Internet Gateway (IGW): lets resources in the VPC reach the public internet.
- NAT Gateway: lets private resources reach the internet (for updates etc.) without being reachable *from* the internet.
- ENI (Elastic Network Interface): a virtual network card attached to an instance.

## 2. Public subnet vs private subnet?
A subnet is "public" simply because its route table sends internet-bound traffic (0.0.0.0/0) to an Internet Gateway. A private subnet has no such route — it can't be reached directly from, or reach out directly to, the internet.

## 3. NAT Gateway vs NAT Instance?
NAT Gateway is a fully managed AWS service — no maintenance, scales automatically, more expensive. NAT Instance is just a regular EC2 instance configured to do NAT — cheaper but you must manage, patch, and scale it yourself.

## 4. Designing a multi-AZ, multi-tier VPC?
Split resources into tiers: public subnets for load balancers, private subnets for app servers, and separate private subnets for databases — each tier duplicated across at least 2 Availability Zones for high availability, with route tables and security groups controlling traffic between tiers.

## 5. VPC Peering and its limits?
VPC Peering directly connects two VPCs so they can talk using private IPs, like a cable between them. Limitation: it's not transitive — if A is peered with B, and B is peered with C, A cannot automatically talk to C. Overlapping IP ranges between the two VPCs also break peering.

## 6. VPC Peering vs Transit Gateway?
Peering works fine for a few VPCs but gets messy at scale (you'd need a peering connection between every pair). Transit Gateway acts like a central hub/router that all VPCs connect to once, making it much easier to manage many VPCs (and even on-prem networks) together.

## 7. Security Groups vs NACLs (recap)?
Security Groups: instance-level, stateful (return traffic auto-allowed). NACLs: subnet-level, stateless (must allow both directions explicitly), and they're evaluated in rule-number order.

## 8. VPC Endpoints (Gateway vs Interface)?
A VPC Endpoint lets resources inside your VPC talk privately to AWS services (like S3) without going over the public internet or through a NAT Gateway.
- Gateway endpoint: used for S3 and DynamoDB only, added as a route table entry, free.
- Interface endpoint: used for most other AWS services, creates a private IP (ENI) inside your subnet, small hourly cost.

## 9. DNS resolution inside a VPC?
AWS provides a built-in DNS resolver inside every VPC. `enableDnsSupport` turns this resolver on, and `enableDnsHostnames` lets instances get automatic public/private DNS names. Route 53 Resolver extends this to also handle hybrid DNS between your VPC and on-prem networks.

## 10. Troubleshooting connectivity between two instances?
Check, in order: are they in the same/peered VPC and can routes reach each other; does the route table have a path; does the Security Group allow the traffic; does the NACL allow it both ways; and finally check VPC Flow Logs to see if traffic is actually being dropped and where.

## 11. CIDR planning?
This means carefully choosing non-overlapping IP address ranges for every VPC/subnet in advance, especially before you have multiple accounts, so you can later peer/connect them or use Transit Gateway without IP conflicts, and so you don't run out of IPs as you grow.

## 12. VPC Flow Logs?
A Flow Log records metadata about the traffic going in and out of network interfaces (source/destination IP, port, accepted/rejected). It's the main tool to diagnose "why is my traffic being silently dropped" issues.

---

## S3 (Simple Storage Service)

## What is S3?
S3 is object storage in the cloud — a place to store files (objects) like images, backups, and logs, organized into "buckets," accessible over the internet or privately.

## 1. Storage classes and lifecycle policies?
Storage classes range from Standard (frequently accessed, most expensive) to Infrequent Access, Glacier, and Glacier Deep Archive (rarely accessed, cheapest, slower to retrieve). A lifecycle policy automatically moves objects between these classes (or deletes them) after a set number of days, to save money without manual work.

## 2. Durability vs Availability?
Durability (11 nines) means your data almost never gets lost or corrupted — S3 automatically stores copies across multiple facilities. Availability means how often the data is actually reachable/usable at any given moment — slightly lower than durability, and varies by storage class.

## 3. Securing an S3 bucket?
Layer several controls: Block Public Access (a simple on/off switch to prevent accidental public exposure), bucket policies (resource-based rules), IAM policies (identity-based rules), and encryption. ACLs are old and mostly discouraged now in favor of policies.

## 4. Bucket policy vs IAM policy for S3?
A bucket policy is attached to the bucket itself and controls who can access it (including people outside your account). An IAM policy is attached to a user/role and controls what that specific identity can do across AWS, including S3. Both are checked together.

## 5. S3 Versioning and protecting against deletes?
Versioning keeps every version of an object instead of overwriting it, so an accidental overwrite or delete can be undone. MFA Delete adds an extra requirement (a physical MFA code) before anyone can permanently delete a version — extra protection against accidents or malicious deletes.

## 6. S3 Event Notifications?
S3 can automatically notify other services (Lambda, SQS, SNS) whenever something happens to an object (like a new upload) — commonly used to trigger a Lambda function that processes a file the moment it's uploaded.

## 7. Encryption options in S3?
- SSE-S3: AWS manages the encryption keys entirely.
- SSE-KMS: AWS KMS manages the keys, giving you more control/auditing (who used the key, when).
- SSE-C: you provide your own encryption key with every request.
- Client-side: you encrypt the data yourself before it even reaches S3.

## 8. Cross-Region Replication (CRR)?
Automatically copies objects from a bucket in one region to a bucket in another region — used for disaster recovery, compliance, or lower latency for users in other regions. Requires versioning to be enabled on both buckets.

## 9. Static website hosting + CloudFront?
S3 can serve HTML/CSS/JS files directly as a website. Putting CloudFront (a CDN) in front of it caches content closer to users worldwide, adds HTTPS, and can restrict direct access to the bucket (using Origin Access Control) so people must go through CloudFront.

## 10. S3 Object Lock?
Makes objects "Write Once Read Many" (WORM) — once locked, nobody (not even the account root user) can delete or modify them until the lock period expires. Used for compliance requirements like financial records retention.

## 11. Multipart uploads?
For large files, S3 lets you split the upload into smaller parts sent in parallel, then reassembles them — faster, more reliable, and if one part fails you only re-upload that part instead of the whole file.

---

## ECR (Elastic Container Registry)

## What is ECR?
ECR is AWS's private storage service for Docker container images — like a private version of Docker Hub, tightly integrated with IAM and other AWS services.

## 1. ECR vs Docker Hub?
Docker Hub is a public, general-purpose registry anyone can use. ECR is private by default, secured through IAM, lives inside your AWS account/region, and integrates natively with ECS/EKS — better for production AWS workloads.

## 2. Authenticating Docker to ECR?
You run `aws ecr get-login-password` to get a temporary token, then pipe it into `docker login` — this proves to ECR that you're allowed in, using your IAM identity instead of a separate username/password.

## 3. Image scanning?
ECR can scan images for known vulnerabilities. Basic scanning checks against a standard vulnerability database on push. Enhanced scanning (powered by Amazon Inspector) does continuous, deeper scanning of both OS and application-level packages.

## 4. Lifecycle policies in ECR?
Rules that automatically clean up old/unused images (e.g., "keep only the last 10 images" or "delete untagged images after 14 days") so your registry doesn't fill up with junk and cost more.

## 5. Replicating images across regions/accounts?
ECR supports cross-region and cross-account replication rules, so an image pushed once automatically copies to other regions/accounts — useful for disaster recovery or multi-region deployments.

## 6. Restricting access with repository policies?
Like an S3 bucket policy, but for a container repo — you can allow specific IAM users, roles, or even other AWS accounts to pull or push images, without giving broader account access.

## 7. Immutable vs mutable tags?
Mutable tags let you push a new image using the same tag (e.g., "latest") — risky because you can't be sure what's actually running. Immutable tags block overwriting a tag once used, so every tag always points to exactly one specific image — safer for production traceability.

---

## ECS (Elastic Container Service)

## What is ECS?
ECS is AWS's own container orchestration service — it runs and manages your Docker containers for you (starts them, restarts them if they crash, scales them).

## 1. ECS vs EKS?
ECS is AWS's simpler, proprietary container orchestrator — easier to learn, tightly integrated with AWS. EKS is managed Kubernetes — more powerful and portable (can run the same way on other clouds), but has a steeper learning curve. Choose ECS for simplicity within AWS; choose EKS if you need Kubernetes features/portability or your team already knows Kubernetes.

## 2. Fargate vs EC2 launch type?
EC2 launch type: you manage the underlying servers your containers run on (more control, can be cheaper at scale, but you patch/scale the servers). Fargate: serverless — AWS runs the containers without you managing any servers at all (simpler, but usually costs more per container).

## 3. Task Definition vs Service vs Cluster?
- Task Definition: the blueprint (which container image, CPU/memory, ports) — like a recipe.
- Service: keeps a desired number of tasks running continuously, replacing any that die, and connects them to a load balancer.
- Cluster: the logical group of resources (EC2 or Fargate capacity) where tasks/services actually run.

## 4. ECS Service Auto Scaling?
ECS can automatically add or remove running tasks based on a metric — most commonly target tracking, like "keep average CPU around 50%" — AWS handles the scaling math itself.

## 5. Rolling / blue-green deployments in ECS?
Rolling deployment: ECS gradually replaces old tasks with new ones. Blue-green (via CodeDeploy): a whole new set of tasks (green) is spun up alongside the old ones (blue), tested, and traffic is switched over all at once — safer, with instant rollback if something looks wrong. The deployment circuit breaker can automatically roll back a failing deployment.

## 6. Task role vs Task execution role?
The task execution role is used by ECS itself to do setup work, like pulling the image from ECR and writing logs to CloudWatch. The task role is used by your actual application code inside the container to call other AWS services (like reading from S3). Different jobs, different roles.

## 7. Service discovery in ECS?
Lets containers find each other by name instead of hardcoded IPs. AWS Cloud Map gives each service a DNS name that automatically updates as tasks come and go; an ALB with target groups is used more for external traffic routing.

## 8. Passing secrets securely?
Instead of hardcoding passwords in the task definition, you reference a secret stored in AWS Secrets Manager or SSM Parameter Store — ECS fetches it securely at container startup and injects it as an environment variable.

## 9. Troubleshooting a task that keeps stopping?
Check the "stopped reason" in the ECS console/CLI (often shows the exit code or an error like "out of memory"), then check CloudWatch Logs for the actual application error output.

---

## EKS (Elastic Kubernetes Service)

## What is EKS?
EKS is AWS's managed Kubernetes service — AWS runs and maintains the Kubernetes "control plane" (the brains) for you, so you can just run your containerized apps as pods without setting up Kubernetes from scratch.

## 1. What does AWS manage vs what you manage?
AWS manages the Kubernetes control plane — the API server, etcd database, scheduler — keeping it highly available and patched. You manage the worker nodes (unless using Fargate) and everything you deploy on top (your pods, deployments, configs).

## 2. How pods get VPC IPs (VPC CNI)?
The Amazon VPC CNI plugin gives each pod a real IP address directly from your VPC's IP range (instead of a separate overlay network), so pods can talk to other AWS resources like normal VPC traffic.

## 3. Managed Node Groups vs self-managed vs Fargate?
- Managed Node Groups: AWS handles provisioning/updating the EC2 worker nodes for you — easiest option.
- Self-managed nodes: you fully control the EC2 instances yourself — more flexibility, more work.
- Fargate profiles: no nodes at all — each pod runs on its own serverless compute, no server management.

## 4. IRSA (IAM Roles for Service Accounts)?
IRSA lets you attach a specific IAM role to just one Kubernetes service account (and therefore just the pods using it), instead of giving every pod on a node the same broad node-level IAM permissions. This is much safer — each app only gets exactly the AWS permissions it needs.

## 5. Cluster/node autoscaling — Cluster Autoscaler vs Karpenter?
Cluster Autoscaler watches for pods that can't be scheduled (not enough capacity) and adds nodes from predefined Auto Scaling Groups; it also removes underused nodes. Karpenter is a newer, faster alternative that provisions exactly the right-sized nodes on demand directly, without needing pre-configured ASGs, often cheaper and quicker to scale.

## 6. Upgrading an EKS cluster with minimal downtime?
Upgrade the control plane first (AWS handles this with no downtime to running pods), then upgrade node groups gradually (rolling replacement of nodes one batch at a time), following Kubernetes's version-skew rules (nodes shouldn't be too many versions behind the control plane).

## 7. Ingress and load balancing (AWS Load Balancer Controller)?
The AWS Load Balancer Controller watches for Kubernetes Ingress or Service objects and automatically creates/configures a real ALB (for Ingress/HTTP) or NLB (for Layer 4 Services) in your VPC to route external traffic into the cluster.

## 8. Managing secrets in EKS?
Plain Kubernetes Secrets are only base64-encoded (not truly encrypted by default) unless you enable encryption at rest. Many teams instead use the Secrets Manager/SSM CSI driver, which mounts secrets from AWS's secret stores directly into pods, keeping the actual secret only in AWS, not sitting in `etcd`.

## 9. Multi-tenancy / namespace isolation?
Use Kubernetes Namespaces to logically separate teams/apps, and Network Policies to control which pods can talk to which other pods (e.g., blocking Namespace A from ever reaching Namespace B's pods) — plus IAM/RBAC to control who can do what in each namespace.

## 10. Monitoring and logging EKS?
CloudWatch Container Insights collects cluster/pod-level metrics automatically. Fluent Bit is commonly used to ship container logs to CloudWatch Logs. Many teams also add Prometheus (metrics) and Grafana (dashboards) for deeper, Kubernetes-native observability.

## 11. StorageClass and EBS CSI driver?
A StorageClass defines what kind of storage a pod can request (e.g., "give me a gp3 EBS volume"). The EBS CSI driver is the plugin that actually talks to AWS to create and attach that real EBS volume when a pod asks for persistent storage.

---

## ELB (Elastic Load Balancing)

## What is a Load Balancer?
A load balancer sits in front of your servers and spreads incoming traffic across them, so no single server gets overwhelmed, and it automatically stops sending traffic to unhealthy servers.

## 1. ALB vs NLB vs CLB?
- ALB (Application Load Balancer): works at Layer 7 (HTTP/HTTPS), understands URLs/paths/hostnames — best for web apps and microservices.
- NLB (Network Load Balancer): works at Layer 4 (raw TCP/UDP), extremely fast, gives static IPs — best for very high performance or non-HTTP traffic.
- CLB (Classic Load Balancer): the old, legacy option — avoid for new projects.

## 2. How ALB routes traffic?
An ALB listens on a port, and listener rules decide where to send each request based on things like the URL path (`/api/*` → one target group) or the hostname — each target group is a pool of servers/containers.

## 3. Health checks?
The load balancer regularly pings each target (e.g., an HTTP GET to `/health`). If a target fails enough checks in a row, it's marked unhealthy and temporarily removed from receiving traffic until it passes again.

## 4. NLB — low latency and static IPs?
Because NLB works at a lower network layer (just forwarding TCP/UDP packets) instead of inspecting the full HTTP request like ALB does, it has much less overhead, giving very low latency and letting each AZ get a fixed static/Elastic IP.

## 5. SSL/TLS termination?
Instead of every backend server managing its own HTTPS certificate, the load balancer "terminates" (decrypts) the HTTPS connection itself, using a certificate from AWS Certificate Manager (ACM), then optionally talks plain HTTP to the backend servers inside the private network.

## 6. Cross-zone load balancing?
Without it, each AZ's load balancer node only sends traffic to targets in its own AZ, which can cause uneven load if one AZ has fewer targets. With cross-zone load balancing on, traffic is spread evenly across targets in all AZs, not just the local one.

## 7. Sticky sessions?
Normally the load balancer can send each request to any server. Sticky sessions (session affinity) use a cookie to make sure the same user keeps landing on the same backend server for their whole session — useful if that server is holding session data in memory.

## 8. Debugging 502/503 errors?
502 usually means the backend gave a bad/invalid response (often the app itself crashed). 503 usually means no healthy targets are available. Check: are targets passing health checks, are security groups allowing the load balancer to reach the targets, and is the deregistration delay too short/long during deployments.

---

## Auto Scaling

## What is Auto Scaling?
Auto Scaling automatically adds or removes EC2 instances based on demand, so you have enough capacity during traffic spikes and aren't paying for idle servers when it's quiet.

## 1. Auto Scaling Group (ASG) core components?
- Launch Template: the blueprint for new instances (AMI, instance type, security groups).
- Min/Max/Desired: the minimum and maximum instance count allowed, and the number ASG currently tries to keep running.
- Scaling policies: the rules that decide when to add/remove instances.

## 2. Types of scaling policies?
- Target tracking: "keep average CPU at 50%" — AWS automatically adjusts, simplest and most common.
- Step scaling: different scaling amounts depending on how far a metric is from the threshold (bigger breach = bigger scale-out).
- Simple scaling: one fixed action per alarm, older/less flexible.
- Scheduled scaling: scale up/down at specific known times (e.g., before a daily traffic spike).

## 3. ASG + ALB target group interaction?
The ASG automatically registers new instances with the ALB's target group so they start receiving traffic, and when an instance is being removed, it's first deregistered and drained (given time to finish existing requests) before it's terminated.

## 4. Launch Template vs Launch Configuration?
Launch Templates are the newer, more flexible option (support versioning, more instance features, Spot integration). Launch Configurations are the old way, can't be edited or updated once created, and AWS recommends using Launch Templates instead.

## 5. Handling stateful workloads?
Warm pools keep pre-initialized (but stopped/hibernated) instances ready to launch quickly instead of starting from scratch, reducing scale-out time. Lifecycle hooks let you pause an instance in a "waiting" state before it's terminated, so you can run cleanup scripts (like finishing a job or deregistering from a service) gracefully.

## 6. Lifecycle hooks — real use case?
Example: before terminating an instance, a lifecycle hook can pause the termination and trigger a script that gracefully finishes any in-progress work or uploads final logs, before finally letting the instance actually shut down.

## 7. Zero-downtime rolling deployment with instance refresh?
Instance Refresh gradually replaces old instances in an ASG with new ones (based on an updated launch template/AMI), a percentage at a time, waiting for new instances to pass health checks before continuing — so the app stays up the whole time.

## 8. Predictive scaling vs target tracking?
Target tracking reacts to what's happening right now. Predictive scaling looks at historical traffic patterns (using machine learning) to scale up *ahead of time*, before a predictable spike even starts, so capacity is ready the moment demand hits.

---

## CloudWatch

## What is CloudWatch?
CloudWatch is AWS's monitoring and observability service — it collects metrics, logs, and events from your AWS resources, and can alert you or trigger automation when something looks wrong.

## 1. Metrics vs Logs vs Alarms vs Events vs Dashboards?
- Metrics: numeric data over time (CPU %, request count).
- Logs: raw text output from your applications/services.
- Alarms: watch a metric and trigger an action (notification, scaling) when a threshold is crossed.
- EventBridge (Events): reacts to things happening (like "an EC2 instance state changed") and routes them to targets like Lambda.
- Dashboards: visual charts combining metrics for a quick overview.

## 2. Standard vs detailed monitoring?
Standard monitoring reports metrics every 5 minutes for free. Detailed monitoring reports every 1 minute for a small extra cost — useful when you need faster reaction to changes (like fast auto-scaling).

## 3. Custom metrics?
You can push your own application-specific numbers (like "orders processed per minute") into CloudWatch using the PutMetricData API or the CloudWatch agent, so you can alarm/dashboard on things AWS doesn't track automatically.

## 4. CloudWatch Alarms — states and evaluation?
An alarm has 3 states: OK (metric is fine), ALARM (threshold breached), or INSUFFICIENT_DATA (not enough data yet to decide). You configure an evaluation period (how many data points, over how long) before an alarm actually fires, to avoid false alarms from a single blip.

## 5. Alarm triggering Auto Scaling or SNS?
An alarm can directly point to an Auto Scaling policy (to scale in/out) or to an SNS topic, which then emails, texts, or pages an on-call engineer (often via something like PagerDuty subscribed to that SNS topic).

## 6. CloudWatch Logs Insights?
A query language/tool to search and analyze huge volumes of log data quickly (filter, aggregate, extract fields) — much faster and more powerful than manually scrolling or basic text search through raw logs.

## 7. Container Insights?
A CloudWatch feature specifically built to automatically collect performance metrics and logs from containerized environments like ECS and EKS (CPU/memory per pod/task, cluster-level stats) without a lot of manual setup.

## 8. Controlling log retention and cost?
By default, CloudWatch Logs never expire (and quietly cost money forever). You should set a retention policy per log group (e.g., keep 30 or 90 days) and optionally export older logs to cheap S3 storage for long-term archiving.

## 9. CloudWatch vs X-Ray?
CloudWatch tells you *that* something is slow or failing (metrics/logs). X-Ray tells you *why*, by tracing a single request as it travels through multiple services, showing exactly which step in the chain caused the delay or error.

---

## Route 53

## What is Route 53?
Route 53 is AWS's DNS (Domain Name System) service — it translates human-readable domain names (like example.com) into the actual IP addresses of your servers, and can also manage domain registration and health-based routing.

## 1. Routing policies — Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multivalue?
- Simple: one record, one answer, no fancy logic.
- Weighted: split traffic by percentage across multiple resources — good for gradual rollouts/A-B testing.
- Latency: sends users to whichever region gives them the fastest response.
- Failover: sends traffic to a primary resource, automatically switches to a backup if the primary's health check fails.
- Geolocation: routes based on the user's actual location (country/continent) — good for legal/content restrictions.
- Geoproximity: like geolocation but lets you shift traffic by "bias" toward or away from certain regions.
- Multivalue: returns several healthy IPs randomly, adding basic load distribution + health checking (not a full load balancer replacement).

## 2. Route 53 health checks + Failover?
Route 53 can regularly check if an endpoint is healthy (e.g., HTTP 200 response). With Failover routing, if the primary resource's health check fails, Route 53 automatically starts answering DNS queries with the backup resource's address instead.

## 3. Alias record vs CNAME?
A CNAME points one name to another name, but AWS doesn't allow a CNAME at the root/apex of a domain (like `example.com` itself) due to DNS rules. An Alias record is an AWS-specific feature that acts like a CNAME but is allowed at the zone apex, and it's free with no extra DNS lookup — commonly used to point a domain straight at an ALB or CloudFront distribution.

## 4. Blue/green or canary deployments with Route 53?
Using Weighted routing, you can send a small percentage of traffic (e.g., 5%) to a new version while the rest goes to the old version, gradually increasing the new version's weight as you gain confidence — a controlled, low-risk rollout.

## 5. Private hosted zone?
A DNS zone that only resolves inside one or more specified VPCs — used for internal service names (like `db.internal`) that shouldn't be visible or resolvable from the public internet.

## 6. Migrating DNS to Route 53 without downtime?
Before switching, lower the TTL (time-to-live) on your existing DNS records well in advance so caches refresh quickly, recreate all records identically in Route 53, verify everything resolves correctly, then update the domain's nameservers — run both in parallel briefly to be safe.

## 7. Route 53 Resolver?
Extends DNS resolution between your VPC and outside networks — for example, letting your VPC resolve on-premises domain names and letting your on-prem network resolve AWS-private domain names, enabling hybrid cloud DNS.

---

## Cross-Service / Scenario Questions — Sample Answer Approach

## 1. Design a highly available 3-tier web app.
Put a Route 53 record pointing at an ALB in public subnets across 2+ AZs. The ALB forwards to an Auto Scaling Group (or ECS/EKS) running app servers in private subnets, also across multiple AZs. The app connects to a Multi-AZ RDS database in an even more restricted private subnet. CloudWatch monitors everything and triggers alarms/scaling. Security Groups tightly control which tier can talk to which.

## 2. Intermittent 5xx errors — troubleshooting order.
Check ALB target health first (are targets failing health checks), then CloudWatch Logs for application errors, then whether ASG/ECS/EKS is scaling fast enough for the load, then Security Group/NACL rules for any blocked connections, and finally downstream dependencies (database, third-party APIs) for slowness.

## 3. CI/CD pipeline for Docker → ECR → ECS/EKS.
Code push triggers a build (CodeBuild/GitHub Actions) that builds the Docker image, tags it, and pushes to ECR. The pipeline then updates the ECS task definition (or Kubernetes manifest) to reference the new image tag and triggers a rolling or blue-green deployment, watching health checks to auto-rollback on failure.

## 4. Securing secrets across EC2/ECS/EKS.
Never hardcode secrets. Store them in Secrets Manager or SSM Parameter Store, and grant the compute's IAM role (instance profile, task role, or IRSA for EKS) permission to read just that secret — the application fetches it at runtime.

## 5. ASG flapping (scaling in/out repeatedly).
Usually caused by a scaling policy threshold set too tight, or cooldown periods too short, causing the metric to bounce back and forth across the threshold. Fix by widening the target range, increasing cooldown time, or switching to a smoother metric.

## 6. Reducing AWS costs.
Use Spot/Reserved instances where appropriate, right-size instances based on actual usage, use S3 lifecycle policies to move cold data to cheaper storage, clean up unused EBS volumes/snapshots, and reduce cross-AZ/cross-region data transfer where possible.

## 7. Monitoring/alerting for EKS microservices.
Enable Container Insights for cluster-wide metrics, ship logs via Fluent Bit to CloudWatch Logs, set CloudWatch Alarms on key metrics (error rate, latency, pod restarts), and route alarms through SNS to a paging tool for on-call response.

## 8. Disaster recovery across regions.
Replicate critical data (S3 Cross-Region Replication, RDS cross-region read replicas), keep infrastructure-as-code ready to redeploy in a second region, and use Route 53 Failover routing so traffic automatically shifts to the backup region if the primary region's health checks fail.

## 9. Full request flow (name every component).
User's browser → Route 53 (DNS lookup) → ALB (public subnet) → target group → EC2/ECS/EKS app servers (private subnet) → application logic → database (RDS/DynamoDB, private subnet) → response travels back the same path.

## 10. Enforcing security/compliance across many accounts.
Use AWS Organizations with Service Control Policies (SCPs) to set hard guardrails, AWS Config to continuously check resource compliance, GuardDuty for threat detection, and a Landing Zone/Control Tower setup to standardize how every new account is created and secured from day one.
