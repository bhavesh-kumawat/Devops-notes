# AWS Interview Questions & Answers

## 🔵 AWS Fundamentals

### 1. What is AWS and what are its core services?

**AWS (Amazon Web Services)** is a comprehensive cloud computing platform provided by Amazon.

- Offers on-demand computing, storage, networking, databases, and other services over the internet.
- Provides scalable, secure, and cost-efficient infrastructure for building applications without managing physical hardware.
- Follows a pay-as-you-go model.

---

### 2. Difference between Security Groups and NACLs (Network Access Control List)?

| **Feature** | **Security Groups** | **Network ACLs (NACLs)** |
| --- | --- | --- |
| Scope / Level | Instance level (EC2, RDS, Lambda ENI) | Subnet level |
| Type | Stateful | Stateless |
| Default Behavior | Deny all inbound, allow all outbound by default | Allow all inbound/outbound by default |
| Rules | Inbound and outbound rules | Separate inbound and outbound rules |
| Rule Action | Allow only | Allow or deny |
| Evaluation | Evaluates all rules and allows if matched | Evaluates rules in number order; first match applies |
| Port / Protocol Control | Yes | Yes |
| Typical Use Case | Control traffic to/from instances | Control traffic at subnet boundary or extra layer of protection |

---

### 3. What is AWS Regions vs Availability Zones?

| **Feature** | **AWS Region** | **Availability Zone (AZ)** |
| --- | --- | --- |
| Definition | A geographical area containing multiple data centers | A physically separate data center within a region |
| Purpose | Provides global reach and geographical separation | Provides fault isolation and high availability within a region |
| Redundancy | Regions are completely independent | AZs in the same region are isolated but connected via low-latency links |
| Use Case | Multi-region deployment for disaster recovery or latency optimization | High availability architecture (e.g., multi-AZ EC2 or RDS) |
| Number per AWS Account | ~30+ globally | Multiple per region (usually 2–6) |
| Distance | Hundreds to thousands of kilometers apart | Typically <100 km apart for low latency |

---

### 4. What is Auto Scaling and how does it work?

**Auto Scaling** is an AWS service that automatically adjusts the number of compute resources (EC2 instances, containers) based on demand.

- Ensures applications have the right capacity at all times
- Helps improve availability, reduce costs, and handle traffic spikes

#### 🔹 Key Components

**1. Auto Scaling Group (ASG)**

- Logical group of instances managed together
- Defines min, max, and desired instance count

**2. Launch Template / Launch Configuration**

- Defines instance type, AMI, key pairs, security groups

**3. Scaling Policies**

- **Dynamic scaling** → reacts to metrics (CPU, memory, custom CloudWatch metrics)
- **Scheduled scaling** → scales at specific times (e.g., daily traffic peak)
- **Target tracking** → maintains a metric at a target value

**4. Health Checks**

- Ensures unhealthy instances are terminated and replaced

#### 🔹 How Auto Scaling Works

1. **Define an Auto Scaling Group**
    - Select AMI, instance type, network settings, min/max/desired size
2. **Set Scaling Policies**
    - Example: scale out if average CPU > 70% for 5 minutes
    - Example: scale in if CPU < 30% for 10 minutes
3. **Monitor Metrics**
    - CloudWatch tracks metrics (CPU, memory, custom metrics)
4. **Trigger Scaling Action**
    - If thresholds are breached, Auto Scaling adds or removes instances
    - Health checks replace unhealthy instances automatically

#### 🔹 Example Use Case

**Web application serving traffic spikes:**

- Min instances: 2
- Max instances: 10
- Target CPU: 50%

Auto Scaling will automatically launch new instances when CPU rises above 50% and terminate them when it drops below 50%.

---

### 5. Difference between Elastic Beanstalk and ECS / EKS?

| **Feature** | **Elastic Beanstalk** | **ECS / EKS** |
| --- | --- | --- |
| Abstraction Level | Platform as a Service (PaaS) | Container orchestration (more control, less abstraction) |
| Focus | Deploying applications (code) | Running containers (Docker) |
| Management | AWS manages underlying EC2 instances, load balancers, scaling | You manage container definitions, tasks, and clusters (though EKS is managed Kubernetes) |
| Supported Workloads | Web apps, APIs (Java, Python, Node.js, Go, etc.) | Any containerized workload |
| Scaling | Auto scaling handled automatically for your app | Auto scaling available but must be configured via ECS service autoscaling or Kubernetes HPA in EKS |
| Complexity | Low — easy to deploy apps | Higher — requires containerization and orchestration knowledge |
| Flexibility / Customization | Limited — mostly standard web app environment | High — full control over networking, tasks, pods, and scheduling |
| Use Case | Quick deployment of applications without managing infrastructure | Running microservices or complex containerized workloads with orchestration |

---

### 6. Difference between Elastic Load Balancer types?

| **Feature** | **Classic Load Balancer (CLB)** | **Application Load Balancer (ALB)** | **Network Load Balancer (NLB)** |
| --- | --- | --- | --- |
| Layer | L4 (TCP) & L7 (HTTP/HTTPS) | L7 (HTTP/HTTPS) | L4 (TCP/UDP) |
| Best For | Legacy apps, simple load balancing | Web apps, microservices, content-based routing | Extreme performance, ultra-low latency, TCP traffic |
| Routing | Round-robin or sticky sessions | Path-based, host-based, query param routing | Connection-based routing |
| Target Type | EC2 instances only | EC2, IP addresses, Lambda | EC2, IP addresses, NLB endpoints |
| TLS Termination | Supported | Supported | Supported (TLS passthrough) |
| Health Checks | Basic (HTTP, TCP) | Advanced (HTTP, HTTPS with path) | TCP-level |
| Scaling / Performance | Auto-scaled, moderate | Auto-scaled, supports millions of requests per second | Handles millions of connections at very low latency |
| Sticky Sessions | Yes | Yes | No |
| WebSockets Support | Limited | Yes | No |
| When to Use | Legacy workloads, simple TCP/HTTP apps | Modern web apps, microservices, containerized apps | High-performance TCP/UDP apps, gaming, IoT, financial services |

---

### 7. What is CloudWatch and what types of monitoring does it offer?

#### 🔹 What is AWS CloudWatch?

**CloudWatch** is a monitoring and observability service for AWS resources and applications.

- Collects and tracks metrics, logs, and events
- Provides dashboards, alarms, and automated actions
- Helps ensure application performance, operational health, and resource optimization

#### 🔹 Types of Monitoring in CloudWatch

**1️⃣ Infrastructure / Resource Monitoring**

- Tracks AWS service metrics (built-in metrics)
- Examples:
    - **EC2:** CPU, memory, disk I/O, network traffic
    - **RDS:** Database connections, CPU utilization, storage space
    - **ELB:** Request count, latency, HTTP 5xx errors
    - **EBS:** Volume read/write operations, throughput

**2️⃣ Custom Metrics Monitoring**

- Applications can publish their own metrics to CloudWatch
- Examples:
    - Number of orders processed per minute
    - Queue length in SQS
    - Custom business KPIs

**3️⃣ Logs Monitoring**

- Collect application, system, and custom logs using CloudWatch Logs
- Features:
    - Centralized log storage
    - Real-time search and analysis
    - Log metric filters (convert log patterns to metrics)

**4️⃣ Alarms and Notifications**

- Trigger alerts based on thresholds or anomalies
- Example:
    - CPU > 80% for 5 mins → send SNS notification
- Can automate actions (scale EC2, restart services)

**5️⃣ Events / Event-Driven Monitoring**

- CloudWatch Events / EventBridge tracks AWS API calls or resource state changes
- Example:
    - EC2 instance terminated → trigger Lambda function
    - S3 object created → trigger notification

**6️⃣ Dashboards**

- Visualize metrics and logs in custom dashboards
- Combine multiple resources or metrics for holistic view

---

### 8. What is AWS CodePipeline / CodeBuild / CodeDeploy?

- These three services—AWS CodePipeline, CodeBuild, and CodeDeploy—are part of Amazon Web Services (AWS) DevOps toolchain. They work together to automate your software delivery process (CI/CD pipeline).

**🔄 1. AWS CodePipeline (The Orchestrator)**
- CodePipeline is a CI/CD pipeline service that automates the entire flow:

- Source → Build → Test → Deploy

- Think of it like:
👉 A manager who tells each step when to run

- Key points:

- Connects tools like GitHub, CodeBuild, CodeDeploy
- Automatically triggers when code changes
- Defines stages (Source, Build, Deploy, etc.)

- Example flow:

- Developer pushes code to GitHub
- CodePipeline detects change
- Triggers CodeBuild
- Then triggers CodeDeploy

**🏗️ 2. AWS CodeBuild (The Builder)**
- CodeBuild is a fully managed build service.

- Think of it like:
👉 A worker who compiles and tests your code

- What it does:

- Compiles source code
= Runs tests
= Creates build artifacts (like .jar, .zip, Docker image)

- Key features:

- No servers to manage
- Uses buildspec.yml file for instructions
- Supports multiple languages (Java, Node.js, Python, etc.)

- Example:

- Takes your Java code → compiles → runs tests → outputs .jar

**🚀 3. AWS CodeDeploy (The Deployer)**
- CodeDeploy is a deployment service.

- Think of it like:
👉 A delivery person who installs your app on servers

- What it does:

- Deploys applications to:
- EC2 instances
- Lambda
- On-prem servers
- Supports deployment strategies:
- Rolling deployment
- Blue/Green deployment

- Example:

- Takes .jar from CodeBuild → deploys it to EC2 instances

| **Service** | **Purpose** | **Key Features** |
| --- | --- | --- |
| CodeCommit | Git-based source control | Store and version your code in private repositories |
| CodeBuild | Build service | Compiles code, runs tests, and produces artifacts |
| CodePipeline | CI/CD orchestration | Automates the workflow from code commit → build → deploy |
| CodeDeploy | Deployment automation | Deploys applications to EC2, Lambda, or on-premises servers |

---

## 🔴 AWS Advanced / Production-Level

### 9. How do you make an application highly available on AWS?

#### 🔹 Definition of High Availability

**High Availability** means designing your application so that it remains operational and accessible even if some components fail, minimizing downtime.

#### 🔹 Steps to Make an Application Highly Available on AWS

**1️⃣ Use Multiple Availability Zones (AZs)**

- Deploy EC2 instances, databases, and load balancers across multiple AZs
- AZs are isolated data centers within a region
- If one AZ fails, traffic can route to healthy AZs
- Example: Web servers in 2 AZs behind an Application Load Balancer (ALB)

**2️⃣ Use Elastic Load Balancers (ELB)**

- Distribute traffic across multiple instances/AZs
- Supports health checks → automatically route traffic away from unhealthy instances

**3️⃣ Use Auto Scaling**

- Automatically increase or decrease EC2 instances based on demand
- Ensures enough capacity during traffic spikes and cost efficiency when idle

**4️⃣ Deploy Multi-AZ or Multi-Region Databases**

- **RDS Multi-AZ** → automated failover if primary DB fails
- **Aurora Global DB / DynamoDB Global Tables** → multi-region redundancy
- Ensures data is highly available and durable

**5️⃣ Use Fault-Tolerant Storage**

- **S3** → stores objects redundantly across multiple AZs
- **EFS** → highly available network file system
- Avoid single points of failure for critical data

**6️⃣ Decouple Components**

- Use SQS, SNS, or EventBridge for asynchronous messaging
- Decoupling reduces dependency on single component failure

**7️⃣ Implement Health Checks & Monitoring**

- Use CloudWatch metrics and alarms for:
    - EC2 instance health
    - Load balancer metrics
    - Database availability
- Integrate with Auto Scaling / Lambda for automated recovery

**8️⃣ Use DNS Failover for Multi-Region HA**

- Route 53 can detect region failure and route traffic to another healthy region
- Ensures disaster recovery across regions

**9️⃣ Backup & Disaster Recovery**

- Regular backups of databases and S3 data
- Test failover procedures regularly

---

### 10. How do you secure data at rest and in transit?

#### 🔹 Securing Data At Rest

Data at rest is stored data in databases, file systems, or object storage.

**1️⃣ AWS Services & Techniques**

| **Service** | **How to Secure Data at Rest** |
| --- | --- |
| S3 | Enable server-side encryption (SSE-S3, SSE-KMS) for objects; optionally use client-side encryption |
| EBS (Volumes) | Use EBS encryption at the volume level (AES-256) |
| RDS / Aurora / DynamoDB | Enable encryption during database creation (KMS-managed keys) |
| EFS | Enable encryption at rest via AWS KMS |

**2️⃣ Key Management**

- Use **AWS KMS** (Key Management Service) to manage encryption keys
- Rotate keys periodically and control access via IAM policies

**3️⃣ OS / Application Level Encryption**

- Optionally encrypt sensitive files with GPG, OpenSSL, or application-level encryption

#### 🔹 Securing Data In Transit

Data in transit is data moving across networks (internet, VPC, or internal service calls).

**1️⃣ Transport Layer Encryption**

| **Protocol / Service** | **How to Secure Data in Transit** |
| --- | --- |
| HTTPS / TLS | Use TLS 1.2 or 1.3 for web traffic (ALB, CloudFront, API Gateway) |
| RDS / Aurora / DynamoDB | Enable SSL/TLS connections for database clients |
| S3 / API Calls | Use HTTPS endpoints or VPC endpoints |
| Internal Services | Use mutual TLS (mTLS) or service mesh (e.g., AWS App Mesh) |

**2️⃣ VPC & Network Security**

- Use VPC endpoints to keep traffic internal (S3, DynamoDB)
- Use VPN or Direct Connect for secure hybrid connectivity

**3️⃣ IAM & Access Control**

- Use IAM policies and roles to ensure only authorized entities access data

---

### 11. What is AWS ECS vs EKS?

| **Feature** | **ECS (Elastic Container Service)** | **EKS (Elastic Kubernetes Service)** |
| --- | --- | --- |
| Type | AWS-native container orchestration | Managed Kubernetes service |
| Control Plane | Fully managed by AWS, no Kubernetes control plane | AWS manages Kubernetes control plane, you manage nodes and workloads |
| Complexity | Simple to set up, AWS-native APIs | More complex, Kubernetes knowledge required |
| Flexibility | Limited to ECS APIs and AWS features | Full Kubernetes ecosystem (custom controllers, CRDs, operators) |
| Deployment Options | EC2 or Fargate (serverless) | EC2 or Fargate (serverless) |
| Scaling | ECS service auto scaling | Kubernetes HPA / Cluster autoscaler |
| Learning Curve | Low | High (requires Kubernetes knowledge) |
| Use Case | Quick deployment of containerized applications with minimal overhead | Running complex microservices, multi-cloud Kubernetes workloads, or leveraging Kubernetes ecosystem |

#### 🔹 Key Differences Explained

**1. ECS**

- AWS-native service → simpler integration with other AWS services (ALB, CloudWatch, IAM)
- No Kubernetes knowledge required
- Managed scheduling, load balancing, and scaling
- Great for AWS-focused, smaller teams or simple microservices

**2. EKS**

- Managed Kubernetes → industry-standard container orchestration
- Can run existing Kubernetes manifests
- Supports complex workloads (stateful apps, service meshes, Helm charts)
- Ideal for teams already using Kubernetes or multi-cloud strategies

---

### 12. What are CloudWatch Logs Insights?

#### 🔹 What is CloudWatch Logs Insights?

**CloudWatch Logs Insights** is a fully managed, interactive log analytics service that allows you to:

- Query, analyze, and visualize log data stored in CloudWatch Logs
- Gain operational and application insights quickly
- Identify errors, performance issues, and trends in real-time

It is especially useful for large-scale logs where searching manually would be inefficient.

#### 🔹 Key Features

**1. Query Logs**

- Use a SQL-like query language to filter, parse, and aggregate logs
- Example query: count errors per service or host

**2. Aggregation & Statistics**

- Calculate sum, average, max, min, percentiles of metrics extracted from logs

**3. Visualization**

- Generate line graphs, bar charts, or tables directly in CloudWatch dashboards

**4. Real-Time Insights**

- Works on logs ingested continuously
- Helps identify spikes, errors, or anomalies quickly

**5. Integration**

- Can be integrated into CloudWatch dashboards or alerts
- Can query multiple log groups at once

---

### 13. Explain AWS Systems Manager (SSM) and its benefits

**AWS Systems Manager (SSM)** is a fully managed service that lets you view, control, and automate operational tasks across your AWS resources.

- Provides a centralized interface for managing EC2 instances, on-prem servers, and hybrid environments
- Helps automate patching, configuration, deployment, and compliance

#### 🔹 Key Features of SSM

| **Feature** | **Description** |
| --- | --- |
| Session Manager | Securely access EC2 or on-prem instances without SSH keys or opening ports |
| Run Command | Execute commands remotely on multiple instances simultaneously |
| State Manager | Maintain consistent configuration and compliance across instances |
| Patch Manager | Automate OS patching for Windows and Linux instances |
| Parameter Store | Securely store configuration data, secrets, and passwords |
| Automation | Automate complex workflows (e.g., backups, deployments, scaling) |

#### 🔹 Benefits of AWS Systems Manager

**1. Centralized Management**

- Manage thousands of instances from a single console
- Works for EC2, on-prem servers, and hybrid environments

**2. Security & Compliance**

- Access instances without opening SSH/RDP ports
- Store secrets securely in Parameter Store or integrate with AWS KMS

**3. Automation & Efficiency**

- Automate repetitive tasks like patching, software deployment, or backups
- Reduce manual errors and operational overhead

**4. Visibility & Auditing**

- Provides logs of commands, actions, and compliance status
- Integrates with CloudWatch and CloudTrail for auditing

**5. Scalability**

- Execute tasks across thousands of instances in parallel
- Supports both cloud and on-premises infrastructure

---

### 14. How do you handle secrets management in AWS?

#### 🔹 Secrets Management in AWS

AWS provides multiple ways to manage secrets securely:

**1️⃣ AWS Systems Manager Parameter Store**

- Securely stores configuration data and secrets
- Supports encryption with KMS
- Can organize parameters by hierarchies and environment
- Useful for application configuration, feature flags, or secrets

*Pros:*

- Free for standard parameters
- Integrated with IAM for access control
- Easy integration with EC2, Lambda, ECS, and EKS

*Cons:*

- Limited secret rotation (manual or via automation scripts)

**2️⃣ AWS Secrets Manager**

- Designed specifically for secrets management (DB passwords, API keys, certificates)
- Supports automatic rotation for AWS RDS, Redshift, and some custom targets
- Encrypts secrets at rest using AWS KMS
- Provides fine-grained access control via IAM

*Pros:*

- Automatic secret rotation
- Integrated with AWS SDKs
- Versioning of secrets

*Cons:*

- Paid service (per secret)

**3️⃣ AWS KMS (Key Management Service)**

- Provides encryption keys to secure secrets stored elsewhere
- Can encrypt/decrypt secrets programmatically
- Used by both Parameter Store and Secrets Manager under the hood

#### 4️⃣ Best Practices for Secrets Management

1. Never hardcode secrets in code or commit to repositories
2. Use IAM roles instead of static credentials wherever possible
    - Example: EC2 or Lambda roles access secrets programmatically
3. Enable encryption at rest and in transit
4. Rotate secrets regularly
5. Audit access using CloudTrail
6. Use environment-specific secrets to isolate environments

#### 5️⃣ Example Usage in DevOps Pipeline

- ECS / EKS / Lambda fetch secrets from Secrets Manager or Parameter Store
- Use IAM roles to grant access dynamically
- Rotate database passwords automatically and update applications without downtime

---

### 15. How do you implement CI/CD for a multi-region deployment?

#### 🔹 Goal

Implement a CI/CD pipeline that deploys applications across multiple AWS regions while ensuring:

- High availability
- Zero or minimal downtime
- Consistency across regions
- Rollback capability

#### 🔹 Step 1: Use a CI/CD Tool

- Use AWS native tools (CodePipeline, CodeBuild, CodeDeploy) or third-party tools (Jenkins, GitHub Actions, GitLab CI).
- Define stages: Build → Test → Deploy.

#### 🔹 Step 2: Build & Test Once

- Build the application once in a primary region.
- Produce immutable artifacts (Docker images, JAR/WAR, Lambda packages).
- Run unit/integration tests to ensure reliability.

*Tip: Use S3 or ECR to store artifacts accessible from all regions.*

#### 🔹 Step 3: Multi-Region Deployment Strategy

**Option 1: Blue/Green Deployment per Region**

- Deploy new version to green environment in each region.
- Route traffic using Route 53 weighted or failover routing.
- Switch traffic gradually from blue → green after validation.

**Option 2: Canary Deployment**

- Deploy new version to small subset of servers in each region.
- Monitor metrics (latency, errors) via CloudWatch/Grafana.
- Gradually increase deployment scope.

**Option 3: Rolling Deployment**

- Deploy new version in batches per region.
- Use Auto Scaling Groups or Kubernetes rolling updates.

#### 🔹 Step 4: Traffic Management

- Use Route 53 DNS routing policies:
    - **Weighted routing** → shift traffic gradually
    - **Failover routing** → send traffic to healthy region if primary fails
- Optionally use CloudFront for global distribution of static content

#### 🔹 Step 5: Monitoring & Rollback

- Monitor application metrics: latency, errors, CPU/memory
- Set CloudWatch alarms integrated with CodeDeploy or Lambda for automatic rollback
- Keep artifact versions immutable for rollback per region

#### 🔹 Step 6: Automation

- Use Infrastructure as Code (IaC) tools like CloudFormation, Terraform, or CDK to deploy infrastructure consistently across regions
- Automate secrets provisioning using Secrets Manager / Parameter Store

---

### 16. How do you troubleshoot performance issues in AWS?

#### 🔹 Step 1: Identify the Symptoms

- High latency, slow application response, timeouts, or failed requests
- Use CloudWatch metrics to detect spikes in CPU, memory, or network utilization

**Key AWS Metrics to Check:**

- **EC2:** CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps
- **RDS / Aurora:** CPU, Connections, Read/Write Latency, FreeStorageSpace
- **ELB / ALB:** Latency, 4xx/5xx errors, RequestCount
- **S3 / DynamoDB:** Throttled requests, latency

#### 🔹 Step 2: Analyze Application Logs

- Check CloudWatch Logs or application logs
- Look for errors, exceptions, or slow queries
- Use CloudWatch Logs Insights to query and filter logs efficiently

#### 🔹 Step 3: Check Resource Utilization

- **EC2 / ECS / EKS:** Are instances under-provisioned?
- **Auto Scaling:** Are scaling policies insufficient or delayed?
- **RDS / DynamoDB:** High IOPS, CPU, or throttling
- **S3:** Request rate limits or eventual consistency issues

#### 🔹 Step 4: Network Performance

- Check VPC Flow Logs for connectivity issues
- Check latency and throughput for ELB, NLB, or API Gateway
- Ensure subnet, route tables, and security groups are configured properly

#### 🔹 Step 5: Database & Storage Bottlenecks

- **RDS / Aurora:** Slow queries, missing indexes, insufficient instance type
- **DynamoDB:** Provisioned throughput limits, hot partitions
- **S3 / EFS:** High latency in reads/writes, consider caching with CloudFront or ElastiCache

#### 🔹 Step 6: Application & Code Level

- Identify CPU/memory-intensive operations or inefficient code
- Check thread pools, connection pools, or queue backlogs
- Profile using X-Ray for distributed tracing

#### 🔹 Step 7: Scaling & Optimization

- Use Auto Scaling Groups or ECS/EKS Horizontal Pod Autoscaling
- Optimize instance types or storage performance
- Consider caching (ElastiCache, CloudFront) to reduce load on DB

#### 🔹 Step 8: Monitor & Alert

- Set CloudWatch alarms for CPU, memory, latency, error rates
- Use Grafana or CloudWatch Dashboards for visualization
- Integrate with SNS for notifications

#### 🔹 Step 9: Document & Prevent

- Identify root cause
- Update runbooks / automated remediation
- Optimize architecture for future growth

---

## ⚙️ Scenario-Based / DevOps-Focused

### 17. Your EC2 instance is unreachable, how do you debug?

#### 🔹 Step 1: Check Security Groups

- Verify inbound rules allow traffic on the port you are trying to reach (e.g., 22 for SSH, 3389 for RDP)
- Verify source IP ranges in the rules
- Confirm outbound rules allow responses

#### 🔹 Step 2: Check Network ACLs (NACLs)

- Ensure the subnet-level NACLs allow inbound and outbound traffic for the relevant ports
- Remember NACLs are stateless, so both inbound and outbound rules must permit traffic

#### 🔹 Step 3: Verify the Route Table

- Ensure the subnet has a route to the Internet Gateway if it's a public instance
- For private instances, ensure NAT Gateway or VPC peering is correctly configured

#### 🔹 Step 4: Check the Instance State

- Confirm the EC2 instance is running
- Check the system status checks and instance status checks in the EC2 console
- If a status check failed, AWS might provide recommended actions

#### 🔹 Step 5: Use Session Manager (SSM)

- If the instance has SSM agent installed and IAM role attached, use Session Manager to access it without SSH
- Helps debug network, firewall, or service issues

#### 🔹 Step 6: Verify Key Pair / Credentials

- For SSH, ensure you're using the correct private key
- Verify user name (e.g., ec2-user for Amazon Linux, ubuntu for Ubuntu)
- Check permissions of private key (chmod 400 key.pem)

#### 🔹 Step 7: Check Operating System

Once accessed (via SSM or serial console):

- Check firewall (iptables / ufw) rules
- Ensure the SSH or RDP service is running
- Check system logs (/var/log/messages or /var/log/syslog)

#### 🔹 Step 8: Check Elastic IP / Public IP

- Ensure the instance has a public IP if connecting over the Internet
- Check Elastic IP association
- For private instances, confirm access via VPN, Direct Connect, or Bastion host

#### 🔹 Step 9: Use AWS Tools

- **VPC Reachability Analyzer** → check connectivity between your source and EC2
- **CloudTrail / CloudWatch Logs** → detect events affecting instance connectivity

#### 🔹 Step 10: Reboot / Replace (if necessary)

- If nothing works, try rebooting the instance
- If the root volume is corrupted, consider launching a new instance and attaching the old volume for investigation

---

### 18. S3 bucket is not accessible by your app — what could be wrong?

#### 🔹 Possible Reasons Why an S3 Bucket Is Not Accessible

**1️⃣ IAM Permissions**

- The role or user your application uses may lack proper permissions.
- Ensure the IAM policy allows s3:GetObject, s3:PutObject, or other required actions for the bucket.
- Check if there are explicit deny statements overriding allow permissions.

**2️⃣ Bucket Policy**

- The bucket may have a policy restricting access to certain IAM users, roles, or IPs.
- Verify that the bucket policy allows the requesting principal to perform the required actions.

**3️⃣ Public Access Settings**

- S3 Block Public Access might prevent access even if the bucket policy allows it.
- Check Account-level and bucket-level block public access settings.

**4️⃣ VPC Endpoint / Private Bucket**

- If the bucket is private and accessed from a VPC, you may need an S3 VPC endpoint.
- Ensure the endpoint policy allows your app to reach the bucket.

**5️⃣ CORS Configuration (for web apps)**

- For browser-based apps, the Cross-Origin Resource Sharing (CORS) rules may prevent access.
- Verify allowed origins, headers, and methods in the bucket CORS configuration.

**6️⃣ Object-Level Permissions**

- Even if the bucket is accessible, individual objects may have restrictive ACLs.
- Check object ownership and ACLs.

**7️⃣ Networking / Connectivity Issues**

- If accessing from a private subnet without internet or S3 endpoint, the app may not reach S3.
- Ensure proper route tables, NAT Gateway, or VPC endpoints are configured.

**8️⃣ Region Mismatch**

- Accessing the bucket in the wrong AWS region may cause errors.
- Confirm your app is using the correct region endpoint.

**9️⃣ KMS Encryption**

- If the bucket or objects are encrypted with KMS, the IAM role must have kms:Decrypt permission.

#### 🔹 How to Troubleshoot Step by Step

1. Check IAM role/user permissions (s3:GetObject, s3:ListBucket)
2. Check bucket policy for explicit denies or IP restrictions
3. Check public access block settings
4. Check networking (VPC endpoints, NAT, routing)
5. Verify region and bucket name in the app config
6. If using KMS encryption, ensure proper key permissions

---

### 19. How do you migrate an on-prem application to AWS?

#### 🔹 Step 1: Assessment & Planning

**1. Inventory your application**

- List servers, databases, storage, networking, and dependencies

**2. Classify workloads**

- Tier applications: critical, non-critical, low/high complexity

**3. Define goals**

- Cost optimization, scalability, high availability, disaster recovery

**4. Select migration approach**

- **Rehost ("lift and shift")** → move as-is to EC2
- **Replatform ("lift, tinker, and shift")** → minimal changes, e.g., migrate database to RDS
- **Refactor / Re-architect** → redesign for cloud-native (microservices, serverless)

#### 🔹 Step 2: Choose AWS Services

- **Compute:** EC2, ECS, EKS, Lambda
- **Storage:** S3, EBS, EFS
- **Databases:** RDS, Aurora, DynamoDB
- **Networking:** VPC, Subnets, Security Groups, Route 53
- **Monitoring:** CloudWatch, X-Ray

#### 🔹 Step 3: Prepare the Environment

**1. Networking**

- Design VPC, subnets, NAT gateways, route tables, security groups

**2. Identity & Access**

- Configure IAM roles and policies

**3. Logging & Monitoring**

- Enable CloudWatch, CloudTrail, and alarms

#### 🔹 Step 4: Migration Execution

**Option A: Lift-and-Shift**

- Use AWS Server Migration Service (SMS) or VM Import/Export
- Move VMs to EC2 with minimal changes

**Option B: Database Migration**

- Use AWS Database Migration Service (DMS)
- Supports homogeneous (MySQL → MySQL) or heterogeneous (Oracle → Aurora) migrations
- Enable ongoing replication to minimize downtime

**Option C: Application Refactoring**

- Break monolith into microservices on ECS/EKS
- Migrate static content to S3 + CloudFront
- Consider Lambda / Step Functions for serverless workflows

#### 🔹 Step 5: Testing

- Validate functionality, performance, and security
- Run load tests to compare with on-prem performance
- Ensure data consistency after migration

#### 🔹 Step 6: Cutover & Optimization

- Switch production traffic to AWS
- Decommission on-prem resources
- Optimize for cost, scalability, and HA
- Use Auto Scaling, ELB, CloudFront, and caching for performance

#### 🔹 Step 7: Monitoring & Ongoing Management

- Use CloudWatch, X-Ray, and AWS Config for ongoing monitoring
- Set up alerts, dashboards, and automated remediation

---

### 20. Your Lambda function is timing out — what do you do?

#### 🔹 Step 1: Check the Timeout Setting

- Each Lambda has a configured timeout (default: 3 seconds, max: 15 minutes).
- If the function is long-running, increase the timeout in the Lambda console, CLI, or IaC template.

#### 🔹 Step 2: Analyze CloudWatch Logs

- Lambda automatically logs start, end, and errors in CloudWatch Logs.
- Look for:
    - Long-running loops
    - External API calls timing out
    - Resource access errors (DB, S3, DynamoDB)

#### 🔹 Step 3: Check Resource Limits

- Memory allocation affects CPU in Lambda → low memory can cause slow execution.
- Consider increasing memory; it often improves performance as CPU scales with memory.

#### 🔹 Step 4: Inspect External Dependencies

Common causes of timeout:

- Slow database queries (RDS, DynamoDB)
- Long HTTP requests / third-party APIs
- Network issues in VPC (private subnets without NAT)

*Tip: If Lambda is in a VPC, ensure it has NAT Gateway or VPC endpoints to access external services.*

#### 🔹 Step 5: Optimize Code & Logic

- Look for inefficient loops, heavy computations, or blocking calls.
- Consider parallel processing or batch processing to reduce execution time.
- For long tasks, consider breaking the function into smaller Lambdas or using Step Functions for orchestration.

#### 🔹 Step 6: Use Retries & Dead Letter Queues (DLQ)

- Lambda automatically retries on errors (if configured).
- Configure DLQ (SNS or SQS) to capture failed events for debugging and reprocessing.

#### 🔹 Step 7: Monitor Metrics

Use CloudWatch Lambda metrics:

- **Duration** → actual execution time
- **Timeouts** → number of timeouts
- **Errors** → failed invocations

This helps identify trends and patterns causing timeouts.

---

### 21. How do you implement blue-green deployment in AWS?

#### 🔹 What is Blue-Green Deployment?

**Blue-Green Deployment** is a strategy where you have two identical environments:

- **Blue environment:** currently serving production traffic
- **Green environment:** new version of the application to deploy

Traffic is switched from blue → green once the new version is tested and validated, reducing downtime and rollback risk.

#### 🔹 Step 1: Prepare Environments

1. Provision two identical environments using AWS services:
    - EC2 / Auto Scaling Groups, or
    - ECS / EKS services, or
    - Elastic Beanstalk environments
2. Ensure both environments are ready to handle full production traffic.

#### 🔹 Step 2: Deploy to Green Environment

- Deploy the new version to the green environment
- Run smoke tests to validate the deployment
- Monitor CloudWatch metrics for errors, latency, or failures

#### 🔹 Step 3: Switch Traffic

- Use AWS **Route 53** for DNS-based traffic switching:
    - **Weighted routing** → gradually shift traffic from blue to green
    - **Failover routing** → route traffic to green if blue fails
- If using Elastic Load Balancers (ALB/NLB):
    - Update target groups to point to green environment

#### 🔹 Step 4: Monitor & Validate

- Monitor application and metrics:
    - CPU, memory, request latency, error rates
- Confirm everything is working before decommissioning blue environment

#### 🔹 Step 5: Rollback Plan

- Keep the blue environment intact until green is fully validated
- If issues occur, switch traffic back to blue immediately

#### 🔹 AWS Tools for Blue-Green Deployment

| **AWS Service** | **How it Helps** |
| --- | --- |
| Elastic Beanstalk | Supports Blue/Green environment swaps out of the box |
| CodeDeploy | Automates blue-green deployment to EC2, ECS, or Lambda |
| Route 53 | DNS-based traffic switching between environments |
| ALB / NLB | Target group switching for traffic routing |
| CloudWatch | Monitor metrics and health during deployment |

#### 🔹 Example Scenario

Deploy a new web application version:

1. Current version on Blue (prod)
2. New version deployed on Green (staging)
3. Test green environment
4. Switch Route 53 weighted traffic: 10% → 50% → 100% to green
5. Monitor metrics → if stable, terminate blue

---

### 22. How would you set up monitoring and alerting for a production web app?

#### 🔹 Step 1: Identify What to Monitor

For a production web app, monitor three main layers:

**1. Infrastructure**

- CPU, memory, disk, network of EC2 instances or containers
- Database health (RDS, DynamoDB, Aurora)
- Load balancer metrics (ALB/NLB)

**2. Application**

- Request latency and throughput
- Error rates (4xx/5xx responses)
- Application logs for exceptions

**3. Business Metrics**

- Transactions per second
- User logins or purchases
- Queue lengths (SQS, Kafka)

#### 🔹 Step 2: Choose Monitoring Tools

**AWS-Native Options:**

- **CloudWatch Metrics** → infrastructure and service-level metrics
- **CloudWatch Logs / Logs Insights** → application and error logs
- **X-Ray** → distributed tracing for latency and bottlenecks
- **CloudTrail** → audit AWS API activity

**Open-Source / Third-Party:**

- **Prometheus + Grafana** → metrics collection and visualization
- **ELK Stack** (Elasticsearch, Logstash, Kibana) → log aggregation and analysis

#### 🔹 Step 3: Set Up Dashboards

- CloudWatch or Grafana dashboards for real-time monitoring
- Combine metrics and logs in a single view:
    - EC2 CPU / Memory vs Request Latency
    - Database IOPS vs Application Errors
- Visualize trends over time to detect anomalies

#### 🔹 Step 4: Define Alerts

- Set threshold-based alerts for critical metrics:
    - CPU > 80% for 5 minutes
    - Request error rate > 5%
    - Latency > 2 seconds
- Configure CloudWatch Alarms / Prometheus Alerts
- Integrate with notification channels:
    - SNS → Email, SMS, Slack
    - PagerDuty or OpsGenie for on-call escalation

#### 🔹 Step 5: Enable Logging & Tracing

- **Application logs** → CloudWatch Logs / ELK
- **Structured logging** → include request IDs and timestamps
- **X-Ray** → trace requests end-to-end for slow transactions

#### 🔹 Step 6: Automate Remediation (Optional)

- Auto-scaling based on metrics (CPU, request latency)
- Lambda functions to restart failed services or notify teams
- Integrate alerts with incident management tools

#### 🔹 Step 7: Test & Review

- Simulate failure scenarios (CPU spike, DB failure)
- Validate alert triggers and dashboard visibility
- Periodically review thresholds and alert rules
