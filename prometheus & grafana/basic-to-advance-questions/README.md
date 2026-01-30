🟢 Prometheus
1. What is Prometheus and why is it used?
Prometheus is an open-source monitoring and alerting system designed for metrics-based monitoring of applications and infrastructure.
It was originally built by SoundCloud and is now part of the Cloud Native Computing Foundation (CNCF).
In simple terms:

👉 Prometheus collects metrics, stores them, lets you query them, and triggers alerts.
🔹 Why is Prometheus used?
Prometheus is used to:
• Monitor applications and servers
• Track performance and availability
• Detect issues before outages
• Generate alerts when thresholds are crossed
• Monitor dynamic cloud and Kubernetes environments
🔹 Why Prometheus is Popular in DevOps
• Cloud-native & Kubernetes-friendly
• Handles ephemeral containers well
• No dependency on external storage
• Open-source & vendor-neutral
• Strong ecosystem (Grafana, Alertmanager)
2. Push vs Pull monitoring — why does Prometheus use pull?
🔹 Push vs Pull Monitoring (Quick Recap)
Push model
• Targets send (push) metrics to the monitoring system
• Example: CloudWatch, older monitoring tools
• Agent decides when and what to send
Pull model
• Monitoring system scrapes (pulls) metrics from targets
• Example: Prometheus
• Prometheus controls when and how often metrics are collected
🔹 Why Prometheus Uses the Pull Model
1️⃣ Better Reliability & Control
Prometheus decides:
• Scrape interval
• Timeout
• Target availability
If a target doesn’t respond → Prometheus knows it’s down

👉 With push, silence could mean network issue or app crash (hard to tell).
2️⃣ Service Discovery (Perfect for Kubernetes)
In dynamic environments:
• Pods are created & destroyed frequently
• IPs change constantly
Prometheus:
• Automatically discovers targets
• Starts scraping new services
• Stops scraping dead ones
🔥 This is huge for Kubernetes & microservices.
3️⃣ No Metrics Loss During Spikes
If Prometheus is slow or down:
• Pull model = targets don’t need to buffer metrics
• Prometheus resumes scraping later
With push:
• Targets may drop metrics
• Or overload the monitoring system
4️⃣ Simpler Security Model
• Only Prometheus needs network access
• Targets expose /metrics endpoint
• Easier firewall rules
With push:
• Every service needs credentials to push metrics
• Higher security risk
5️⃣ Consistent & Predictable Data
• Same scrape interval for all targets
• Easier comparisons across services
• Clean, uniform time-series data
6️⃣ Built-in Health Checking
If Prometheus can’t scrape:
• Target is unhealthy or unreachable
• This itself becomes a metric (up == 0)
💡 Monitoring system monitors availability by design.
🔹 But… When Is Push Useful?
Prometheus still supports push via Pushgateway, for:
• Short-lived jobs (batch jobs, cron jobs)
• Jobs that finish before scrape happens
⚠️ Pushgateway is an exception, not the default.
3. What are metrics? Name the metric types in Prometheus.
Metrics are numerical measurements collected over time that describe the state or performance of a system.
In monitoring, metrics answer questions like:
• How many requests are coming in?
• How fast is the application responding?
• How much CPU or memory is being used?
• Is the system healthy or failing?
In Prometheus, metrics are stored as time-series data, meaning:metric name + labels + timestamp + value
🔹 Metric Types in Prometheus
Prometheus supports four main metric types:
1️⃣ Counter
What it is:
• A value that only increases (or resets to zero on restart)
Used for:
• Counting events
Examples:
• HTTP requests
• Errors
• Jobs completed
Example metric:

http_requests_total

Important interview note:
• Counters are commonly used with rate() or increase() in PromQL
2️⃣ Gauge
What it is:
• A value that can go up and down
Used for:
• Current state measurements
Examples:
• CPU usage
• Memory usage
• Number of active connections
• Queue length
Example metric:

node_memory_available_bytes

3️⃣ Histogram
What it is:
• Samples observations and counts them in buckets
Used for:
• Request latency
• Response size
Provides:
• Bucket counts
• Sum of observations
• Total count
Example metrics:

http_request_duration_seconds_bucket
http_request_duration_seconds_sum
http_request_duration_seconds_count

4️⃣ Summary
What it is:
• Similar to histogram, but calculates quantiles (e.g. 95th percentile)
Used for:
• Latency percentiles
Key difference from histogram:
• Quantiles are calculated on the client side
• Not aggregatable across instances
Example metric:

http_request_duration_seconds{quantile="0.95"}

4. What is an exporter?
An exporter is a service or agent that collects metrics from a system and exposes them in Prometheus format, usually over an HTTP endpoint like /metrics.
👉 Prometheus scrapes exporters to collect metrics.
🔹 Why Are Exporters Needed?
Many systems:
• Don’t natively expose Prometheus metrics
• Use proprietary or legacy formats
Exporters act as a bridge between those systems and Prometheus.
🔹 How Exporters Work (Simple Flow)

System → Exporter → /metrics endpoint → Prometheus

Example:

Linux Server → Node Exporter → Prometheus

🔹 Common Prometheus Exporters
Here are some you should name in interviews:
🔹 Node Exporter
• Exposes OS-level metrics
• CPU, memory, disk, network
🔹 Blackbox Exporter
• Probes endpoints (HTTP, TCP, ICMP)
• Checks availability & latency
🔹 cAdvisor
• Container-level metrics
🔹 MySQL / PostgreSQL Exporter
• Database metrics
🔹 JVM Exporter
• Java application metrics
🔹 NGINX / Apache Exporter
• Web server metrics
5. What is a scrape interval?
A scrape interval is the frequency at which Prometheus pulls (scrapes) metrics from a target (application or exporter).
👉 In short:
How often Prometheus collects metrics.
🔹 Example
If:

scrape_interval: 15s

Prometheus will collect metrics every 15 seconds from the target.
🔹 Where Is It Defined?
• Globally (applies to all targets)
• Per job (overrides global value)
Example:

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "node"
    scrape_interval: 30s

🔹 Why Scrape Interval Matters
Choosing the right interval is a trade-off:
Short interval (e.g., 5s)
✅ Faster alerting

✅ More granular data

❌ Higher CPU, memory, and storage usage
Long interval (e.g., 60s)
✅ Lower resource usage

❌ Slower detection of issues

❌ Less detailed graphs
6. Difference between Prometheus and traditional monitoring tools (Nagios, Zabbix)?
FeaturePrometheusNagios / ZabbixMonitoring typeMetrics-basedCheck-basedData modelTime-seriesStatus checksArchitecturePull-basedMostly push-basedInfrastructureDynamicStaticCloud/K8s supportNativeLimited / complexService discoveryBuilt-inManualAlertingPromQL-basedThreshold-basedVisualizationGrafanaBuilt-in UIScalingHorizontalHarderUse caseMicroservices, K8sServers, legacy systems
7. Explain PromQL. What is it used for?
PromQL (Prometheus Query Language) is the query language used by Prometheus to retrieve, filter, aggregate, and analyze time-series metrics stored in Prometheus.
👉 In short:
PromQL is used to turn raw metrics into meaningful insights.
🔹 What is PromQL Used For?
PromQL is used to:
• Query metrics from Prometheus
• Aggregate and calculate rates
• Build Grafana dashboards
• Define alerting rules
• Perform troubleshooting & root-cause analysis
8. Difference between rate() and irate()?
Both functions calculate the per-second rate of increase of a counter metric.
The difference is how much data they use.
🔹 rate()
✅ What it does
• Calculates the average rate of increase over a time range
• Uses multiple data points in the range
📌 Syntax

rate(http_requests_total[5m])

👍 Best for
• Dashboards
• Alerting
• Stable trends
• SLO/SLA calculations
💡 Behavior
• Smooths out spikes
• Less noisy
• More reliable
🔹 irate()
⚡ What it does
• Calculates the instant rate
• Uses only the last two data points
📌 Syntax

irate(http_requests_total[5m])

👍 Best for
• Debugging
• Spotting sudden spikes
• Short-term analysis
⚠️ Behavior
• Very sensitive to spikes
• Can be noisy
• Not recommended for alerts
9. What is service discovery in Prometheus?
Service discovery in Prometheus is the mechanism by which Prometheus automatically finds targets (services, pods, nodes) to scrape metrics from without manual configuration.
👉 Instead of hard-coding IPs and ports, Prometheus dynamically discovers them.
🔹 Why Service Discovery Is Needed
Modern environments are dynamic:
• Pods are created and destroyed
• IPs change frequently
• Auto-scaling adds/removes instances
Manual target configuration does not scale.
🔹 How Service Discovery Works
1. Prometheus queries a service discovery source
2. Gets a list of targets + metadata (labels)
3. Applies relabeling rules
4. Starts scraping valid targets
5. Stops scraping removed targets
🔹 Types of Service Discovery in Prometheus
You should name at least 3–4 in interviews:
🔹 Static Config
• Manually defined targets
• Used in small or static setups

static_configs:
  - targets: ["10.0.0.1:9100"]

🔹 Kubernetes Service Discovery (Most Important)
• Discovers:
    ◦ Pods
    ◦ Services
    ◦ Nodes
    ◦ Endpoints

kubernetes_sd_configs:
  - role: pod

Labels from Kubernetes become Prometheus labels.
🔹 Cloud Providers
• AWS EC2
• GCP
• Azure
Prometheus discovers instances via cloud APIs.
🔹 File-Based Discovery
• Targets defined in files
• Files can be updated dynamically
10. How does Prometheus store data?
  Prometheus stores data as time-series data in a local time-series database (TSDB) on disk.
Each data point is stored as:

metric name + labels + timestamp + value

Example:

http_requests_total{method="GET", status="200"} @ 1700000000 → 15432

🔹 What Is a Time Series?
A time series is a stream of values over time for a unique combination of metric name and labels.
👉 Each unique label set = one time series.
🔹 Prometheus Storage Architecture (High Level)
Prometheus stores data in blocks on local disk:
1️⃣ In-Memory (Head Block)
• Recent samples are stored in memory
• Fast reads & writes
• Periodically flushed to disk
2️⃣ Persistent Storage (TSDB Blocks)
• Data written in immutable blocks
• Each block typically covers 2 hours
• Blocks contain:
    ◦ Samples
    ◦ Index
    ◦ Metadata
3️⃣ Write-Ahead Log (WAL)
• Every incoming sample is written to WAL
• Ensures crash recovery
• On restart, WAL is replayed
🔹 Retention Policy
Prometheus stores data for a limited time:
Example:

--storage.tsdb.retention.time=15d

Older data is automatically deleted.
11. What is Alertmanager?
Alertmanager is a component of the Prometheus ecosystem that manages alerts generated by Prometheus.
👉 Prometheus detects problems.

👉 Alertmanager handles and delivers those alerts.
🔹 What Does Alertmanager Do?
Alertmanager is responsible for:
• Deduplication – avoids duplicate alerts
• Grouping – combines related alerts
• Routing – sends alerts to the right team
• Inhibition – suppresses lower-priority alerts
• Notification delivery – email, Slack, PagerDuty, etc.
🔹 How Alertmanager Works (Flow)

Metrics → Prometheus → Alert rules → Alertmanager → Notifications

1. Prometheus evaluates alert rules
2. Fires alerts when conditions are met
3. Sends alerts to Alertmanager
4. Alertmanager processes & routes them
5. Notifications are sent
12. How do you monitor Kubernetes with Prometheus?
We monitor Kubernetes with Prometheus by deploying Prometheus inside the cluster, using exporters and Kubernetes service discovery to collect metrics from nodes, pods, containers, and cluster components, and then visualizing and alerting on those metrics.
🔹 Architecture (Interview-Friendly Flow)

Kubernetes Cluster
│
├─ Node Exporter → Node metrics
├─ kube-state-metrics → K8s object state
├─ cAdvisor → Container metrics
├─ App metrics (/metrics)
│
└─ Prometheus → Alertmanager → Grafana

13. How do you handle high cardinality issues?
High cardinality occurs when a metric has too many unique label combinations, creating a large number of time series.
Example of bad labels:
• user_id
• session_id
• request_id
• timestamp
Each unique value = new time series → memory & CPU explosion.
🔹 Why High Cardinality Is a Problem
• High memory usage
• Slow queries
• Increased disk usage
• Prometheus instability or crashes
💡 Prometheus performance depends heavily on number of time series, not just metrics count.
🔹 How Do You Handle High Cardinality Issues?
1️⃣ Avoid High-Cardinality Labels (Best Fix)
Do not use:
• User IDs
• Request IDs
• IP addresses (sometimes)
• Dynamic URLs
Instead:
• Use aggregated labels
• Bucket values (e.g. status code ranges)
2️⃣ Use Metrics, Not Logs
• Metrics = aggregated data
• Logs = detailed per-request data
👉 Use Loki / ELK for request-level details.
3️⃣ Relabeling to Drop Labels
Drop unnecessary labels before ingestion:

metric_relabel_configs:
  - regex: "request_id|user_id"
    action: labeldrop

4️⃣ Drop Unneeded Metrics
Reduce ingestion:

metric_relabel_configs:
  - source_labels: [__name__]
    regex: "http_request_duration_seconds_bucket"
    action: drop

5️⃣ Control Histogram Buckets
Too many buckets = more series.
• Reduce bucket count
• Use reasonable bucket ranges
6️⃣ Limit Scraped Targets
• Avoid scraping unnecessary pods
• Use annotations or ServiceMonitor selectors
• Filter targets with relabeling
7️⃣ Use Recording Rules
Precompute heavy queries:

record: job:http_requests:rate5m
expr: sum(rate(http_requests_total[5m])) by (job)

Benefits:
• Faster queries
• Lower CPU usage
8️⃣ Monitor Cardinality Itself (Interview Gold)
Use Prometheus metrics:

count({__name__=~".+"}) by (__name__)

or:

prometheus_tsdb_head_series

14. How does Prometheus scale?
Prometheus scales vertically by default, and for large or distributed systems it scales horizontally using federation or external systems like Thanos, Cortex, or Mimir.
🔹 Why Scaling Is Non-Trivial in Prometheus
• Prometheus is single-node per instance
• Uses local TSDB
• No built-in clustering
👉 This design keeps Prometheus simple and reliable, but requires architectural scaling.
🔹 Scaling Methods in Prometheus
1️⃣ Vertical Scaling (Default)
Increase:
• CPU
• Memory
• Disk
Used when:
• Small to medium environments
• Single cluster monitoring
⚠️ Limited by hardware.
2️⃣ Horizontal Scaling by Sharding
Split scrape targets across multiple Prometheus servers:
• Prometheus A → half the services
• Prometheus B → other half
Benefits:
• Reduces load per instance
• Independent failures
3️⃣ Federation (Hierarchical Scaling)
Lower-level Prometheus servers scrape metrics and a central Prometheus scrapes aggregated metrics from them.
Used for:
• Multi-cluster setups
• Global dashboards
Example:

Cluster Prometheus → Global Prometheus

4️⃣ Remote Storage / Long-Term Storage (Production Standard)
Use systems built on top of Prometheus:
🔹 Thanos
• Global querying
• Long-term storage (S3, GCS)
• High availability
🔹 Cortex / Mimir
• Horizontally scalable Prometheus backend
• Multi-tenant
• Used at very large scale
💡 Interview tip:“Prometheus handles collection; Thanos handles scale.”
5️⃣ High Availability (HA)
Run two Prometheus instances scraping the same targets:
• One active
• One standby
Alertmanager deduplicates alerts.
🔹 How Prometheus Scales in Kubernetes
• Multiple Prometheus instances per cluster
• Prometheus Operator manages lifecycle
• Sharding via shards config
• External storage via Thanos
15. What is federation? When would you use it?
Federation is a Prometheus feature where one Prometheus server scrapes metrics from other Prometheus servers instead of scraping exporters directly.
👉 In short:
Prometheus scraping Prometheus.
🔹 How Federation Works
• Child Prometheus instances:
    ◦ Scrape exporters in their local environment
• Parent (Federation) Prometheus:
    ◦ Scrapes selected metrics from child Prometheus servers via /federate
Example flow:

Exporters → Child Prometheus → Parent Prometheus

Only aggregated or important metrics are federated.
🔹 Example Use Case
• Each Kubernetes cluster has its own Prometheus
• A central Prometheus federates metrics from all clusters
• Used for global dashboards & alerts
🔹 Why Use Federation?
1️⃣ Scalability
• Reduces load on a single Prometheus
• Each instance handles a subset of targets
2️⃣ Multi-Cluster / Multi-Region Monitoring
• Local Prometheus per cluster
• Central Prometheus for high-level visibility
3️⃣ Isolation & Reliability
• If one cluster fails, others still work
• Prometheus instances fail independently
4️⃣ Reduced Cardinality at Top Level
• Only aggregated metrics are federated
• Prevents high cardinality explosion in central Prometheus
🔹 When Would You Use Federation?
Use federation when:
• You have multiple clusters or regions
• You need global dashboards
• You want hierarchical monitoring
• You don’t need raw per-pod metrics centrally
🔹 When NOT to Use Federation
• If you need long-term storage
• If you need raw metrics across clusters
• If you need global querying with deduplication
👉 Use Thanos / Cortex / Mimir instead.
16. Explain Thanos / Cortex / Mimir.
🔹 Why Thanos / Cortex / Mimir Exist (Context)
Prometheus by itself:
• Is single-node
• Stores data locally
• Has limited retention
• Has no global query
👉 Thanos, Cortex, and Mimir extend Prometheus to solve these problems at scale.
🔹 Thanos
🔸 What is Thanos?
Thanos is a high-availability and long-term storage layer built on top of Prometheus.
It keeps Prometheus unchanged and adds global visibility.
🔸 Key Features
• Global querying across Prometheus instances
• Long-term storage (S3, GCS, Azure Blob)
• High availability with deduplication
• Downsampling for cheaper storage
🔸 How Thanos Works
• Prometheus runs normally
• Thanos Sidecar ships blocks to object storage
• Thanos Querier reads from:
    ◦ Prometheus (real-time)
    ◦ Object storage (historical)
🔸 When to Use Thanos
✅ You already use Prometheus

✅ Multi-cluster / multi-region

✅ Need HA & long-term storage

✅ Want minimal changes
🔹 Cortex
🔸 What is Cortex?
Cortex is a horizontally scalable Prometheus backend.
Instead of local storage, metrics are:
• Pushed via remote_write
• Stored in object storage
• Queried centrally
🔸 Key Features
• Massive horizontal scalability
• Multi-tenancy
• Long-term storage
• Cloud-native
🔸 When to Use Cortex
✅ SaaS platforms

✅ Very large scale (millions of series)

✅ Strong multi-tenancy requirements
⚠️ More complex to operate.
🔹 Mimir
🔸 What is Mimir?
Grafana Mimir is the successor to Cortex, designed to be:
• Simpler
• Faster
• More scalable
Used by Grafana Cloud.
🔸 Key Features
• All Cortex features
• Better performance
• Easier operations
• Active development
🔸 When to Use Mimir
✅ New large-scale deployments

✅ Grafana ecosystem users

✅ Cloud-native monitoring at scale
17. How do you make Prometheus highly available?
🔹 Why Prometheus Needs HA
• Prometheus is single-node by default
• If it crashes or a node goes down → metrics collection stops
• Alerts may be missed
💡 HA ensures continuous monitoring and alerting, even if a Prometheus instance fails.
🔹 Ways to Make Prometheus Highly Available
1️⃣ Run Multiple Prometheus Instances
• Deploy two or more Prometheus servers scraping the same targets
• Each instance runs independently
• Use Alertmanager to deduplicate alerts
Example:

Prometheus-A → scrapes all targets
Prometheus-B → scrapes same targets
Alertmanager → deduplicates alerts

✅ Simple and effective

⚠️ Memory usage doubles (each instance stores its own data)
2️⃣ Use Alertmanager for HA
• Alertmanager should also be redundant
• Multiple Alertmanager instances:
    ◦ Clustered
    ◦ Deduplicate alerts
    ◦ Failover automatically
3️⃣ Use Remote Storage for HA
• Prometheus writes to Thanos, Cortex, or Mimir
• Benefits:
    ◦ Long-term storage
    ◦ Query from multiple replicas
    ◦ Failover if one Prometheus crashes
Thanos example:
• Prometheus Sidecar → object storage
• Thanos Querier → global queries
• Multiple Prometheus instances → deduplication
4️⃣ Sharding Targets
• For large environments:
    ◦ Split scrape targets across Prometheus instances
    ◦ Reduces load per instance
    ◦ Combined with remote storage for full picture
5️⃣ Kubernetes HA Deployment (Optional)
• Prometheus Operator supports replicas
• StatefulSets + PersistentVolumeClaims
• Works with Thanos Sidecar for HA
🔹 HA Best Practices
• Always deduplicate alerts in Alertmanager
• Use remote storage for backup & global querying
• Monitor Prometheus itself
• Avoid single point of failure (storage, network)
18. How do you secure Prometheus endpoints?
🔹 Why Securing Prometheus Endpoints Is Important
• Prometheus exposes metrics via HTTP /metrics endpoints
• Contains sensitive info (system metrics, app internals)
• Default Prometheus has no authentication
• Open endpoints can lead to:
    ◦ Information leakage
    ◦ Unauthorized access
    ◦ Security breaches
🔹 Key Strategies to Secure Prometheus
1️⃣ Use Network-Level Security
• Restrict access with firewalls or security groups
• Only allow access from:
    ◦ Monitoring servers
    ◦ Trusted admin IPs
• In Kubernetes:
    ◦ Use NetworkPolicies
    ◦ Limit pod access
2️⃣ Enable Authentication & Authorization
• Prometheus itself doesn’t support auth natively
• Use a reverse proxy (Nginx, Envoy, Traefik) in front
    ◦ Basic Auth or OAuth2
    ◦ Role-based access control (RBAC)
Example with Nginx:

location / {
    auth_basic "Prometheus Login";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://prometheus:9090;
}

3️⃣ Use TLS/HTTPS
• Encrypt traffic between Prometheus and clients
• Especially important for:
    ◦ Remote Prometheus queries
    ◦ Federated Prometheus setups
• Terminate TLS at reverse proxy or load balancer
4️⃣ Protect /metrics Endpoints of Targets
• Node Exporter, cAdvisor, kube-state-metrics, etc. also expose /metrics
• Options:
    ◦ Reverse proxy + auth
    ◦ Network isolation
    ◦ Use service accounts / RBAC in Kubernetes
5️⃣ Limit Remote Write Access
• Remote write endpoints should be authenticated
• Use API keys or TLS mutual auth if supported
6️⃣ Kubernetes Best Practices
• Run Prometheus in dedicated namespace
• Use NetworkPolicy to allow only scraping pods
• Avoid exposing Prometheus publicly unless needed
7️⃣ Monitor & Audit
• Log all access to Prometheus endpoints
• Watch for unusual queries or scraping attempts
19. Explain histogram vs summary — when would you choose one over the other?
FeatureHistogramSummaryPurposeObserves and counts values in bucketsCalculates quantiles (percentiles) directlyData storedCount per bucket + sum + total countQuantiles (e.g., 0.5, 0.95) + sum + countAggregation✅ Can aggregate across instances❌ Cannot aggregate quantiles across instancesUse caseLatency, request size, other distributionsLatency per instance when aggregation isn’t neededScraping overheadHigher (many series for buckets)Lower (fewer series)
🟢 Grafana
20. What is Grafana?
Grafana is an open-source visualization and analytics platform for monitoring and observability.
It allows you to:
• Query data from various sources (Prometheus, Loki, Elasticsearch, InfluxDB, MySQL, etc.)
• Visualize metrics in dashboards (graphs, charts, heatmaps, tables)
• Set up alerts based on metric thresholds
👉 In short: Prometheus collects the data, Grafana makes it human-readable and actionable.
🔹 Key Features of Grafana
1. Multiple Data Sources
    ◦ Prometheus, Loki, Elasticsearch, PostgreSQL, InfluxDB, etc.
2. Dashboards & Panels
    ◦ Interactive visualizations: graphs, tables, heatmaps, bar charts
3. Alerting
    ◦ Alerts based on thresholds or PromQL queries
    ◦ Integrated with Slack, Email, PagerDuty
4. Annotations
    ◦ Mark events (deployments, incidents) on dashboards for context
5. Templating & Variables
    ◦ Dynamic dashboards (filter by service, cluster, or node)
6. Plugins & Extensibility
    ◦ Community plugins for new panels, data sources, and apps
🔹 How Grafana Works with Prometheus
• Prometheus exposes metrics via /metrics
• Grafana connects to Prometheus as a data source
• Users create dashboards with PromQL queries to visualize metrics
• Alerts can be configured in Grafana using the same queries
21. Difference between Grafana and Prometheus?
FeaturePrometheusGrafanaPurposeMonitoring & alertingVisualization & analyticsFunctionCollects, stores, and queries metricsVisualizes metrics and dashboardsData SourceMetrics from exporters or instrumented appsConnects to Prometheus, Loki, Elasticsearch, InfluxDB, etc.AlertingBuilt-in alerting via rules → AlertmanagerAlerts based on dashboard queriesStorageLocal TSDB or remote storage (Thanos/Cortex)No native metric storageQuery LanguagePromQLUses PromQL (for Prometheus) or SQL (for other sources)VisualizationMinimal (basic graphs in Prometheus UI)Rich, interactive dashboards and panels
22. What are dashboards and panels?
Dashboards are collections of panels that provide a single view of your metrics and observability data.
• Think of a dashboard as a monitoring page for a system, service, or cluster.
• Dashboards can include multiple panels, variables, and filters.
• They help you visualize, track trends, and detect anomalies in one place.
Examples:
• Kubernetes cluster health dashboard
• Application latency dashboard
• Node or server resource usage dashboard
🔹 What Are Panels?
Panels are the individual visualization units inside a dashboard.
• Each panel displays one query or a set of queries.
• Panels can take many forms:
    ◦ Graphs (time-series)
    ◦ Tables
    ◦ Singlestat (single numeric value)
    ◦ Heatmaps, pie charts, bar charts
Examples:
• CPU usage graph for a node
• HTTP request rate per service
• Error count table per microservice
23. What data sources does Grafana support?
A data source in Grafana is the system from which Grafana queries metrics, logs, or traces to display in panels.
• Grafana connects to many types of systems.
• Each data source has its own query language and features.
🔹 Key Data Sources Grafana Supports
1️⃣ Prometheus
• Query language: PromQL
• Metrics-focused
• Example use: application latency, request rate, CPU/memory metrics
2️⃣ Loki
• Log aggregation system
• Query language: LogQL
• For logs visualization and correlation with metrics
3️⃣ Elasticsearch
• Query language: Lucene / Elasticsearch Query DSL
• Logs or time-series data
• Useful for monitoring, search, and logs analytics
4️⃣ InfluxDB
• Time-series database
• Query language: InfluxQL / Flux
• Works for metrics like sensor data, server stats, IoT
5️⃣ Graphite
• Legacy time-series database
• Query language: Graphite functions
• Still used in some legacy monitoring environments
6️⃣ MySQL / PostgreSQL / MSSQL
• SQL databases
• Query language: SQL
• Useful for business metrics, custom dashboards
7️⃣ Cloud Monitoring Systems
• AWS CloudWatch
• Google Stackdriver (GCP Monitoring)
• Azure Monitor
• Query metrics, logs, and alarms from cloud providers
8️⃣ Tracing & Observability
• Jaeger (distributed tracing)
• Tempo (traces)
• OpenTelemetry collectors
9️⃣ Others
• JSON API
• Microsoft SQL Server
• ClickHouse, Loki, Tempo
• Many community plugins for new sources
24. How does Grafana query Prometheus?
🔹 How Grafana Queries Prometheus
1. Grafana connects to Prometheus as a data source
    ◦ In Grafana, you configure Prometheus with:
        ▪ URL (e.g., http://prometheus:9090)
        ▪ Access method (proxy or direct)
        ▪ Optional authentication (basic auth, TLS)
2. Panels use PromQL queries
    ◦ Each panel has a query editor
    ◦ You write PromQL to fetch metrics
    ◦ Grafana sends the query via HTTP to Prometheus’s /api/v1/query or /api/v1/query_range endpoints
3. Prometheus executes the query
    ◦ Returns JSON results:
        ▪ Time series
        ▪ Labels
        ▪ Values
    ◦ Can be:
        ▪ Instant query → single timestamp
        ▪ Range query → series over a time range
4. Grafana renders the panel
    ◦ Converts JSON results into:
        ▪ Graphs
        ▪ Tables
        ▪ Singlestat panels
        ▪ Heatmaps, etc.
25. Variables in Grafana — why are they useful?
🔹 What Are Variables in Grafana?
Variables are dynamic placeholders you can use in queries, panel titles, or dashboards.
• Defined at the dashboard level
• Can be reused across multiple panels
• Can be dropdowns, multi-select, or custom values
🔹 Why Variables Are Useful
1️⃣ Dynamic Dashboards
• One dashboard can monitor multiple clusters, services, or environments
• Instead of creating separate dashboards, you use a variable:

rate(http_requests_total{job="$job"}[5m])

• $job can be a dropdown with all jobs (services) discovered by Prometheus
2️⃣ Reduce Dashboard Duplication
• Variables allow reuse of panels
• Example: same CPU panel works for multiple nodes
3️⃣ Interactive Filtering
• Users can select:
    ◦ Service
    ◦ Namespace
    ◦ Pod
    ◦ Region
• Panels automatically update with the selection
4️⃣ Powerful Query Composition
• Variables can be used in PromQL, table queries, or annotations
• Enable dashboards to adapt to changing infrastructure
5️⃣ Multi-Select & Regex
• Variables can select multiple values at once
• Regex can filter which targets to show
26. How do you create alerts in Grafana?
🔹 How Alerts Work in Grafana
Grafana alerts are triggered based on data from panels, usually from metrics in Prometheus.
• Alerts evaluate a condition on a time-series query.
• If the condition is met, Grafana sends notifications to configured channels.
• Grafana supports alert rules at the panel level or centralized alerting.
🔹 Steps to Create Alerts in Grafana
1️⃣ Configure a Data Source
• Ensure Prometheus (or another metric source) is added to Grafana.
2️⃣ Create a Panel
• Create a graph or visualization panel displaying the metric you want to monitor.
3️⃣ Define an Alert Rule
• In the panel:
    ◦ Go to the Alert tab
    ◦ Click “Create Alert”
• Set:
    ◦ Condition (threshold, increase, rate)
    ◦ Evaluation interval (e.g., every 1 minute)
    ◦ For duration (e.g., fire alert only if condition persists 5 min)
Example condition:

avg(rate(http_requests_total{job="auth-service"}[5m])) > 100

• Fires alert if average request rate > 100 for 5 minutes
4️⃣ Set Notification Channel
• Configure channels for alert delivery:
    ◦ Slack
    ◦ Email
    ◦ PagerDuty
    ◦ Webhook
• Link the alert rule to the notification channel.
5️⃣ Test & Save
• Verify the alert triggers correctly using Test Rule
• Save the dashboard and alert configuration
🔹 Advanced Options
• Multiple conditions per alert (AND/OR)
• Annotations on graphs when alerts fire
• Templated dashboards: Alerts dynamically change based on variables
• Contact policies: Route alerts to different teams
27. What is a Grafana annotation?
🔹 What Is a Grafana Annotation?
Annotations are markers or notes on a Grafana graph/panel that indicate events or incidents.
• They overlay on time-series graphs.
• Provide context to metrics.
• Can be manual or automatic.
🔹 Types of Annotations
1️⃣ Manual Annotations
• Added directly in the Grafana UI.
• Example: “Deployment started” at 10:05 AM.
• Useful for marking deployments, incidents, or maintenance.
2️⃣ Automatic / Query-Based Annotations
• Fetched from a data source (Prometheus, Elasticsearch, MySQL, etc.)
• Example: Prometheus alert firing → annotation automatically appears on the graph.
• Useful for linking metrics with events programmatically.
🔹 Why Annotations Are Useful
1. Provide Context
    ◦ Helps explain spikes or anomalies in metrics.
    ◦ Example: “CPU spike corresponds to deployment rollout.”
2. Correlation Between Metrics and Events
    ◦ Correlate alerts, deployments, or incidents with time-series data.
3. Audit / Troubleshooting
    ◦ Helps teams investigate incidents faster.
🔹 Example Use Case
1. Prometheus fires an alert: High CPU usage
2. Grafana query-based annotation appears at alert time on the dashboard
3. Overlay shows: deployment, crash, or incident
4. Engineers immediately understand why CPU spiked
28. How do you manage dashboards as code?
🔹 What Does “Dashboards as Code” Mean?
Dashboards as code means managing Grafana dashboards in a version-controlled format (JSON, YAML, or via tooling) rather than manually through the UI.
• Dashboards are stored in Git
• Changes are tracked and auditable
• Dashboards are automatically deployed to Grafana
🔹 Ways to Manage Grafana Dashboards as Code
1️⃣ JSON Export / Import
• Grafana dashboards are stored as JSON
• Export a dashboard → store in Git → import to Grafana
• Simple but manual deployment
2️⃣ Grafana Provisioning
• Grafana supports provisioning dashboards via YAML/JSON
• Place dashboard files in /etc/grafana/provisioning/dashboards/
• Grafana loads dashboards on startup automatically
• Example:

apiVersion: 1
providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    options:
      path: /var/lib/grafana/dashboards

3️⃣ Grafana Terraform Provider
• Use Terraform to define dashboards as code
• Example:

resource "grafana_dashboard" "app_dashboard" {
  config_json = file("${path.module}/dashboards/app.json")
}

• Supports full lifecycle management: create, update, delete
• Ideal for CI/CD pipelines
4️⃣ Grafana API
• Grafana provides a REST API to create/update dashboards
• Can be automated via scripts or CI/CD pipelines
5️⃣ Grafonnet / Jsonnet
• Jsonnet library to generate dynamic, templated dashboards
• Useful for multi-environment dashboards
🔹 Benefits of Managing Dashboards as Code
1. Version Control
    ◦ Track changes, rollbacks, and approvals in Git
2. Consistency
    ◦ Same dashboards deployed across environments
3. Automation
    ◦ CI/CD pipelines can deploy dashboards automatically
4. Collaboration
    ◦ Teams can review changes via pull requests
29. Difference between table panel and time-series panel?
FeatureTime-Series PanelTable PanelPurposeVisualize metrics over timeDisplay metrics or logs in a tabular formatData TypeTime-series (timestamps + values)Tabular data (rows and columns)Best ForTrends, graphs, spikes, latency, CPU/memory usageDetailed lists, status tables, top N metrics, logsVisualizationLine, bar, area, heatmapTable with sortable columns, thresholds, linksQuery Examplerate(http_requests_total[5m])topk(5, sum(rate(http_requests_total[5m])) by (service))Annotations / AlertsCan overlay annotations, visualize thresholdsAlerts can be applied, but more for visual reporting
30. How do you handle multi-tenant dashboards?
🔹 What Is a Multi-Tenant Dashboard?
Multi-tenant dashboards allow multiple users or teams (“tenants”) to:
• Use the same Grafana instance
• Have separate dashboards, data sources, and permissions
• Avoid interfering with each other’s metrics or alerts
🔹 Strategies to Handle Multi-Tenancy in Grafana
1️⃣ Grafana Organizations
• Grafana natively supports organizations.
• Each tenant/team can have:
    ◦ Separate dashboards
    ◦ Separate data sources
    ◦ Separate users
• Users belong to one or more orgs, with role-based permissions (Admin, Editor, Viewer)
Example:

Org A → Dev dashboards + Prometheus data source
Org B → QA dashboards + Prometheus data source

2️⃣ Folder & Dashboard Permissions
• Dashboards can be grouped in folders
• Set read/write permissions per folder for teams
3️⃣ Data Source Isolation
• Each tenant can have dedicated data sources
• Useful if tenants have:
    ◦ Separate Prometheus instances
    ◦ Separate clusters
• Prevents one team from querying another team’s data
4️⃣ Dashboard Variables / Templating
• Use templated variables for shared dashboards across tenants
• Example:

rate(http_requests_total{cluster="$cluster"}[5m])

    ◦ $cluster variable changes per tenant
• Reduces dashboard duplication while still isolating views
5️⃣ API & Automation
• Manage tenants programmatically with Grafana API or Terraform provider
• Automate:
    ◦ Dashboard provisioning
    ◦ Folder & permission assignment
    ◦ Data source creation per tenant
31. Grafana performance tuning techniques?
🔹 Key Areas for Grafana Performance Tuning
1️⃣ Optimize Data Sources & Queries
• Use Prometheus recording rules instead of heavy PromQL queries in dashboards
    ◦ Precompute rates, aggregates, or percentiles
• Avoid count/sum on massive time-series without aggregation
• Use time ranges and resolution carefully
    ◦ Reduce panel queries frequency (Min interval setting)
2️⃣ Reduce Panel & Dashboard Load
• Avoid too many panels in a single dashboard
• Combine metrics where possible
• Use templating/variables carefully:
    ◦ Multi-select variables with many values can explode queries
• Use row collapse or multiple dashboards instead of one mega-dashboard
3️⃣ Caching & Refresh Intervals
• Set reasonable dashboard refresh intervals (e.g., 30s–1min)
• Enable query caching (depends on data source, e.g., Prometheus proxy caching)
4️⃣ Grafana Server Tuning
• Increase Grafana server resources (CPU/memory) for large installations
• Use Grafana 9+ or newer versions for better rendering performance
• Enable parallel queries (concurrent_render_limit) for panels with many queries
5️⃣ Use Provisioned Dashboards / Dashboards as Code
• Pre-load dashboards via provisioning instead of creating them in the UI
• Reduces UI rendering overhead
6️⃣ Scale Grafana for Multi-User Access
• For large teams or enterprise setups:
    ◦ Run multiple Grafana instances behind a load balancer
    ◦ Use Redis or database caching for session & alert storage
• Use Grafana Enterprise for built-in scaling features (optional)
7️⃣ Minimize Heavy Visualizations
• Avoid too many heatmaps or high-cardinality visualizations in one panel
• Use singlestat or table panels for high-density metrics
8️⃣ Alerting Performance
• Use Prometheus alerting rules instead of Grafana alerts when possible
• Alerts in Grafana query Prometheus frequently → can increase load
32. How do you integrate Grafana with authentication systems?
🔹 Common Methods to Integrate Authentication
1️⃣ LDAP / Active Directory
• Grafana connects to LDAP/AD server
• Users can log in with corporate credentials
• Supports mapping LDAP groups to Grafana roles (Admin, Editor, Viewer)
Example Config:

[auth.ldap]
enabled = true
config_file = /etc/grafana/ldap.toml
allow_sign_up = true

2️⃣ OAuth / SSO
• Grafana supports OAuth2 providers: Google, GitHub, Okta, Azure AD
• Users log in via the provider
• Roles can be assigned based on claims or groups
• Great for multi-tenant environments or cloud setups
Example:

[auth.google]
enabled = true
client_id = YOUR_CLIENT_ID
client_secret = YOUR_CLIENT_SECRET
scopes = https://www.googleapis.com/auth/userinfo.profile https://www.googleapis.com/auth/userinfo.email

3️⃣ SAML
• Enterprise SSO via SAML providers (Okta, OneLogin, Azure AD)
• Centralized authentication for Grafana
• Supports role mapping from SAML attributes
4️⃣ Proxy Authentication
• Reverse proxy handles auth (e.g., Nginx, Traefik)
• Grafana trusts headers like X-WEBAUTH-USER
• Useful for internal deployments behind a corporate gateway
5️⃣ Role-Based Access Control (RBAC)
• After authentication, assign roles: Admin, Editor, Viewer
• Can be org-level or folder-level
• Combined with multi-tenant dashboards for access isolation
33. How do you restrict dashboard access?
🔹 Ways to Restrict Dashboard Access in Grafana
1️⃣ Organizations
• Grafana supports multiple organizations.
• Users belong to one or more orgs.
• Dashboards are isolated per organization.
Example:

Org A → Dev dashboards
Org B → QA dashboards

2️⃣ Folders and Folder Permissions
• Dashboards can be grouped in folders.
• Permissions can be set per folder:
    ◦ Viewer → can only view dashboards
    ◦ Editor → can edit dashboards
    ◦ Admin → full control
• Useful for separating dashboards by team, environment, or service.
3️⃣ Dashboard Permissions
• Grafana allows dashboard-level permissions.
• You can explicitly allow or deny:
    ◦ Individual users
    ◦ Teams
    ◦ Roles (Viewer, Editor, Admin)
Example:

Dashboard X → Editors: Team A, Viewers: Team B

4️⃣ Role-Based Access Control (RBAC)
• Roles can be applied:
    ◦ Organization level
    ◦ Folder level
    ◦ Dashboard level
• Admin, Editor, Viewer roles control what actions users can perform.
5️⃣ Data Source Restrictions
• Limit which data sources a user or team can access.
• Users with no access to a data source cannot view dashboards using that source.
6️⃣ Authentication Integration
• Use SSO / LDAP / OAuth / SAML to map users to Grafana roles or teams.
• Combine with folder and dashboard permissions for fine-grained access control.
7️⃣ Proxy or Network Restrictions
• Restrict access to Grafana using network policies, firewalls, or reverse proxies.
• Useful for internal dashboards or multi-tenant isolation.
⚙️ Scenario-Based Questions
34. CPU usage is spiking — how do you debug using Prometheus & Grafana?
🔹 Step 1: Confirm the Spike
1. Open Grafana dashboard for CPU metrics.
2. Check time-series graphs for the spike:
    ◦ Node CPU usage (node_cpu_seconds_total)
    ◦ Pod/container CPU usage (container_cpu_usage_seconds_total) if Kubernetes
3. Verify if the spike is sustained or a short burst.
💡 Look at raw metrics and aggregated values.
🔹 Step 2: Identify the Scope
• Is the spike system-wide or application-specific?
• Use PromQL queries to break it down:
Example: Per Node CPU Usage

100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

Example: Per Pod in Kubernetes

sum by(pod) (rate(container_cpu_usage_seconds_total[5m]))

• This helps isolate which nodes or pods are contributing most to CPU usage.
🔹 Step 3: Check Process / Service Contribution
• If Kubernetes, use label filters:

sum by(namespace, pod, container) (rate(container_cpu_usage_seconds_total[5m]))

• Identify top CPU-consuming pods/services using topk():

topk(5, sum by(pod) (rate(container_cpu_usage_seconds_total[5m])))

• This shows which services or pods are causing the spike.
🔹 Step 4: Correlate with Other Metrics
1. Memory usage (container_memory_usage_bytes) → Is CPU spike correlated with memory pressure?
2. Disk / I/O (node_disk_io_time_seconds_total) → Are there I/O bottlenecks?
3. Network metrics → Is network saturation affecting CPU?
4. Application metrics → Requests per second, error rates
💡 Use Grafana panels and annotations to correlate events (deployments, cron jobs, alerts).
🔹 Step 5: Check Events / Logs
• Use Loki or other log data source in Grafana:
    ◦ Filter logs for the time range of the CPU spike
    ◦ Look for errors, slow queries, or heavy tasks
🔹 Step 6: Drill Down / Alert Investigation
• Check Prometheus alerting rules (if CPU threshold alerts exist)
• Check if the spike triggered Alertmanager notifications
• Correlate with dashboards: which pods, namespaces, or nodes were affected
🔹 Step 7: Long-Term Analysis
• Compare spike with historical trends:
    ◦ Is it daily/hourly load pattern?
    ◦ Are specific jobs or cron tasks causing the spike?
• Use Prometheus recording rules to pre-aggregate metrics for easier comparison.
🔹 Step 8: Take Action
• Based on analysis, options may include:
    ◦ Scaling pods or nodes
    ◦ Optimizing the application/service
    ◦ Throttling batch jobs
    ◦ Scheduling heavy tasks at off-peak times
35. A Prometheus alert is firing but everything looks fine — what do you do?
🔹 Step 1: Don’t Panic — Confirm the Alert
• Check Alertmanager or Grafana notification
• Look at the alert name, labels, and firing time
• Note which targets/instances triggered the alertExample: HighCPUUsage alert fired for pod auth-service
🔹 Step 2: Check the Query in Prometheus
1. Go to Prometheus “Alerts” or “Graph” tab
2. Copy the PromQL expression from the alert rule
3. Run it manually for the same time range
• Check if the query is currently evaluating to true
• If the metric looks normal, it may be a query or threshold issue
🔹 Step 3: Check Evaluation Interval & ‘For’ Duration
• Alerts in Prometheus can have a for clause:

alert: HighCPU
expr: rate(cpu_usage_seconds_total[5m]) > 80
for: 5m

• If spike was shorter than 5 minutes, alert may still fire due to evaluation delay
• Similarly, very short intervals may cause flapping alerts
🔹 Step 4: Look for Stale or Missing Metrics
• Metrics may be temporarily missing or delayed due to:
    ◦ Node exporter scrape failure
    ◦ Pod restart
    ◦ Network issues
• Prometheus treats absent metrics as zero or may trigger alert depending on alert logic
Check with PromQL:

absent(cpu_usage_seconds_total{job="auth-service"})

🔹 Step 5: Check for High Cardinality or Label Changes
• Alerts may fire unexpectedly if labels change or new pods are added
• Example: rate(cpu_usage_seconds_total[5m]) with per-pod aggregation may include new pods temporarily
🔹 Step 6: Check Alertmanager Deduplication & Silences
• Alert may appear firing in Alertmanager, but:
    ◦ It could be a duplicate firing
    ◦ A previous alert was silenced or pending
• Verify Alertmanager UI for pending or silenced alerts
🔹 Step 7: Check Grafana or External Dashboards
• Cross-check metrics in Grafana
• Compare time ranges used in alert rule vs dashboard
• Confirm the spike or anomaly exists
🔹 Step 8: Adjust Alert Rule if Needed
• Increase for duration to avoid flapping
• Add unless absent() clauses to prevent firing on missing metrics
• Refine PromQL expression for better accuracy
🔹 Step 9: Document and Monitor
• Log the false positive and adjust rules if needed
• Set up dashboard panels to cross-verify alerts before notifying team
36. Metrics are missing for a service — troubleshooting steps?
🔹 Step 1: Verify the Service is Up
• Ensure the service or exporter is running and healthy
• Check:
    ◦ Service pod/container status (kubectl get pods)
    ◦ Logs for errors
    ◦ Network connectivity
🔹 Step 2: Check the Exporter
• If Prometheus scrapes metrics via an exporter (Node Exporter, App Exporter):
    ◦ Is the exporter running?
    ◦ Can you access the /metrics endpoint directly?

curl http://<exporter-host>:9100/metrics

    ◦ Ensure the metrics for your service are actually exposed
🔹 Step 3: Verify Prometheus Scrape Config
• Check prometheus.yml or config map:
    ◦ Job name exists
    ◦ Targets include the service
    ◦ Correct port, path (/metrics)
• Access Prometheus Targets page (/targets)
    ◦ Check if the target is UP or DOWN
    ◦ Look at last scrape time
🔹 Step 4: Check Labels and Relabeling
• Misconfigured labels or relabeling can cause metrics to be missing
• Ensure metric name and labels match Prometheus queries
• Example: If a service exposes http_requests_total, check that your query is correct:

rate(http_requests_total{job="my-service"}[5m])

🔹 Step 5: Check Prometheus Logs
• Prometheus logs may indicate:
    ◦ Scrape failures
    ◦ Connection timeouts
    ◦ TLS/authentication errors
🔹 Step 6: Check Metric Retention & Query Time Range
• Ensure query is within retention period
• Metrics older than --storage.tsdb.retention.time may not appear
• Adjust time range in Grafana accordingly
🔹 Step 7: Check for High Cardinality / Missing Series
• Sometimes metrics exist but with unexpected labels
• Use Prometheus query to list series:

{job="my-service"}

• Verify that expected series appear
🔹 Step 8: Network / Firewall Checks
• Ensure Prometheus can reach the service/exporter over network
• Check firewall rules, service endpoints, security groups
🔹 Step 9: Check for Application Changes
• Metrics may have been removed or renamed in a new release
• Check changelogs or application code for exposed metrics
🔹 Step 10: Test in Isolation
• Temporarily use direct Prometheus query or curl to fetch metrics
• Verify that the service produces metrics correctly
37. How would you monitor a batch job vs a long-running service?
🔹 Monitoring a Long-Running Service
Examples: Web service, API server, microservice.
Key Metrics to Monitor
1. CPU / Memory / Disk / Network
    ◦ node_cpu_seconds_total, container_memory_usage_bytes, container_fs_usage_bytes
2. Application metrics
    ◦ Request rate, latency, error rate (http_requests_total, http_request_duration_seconds)
3. Availability
    ◦ Uptime, pod or process status
4. Custom business metrics
    ◦ Orders processed, queue depth
Monitoring Approach
• Scrape metrics continuously via Prometheus exporters or instrumentation libraries (client libraries in Go, Java, Python, etc.)
• Dashboards in Grafana with time-series panels
• Alerts via Prometheus Alertmanager
    ◦ e.g., high CPU for >5 mins, error rate >5%
• Annotations for deployments or incidents
Key Points:
• Metrics are continuous → Prometheus works well
• Use histograms or summaries for latency metrics
• Monitor trends over time
🔹 Monitoring a Batch Job
Examples: Data processing job, nightly ETL, cron job.
Key Metrics / Checks
1. Job completion
    ◦ Did it finish successfully?
    ◦ Status: success/failure
2. Duration / Runtime
    ◦ How long did the job take?
    ◦ Compare to historical averages
3. Errors or warnings
    ◦ Failed records, exceptions
4. Resource usage (optional)
    ◦ CPU, memory, disk I/O during job run
Monitoring Approach
• Push metrics at the end of the job (Prometheus pushgateway)
    ◦ Since the job is short-lived, Prometheus can’t scrape it while running
• Expose metrics via job logs and parse them into Prometheus or Loki
• Alerts on failures or unusually long runtimes
• Dashboards can track job history over time
Key Points:
• Jobs are ephemeral → Prometheus pull model won’t see them unless they push metrics
• Focus is on completion and performance, not continuous resource usage
