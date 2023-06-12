# AWS Certified Developer Notes

---

# IAM AWS CLI

- **Groups**
    - Only contain users, not other groups.
    - A user doesn't have to belong to a group. Users can belong to multiple groups.

- **Policies**
    - Give permissions to users and groups.
    - Users can be assigned many policies, belonging to different groups.
  
    example of Dynamic policy
  ```
  {
    "Effect": "AllowAllS3ActionsInUserFolder",
    "Action": "s3:*",
    "Resource": "arn:aws:s3:::your-bucket-name/user-folder/${aws:username}/*"
  }
  ```
- **MFA**
    - Google Authenticator (phone only)
    - Authy (multi-device), support for multiple tokens on a single device
    - Universal 2nd Factor (U2F) Security Key, support multiple users using 1 key

- **Access Keys**
    - To use CLI and SDK you need access keys.
    - They are generated through the AWS Console and users manage their own keys.

- **Roles**
    - AWS services need permissions, we grant those by attaching Roles.

---

# EC2 Instance and Storage

### Naming Example:

- Example: m5.2xlarge
    - m: Instance class
    - 5: Generation
    - 2xlarge: Size within the instance class

### Types:

- General Purpose
    - t2
- Compute Optimized
    - C
- Memory Optimized
    - R
- Storage Optimized
    - ...

### Instance Purchasing Options:

- On-Demand: Short workloads, predictable pricing, pay per second
- Reserved: 1-year and 3-year terms, for long workloads
- Saving Plans: 1-year and 3-year commitments, for a specific amount of usage, for long workloads
- Spot Instances: Short workloads, cost-effective, can lose instances
- Dedicated Hosts: Book an entire physical server, control instance placement
- Dedicated Instances: No other customers will share your hardware
- Capacity Reservations: Reserve capacity in a specific Availability Zone for any duration

### EBS (Elastic Block Store):

- Network drive
- Persists data
- Mount to only one instance
- Only available in specific Availability Zones
- Snapshots can be taken and used to move data between Availability Zones

### EBS Snapshots:

- Backups of EBS volumes
- Can be restored to another Availability Zone
- Can be archived for cost savings (75% cheaper)
- Snapshots can go to a recycle bin with specified retention
- Fast Snapshot Restore (FSR) for low-latency on first use (expensive)

### AMI (Amazon Machine Image):

- Customizations of EC2 instances
- Includes your own software, configurations, and OS
- Faster boot and configuration time due to pre-packaged software
- Limited to a specific AWS Region
- Custom AMIs can be created from existing EC2 instances and launched in new Availability Zones

### EC2 Instance Store:

- High-performance hardware disk
- Best I/O performance
- Ephemeral storage (risk of data loss)
- Suitable for buffer/cache/scratch data, temporary content, etc.

### EBS Volume Types:

- gp2/gp3 (SSD): General purpose, balanced, IOPS increase with disk size
- io1/io2 (SSD): Highest performance, independent IOPS configuration
- st1 (HDD): Low-cost, for frequently accessed throughput-intensive workloads
- sc1 (HDD): Lowest cost, for less frequently accessed workloads

### EBS Multi-Attach (io1/io2):

- Attach a single EBS volume to multiple EC2 instances in the same Availability Zone
- Achieves higher availability in clustered systems
- Supports up to 16 instances simultaneously

### EFS (Elastic File System):

- Network file system that can be mounted on multiple EC2 instances
- Supports multi-AZ deployment
- Highly available and scales automatically (but expensive)
- Used in content management, web serving, data sharing, WordPress hosting, etc.
- Compatible only with Linux-based AMIs (not Windows)
- EFS Scale: 1000 concurrent NFS clients, 10 GB+/s throughput
- Performance Modes:
    - General (default): For web servers and CMS
    - Max I/O: Higher latency, throughput, highly parallel (big data, media processing)
- Throughput Modes:
    - Bursting
    - Provisioned, set throughput

### EFS Storage Classes

#### Storage Tiers

- Standard: Frequent access
- Infrequent Access (EFS-IA): Costs to retrieve files, lower price to store

#### Availability and Durability

- Standard: Multi-AZ, great for production
- One Zone: Single AZ, great for development, default backup, compatible with IA

---

# ELB, ASG

### Types:

- Classic Load Balancer (deprecated)
- Application Load Balancer
    - Layer 7 (HTTP, HTTPS, WebSocket)
- Network Load Balancer
    - TCP, TLS, UDP
- Gateway Load Balancer
    - Layer 3

### ELB (Elastic Load Balancer):

- Spreads load across multiple instances
- Exposes a single point of access to your app
- Handles failures of instances
- Performs health checks
- Provides SSL termination (HTTPS) for websites
- Enforces stickiness with cookies
- Ensures high availability across Availability Zones
- Separates public from private traffic

### ALB (Application Load Balancer):

- Layer 7 (HTTP)
- Load balancing to multiple HTTP apps across machines
- Load balancing to multiple apps on the same machine (e.g., containers)
- Supports redirects (e.g., HTTP to HTTPS)
- Supports routing based on path in URL, hostname, query string, headers, etc.
- Port mapping feature
- Great for microservices and container-based apps

### NLB (Network Load Balancer):

- Forwards TCP and UDP traffic to instances
- Handles millions of requests per second
- Low latency
- Provides one static IP per Availability Zone, supports assigning Elastic IP
- Not free
- Can communicate with ALB

### GLB (Gateway Load Balancer):

- Deploys, scales, and manages 3rd-party traffic in AWS
- Examples: firewalls, intrusion detection/prevention systems, deep packet inspection, payload manipulation
- Layer 3 (IP packets)
- Acts as a transparent network gateway, serving as a single entry/exit point for all traffic
- Uses GENEVE protocol on port 6081

### Sticky Sessions (Affinity):

- With stickiness, the same client will always be redirected to the same instance behind a load balancer
- Available for Classic Load Balancer and Application Load Balancer
- Use case: Ensuring a user doesn't lose their session data

### Cross-Zone Balancing:

- When enabled, each load balancer distributes traffic evenly across all registered instances in all Availability Zones
- Enabled by default only on Application Load Balancer

### SSL/TLS Certs

- With an SSL (or TLS) cert, traffic is encrypted in transit (in-flight) between clients and the load balancer.
- SSL certs can be managed with ACM (AWS Certificate Manager).
- For HTTPS listener:
    - Must specify a default cert.
    - Can add an optional list of certs for many domains.
    - Clients can use SNI to specify the hostname they reach.

### SNI (Server Name Indication)

- Used for loading multiple SSL certs on one web server (for multiple websites).
- Does not work with CLB (Classic Load Balancer).
- The client is required to specify the hostname of the target in the SSL handshake.

### Connection Draining (or Deregistration Delay)

- Specifies the time to complete existing connections/requests while the particular instance is de-registering or
  unhealthy.
- Can be disabled, but not recommended for long processes (e.g., uploading something).

### ASG (Auto Scaling Groups)

- Automatically adds or moves instances based on load (scale-in and scale-out).
- Automatically registers new instances with the load balancer.
- Free to use.
- ASG can scale based on CloudWatch alarms.
- Scaling policies:
    - Target tracking
    - Simple/Step
    - Scheduled actions
    - Predictive
- Scaling metrics:
    - CPUUtilization
    - RequestCountPerTarget
    - Avg network I/O
    - Other custom metrics
- Scaling cooldown:
    - Cooldown period after a scaling activity.
    - During this period, ASG will not launch or terminate new instances to allow metrics to stabilize.

---

# RDS (Relational Databases)

- Storage auto-scaling, useful for cases of unpredictable workloads.
- Read replicas for scalability:
    - Up to 5 replicas.
    - Cross AZ or cross Region replication (no fee for same region replication).
    - Asynchronous, eventually consistent reads.
    - Replicas can be promoted to their own database.
- Read replicas can be set up as multi-AZ for disaster recovery:
    - Failover in case of losing an entire AZ.
    - Increased availability.
    - Synchronous replication.
    - Zero downtime setup (no need to stop the database):
        - Snapshot taken automatically and restored to a new standby database.
        - Sync will happen afterward.

### Aurora

- Proprietary database compatible with other open-source databases, cloud-optimized, and high-performance.
- Can have 15 replicas with very fast replication.
- Instantaneous failover.
- Stores 6 copies of your data across 3 AZs:
    - Some for reads, some for writes.
    - Self-healing.
    - Storage stripped across hundreds of volumes.
- One instance takes writes (master).
- Cross-region replication.
- DB cluster:
    - Writer endpoint points to the master.
    - Reader endpoint points to read replicas in an auto-scaling cluster.

### RDS & Aurora Security

- At-rest encryption:
    - DB master and replicas encrypted with AWS KMS.
- In-flight encryption:
    - TLS-ready by default.
    - Use AWS TLS root certs client-side.
- IAM authentication:
    - Use roles to connect to the database instead of username/password.
- Security groups:
    - Control network access.
- No SSH available.
- Audit logs option.

### ElastiCache

- Provides very high performance and low latency by reducing the load off of databases through caching.
- Managed Redis or Memcached.
- Must have an invalidation strategy to ensure current data is used.
- Can also be used for storing session data to retrieve when the user hits another instance of the app, avoiding the
  need to log in again.

### Redis

- Multi-AZ with auto-failover.
- Read replicas for scaling reads with high availability.
- Data durability with AOF persistence.
- Backup and restore feature.

### Memcached

- Multi-node for partitioning (sharding).
- No high availability (replication).
- No persistence.
- No backup or restore.
- Multi-threaded features.

### Caching Considerations

- Safety: Data may be outdated.
- Effectiveness: Data change rate.
- Structure: Key-value caching, caching of aggregations.
- Strategies:
    - Lazy loading:
        - Cache is populated only with requested data.
        - Cache misses more than 3 times can cause noticeable delay.
        - Stale data (outdated) could be a problem.
    - Write-through:
        - Update cache whenever the database is updated.
        - Leads to fast reads, but writes will generally take longer.
        - Missing data until the database is updated can be a problem.
        - Cache churn can also occur.
    - It's a good idea to combine the two strategies.
- Cache evictions and TTL:
    - Removing data from the cache after a specified time (TTL) or out of memory.

### ElastiCache Redis Cluster Mode

- Disabled:
    - One primary cache node for read/write and 5 read replicas.
    - Asynchronous replication.
    - One shard, all nodes have all the data.
        - Guards against data loss if a node fails.
    - Multi-AZ enabled by default for failover.
    - Helps scaling read performance.
- Enabled:
    - Data is partitioned across many shards (scales writes).
    - Each shard has one primary cache node and up to 5 replicas.
    - Multi-AZ enabled.
    - Up to 500 nodes per cluster.

---

# Route 53

- High availability and scalable DNS.
- Also functions as a domain registrar.
- Records:
    - Domain/subdomain name, RecordType, Value, Routing Policy, TTL.
    - Types:
        - A: Maps hostname to IPv4.
        - AAAA: Maps hostname to IPv6.
        - CNAME: Maps hostname to hostname (domain names that must have A or AAAA record).
        - NS: Name servers of the hosted zone.
    - Hosted zones:
        - Container of records defining how to route traffic to domains.
        - Public hosted zone: Routes traffic on the internet (public domains).
        - Private hosted zone: Routes traffic within one or more VPCs.
        - $0.50 per month per hosted zone.
    - TTL:
        - Client will cache the DNS result for the TTL of the record.
        - High TTL: Less traffic, but possibly outdated records.
        - Low TTL: More traffic to DNS, but records are outdated for less time and easy to change.
    - CNAME vs Alias:
        - CNAME:
            - Exposes hostname to another (app.mydomain.com => blabla.anything.com), only for non-root domain.
        - Alias:
            - Points hostname to AWS resource (app.mydomain.com => blabla.amazonaws.com).
            - Works for both root and non-root domain.
            - Free of charge.
            - Only supports A or AAAA records.
- Route 53 routing policies:
    - Define how Route 53 responds to DNS queries.
    - Simple, weighted, failover, latency-based, geolocation, multi-value answer, geoproximity.

---

### VPC

- VPCs are private networks for deploying resources (regional).
- Subnets are used to partition the network inside the VPC.
    - Subnets are availability zone-specific resources.
    - Public subnets:
        - Accessible from the internet.
        - Connect to the internet with an Internet Gateway.
    - Private subnets:
        - Not accessible from the internet.
        - Connect to the internet with a NAT Gateway or NAT Instance.
        - The NAT Gateway will forward traffic to and from the Internet Gateway.
- NACL (Network ACL):
    - Allows or denies traffic (IP).
    - Attached at the subnet level.
- Security Groups:
    - Control traffic to and from instances.
    - Reference IPs or other security groups.
    - Only allow traffic.
- VPC Peering:
    - Connects two VPCs privately within AWS.
    - No overlapping CIDR.
    - Not transitive.
- VPC Endpoints:
    - Endpoints to connect to AWS services using a private network.
    - Enhanced security and lower latency.
    - VPC Endpoint Gateway is only for S3 or DynamoDB.
    - VPC Endpoint Interface is for everything else.
- Site-to-Site VPN:
    - Connects on-premises VPN to AWS.
    - Automatically encrypted connection.
    - Goes over the public internet.
    - Cannot access VPC endpoints.
- Direct Connect (DX):
    - Physical connection between on-premises and AWS.
    - Connection is private, secure, and fast.
    - Cannot access VPC endpoints.
- 3-tier architecture:
    - Public subnet (ELB, Route 53).
    - Private subnet (ASG with many AZs and instances).
    - Data subnet (ElastiCache, RDS).
- LAMP stack:
    - Linux OS for EC2.
    - Apache web server.
    - MySQL.
    - PHP.
    - Can add Redis/Memcached, EBS.

---

# S3

- At its core, it is storage.
- Used for:
    - Backup & storage
    - Disaster recovery
    - Archiving
    - Hybrid cloud storage
    - Application hosting
    - Media hosting
    - Data lakes & big data analytics
    - Software delivery updates
    - Static website

### Buckets

- Store objects (files) in "buckets" (dirs)
- Defined at the region level
- Objects (files) are also called keys
    - s3://my-bucket/my_file.txt
    - s3://my-bucket/my_folder1/another_folder/my_file.txt
- Maximum size: 5 TB; for bigger files, "multi-part upload" must be used

### Security

- User-based:
    - IAM policies
- Resource-based:
    - Bucket policies
    - Object access control list (ACL) - fine-grained
    - Bucket access control list (ACL) - less common
- IAM principal can access S3 if:
    - User IAM permissions allow it or the resource policy allows it
    - There's no explicit deny

### Static Web Hosting

- URL: http://bucket-name.s3-website.aws-region.amazonaws.com

### Versioning

- Files are versioned if enabled at the bucket level
- Easy rollback

### Replication

- CRR (Cross-Region Replication):
    - Compliance
    - Lower latency access
    - Replicate data across accounts
- SRR (Same-Region Replication):
    - Log aggregation
    - Live replication between prod and test accounts
- S3 Batch Replication (optional):
    - For existing objects as well as objects that failed replication

### S3 Storage Classes

- Standard - General Purpose:
    - 99.99% availability
    - Low latency and high throughput
    - Frequently accessed data cases: big data analytics, mobile & gaming apps, content distribution
- Standard - Infrequent Access (IA):
    - For less frequently accessed data, but with rapid access when needed (disaster recovery, backups)
    - Lower cost
- One Zone Infrequent Access:
    - High durability in a single AZ (data lost when AZ is destroyed)
    - Storing secondary backups or data you can recreate
- Glacier:
    - Low cost meant for archiving/backup
    - Instant retrieval
    - Flexible retrieval
    - Deep Archive: for long-term storage, longer retrieval time
- S3 Intelligent-Tiering:
    - Moves objects automatically between access tiers based on usage

---

# MFA with CLI

To use MFA with CLI you need a temporary session, use STS GetSessionToken API call.

    aws sts get-session-token --serial-number arn-of-the-mfa-device --token-code code --duration-seconds 3600

---

# AWS Limits (Quotas)

### API Rate Limits

- DescribeInstances API for EC2 has a limit of 100 calls per second.
- GetObject on S3 has a limit of 5500 GET requests per second per prefix.
- Exponential backoff is recommended for intermittent errors.
- API throttling limit increase should be requested for consistent errors.

### Service Quotas (Service Limits)

- Running on-demand standard instances: 1152 vCPU.
- Service limit increases can be requested through tickets.
- Service quota increases can be requested using the Service Quotas API.

### Exponential Backoff

- Exponential backoff is a retry mechanism, typically used when encountering ThrottlingException.
- Retries should be implemented for 5xx server errors and throttling.
- It should not be implemented for 4xx errors.

---

# Credentials Provider Chain

The AWS Command Line Interface (CLI) follows a specific order when looking for credentials:

1. Command line options: Credentials can be provided as command line options, such as `--region`, `--output`,
   and `--profile`.
2. Environment variables: Credentials can be set as environment variables,
   including `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_SESSION_TOKEN`.
3. CLI credentials file: The CLI looks for credentials in the `~/.aws/credentials` file, which can be configured using
   the `aws configure` command.
4. CLI config file: The CLI also checks the `~/.aws/config` file for additional configuration options.
5. Container credentials: For Amazon Elastic Container Service (ECS) tasks, the CLI can retrieve credentials from the
   task's container environment.
6. Instance profile credentials: For Amazon EC2 instance profiles, the CLI can obtain credentials from the instance's
   IAM role.

### SigV4

SigV4 is the authentication method used to sign AWS HTTP requests. It ensures the integrity and authenticity of the
requests sent to AWS services. SigV4 signs the requests with access keys and includes the signature in the request
headers for authentication purposes.

---

# S3 Advanced

### Transitioning Objects Between Storage Classes

To optimize storage costs, you can transition objects between different storage classes:

- For infrequently accessed objects, you can move them from Standard to Standard-IA (Infrequent Access).
- For archive objects that don't require fast access, you can move them to Glacier or Glacier Deep Archive using S3
  Glacier Select.

### Lifecycle Rules

Lifecycle rules allow you to automate actions on objects in S3:

- Transition actions: Configure objects to transition to another storage class after a specific time period.
- Expiration actions: Configure objects to be deleted after a certain time, including old versions, incomplete multipart
  uploads, and more.

### Storage Class Analysis

S3 provides storage class analysis, which offers recommendations for using the Standard and Standard-IA storage classes
based on the access patterns of your objects.

### S3 Event Notifications

You can configure S3 event notifications to trigger actions when certain events occur in your S3 bucket. Events include
ObjectCreated, ObjectRemoved, ObjectRestore, Replication, and more. Notifications can be sent to various AWS services
such as SNS, SQS, Lambda Functions, and can also be forwarded to Amazon EventBridge for additional processing and
routing.

### S3 Performance

S3 offers high request rate and low latency for accessing objects. To improve performance:

- Use multipart upload for files larger than 100MB (required for files larger than 5GB) to parallelize uploads.
- Enable S3 Transfer Acceleration to increase transfer speed by leveraging AWS edge locations.
- Use S3 byte-range fetches to parallelize GET requests by requesting specific byte ranges, improving download speed and
  resilience.

### S3 Select & Glacier Select

S3 Select allows you to use SQL-like expressions to filter data on the server-side, reducing network transfer and
client-side processing. Glacier Select provides similar functionality for data stored in S3 Glacier.

### Object Encryption

- Server-side encryption (SSE):
    - SSE-S3: Server-side encryption with Amazon S3-managed keys (AES-256).
    - SSE-KMS: Server-side encryption with AWS Key Management Service (KMS) keys, offering additional user control and
      key usage auditing.
    - SSE-C: Server-side encryption with customer-provided keys, where you encrypt the data before sending it to S3, and
      S3 doesn't store the encryption key.
- Client-side encryption: Encrypt data client-side before sending it to S3 and decrypt it when retrieving.
- Encryption in transit: S3 provides both HTTP (non-encrypted) and HTTPS (encrypted) endpoints.

### Default Encryption vs. Bucket Policies

You can enforce encryption using bucket policies by refusing API PUT calls on S3 objects without encryption headers.
Bucket policies are evaluated before default encryption settings.

### CORS (Cross-Origin Resource Sharing)

CORS is a web browser feature that allows requests to be made from one origin (protocol + host/domain + port) to
another. In the context of S3 buckets, CORS configuration allows clients to make cross-origin requests to the bucket.
The following CORS headers can be set in the response from the S3 bucket:

- Access-Control-Allow-Origin: Specifies the allowed origin(s) for cross-origin requests.
- Access-Control-Allow-Methods: Specifies the allowed HTTP methods for cross-origin requests.

### MFA Delete

MFA Delete provides an extra layer of protection against accidental or unauthorized permanent deletion of object
versions or suspension of versioning on an S3 bucket. It requires versioning to be enabled on the bucket and can only be
enabled or disabled by the bucket owner (root account).

### Access Logs

S3 access logs allow you to capture detailed information about every request made to an S3 bucket. The logs are stored
in a separate S3 bucket (logging bucket) and must be in the same AWS region as the bucket being logged.

### Pre-signed URLs

Pre-signed URLs are generated using the S3 console, AWS CLI, or SDKs. They grant temporary access to specific S3 objects
and inherit the permissions of the user who generated the URL. Pre-signed URLs are commonly used for scenarios such as:

- Allowing only logged-in users to download premium videos from S3.
- Enabling an ever-changing list of users to download files by dynamically generating URLs.
- Allowing temporary users to upload files to S3.

### S3 Object Lambda

S3 Object Lambda allows you to modify or transform objects retrieved from S3 before they are returned to the caller.
This can be useful for various use cases, including:

- Redacting personally identifiable information (PII) from data retrieved for analytics purposes.
- Converting data across different formats.
- Dynamically resizing and watermarking images on the fly.

---

# CloudFront

CloudFront is a content delivery network (CDN) service provided by AWS. It improves read performance by caching content
at the edge locations, which are distributed globally across 216 locations. CloudFront offers DDoS protection and
integrates with AWS Shield and AWS Web Application Firewall.

### CloudFront Origins

CloudFront supports the following types of origins:

- S3 bucket: Used for distributing files and caching them at the edge. Enhanced security can be achieved using Origin
  Access Control (OAC).
- Custom origin (HTTP): Can be an Application Load Balancer (ALB), EC2 instance, S3 website, or any other HTTP backend.

### CloudFront vs Cross Region Replication

CloudFront and S3 Cross Region Replication serve different purposes:

- CloudFront: Provides a global edge network where files are cached for a specified time-to-live (TTL). It is suitable
  for static content that needs to be available worldwide.
- S3 Cross Region Replication: Must be set up separately in each region requiring replication. It offers near real-time
  uploads and is read-only. It is suitable for dynamic content that needs to be available with low latency in specific
  regions.

### Security Group Configuration

ALBs and EC2 instances associated with CloudFront need to have security groups that allow public IPs of edge locations.

### Geo Restriction

CloudFront supports geo restriction, allowing you to allowlist or blocklist specific countries. This feature can be used
to control access to content based on copyright laws or other requirements.

### Signed URLs / Cookies

CloudFront supports signed URLs and signed cookies for distributing paid or shared content to premium users worldwide.
Signed URLs provide access to individual files, while signed cookies allow access to multiple files.

### CloudFront Signed URLs vs S3 Pre-signed URLs

CloudFront signed URLs:

- Allow access to paths regardless of the origin.
- Managed at the account level, and only the root account can manage the key pair.
- Can filter access based on IP, path, date, and expiration.
- Can leverage CloudFront caching features.

S3 pre-signed URLs:

- Issue a request as the IAM principal who pre-signed the URL.
- Use the IAM key of the signing IAM principal.
- Have a limited lifetime.

### Pricing

To reduce costs, you can reduce the number of edge locations used by CloudFront. CloudFront offers different price
classes:

- All: All regions, best performance.
- 200: Most regions, excluding the most expensive.
- 100: Only the least expensive regions.

### Multiple Origins

CloudFront allows you to route requests to different origins based on the content type, enabling flexible handling of
different kinds of content.

### Origin Groups

Origin groups provide high availability and failover capabilities. You can configure a primary and secondary origin
group. If the primary fails, CloudFront automatically falls back to the secondary group. Origin groups can also be used
with S3 buckets, allowing content replication from one bucket to another.

### Field Level Encryption

Field level encryption adds an additional layer of security to protect sensitive user information. It works in
conjunction with HTTPS and encrypts the specified set of fields in POST requests. Asymmetric encryption is used, and you
need to specify the public key to encrypt the fields.

---

# ECS (Elastic Container Service)

ECS is a service provided by AWS for launching and managing Docker containers. It allows you to run containers as tasks
on ECS clusters.

### EC2 Launch Type

- Provision and maintain the infrastructure (EC2 instances).
- Docker containers are placed on pre-provisioned EC2 instances.

### Fargate Launch Type

- No need to provision infrastructure (no EC2 instances to manage).
- Serverless approach.
- Task definitions are created, and AWS runs ECS tasks based on CPU and RAM usage.
- Scaling is achieved by increasing the number of tasks.

### IAM Roles for ECS

- EC2 launch type uses EC2 instance profiles.
- ECS task role is specific to the EC2 task and is defined in the task definition.

### Load Balancer Integration

- Application Load Balancer (ALB) is recommended for most use cases.
- Network Load Balancer (NLB) is suitable for high-performance scenarios.
- Classic Load Balancer (ELB) is supported but not recommended, as it lacks advanced features for Fargate.

### Data Volumes (EFS)

- Mount Amazon Elastic File System (EFS) onto ECS tasks.
- Provides persistent, multi-AZ shared storage for containers.

### AWS Application Auto Scaling

- Auto scaling based on average CPU utilization, average memory usage, or ALB request count per target.
- Can work with CloudWatch to monitor metrics such as CPU usage, and the associated alarm triggers a scaling event for
  the auto scaling group.

### Rolling Updates

- Control the number of tasks that can be started and stopped during an update, as well as the order in which they
  occur.

### ECS Solutions Architecture

- ECS tasks can be invoked by Amazon EventBridge:
    1. Upload an object to S3.
    2. S3 sends the relevant event to EventBridge.
    3. EventBridge triggers ECS tasks based on a predefined rule.
    4. Fargate, based on the ECS task role, can perform actions like fetching the object from S3 and saving it to
       DynamoDB.
- ECS tasks can be invoked by an EventBridge schedule, such as running a task every hour.
- ECS can also integrate with an SQS queue, where messages are sent to the queue, and the ECS service polls for
  messages, autoscaling according to throughput.

### Task Definitions

- Metadata in JSON format that describes the container configuration.
- Contains information such as the image name, port binding, memory/CPU requirements, environment variables, network
  configuration, IAM role, and logging configuration.
- Up to 10 containers can be defined per task definition.
- For EC2 launch type:
    - Dynamic host port mapping: If you only define the container port, the ALB will find the appropriate port on the
      EC2 instances.
    - EC2 security group must allow any port from the ALB security group.
- For Fargate:
    - Each task gets a unique private IP.
    - Only the container port is defined; there is no need to specify the host port.
    - The ALB connects to all ECS tasks on the same port (e.g., port 80).
- IAM roles in ECS are defined and assigned at the task definition level.
- Environment variables can be hardcoded, sourced from SSM Parameter Store (e.g., API keys, shared configurations),
  retrieved from Secrets Manager (e.g., database passwords), or fetched from S3.
- Data volumes (bind mounts) allow sharing data (e.g., metrics, logs, or ephemeral data) between multiple containers
  within the same task definition.

### ECS Tasks Placement

ECS provides several strategies for task placement, which are not applicable to Fargate:

- ECS places tasks based on the following criteria:
    1. Instances that meet the CPU, memory, and port requirements defined in the task definition.
    2. Instances that satisfy any task placement constraints.
    3. Task placement strategies.

### Task Placement Strategies

1. Binpack: Places tasks on instances with the least available CPU or memory.
    - Minimizes the number of instances used, optimizing costs.

2. Random: Places tasks randomly on instances.

3. Spread: Achieves an even placement of tasks based on a specified value (e.g., instance ID, availability zone).

### Task Placement Constraints

- Constraints allow for more fine-grained control over task placement. Some examples include:
    - DistinctInstance: Ensures that each task is placed on a different container instance.
    - MemberOf: Places tasks on instances that satisfy a specified expression.

### ECR (Elastic Container Registry)

- ECR is a repository provided by AWS to store and manage Docker images.
- ECR supports public or private repositories and is backed by Amazon S3.
- Integrated with ECS and allows IAM-controlled access to repositories.

### EKS (Elastic Kubernetes Service)

- EKS is an alternative to ECS, offering a different API and Kubernetes compatibility.
- Supports EC2 launch mode for deploying worker nodes or Fargate for serverless containers.
- Can use private or public load balancers to expose EKS services.
- Data volumes in EKS:
    - Can be attached by specifying a StorageClass manifest on the EKS cluster.
    - Uses Container Storage Interface (CSI) compliant drivers.
    - Supports various storage options, including EBS, EFS (for Fargate), FSx for Lustre, and FSx for NetApp ONTAP.

---

# Elastic Beanstalk

Elastic Beanstalk is a tool provided by AWS for deploying applications. It automates various tasks such as capacity
provisioning, load balancing, scaling, application health monitoring, and instance configuration. Here are the key
components and features of Elastic Beanstalk:

### Components

1. Application: Represents your application in Elastic Beanstalk.
2. Version: Each deployment of your application is assigned a version.
3. Environment (Env): Represents a runtime environment in which your application is deployed. You can create one or
   multiple environments.
    - Web server tier: In a web server environment, traffic is sent to an Auto Scaling group (ASG) with multiple EC2
      instances.
    - Worker tier: In a worker environment, EC2 instances (workers) pull messages from an SQS queue.

### Deployment Modes

1. Single instance: Suitable for development environments, where you have only one EC2 instance running your
   application.
2. High availability with load balancing: Suitable for production environments, where an Elastic Load Balancer (ELB)
   distributes traffic to an ASG with multiple EC2 instances.

Deployment options include:

- All at once: Fastest deployment, but your application experiences downtime during the deployment. Suitable for quick
  iterations in development but not recommended for production.
- Rolling: Your application runs below capacity, and Elastic Beanstalk deploys the new version while keeping the old
  version running. Can take some time to complete the deployment.
- Rolling with additional batches: Your application runs at capacity, and Elastic Beanstalk adds additional batches of
  instances with the new version while rolling out the old versions. Can be costlier and takes longer to complete.
- Immutable: Zero downtime deployment. The new version is deployed to new instances on a temporary ASG, and the old ASG
  is terminated. Suitable for production but can be the longest deployment option.

### Blue/Green Deployment

- In a blue/green deployment, you create a stage environment for deploying version 2 of your application.
- Route 53 is set up with weighted policies to redirect some traffic to the stage environment.
- Once the testing is complete, you can swap URLs with Elastic Beanstalk to make the stage environment the production
  environment.

### Lifecycle Policy

- Elastic Beanstalk can store up to 1,000 application versions.
- The lifecycle policy automatically phases out old application versions based on time or space constraints. The
  currently used version will not be deleted.

### Extensions

- Elastic Beanstalk provides the ability to configure resources through the AWS UI or using extensions.
- Extensions can be defined in the `.ebextension/` directory or as `.config` files.
- You can add resources like RDS, ElastiCache, etc., using extensions.
- CloudFormation resources can also be defined in `.ebextensions/` for more advanced configurations.

### Beanstalk from HTTPS

To load an SSL certificate onto the load balancer in Elastic Beanstalk:

- This can be done from the AWS Management Console, specifically in the Elastic Beanstalk console or the load balancer
  configuration.
- You can also configure this through code using `.ebextensions/securelistener-alb.config`.
- The SSL certificate can be provisioned using AWS Certificate Manager (ACM) or the AWS CLI.
- Ensure that the security group rules allow incoming traffic on port 443.

To redirect HTTP to HTTPS:

- You can configure your instances to redirect HTTP traffic to HTTPS.
- Alternatively, if you are using an Application Load Balancer (ALB), you can set up a relevant rule to redirect HTTP to
  HTTPS.

### Web Server vs. Worker Environment

- If your application performs tasks that take a long time to complete, you can offload them to a dedicated worker
  environment.
- This decouples your application into two tiers: web server and worker.
- Examples of tasks suitable for the worker environment include processing videos or generating ZIP files.
- Periodic tasks can be added to `cron.yaml` in the web tier, which sends messages to an SQS queue in the worker tier.

---

# AWS CICD

- CodeCommit, storing code
- CodePipeline, automating our pipeline from code to elastic beanstalk
- CodeBuild, building and testing code
- CodeDeploy, deploying the code to ec2 instances
- CodeStar, manage software development activities in one place
- CodeArtifact, store publish and share software packages
- CodeGuru, automated code reviews using ml

---

# AWS CloudFormation

- Infrastructure as code
- Templates uploaded to S3 and referenced in CloudFormation (CF)
- To update, re-upload a new version
- Deleting a stack deletes all artifacts created by CF

### Template Components

- Resources
- Parameters
- Mappings
- Outputs
- Conditionals
- Metadata

### Intrinsic Functions

- `Fn::Ref`: References parameters and resources
- `Fn::GetAtt`: Retrieves attributes of resources
- `Fn::FindInMap`: Returns a named value from a mapping
- `Fn::ImportValue`: Imports values exported in other templates
- `Fn::Join`: Joins values with a delimiter
- `Fn::Sub`: Substitutes variables in a text

### Condition Functions

Example:

```yaml
CreateProdResources: !Equals [ !Ref EnvType, prod ]
```

### Rollbacks

- If stack creation fails at any point, everything is deleted.
- If an update fails, the stack automatically rolls back to the previous state.

### ChangeSets

- Help with viewing changes before they are applied.

### Nested Stacks

- Allow for updating and isolating repeated patterns in separate stacks.
- Nested stacks can be called from other stacks and are shared across stacks.
- Common use cases include LB configurations or reusable security groups.
- Considered a good practice.

### Cross Stacks

- Useful when stacks have different lifecycles.
- Utilize output export and `Fn::ImportValue` to pass exported values to multiple stacks.
- Example use case: reusing a VPC stack across different stacks.

### Drift

- Helps identify if there were manual changes that affect the CloudFormation-created infrastructure.
- Indicates whether resources have "drifted" from the expected configuration.

---

# Monitoring

- CloudWatch includes metrics, logs, events, alarms.
    - EC2 instances have CloudWatch Logs & Unified Agent to push log files to CloudWatch.

- CloudWatch Logs Metric Filter:
    - Used to filter expressions, e.g., find specific IP in a log, count occurrences of errors.
    - Can be used to trigger alarms.

- Alarms:
    - Trigger notifications for any metric.
    - States: OK, INSUFFICIENT_DATA, ALARM.
    - Alarm targets:
        - Stop, terminate, reboot, or recover EC2 instance.
        - Trigger auto scaling action.
        - Send notification to SNS.
    - To test an alarm:
      ```
      aws cloudwatch set-alarm-state --alarm-name "testalarm" --state-value ALARM --state-reason "testing"
      ```

- Events:
    - Intercept events from AWS services.
    - Can also intercept any API call with CloudTrail integration.

- EventBridge:
    - Extends Events and allows adding event buses for custom apps and third-party SaaS apps.

### X-Ray

- X-Ray is for troubleshooting app performance and errors and distributed tracing of microservices.
- Instead of debugging by testing locally, then adding logs and re-deploying, we can use X-Ray.
- Provides a visual analysis of the app, making it easy to trace interactions with services like EC2 instances.
    - Troubleshoot performance, identify bottlenecks.
    - Understand dependencies better.
    - Pinpoint service issues.
    - Review request behavior.
    - Find errors/exceptions.
    - Identify impacted users.
    - Find throttling.
    - Ability to trace every request.
- To enable X-Ray:
    - Import the AWS X-Ray SDK.
    - Modify the code to capture calls to AWS services, HTTP/HTTPS requests, database calls, queue calls, etc.
    - Install X-Ray daemon or enable X-Ray AWS integration. The app code + AWS X-Ray will send traces to the daemon,
      which will send batches to AWS X-Ray every second.
- Instrumentation:
    - Use the X-Ray SDK to instrument the application.
- Segments:
    - Each app/service will send segments, with subsegments providing more details.
- Traces:
    - Segments are collected together to form traces.
- Sampling:
    - Control the amount of data recorded.
    - The X-Ray SDK records the first requests each second and 5% of any additional requests.
- X-Ray APIs:
    - `xray:PutTraceSegments`: Uploads segment documents to X-Ray.
    - `xray:PutTelemetryRecords`: Daemon uploads telemetry.
    - `xray:GetSamplingRules`: Retrieve all sampling rules to know what and when to send.
    - `xray:GetSamplingTargets`.
    - `xray:GetSamplingStatisticsSummaries`.
    - `xray:GetServiceGraph`: The main graph.
    - `xray:BatchGetTraces`: List of traces by ID.
    - `xray:GetTraceGraph`: Service graph for trace ID.
- Beanstalk includes X-Ray daemon. It can be run using the console or `.ebextensions/xray-daemon.config`
  with `XrayEnabled: true`.
- ECS + X-Ray integration:
    - In an ECS cluster, use X-Ray container as a daemon, one per EC2 instance.
    - Launch app containers on the EC2 instances.
    - Another way is to run an X-Ray daemon along with each app container, one per container.
    - For Fargate, where we don't have control over the underlying instances (serverless), X-Ray will be packaged along
      each app container.

### CloudTrail

- CloudTrail is for internal monitoring of API calls made and audit changes to AWS resources by users.
- Provides governance, compliance, and audit for the AWS account.
- Works in one or many regions.
- Get a history of events and API calls made within the AWS account.
- SDK, CLI, console, IAM users, and roles, all their actions appear in the CloudTrail console.
- Events:
    - Management events (default).
    - Data events (not logged by default).
    - CloudTrail Insights: Detects unusual activity, such as hitting service limits, gaps, inaccurate resource
      provisioning, etc.
- Events are stored for 90 days, but we can send them to S3 and use Athena to analyze them from S3.

---

# SQS, SNS, Kinesis

### SQS (Simple Queue Service)

- SQS is used for decoupling applications and enables communication between producers and consumers.
- It provides unlimited throughput and supports an unlimited number of messages in the queue.
- Default retention period is 4 days, with a maximum of 14 days.
- Each message can have a maximum size of 256 KB.
- Messages can be duplicated, and their order is not guaranteed.
- CloudWatch metrics can be used to monitor the queue length, and alarms can trigger actions such as auto-scaling
  consumer ASGs.
- SQS queue access policies:
    - Cross-account access:
        - One account pushes messages, and the other polls them. An `Allow` action `sqs:ReceiveMessage` is required.
    - S3 bucket sends messages to SQS about uploaded items:
        - Add a policy to allow `sqs:SendMessage` with conditions `aws:SourceArn` (bucket ARN) and `aws:SourceAccount`.
- Message visibility timeout:
    - After a message is polled, it becomes invisible to other consumers for a default visibility timeout of 30 seconds.
    - If the timeout is exceeded, the message will be processed again, possibly resulting in duplicates.
    - Consumers can call the `ChangeMessageVisibility` API to extend the visibility timeout.
- Dead Letter Queue (DLQ):
    - Set a threshold for how many times a message can be returned to the queue.
    - After the `MaximumReceives` threshold is exceeded, the message goes to the DLQ.
    - Useful for debugging, and the retention time can be set.
    - Messages from the DLQ can be consumed to check what went wrong, and they can be redriven back into the SQS queue.
- Delay Queue:
    - Delays messages so that consumers do not immediately receive them.
    - Default delay is 0 seconds.
- Long polling:
    - Consumers can wait if messages are not available yet.
    - Reduces the number of API calls made to SQS, improving efficiency and latency.
    - Can be set at the queue level or API level using the `ReceiveMessageWaitTimeSeconds` parameter.
- Extended client:
    - For large messages (> 256 KB), the message is sent to a specified S3 bucket, and only metadata is pushed to the
      queue.
    - The metadata informs the consumer to retrieve the large message from S3.
- API:
    - `CreateQueue`, `DeleteQueue`, `PurgeQueue`, `SendMessage`, `ReceiveMessage`, `DeleteMessage`, `MaxNumberOfMessages` (
      default 1), `ReceiveMessageWaitTimeSeconds`, `ChangeMessageVisibility`.

### FIFO Queues

- FIFO queues provide first-in-first-out ordering, with a maximum throughput of 300 messages/second without batching and
  3,000 messages/second with batching.
- FIFO queues guarantee exactly-once send capability by removing duplicates.
- Deduplication:
    - If the same message is sent within 5 minutes, the new message will be discarded.
    - Deduplication can be based on content or by using a message deduplication ID.
- Message grouping:
    - Messages with the same `MessageGroupId` value are treated as a single group, ensuring that they are processed in
      order within the group.
    - Ordering across groups is not guaranteed.

# SNS (Simple Notification Service)

- SNS is used for one sender to send messages to many different receivers.
- Subscribers to SNS topics receive the same message.
- Each subscription to a topic receives all the messages.
- Up to 12,500 subscriptions per topic and a limit of 100,000 topics.
- SNS integrates with SQS, Lambda, Firehose, and many other services.
- To publish:
    - Topic publish:
        - Using the SDK, create a topic, create one or more subscriptions, and publish the topic.
    - Direct publish (for mobile app SDK):
        - Create a platform app, create a platform endpoint, and publish to it.
- Security:
    - Encryption in transit with HTTPS API.
    - At-rest encryption with KMS.
    - Client-side encryption.
    - SNS access policies for cross-account access to SNS topics and allowing other services to write to them.

### SQS + SNS Fan-out

- Messages are pushed once to SNS and received in all SQS queues that are subscribers.
- Provides full decoupling, data persistence, delayed processing, retries, and cross-region delivery.
- Fan-out is used to send one event to multiple queues.

### SNS as FIFO

- SNS can be used as a FIFO system by using message group ID for ordering and deduplication with deduplication ID or
  content-based deduplication.
- Only SQS FIFO queues can be subscribers, with the same throughput as SQS.

### SNS Message Filtering

- JSON policy can be used to filter messages sent to subscribers of SNS topics.
- Subscribers without filter policies receive every message.

### Kinesis

- Kinesis is used to collect, process, and analyze streaming data in real-time, such as application logs, metrics,
  website clickstreams, IoT telemetry, etc.
- Data Streams:
    - Streams consist of multiple shards that are configurable and scalable.
    - Producers send records (including a partition key and data blob), and shards receive them in parallel.
    - Producers can be the AWS SDK, Kinesis Producer Library (KPL), or the Kinesis Agent (monitors log files).
    - `PutRecord` API is used to send records.
    - Records with the same partition key go to the same shard.
    - If overproducing into a shard, a `ProvisionedThroughputExceeded` error can occur. To mitigate, use highly
      distributed partition keys, retries with exponential backoff, and increase the number of shards (scaling).
    - Consumers receive records (with partition key, sequence number, and data blob).
    - Consumers can be applications (KCL, SDK), Lambda, Kinesis Firehose, Kinesis Analytics.
    - Shared (classic) fan-out consumer:
        - `GetRecords()`, with a throughput of 2 MB/s per shard across all consumers, low cost, and latency of
          approximately 200 ms.
    - Enhanced fan-out consumer:
        - `SubscribeToShard()`, with a throughput of 2 MB/s per consumer per shard, higher cost, and latency of
          approximately 70 ms.
    - AWS Lambda:
        - `GetBatch()`, Lambda functions process records and can save to DynamoDB, for example.
        - Retries, configurable batch size, and window.
    - Retention period is configurable from 1 to 365 days.
    - Kinesis data cannot be deleted; however, replayability is possible.
    - Data Streams offer replayability, and data with the same partition key goes to the same shard.
    - Capacity modes:
        - Provisioned: Each shard allows 1 MB/s in (or 1,000 records/s) and 2 MB/s out. Pay per shard per hour.
        - On-demand: Capacity is adjusted automatically on demand.
    - Shard splitting can be used to increase stream capacity and avoid "hot shards." Shard merging can be done to
      reduce capacity.

### Data Firehose

- Fully managed, automatic scaling, serverless, pay for data, near real-time.
- Kinesis Data Streams can produce data to Data Firehose, along with other sources.
- Firehose can send data to Lambda for custom transformations and then batch writes the data to destinations.
- Destinations include S3, Redshift (copied from S3), Elasticsearch, and some third-party or custom HTTP endpoints.
- Failed data can be sent to a backup location in S3.

### Data Analytics

- Real-time analytics on data streams and Data Firehose.

---

# AWS Lambda

AWS Lambda is a serverless computing service provided by Amazon Web Services. It allows you to run code without
provisioning or managing servers. Lambda functions are virtual functions that run on-demand, have a limited execution
time, and offer automated scaling.

### Features

- Virtual functions with no servers to manage
- Pay-per-request and compute time
- Automated scaling based on demand
- Support for Lambda container images
- Integration with various AWS services
- Synchronous and asynchronous invocation options
- Integration with ALB (Application Load Balancer)
- Support for Lambda@Edge for edge computing

### Examples

- Thumbnail generation from S3 bucket events
- Serverless cron jobs triggered by CloudWatch Events

### Synchronous Invocation

Synchronous invocation of Lambda functions can be done using the CLI, SDK, API Gateway, or ALB. When making a
synchronous invocation, you wait for the results, and errors must be handled client-side with retries and exponential
backoff.

### Integration with ALB

To expose Lambda functions as HTTP endpoints, you can use ALB (Application Load Balancer) or API Gateway. For ALB
integration, the Lambda function needs to be registered in a target group, and ALB will convert the HTTP request into a
Lambda synchronous invocation. ALB also supports multi-header values.

### Lambda@Edge

Lambda@Edge allows you to run Lambda functions globally across edge locations or implement request filtering before
reaching your application. It can be used to modify CloudFront requests and responses at any stage. There are various
use cases, including website security, dynamic web app at the edge, SEO, bot mitigation, A/B testing, image
transformation, user tracking, analytics, user authentication, etc.

### Async Invocations

Lambda functions can be invoked asynchronously for event-driven processing. For example, when a new file is added to an
S3 bucket, the event is placed in an internal event queue, and the Lambda function reads it and retries up to 3 times on
errors. It is important to ensure that the Lambda function is idempotent. You can also define dead letter queues for
handling failures. Supported async invocation services include S3, SNS, CloudWatch, CodeCommit, CodePipeline, and IoT.

### Event Source Mapping

Lambda provides an internal mechanism for event source mapping, where Lambda needs to invoke services to poll for
records. For example, from a Kinesis stream. The response from the service is then directed from the event source
mapping to a Lambda invocation.

### Execution Roles

Execution roles are attached to Lambda functions and grant permissions to access AWS services and resources. There are
various predefined execution roles, such as AWSLambdaBasicExecutionRole, AWSLambdaKinesisExecutionRole,
AWSLambdaDynamoDBExecutionRole, AWSLambdaSQSQueueExecutionRole, AWSLambdaVPCAccessExecutionRole, and
AWSXRayDaemonWriteAccess. When using event source mapping to invoke a function, Lambda uses the execution role to read
the event data. It is recommended to follow the best practice of having one role per function. IAM principals can access
Lambda functions if authorized by an IAM policy attached to the principal or through resource-based policies.

### VPC Access and Internet Access

Lambda functions can access external APIs but cannot access resources in another VPC with a private subnet. To access a
private VPC, you need to define the VPC ID, subnets, and security groups. Lambda will create an ENI (Elastic Network
Interface) in the specified subnets and use it to access the private VPC. To create this ENI, the Lambda function needs
the AWSLambdaVPCAccessExecutionRole. By default, Lambda functions do not have internet access, even if deployed

### Resource Allocation and Performance

Lambda functions can have a memory allocation ranging from 128MB to 10GB. Increasing the memory allocation also
increases the allocated CPU credits. After 1782MB, Lambda functions get more than one vCPU, and you may need to use
multithreading to benefit from it. If your application is computation-heavy, increasing the Lambda function's memory
allocation can improve performance.

### Limits

Here are some important limits related to AWS Lambda:

- Execution Limits:
    - Memory allocation: 128MB to 10GB
    - Max execution time: 15 minutes
    - Environment variables: 4KB
    - Disk capacity in /tmp directory: 512MB to 10GB
    - Concurrency executions: 1000 (can be increased)

- Deployment Limits:
    - Lambda function deployment size (zip): 50MB
    - Size of uncompressed deployment (code + dependencies): 250MB

### Best Practices

To make the most of AWS Lambda, here are some best practices to follow:

- Perform heavy work outside your function handler to minimize the execution time.
- Use environment variables for configuration that may change over time.
- Minimize the deployment package size to include only the necessary runtime dependencies.
- Avoid recursion in your Lambda functions to prevent infinite loops.

---

# DynamoDB

DynamoDB is a NoSQL serverless database provided by Amazon Web Services. It offers a fully managed, highly available,
and horizontally scalable solution for storing and retrieving data. DynamoDB is designed to handle massive workloads,
with support for millions of requests per second, trillions of rows, and hundreds of terabytes of storage.

### Features

- NoSQL database with serverless architecture
- Fully managed and highly available with replication across multiple Availability Zones (AZs)
- Scales automatically to handle large workloads
- Fast and consistent performance with low retrieval latency
- Cost-effective with auto-scaling capabilities

### Key Concepts

- Maximum size of an item in a table is 400KB.
- Primary keys:
    - Partition key (hash)
    - Partition key + Sort key (hash + range)

### Read/Write Capacity Modes

- Provisioned mode (default):
    - Specify the number of reads and writes per second (capacity units).
    - Pay for the provisioned capacity.
    - Read Capacity Units (RCUs) for reads and Write Capacity Units (WCUs) for writes.
    - Supports strongly consistent and eventually consistent reads.
- On-demand mode:
    - Reads and writes automatically scale up and down based on workload.
    - No capacity planning required.
    - Unlimited RCUs and WCUs, but more expensive (2.5x more than provisioned mode).

### Partitions and Throttling

- Data is written to partitions based on the partition key.
- WCUs and RCUs are spread evenly across partitions.
- Throttling occurs when exceeding provisioned RCUs and WCUs.
- Throttling solutions include exponential backoff, distributing partition keys, and using DynamoDB Accelerator (DAX)
  for caching.

### Writing and Reading Data

- PutItem: Consumes WCUs to create or replace an item.
- UpdateItem: Edits an existing item or adds a new item.
- Conditional writes: Accept writes/updates/deletes only if conditions are met.
- GetItem: Retrieves an item (eventually consistent by default).
- Query: Retrieves items based on specified criteria.
- Scan: Reads the entire table and filters items (inefficient).

### Indexing

- Local Secondary Index: Alternative sort key, defined at table creation.
- Global Secondary Index: Alternative primary key, speeds up queries on non-key attributes.

### PartiQL

- DynamoDB supports PartiQL, a SQL-like syntax for performing operations such as INSERT, UPDATE, SELECT, DELETE, and
  batch operations.

### DynamoDB Accelerator (DAX)

- DAX is an in-memory cache for DynamoDB.
- Provides low-latency cached reads and queries.
- Solves hot key issues.
- Provision a DAX cluster in advance with cache nodes.

### Streams

- DynamoDB Streams provide a stream of item modifications (create/update/delete).
- Stream records can be sent to Kinesis Data Streams, AWS Lambda, or Kinesis Client Library applications.
- Streams have retention up to 24 hours.
- Use cases include reacting to real-time changes, analytics, cross-region replication, and inserting into
  Elasticsearch.

### Time-to-Live (TTL)

- TTL automatically deletes items after the expiry timestamp.
- Expired items are deleted after 48 hours.
- Useful for reducing stored data and adhering to regulatory obligations.

### Transactions

- DynamoDB supports coordinated, all-or-nothing transactions for multiple items in one or more tables.
- ACID properties (Atomicity, Consistency, Isolation, Durability).
- Read modes: Eventual consistency, Strong consistency, Transactional reads.
- Write modes: Standard writes, Transactional writes.

### Session State

- DynamoDB can be used to store session state.
- Compare with ElastiCache (in-memory cache) and EFS (Elastic File System).
- ElastiCache is recommended for more computationally expensive logic.

### Other Features

- Encryption at rest using AWS Key Management Service (KMS) and in-transit using SSL/TLS.
- Backup and restore capabilities, including point-in-time recovery.
- Global Tables for multi-region replication.
- Integration with AWS Database Migration Service (DMS) for data migration.

### Security

- Cognito Identity Pools can be used for user credentials.
- IAM roles can be assigned to users to allow actions with limitations.

---

# API Gateway

API Gateway is a fully managed service provided by Amazon Web Services that allows you to create and manage REST APIs.
It can be used to invoke Lambda functions or direct traffic to Kinesis Data Streams. API Gateway is serverless, meaning
there is no infrastructure to manage. It supports WebSocket for real-time communication and provides features for API
versioning, handling different environments, security, request throttling, Swagger support, and more.

### Endpoint Types

API Gateway supports different endpoint types:

- Edge-optimized: Optimized for global clients by using Amazon CloudFront.
- Regional: Used for clients within the same region as the API Gateway.
- Private: Accessed only through your VPC using Elastic Network Interfaces (ENI).

### Security

API Gateway provides several options for security:

- Authentication: Supports IAM roles, Cognito (for identifying external users), or custom authorizers.
- Resource Policies: Allow cross-account access combined with IAM security or specific access from a VPC endpoint or
  source IP.
- Cognito: Can manage user lifecycles, where users are authenticated with Cognito and authorized at the API Gateway.
  Users receive a token from the Cognito pool and pass it to the API Gateway for evaluation.
- Lambda Authorizer: Allows you to use a Lambda function to authenticate and authorize requests. For example, you can
  pass request parameters to a Lambda authorizer, which evaluates them and returns an IAM policy. Authentication happens
  externally, but authorization occurs in the Lambda function.

### Deployment Stages

API Gateway uses deployment stages to manage changes and make them take effect. Stages can represent different
environments such as development, testing, and production. For example, a new version of a Lambda function can be
accessed through a new version stage, and clients can be gradually migrated to the new URL. Stage variables can be used
as environment variables for API Gateway stages and can also be accessed from Lambda function contexts. Canaries and
blue/green deployments are also supported.

### Integration Types

API Gateway supports different integration types:

- MOCK: Allows you to define a response without sending a request to a backend. Useful for development and testing
  purposes.
- HTTP (Lambda & AWS Services): Requires configuration of data mapping using mapping templates for integration requests
  and responses.
- AWS_PROXY (Lambda Proxy): The Lambda function receives the request from the client and is responsible for the logic of
  handling the request and generating the response. No mapping templates, headers, or query string parameters are passed
  as arguments to the Lambda function.
- HTTP_PROXY: Forwards the HTTP request to a backend, such as an Application Load Balancer (ALB), and returns the
  response to the API Gateway.
- Mapping templates: Used to modify requests and responses. They use Velocity Template Language (VTL) and can add
  headers, modify the body, query string parameters, and more. For example, you can use mapping templates to integrate a
  SOAP API with API Gateway by transforming JSON payloads to XML and vice versa.

### Caching
Caching in API Gateway can be defined per stage and ranges from 0.5 GB to 237 GB. However, caching can be quite expensive, so it's recommended to use it only in production environments. Invalidation of the cache can be done either by flushing the entire cache or by including the header `Cache-Control: max-age=0` in the request.

### API Keys
API Gateway allows you to create APIs and configure methods to require an API key. You can then deploy these APIs to different stages. To distribute the API keys to application developers, you can generate or import keys. Additionally, you can create a usage plan with throttling and quota settings and associate the API stages and keys with the plan. Callers need to supply the API key in the `x-api-key` header with their requests.

### WebSocket API
WebSocket API in API Gateway enables two-way interaction between clients and servers, allowing the server to push data to clients. This makes WebSocket API stateful and suitable for use in scenarios such as chats, collaborations, multiplayer gaming, and forex trading. WebSocket API can work with different AWS services including Lambdas, RDS, and HTTP endpoints. The URL for a WebSocket API is `wss://[id].api.[region].amazonaws.com/[stage]`.

Clients connect to the API WebSocket gateway, which then invokes a Lambda function with a `connectionId`. This `connectionId` can be persisted, for example, in DynamoDB. Clients can send messages that are forwarded from the gateway to another Lambda function, using the same `connectionId`. The server uses a Connection URL callback, where the Lambda function makes an HTTP signed call with IAM Sig V4 to the Connection URL callback, passing the `connectionId`. This allows the server to send a message from the gateway back to the client, enabling two-way communication.

### Routing
In a WebSocket API, incoming JSON messages can be routed to different backends or a `$default` backend. Route selection is based on a route selection expression that selects a JSON field to route from. API Gateway has a route key table defined, which is used to check the incoming JSON for matches. If a match is found, the message is routed accordingly. If no route is matched, the message is sent to the `$default` backend.

---
# SAM (Serverless Application Model)

The Serverless Application Model (SAM) is a framework for developing and deploying serverless applications using a YAML-based syntax. It simplifies the creation of complex CloudFormation configurations.

SAM provides several resources for defining serverless applications:

- `Transform: 'AWS::Serverless-2023-01-01'`: Specifies the version of the SAM specification.
- `AWS::Serverless::Function`: Defines a serverless function resource.
- `AWS::Serverless::Api`: Defines a serverless API resource.
- `AWS::Serverless::SimpleTable`: Defines a DynamoDB table resource.

The following commands are commonly used with SAM:

- `aws cloudformation package` or `sam package`: Packages the SAM template, zips and uploads the code to an S3 bucket.
- `aws cloudformation deploy` or `sam deploy`: Deploys the CloudFormation stack using the packaged artifacts from the S3 bucket.

SAM provides policy templates for defining permissions for Lambda functions. These templates include:

- S3ReadPolicy: Grants read access to Amazon S3.
- SQSPollerPolicy: Grants permissions to poll Amazon Simple Queue Service (SQS).
- DynamoDBCrudPolicy: Grants CRUD (Create, Read, Update, Delete) access to DynamoDB.

---
# CDK (Cloud Development Kit)

CDK (Cloud Development Kit) is a framework that allows you to define cloud infrastructure using a programming language. It provides a higher-level abstraction compared to writing raw CloudFormation templates, enabling you to define infrastructure using familiar programming languages.

Here's an example of defining an AWS CloudFormation stack using the CDK in Java:

```java
import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.core.StackProps;
import software.amazon.awscdk.services.s3.Bucket;

public class MyStack extends Stack {

    public MyStack(final Construct scope, final String id) {
        this(scope, id, null);
    }

    public MyStack(final Construct scope, final String id, final StackProps props) {
        super(scope, id, props);

        Bucket bucket = Bucket.Builder.create(this, "MyBucket")
                .bucketName("my-bucket-name")
                .build();
    }
}
```
With CDK, you define your cloud infrastructure using the CDK constructs, such as Bucket, and the resulting code is compiled to a CloudFormation template. You can then deploy both the infrastructure and your application runtime code together.

---
# Cognito

Cognito is a service provided by AWS that allows you to manage user identities and provide authentication and authorization functionality in your applications.

### User Pools

User pools in Cognito are used to handle user sign-in functionality. They provide a serverless database for managing users of web and mobile applications. With user pools, you can handle user registration, sign-in, and account recovery. It also supports features like multi-factor authentication, email and SMS verification, and user attribute customization. User pools can be integrated with other AWS services and can invoke Lambda functions synchronously on triggers.

### Identity Pools

Identity pools, also known as federated identities, are used for external users who are already authenticated through other identity providers. With identity pools, you can create identities for your users, which allows them to obtain temporary credentials for direct access to AWS services. This enables you to control access to your AWS resources based on user identities obtained from external identity providers, such as Amazon, Google, or Facebook.

### Sync

Cognito Sync is a feature that allows you to synchronize data to Cognito. It can be used to store preferences, configuration settings, and the state of your application across multiple devices. Sync provides cross-device synchronization, ensuring that the data remains consistent across various platforms. It also supports push sync, which allows your application to be notified when there are changes to the identity data.

Cognito provides a comprehensive solution for managing user identities, authentication, and authorization in your applications. It offers both user pools and identity pools to cater to different use cases.

---
# Step Functions

Step Functions is a fully managed service provided by AWS that allows you to model and execute workflows as state machines. Workflows can be modeled using JSON, and Step Functions provides visualization and monitoring capabilities.

### Tasks

Step Functions supports a variety of tasks that can be included in your workflows. These tasks can invoke Lambda functions, run batch jobs, execute ECS tasks, insert data into a database, publish messages to SNS, or even launch another Step Function. Additionally, you can run Activities, which are external processes or services that perform work and return results back to the Step Function.

### States

Step Functions provide various states that can be used in your workflows. These states include choice, fail or succeed, pass, wait, map, and parallel. You can also use retry and catch to handle errors. Errors can be categorized into different types such as States.ALL, States.Timeout, States.TaskFailed, and States.Permissions.

### Standard Workflows

Standard workflows in Step Functions have a maximum duration of 1 year. They support an execution start rate of over 2,000 per second and a state transition rate of over 4,000 per second per account. Pricing is based on the number of state transitions. Execution history can be listed and described using Step Functions APIs and can be inspected using CloudWatch. Standard workflows provide exactly-once workflow execution.

### Express Workflows

Express workflows in Step Functions have a maximum duration of 5 minutes. They support an execution start rate of over 100,000 per second and have nearly unlimited state transitions. Pricing for express workflows is based on the number of executions, duration, and memory consumption. Execution details can be inspected through CloudWatch. Express workflows provide at-least-once workflow execution.

Step Functions is a powerful service that allows you to orchestrate and coordinate complex workflows. It provides reliability, scalability, and monitoring capabilities for your workflows.

---
# AppSync

AppSync is a managed service provided by AWS that enables you to build and deploy GraphQL APIs. It allows you to include data from multiple sources and provides real-time data retrieval capabilities using WebSocket or MQTT protocols.

### Features

- **Managed service:** AppSync is a fully managed service that takes care of the infrastructure and scaling for you, allowing you to focus on building your GraphQL API.

- **GraphQL:** AppSync uses the GraphQL query language to define your API schema and interact with your data sources. GraphQL provides a flexible and efficient way to fetch data and allows clients to specify exactly what data they need.

- **Multiple data sources:** AppSync can integrate with various data sources such as DynamoDB, RDS, Lambda, Elasticsearch, and more. You can combine data from multiple sources and define resolvers to fetch the data and respond to client queries.

- **Real-time data retrieval:** AppSync supports real-time data retrieval using WebSocket or MQTT protocols. This allows clients to subscribe to data changes and receive updates in real-time, enabling reactive and interactive applications.

### How it works

1. **Upload schema:** You start by uploading your GraphQL schema to AppSync. The schema defines the types, fields, and operations supported by your API.

2. **Client sends GraphQL query:** Clients can send GraphQL queries to AppSync, specifying the data they need and the structure of the response.

3. **Resolving queries:** When a query is received, AppSync resolves it by fetching the data from the configured data sources. This can involve making requests to DynamoDB, invoking Lambda functions, or querying other services.

4. **Sending response:** Once the data is retrieved, AppSync sends the response back to the client in JSON format, containing the requested data.

AppSync simplifies the process of building and deploying GraphQL APIs, providing real-time capabilities and supporting multiple data sources. It enables you to build efficient and flexible APIs that meet the needs of your applications.

---
# Security Token Service (STS)

The Security Token Service (STS) is an AWS service that provides temporary credentials for accessing AWS resources. These temporary credentials are valid for a specified duration, ranging from 15 minutes up to 1 hour.

### Key Operations

- **AssumeRole:** Allows you to assume a specified IAM role and obtain temporary credentials that are associated with that role.

- **AssumeRoleWithSAML:** Enables you to assume an IAM role using a SAML-based identity provider (IdP) for authentication.

- **AssumeRoleWithWebIdentity:** Allows you to assume an IAM role using a web identity provider, such as Amazon Cognito, for authentication.

- **GetSessionToken:** Retrieves a set of temporary security credentials (access key, secret access key, and session token) for a user or federated user.

- **GetFederationToken:** Generates a set of temporary security credentials for a federated user.

- **GetCallerIdentity:** Returns details about the AWS account that owns the calling identity (the user or role whose credentials are used to call the operation).

- **DecodeAuthorizationMessage:** Decodes an encoded authorization message, which can be useful for troubleshooting authorization issues.

### Usage

STS allows you to assume roles, which involves defining an IAM role and specifying which principals are allowed to assume that role. Once the role is defined, you can use the AWS STS API to retrieve temporary credentials and impersonate the IAM role.

### Multi-Factor Authentication (MFA)

STS also supports Multi-Factor Authentication (MFA) when obtaining temporary credentials. By using the `GetSessionToken` operation with the appropriate IAM policy that includes IAM conditions, such as `aws:MultiFactorAuthPresent:true`, you can enforce MFA requirements for accessing AWS resources.

STS provides a secure and flexible way to manage temporary credentials and control access to AWS resources. It enables you to implement granular access control policies and enforce additional security measures, such as MFA, to protect your AWS infrastructure.

---
# Directory Services

AWS Directory Services is a managed service that allows you to create and manage directories in the AWS Cloud. It provides several options for integrating with Microsoft Active Directory (AD) and simplifies the management of users, groups, and directory resources.

### AWS Managed Microsoft AD

AWS Managed Microsoft AD allows you to create your own Active Directory in the AWS Cloud. It provides a fully managed AD service with features like user and group management, integrated DNS, and support for multi-factor authentication (MFA). You can use AWS Managed Microsoft AD to manage your users locally within AWS and leverage AWS services that integrate with Active Directory.

One of the key features of AWS Managed Microsoft AD is the ability to establish trust connections with on-premises Active Directory environments. This enables you to extend your existing on-premises AD to the AWS Cloud and seamlessly integrate your on-premises and AWS resources.

### AD Connector

AD Connector is a directory gateway that allows you to redirect directory requests from AWS services to your on-premises Active Directory. It acts as a proxy between AWS services and your on-premises AD. AD Connector supports multi-factor authentication (MFA) and provides secure access to your on-premises directory resources.

With AD Connector, users and groups are managed on the on-premises AD, and AWS services can use the AD Connector to authenticate and authorize access to your on-premises resources.

### Simple AD

Simple AD is an Active Directory-compatible managed directory service provided by AWS. It is a cost-effective option for organizations that need basic AD functionality without the complexity of managing a full-scale Active Directory infrastructure.

Unlike AWS Managed Microsoft AD and AD Connector, Simple AD cannot be joined with an on-premises Active Directory environment. It is designed to be a standalone directory service within the AWS Cloud.

Directory Services simplifies the deployment and management of directory resources in the AWS Cloud, providing flexibility and scalability for your directory needs.
