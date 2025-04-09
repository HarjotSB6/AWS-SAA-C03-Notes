# AWS SAA-C03 Notes

*Harjot Singh*

# Access Management

## Identity & Access Management (IAM)

- **Global Service** - IAM entities like roles can be used in any region without recreation

**Users & Groups**

- Groups are collections of users with attached policies
- Groups cannot be nested
- Users can belong to multiple groups
- **Root User** has full access to the account
- **IAM User** has limited permissions to the account
- Best practice: Use IAM users with admin access instead of root

**Policies**

- JSON documents that outline permissions
- **User-based policies**: Define allowed API calls for specific users
- **Resource-based policies**: Control access to AWS resources
- Access principle: A principal can access a resource if user policy ALLOWS it OR the resource policy ALLOWS it AND there's no explicit DENY
- Follow **least privilege principle**

**Trust Policies**

- Define which principal entities can assume the role
- IAM roles are both an identity and a resource that supports resource-based policies
- Must attach both a trust policy and an identity-based policy to an IAM role

**Roles**: Collection of policies for AWS services

**Protection Mechanisms**

- **Password Policy**: Enforces standards (rotation, reuse) and prevents brute force attacks
- **Multi-Factor Authentication (MFA)**: Recommended for root and IAM users

**Reporting Tools**

- **Credentials Report**: Lists all users and credential status (MFA, rotation)
- **Access Advisor**: Shows service permissions granted to users and when last accessed

**Access Keys**

- Required for AWS CLI and SDK
- Never share access keys
- Access Key ID ~ username
- Secret Access Key ~ password

**Assume Role vs Resource-based Policy**

- When assuming a role, you give up original permissions for role permissions
- With resource-based policies, principal retains original permissions

**IAM Permission Boundaries**

- **Apply to IAM users & roles (not groups).**
- **Set max permissions** an IAM entity can receive via a managed policy.
- **Can work alongside AWS Organizations SCP.**
    
    **Use Cases**
    
    - Delegate responsibilities **without full admin rights** (e.g., create IAM users).
    - Allow developers to **self-manage policies** while preventing privilege escalation.
    - **Restrict permissions** for specific users **without affecting the whole account.**

**AWS IAM Identity Center (Successor to AWS SSO)**

- **Single sign-on (SSO)** for AWS accounts & cloud apps.
- **Supports:** AWS Organizations, SAML 2.0 apps, EC2 Windows, & Identity Providers.
- **Identity Sources:** Built-in store, Active Directory (AD), Okta, OneLogin, etc.

![](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220505202904.jpg)

## **Amazon Cognito**

- **Provides user identity** for web & mobile apps.
- **Cognito User Pools:** Sign-in for app users, integrates with API Gateway & ALB.
- **Cognito Identity Pools (Federated Identity):** Grants AWS credentials, integrates with User Pools.
- **Cognito vs IAM:** Scales for "hundreds of users", mobile users, & SAML authentication.

## **AWS Organizations**

- **Global service** to manage multiple AWS accounts.
- **Management account** controls **member accounts** (each part of only one organization).
- **Consolidated billing** with a single payment method & volume-based discounts (EC2, S3).
- **Share reserved instances & Savings Plans** across accounts.
- **API available** for automated AWS account creation.

**Advantages of AWS Organizations**

- **Multi-Account vs. One Account Multi-VPC** for better isolation & security.
- **Tagging standards** for cost allocation & billing.
- **Centralized logging**: Enable **CloudTrail** on all accounts & send logs to a central S3.
- **Centralized monitoring**: Send **CloudWatch Logs** to a logging account.
- **Cross-account roles** for centralized admin access.
- **Security Control**: Use **Service Control Policies (SCP)** to restrict IAM roles & users at OU/account level.
- **SCPs don’t apply to the management account** & require explicit allow from the root.

![image.png](image%207.png)

# Compute Services

## Elastic Compute Cloud (EC2)

- **Regional Service**
- **Infrastructure as a Service (IaaS)**
- Stopping/starting may change public IP but not private IP
- **AWS Compute Optimizer** recommends optimal resources
- vCPU-based On-Demand Instance soft limit per region

User Data

- Commands run only on first instance launch
- Automates dynamic boot tasks (updates, software installation)
- Runs with root user privileges

Instance Classes

- **General Purpose**: Web servers, code repositories
- **Compute Optimized**: Batch processing, transcoding, HPC
- **Memory Optimized**: In-memory databases, caches
- **Storage Optimized**: OLTP, distributed file systems

Security Groups

- **Only contain Allow rules**
- External firewall for EC2 instances
- Can reference resources by IP or Security Group
- Default settings:
    - Default SG: Inbound from same SG allowed, all outbound allowed
    - New SG: All inbound blocked, all outbound allowed
- Can attach to multiple instances and vice versa
- Bound to a VPC/region
- Blocked requests result in timeout errors

IAM Roles for EC2

- Never enter AWS credentials directly into EC2 instances
- Attach IAM roles instead

Purchasing Options

1. On-demand Instances
    - Pay per use (no upfront payment)
    - Highest cost
    - No long-term commitment
    - Recommended for short-term, uninterrupted and **unpredictable** workloads
2. Standard Reserved Instances
    - Reservation Period: 1 year or 3 years
    - Recommended for steady-state applications (like database)
    - **Sell unused instances** on the Reserved Instance Marketplace
3. Convertible Reserved Instances[](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/elastic%20compute%20cloud%20(ec2)/#convertible-reserved-instances)
    - Can change the instance type
    - Lower discount
    - **Cannot sell unused instances** on the Reserved Instance Marketplace
4. Scheduled Reserved Instances[](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/elastic%20compute%20cloud%20(ec2)/#scheduled-reserved-instances)
    - Reserved for a time window (ex. everyday from 9AM to 5PM)
5. Spot Instances[](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/elastic%20compute%20cloud%20(ec2)/#spot-instances)
    - Work on a bidding basis where you are willing to pay a specific max
    hourly rate for the instance. Your instance can terminate if the spot
    price increases.
    - **Spot blocks** are designed not to be interrupted
    - Good for workloads that are resilient to failure
        - Distributed jobs (resilient if some nodes go down)
        - Batch jobs
6. Dedicated Hosts[](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/elastic%20compute%20cloud%20(ec2)/#dedicated-hosts)
    - Server hardware is allocated to a specific company (not shared with other companies)
    - **3 year reservation** period
    - Billed per host
    - Useful for software that have **BYOL (Bring Your Own License)** or for companies that have strong regulatory or compliance needs
7. Dedicated Instances[](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/elastic%20compute%20cloud%20(ec2)/#dedicated-instances)
    - Dedicated hardware
    - Billed per instance
    - No control over instance placement
8. On-Demand Capacity Reservations[](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/elastic%20compute%20cloud%20(ec2)/#on-demand-capacity-reservations)
    - Ensure you have the available capacity in an AZ to launch EC2 instances when needed
    - Can reserve for a recurring schedule (ex. everyday from 9AM to 5PM)
    - No need for 1 or 3-year commitment (independent of billing discounts)
        - Need to specify the following to create capacity reservation: - AZ - Number of instances - Instance attributes
    

**On-demand Instances**: Pay per use, highest cost, no commitment

**Standard Reserved Instances**: 1-3 year commitment, steady workloads

**Convertible Reserved Instances**: Can change instance type

**Scheduled Reserved Instances**: Reserved for specific time windows

**Spot Instances**: Bidding-based, can be terminated if price increases

**Dedicated Hosts**: Hardware for specific company, BYOL

**Dedicated Instances**: Dedicated hardware, less control

**On-Demand Capacity Reservations**: Reserve capacity without long commitment

![](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/attachments/Pasted%20image%2020220516114146.jpg)

Spot Instances Details

- **Spot Requests**: One-time or persistent
- **Spot Fleets**: Combination of spot and on-demand instances
    - Launch Templates must be used to have on-demand instances in the fleet
    - Allocation strategies: lowestPrice, diversified, capacityOptimized

Elastic IP

- Static public IP you own
- Can be attached to stopped instances
- Limit of 5 per account
- Free when associated with running instance with only one EIP

Placement Groups

- **Cluster**: Same rack, great network, used for HPC
- **Spread**: Separate racks, max availability, max 7 instances per AZ
- **Partition**: Instances share racks within partitions, for big data

Elastic Network Interface (ENI)

- Virtual network card giving private IP
- Primary ENI created automatically
- Additional ENIs can be attached/detached
- Tied to subnet/AZ

Instance States

- **Stop**: EBS root volume preserved
- **Terminate**: EBS root volume destroyed
- **Hibernate**: RAM contents saved to EBS, faster restart
- **Standby**: Instance remains in Auto Scaling Group but out of service

Other EC2 Features

- **EC2 Nitro**: New virtualization tech, better networking/security
- **vCPU & Threads**: vCPU is total concurrent threads
- **Amazon Machine Image (AMI)**: Pre-packaged instance image
- **EC2 Classic & ClassicLink**: Legacy single network (deprecated)
- **Run Command**: Systems Manager feature for remote configuration

Instance Tenancy[](https://tahseer-notes.netlify.app/notes/aws%20solutions%20architect%20associate/elastic%20compute%20cloud%20(ec2)/#instance-tenancy)

- **Default**: Instance runs on shared hardware
- **Dedicated**: Instance runs on single-tenant hardware
- **Host**: Instance runs on dedicated host

> Tenancy of an instance can only be changed from host to dedicated or dedicated to host after the instance has been launched. Dedicated instance tenancy takes precedence over Default instance tenancy
> 

# Containerization

## Amazon ECS (Elastic Container Service)

Launch Docker containers

- **EC2 Launch Type**: You provision infrastructure
- **Fargate Launch Type**: Serverless, no infrastructure provisioning
    - Fargate + EFS = Serverless
- Task definitions specify port, image, storage details
- ECS Service spins up new tasks
    
    ![image.png](image%208.png)
    

**ECS Service Auto Scaling :**

- Automatically increases or decreases the desired number of ECS tasks
- Uses AWS Application Auto Scaling
    - ECS Service Average CPU Utilization
    - ECS Service Average Memory Utilization (RAM)
    - ALB Request Count Per Target

**Scaling Methods:**

- Target Tracking: Scales based on a target value for a CloudWatch metric
- Step Scaling: Scales based on a specified CloudWatch Alarm
- Scheduled Scaling: Scales based on specified date/time for predictable changes

- ECS Service Auto Scaling (task level) ≠ EC2 Auto Scaling (instance level)
- Fargate Auto Scaling is noted as being easier to set up due to its serverless nature

## Amazon ECR (Elastic Container Registry):

- Used to store and manage Docker images on AWS
- Offers both private and public repositories (
- Fully integrated with ECS  backed by Amazon S3 for storage
- Access control managed through IAM (permission errors indicate policy issues)
- Supports multiple container image management features:
    - Vulnerability scanning
    - Versioning
    - Image tags
    - Image lifecycle management

## Amazon EKS (Elastic Kubernetes Service):

- A service for launching managed Kubernetes clusters on AWS
- Presented as an alternative to ECS with similar goals but different API
- Supports EC2 for deploying worker nodes or Fargate for serverless containers

> Kubernetes is cloud-agnostic (can be used in any cloud including Azure, GCP, etc.)
> 

![image.png](image%209.png)

EKS Node Types:

**Managed Node Groups:**

- Creates and manages Nodes (EC2 instances) for you
- Nodes are part of an ASG (Auto Scaling Group) managed by EKS
- Supports On-Demand or Spot Instances

**Self-Managed Nodes:**

- Nodes created by you and registered to the EKS cluster and managed by an ASG
- You can use prebuilt AMI - Amazon EKS Optimized AMI
- Supports On-Demand or Spot Instances

**AWS Fargate:**

- No maintenance required; no nodes managed

EKS Data Volumes:

**Requirements:**

- Need to specify StorageClass manifest on your EKS cluster
- Leverages a Container Storage Interface (CSI) compliant driver

**Supported Storage Options:**

- Amazon EBS
- Amazon EFS (works with Fargate)
- Amazon FSx for Lustre
- Amazon FSx for NetApp ONTAP

## AWS App Runner:

- Fully managed service for easy deployment of web applications and APIs at scale
- No infrastructure experience required

**Key Features:**

- Start with your source code or container image
- Automatically builds and deploys the web app
- Provides automatic scaling, high availability, load balancing, and encryption
- Supports VPC access
- Connects to database, cache, and message queue services

**Use Cases:** Web applications, APIs, Microservices, Rapid production deployments

# Database Services

## Relational Database Service (RDS)

- Managed database service for:
    - PostgreSQL
    - MySQL
    - MariaDB
    - Oracle
    - Microsoft SQL Server
    - Aurora
- **Managed Service Features**:
    - Automated provisioning and OS patching
    - Backups, point-in-time restore
    - Monitoring dashboards
    - Read replicas
    - Multi-AZ deployment
    - Storage backed by EBS
- **Backups**:
    - Automatic with logs (7-35 day retention)
    - Manual snapshots (infinite retention)
    - Auto scaling storage
- **Read Replicas**:
    - Same region, different AZ (free)
    - Up to 5 available
    - For scaling read performance
- **Multi-AZ**:
    - For high availability, not scaling
    - Can enable read replicas as Multi-AZ backup
- **RDS Custom**:
    - Access to underlying DB and OS
    - Can SSH, install patches, configure settings

![image.png](image%2010.png)

## Amazon Aurora

- Compatible with PostgreSQL and MySQL
- Cloud-optimized (5x MySQL performance, 3x PostgreSQL)
- Automatic storage growth
- 15 replicas, high availability
- **Aurora Serverless:** for unpredictable/intermittent workloads, no capacity planning
- **Aurora Multi-Master:** for continuous writes failover (high write availability)
- **Aurora Global:** up to 16 DB Read Instances in each region, < 1 second storage replication
- **Aurora Machine Learning:** perform ML using SageMaker & Comprehend on Aurora
- **Aurora Database Cloning:** new cluster from existing one, faster than restoring a snapshot

![image.png](image%2011.png)

## ElastiCache

- Need to change application code to leverage
- Key/Value Store
- Must provisions EC2 Instance Type
- **Caching Patterns**:
    - Lazy Loading
    - Write Through
    - Session Store

![image.png](image%2012.png)

## Amazon DynamoDB

- Fully managed, highly available with replication across multiple AZs
- NoSQL database - not a relational database - with transaction support
- Scales to massive workloads, distributed database
- Millions of requests per seconds, trillions of row, 100s of TB of storage
- Fast and consistent in performance (single-digit millisecond)
- Integrated with IAM for security, authorization and administration
- Low cost and auto-scaling capabilities
- No maintenance or patching, always available
- Standard and Infrequent Access (IA) Table Class

**DynamoDB – Read/Write Capacity Modes**

- Control how you manage your table's capacity (read/write throughput)
- Provisioned Mode (default)
    - You specify the number of reads/writes per second
    - You need to plan capacity beforehand
    - Pay for provisioned Read Capacity Units (RCU) & Write Capacity Units (WCU)
    - Possibility to add auto-scaling mode for RCU & WCU
- On-Demand Mode
    - Read/writes automatically scale up/down with your workloads
    - No capacity planning needed
    - Pay for what you use, more expensive ($$$)
    - Great for unpredictable workloads, steep sudden spikes
    

**DynamoDB Accelerator (DAX)**

- Fully-managed, highly available, seamless in-memory cache for DynamoDB
- Help solve read congestion by caching
- Microseconds latency for cached data
- Doesn't require application logic modification (compatible with existing DynamoDB APIs)
- 5 minutes TTL for cache (default)

**DynamoDB – Stream Processing**

- DynamoDB Global Tables Read/Write Replica
- TTL
- Ordered stream of item-level modifications (create/update/delete) in a table
- Use cases:
    - React to changes in real-time (welcome email to users)
    - Real-time usage analytics
    - Insert into derivative tables
    - Implement cross-region replication
    - Invoke AWS Lambda on changes to your DynamoDB table

**DynamoDB Streams**

- 24 hours retention
- Limited # of consumers
- Process using AWS Lambda Triggers, or DynamoDB Stream Kinesis adapter

**Kinesis Data Streams (newer)**

- 1 year retention
- High # of consumers
- Process using AWS Lambda, Kinesis Data Analytics, Kinesis Data Firehose, AWS Glue Streaming ETL...

![image.png](image%2013.png)

## **DocumentDB**

- Aurora is an “AWS-implementation” of PostgreSQL / MySQL …
- **DocumentDB is the same for MongoDB (which is a NoSQL database)**
- MongoDB is used to store, query, and index JSON data
- Similar “deployment concepts” as Aurora
- Fully Managed, highly available with replication across 3 AZ
- Aurora storage automatically grows in increments of 10GB, up to 64TB.
- Automatically scales to workloads with millions of requests per second

## **Amazon Neptune**

- Fully managed **graph** database
- Highly available across **3 AZ**, with up to **15 read replicas**
- Build and run applications working with highly connected datasets – optimized for these complex and hard queries
- Can store up to **billions of relations** and query the graph with **milliseconds latency**
- Highly available with replications across **multiple AZs**
- Great for **knowledge graphs** (Wikipedia), **fraud detection, recommendation engines, social networking**

## Amazon Keyspaces (for Apache Cassandra)

- Apache Cassandra is an open-source NoSQL distributed database
- A managed Apache Cassandra-compatible database service
- Serverless, Scalable, highly available, fully managed by AWS
- Automatically scale tables up/down based on the application's traffic
- Tables are replicated 3 times across multiple AZ
- Using the Cassandra Query Language (CQL)
- Single-digit millisecond latency at any scale, 1000s of requests per second
- Capacity: On-demand mode or provisioned mode with auto-scaling
- Encryption, backup, Point-In-Time Recovery (PITR) up to 35 days
- Use cases: store IoT devices info, time-series data, ...

## Amazon QLDB

- QLDB stands for ”Quantum Ledger Database”
- A ledger is a book recording financial transactions
- Fully Managed, Serverless, High available, Replication across 3 AZ
- Used to review history of all the changes made to your application data over time
- Immutable system: no entry can be removed or modified, cryptographically verifiable
- 2-3x better performance than common ledger blockchain frameworks, manipulate data using SQL
- Difference with Amazon Managed Blockchain: no decentralization component, in accordance with financial regulation rules

## Amazon Timestream

- Fully managed, fast, scalable, serverless time series database
- Automatically scales up/down to adjust capacity
- Store and analyze trillions of events per day
- 1000s times faster & 1/10th the cost of relational databases
- Scheduled queries, multi-measure records, SQL compatibility
- Data storage tiering: recent data kept in memory and historical data kept in a cost-optimized storage
- Built-in time series analytics functions (helps you identify patterns in your data in near real-time)
- Encryption in transit and at rest
- Use cases: IoT apps, operational applications, real-time
analytics, …

## Database Types

- **RDBMS (= SQL / OLTP):** RDS, Aurora – great for joins
- **NoSQL database – no joins, no SQL:** DynamoDB (~JSON), ElastiCache (key/value pairs), Neptune (graphs), DocumentDB (for MongoDB), Keyspaces (for Apache Cassandra)
- **Object Store:** S3 (for big objects) / Glacier (for backups / archives)
- **Data Warehouse (= SQL Analytics / BI):** Redshift (OLAP), Athena, EMR
- **Search:** OpenSearch (JSON) – free text, unstructured searches
- **Graphs:** Amazon Neptune – displays relationships between data
- **Ledger:** Amazon Quantum Ledger Database
- **Time series:** Amazon Timestream

# Storage Services

## Simple Storage Service (S3)

- **Regional Service**
- Use cases: Backup, archive, application hosting, analytics, recovery
- Requestor Pays — Owner Pays Storage | Requestor Pays Transfer Cost
- **Access Security**:
    - IAM/Roles
    - Bucket Policies (JSON)
    - Object/Bucket ACLs
    - Encryption
- **CORS**: Cross Origin Resource Sharing
- **Storage Classes**:
    1. **Standard - General**: Frequent access (free retrieval)
    2. **Standard - Infrequent Access (IA)**: Backups
    3. **One Zone - IA**: Secondary backups (single AZ)
    4. **Glacier Instant Retrieval**: 90-day access duration
    5. **Glacier Flexible Retrieval**: Various retrieval speeds
    6. **Glacier Deep Archive**: Lowest cost, 180-day access duration
    7. **Intelligent Tiering**: Auto-tier with monitoring fee (free retrieval)
- **Lifecycle Management**:
    - Transition rules
    - Expiration rules
    - Cannot revert from Glacier to Standard
- **Performance Features**:
    - 3500 PUT/COPY/POST/DELETE per second per prefix
    - 5500 GET/HEAD per second per prefix
    - Multi-Part Upload (>100MB recommended, >5GB required)
    - Transfer Acceleration (via Edge Locations)
    - Byte Range Fetches (partial retrieval)

![image.png](image%2014.png)

![image.png](image%2015.png)

![image.png](image%2016.png)

![image.png](image%2017.png)

All Storage 

![image.png](image%2018.png)

![image.png](image%2019.png)

## API Gateway

Integrates with Lambda to expose REST API

HTTP Endpoints, API and AWS API

- **Endpoint Types**:
    - Edge-optimized (CloudFront)
    - Regional
    - Private (ENI/VPC Endpoint)
- **Security**:
    - IAM, Cognito, Custom Auth
    - Custom Domain HTTPS AWS Certificate Manager integration
    - CNAME/Alias requirements in Route 53
    

# Networking

## Virtual Private Cloud (VPC)

![image.png](image%2020.png)

- Private network in AWS
- Subnet configuration
- Internet and NAT gateways

![image.png](image%2021.png)

- **Security**:
    - Security Groups (stateful, allow rules only)
    - Network ACLs (stateless, allow/deny rules)
    
    ![image.png](image%2022.png)
    

## **VPC Endpoints (AWS PrivateLink)**

- **Connect privately** to AWS services **without public internet** (no IGW/NATGW needed).
- **Redundant & horizontally scalable** for reliability.
- **Troubleshooting:** Check **DNS settings & route tables**.

**Types of Endpoints:**

- **Interface Endpoints (PrivateLink)**
    - **Creates an ENI (private IP)** as an entry point.
    - **Requires a Security Group**.
    - **Supports most AWS services**.
    - **Paid** ($ per hour + $ per GB).
- **Gateway Endpoints**
    - **Creates a gateway** (must be in a **route table**).
    - **Supports only S3 & DynamoDB**.
    - **Free**.

## **AWS Site-to-Site VPN**

- **Secure connection** between on-premises networks and AWS **over IPsec tunnels**.
- **Components:**
    - **Virtual Private Gateway (VGW):** AWS-side VPN concentrator, attached to the VPC.
    - **Customer Gateway (CGW):** On-premises device/software managing the VPN connection.
- **Public IP Requirement:**
    - CGW must have a **public Internet-routable IP** (or the NAT device's public IP if behind NAT-T).
- **Route Propagation:** Enable for the **Virtual Private Gateway** in route tables.
- **ICMP Traffic:** Allow in **Security Groups** if you need to ping EC2 from on-premises.
- **Customizable ASN** (Autonomous System Number) for BGP routing.

## **AWS Direct Connect (DX)**

- **Dedicated private network connection** from an on-premises data center to AWS.
- **Bypasses the public Internet** → **lower latency & higher reliability**.
- **Supports both public (S3) & private (EC2) resources** on the same connection.
- **Use Cases:**
    - High bandwidth needs (large datasets, hybrid environments).
    - Low-latency & consistent network experience (real-time data feeds).
- Supports **IPv4 & IPv6**.

**Direct Connect Gateway**

- Required to **connect multiple VPCs across different regions** within the same AWS account.

**Connection Types:**

1. **Dedicated Connection** (1, 10, 100 Gbps) - Direct AWS request.
2. **Hosted Connection** (50 Mbps - 10 Gbps) - Provided via AWS Direct Connect Partners.

**Encryption**

- **DX does not encrypt traffic by default** (but it's private).
- **Use DX + VPN** for **IPsec encryption** (adds security but increases complexity).
- Lead times for setup can **exceed 1 month** for new connections.

## **AWS Transit Gateway**

- **Hub-and-spoke model** for connecting **thousands of VPCs & on-premises networks**.
- **Regional resource** → supports **cross-region peering**.
- **Cross-account sharing** via **AWS Resource Access Manager (RAM)**.
- **Route Tables** → control which VPCs can communicate.
- **Works with**:
    - **Direct Connect Gateway** (DX integration).
    - **VPN connections**.
- **Supports IP Multicast** (unique feature, not supported by other AWS services).
- **Site-to-Site VPN ECMP (Equal-Cost Multi-Path Routing)** → **spreads traffic** across multiple best paths for **higher bandwidth & redundancy**.

## **VPC Traffic Mirroring**

- **Captures & inspects network traffic** within a **VPC**.
- **Routes traffic to security appliances** for analysis.
- **Source**: ENIs (Elastic Network Interfaces).
- **Target**: ENI or Network Load Balancer (NLB).
- **Flexible filtering**: capture **all packets** or **specific ones** (can truncate).
- **Works across VPCs** using **VPC Peering**.
- **Use cases**: **content inspection, threat monitoring, troubleshooting**.

# Serverless & Application Services

## Lambda

- Serverless compute

**Execution:**

- Memory allocation: 128 MB – 10GB (1 MB increments)
- Maximum execution time: 900 seconds (15 minutes)
- Environment variables: 4 KB limit
- Disk capacity in the "function container" (/tmp): 512 MB to 10GB
- Concurrency executions: 1000 (can be increased)

**Deployment:**

- Lambda function deployment size (compressed .zip): 50 MB
- Size of uncompressed deployment (code + dependencies): 250 MB
- Can use the /tmp directory to load other files at startup
- Size of environment variables: 4 KB
- Integrated with Step Functions for workflows

Lambda@Edge allows you to use Lambda functions to modify CloudFront requests and responses

**Four Integration Points:**

1. After CloudFront receives a request from a viewer (viewer request)
2. Before CloudFront forwards the request to the origin (origin request)
3. After CloudFront receives the response from the origin (origin response)
4. Before CloudFront forwards the response to the viewer (viewer response)

## Step Functions (Visual Workflow)

- Build serverless workflows for Lambda functions
- Visual flow/algorithm charts

# Machine Learning & AI Services

**Amazon Rekognition**

- Image/video recognition
- Confidence threshold
- Content Moderation
- JSON responses

**Amazon Transcribe**

- Speech-to-text
- Can remove personal info (PII)
- Subtitle generation

**Amazon Polly**

- Text-to-speech
- Lexicons and SSML support

**Amazon Translate**

- Language translation

**Amazon Lex & Connect**

- Chatbot functionality
- Virtual call center (CRM)

**Amazon Comprehend**

- Natural Language Processing
- Specialized medical version available

**SageMaker**

- Build and deploy ML models

**Amazon Forecast**

- ML-based forecasting

**Amazon Kendra**

- Document search using ML/indexing

**Amazon Personalize**

- Recommendation systems

**Amazon Textract**

- Extract data from documents/media

# Monitoring & Management

## CloudWatch

- Metrics for all AWS services
- Near real-time metric streams
- **Logs**:
    - Log Groups and Streams
    - Expiration policies
    - Integration with S3, Kinesis, Lambda, etc.
- **Alarms**:
    - Notification triggers
    - States: OK, INSUFFICIENT_DATA, ALARM
    - Composite alarms (AND/OR logic)
- **Container Insights**: Metrics for ECS, EKS, Kubernetes
- **Lambda Insights**: Lambda monitoring
- **Contributor Insights**: Analyze log data for top contributors
- **Application Insights**: Automated dashboards
- All Insights
    
    CloudWatch Container Insights
    
    - Collect, Summarize metrics and logs from containers
    - For ECS, EKS, Kubernetes on EC2 and Fargate
    - Containerized Versions of Agent for Containers
    
    CloudWatch Lambda Insights
    
    - Monitoring and Troubleshoots for lambda
    - Diagnostics like Cold Starts, Worker Shutdowns, System-level metrics
    - Insights is provided as lambda layer
    
    **CloudWatch Contributor Insights**
    
    - Analyze log data and create time series that display contributor data.
        - See metrics about the top-N contributors.
        - The total number of unique contributors, and their usage.
    - This helps you find top talkers and understand who or what is impacting system performance.
    - Works for any AWS-generated logs (VPC, DNS, etc.)
    - For example, you can find bad hosts, identify the heaviest network users, or find the URLs that generate the most errors.
    - You can build your rules from scratch, or you can also use sample rules that AWS has created - leverages your CloudWatch Logs.
    
    **CloudWatch Application Insights**
    
    - Provides automated dashboards that show potential problems with monitored applications to help isolate ongoing issues.
    - Your applications run on Amazon EC2 Instances with select technologies only (Java, .NET, Microsoft IIS Web Server, databases...).
    - And you can use other AWS resources such as Amazon EBS, RDS, ELB, ASG, Lambda, SQS, DynamoDB, S3 bucket, ECS, EKS, SNS, API Gateway...
    - Powered by SageMaker.
    - Enhanced visibility into your application health to reduce the time it will take you to troubleshoot and repair your applications.
    - Findings and alerts are sent to Amazon EventBridge and SSM OpsCenter.
        
        

![image.png](image%2023.png)

## CloudTrail

- **Provides governance, compliance, and audit for AWS accounts:**
    - CloudTrail is enabled by default.
    - Get a history of events and API calls made within your AWS account by:
        - Console.
        - SDK.
        - CLI.
        - AWS Services.
- **Logs:**
    - Logs from CloudTrail can be put into CloudWatch Logs or S3.
    - A trail can be applied to all regions (default) or a single region.
- **Recommendation:**
    - If a resource is deleted in AWS, it is recommended to investigate CloudTrail first.

**AWS CloudTrail Events**

**Management Events**

- Track **resource operations** in AWS (e.g., IAM role changes, EC2 subnet creation, CloudTrail setup).
- **By default, CloudTrail logs management events** (Read & Write can be separated).

**Data Events**

- Track **high-volume** operations (not logged by default).
- Example: **S3 object-level activity** (GetObject, PutObject, DeleteObject).

**CloudTrail Insights**

- Detects **unusual activity** (e.g., service limits, IAM bursts, maintenance gaps).
- **Baseline analysis** of normal events, alerts for anomalies.
- Logs anomalies in CloudTrail, sends them to S3, and generates **EventBridge events** for automation.

CloudTrail Events Retention

- Events stored for 90 days
- Beyond this period, log them to S3 and use Athena

## AWS Config

- Audits and records compliance
- Answers questions about resource configurations
- SNS notifications for changes
- **Config Rules**:
    - AWS managed or custom (Lambda)
    - Triggered by changes or time intervals

## **Amazon EventBridge**

- **Event buses** can be shared across AWS accounts via **Resource-based Policies**.
- **Event archiving & replay** for past events (all or filtered) with indefinite or time-limited retention.
- **Scheduling (Cron jobs):** Automate scripts on a set schedule.
- **Event patterns:** Define rules to react to AWS service events.
- **Triggers:** Invoke Lambda, send messages to SQS/SNS, and more.

**Amazon EventBridge – Schema Registry**

- **Schema inference:** EventBridge analyzes events to determine their structure.
- **Code generation:** Generates code for applications to process event data efficiently.
- **Versioning:** Schema updates are managed with version control.

**Amazon EventBridge – Resource-Based Policy**

- **Manages permissions** for a specific Event Bus.
- **Controls event access** across AWS accounts or regions.
- **Use case:** Centralize event aggregation for an AWS Organization.

# Messaging & Integration

## Simple Queue Service (SQS)

- **Standard Queue**:
    - 4-14 day retention
    - 256KB per message
- Producer to multiple parallel consumers
- **Security**:
    - In-flight encryption (HTTPS)
    - At-rest encryption (KMS)
    - Client-side encryption
- **Long Polling**: Wait for messages
- **FIFO Queue**: Ordered delivery
    - Limited throughput: 300 msg/s without batching
    - 3000 msg/s with batching

## Simple Notification Service (SNS)

- Producer(Subscribers) —> Topic —> Receiver(Subscription)
- **Security**:
    - In-flight encryption (HTTPS)
    - At-rest encryption (KMS)
    - Client-side encryption
- SNS Access Polices for cross account
- FIFO Queue
- JSON Message filtering

![image.png](image%2024.png)

## Amazon Kinesis

- Makes it easy to collect, process, and analyze streaming data in real-time
- Ingests real-time data such as application logs, metrics, website clickstreams, IoT telemetry data

Types

1. **Kinesis Data Streams**: capture, process, and store data streams
2. **Kinesis Data Firehose**: load data streams into AWS data stores
3. **Kinesis Data Analytics**: analyze data streams with SQL or Apache Flink
4. **Kinesis Video Streams**: capture, process, and store video streams

![image.png](image%2025.png)

**Kinesis Data Streams – Capacity Modes**

**Provisioned mode:**

- You choose the number of shards provisioned, scale manually or using API
- Each shard gets 1MB/s in (or 1000 records per second)
- Each shard gets 2MB/s out (classic or enhanced fan-out consumer)
- You pay per shard provisioned per hour

**On-demand mode:**

- No need to provision or manage the capacity
- Default capacity provisioned (4 MB/s in or 4000 records per second)
- Scales automatically based on observed throughput peak during the last 30 days
- Pay per stream per hour & data in/out per GB

**Kinesis Data Firehose**

![image.png](image%2026.png)

- Near Real Time
- Pay for data going through Firehose

![image.png](image%2027.png)

![image.png](image%2028.png)

# Disaster Recovery & Migration

### **Disaster Recovery Strategies**

![rto-vs-rpo-what-is-the-difference-no-white-spaces.png](rto-vs-rpo-what-is-the-difference-no-white-spaces.png)

RTO - Recovery Time Objective

RPO - Recovery Point Objective

![image.png](70adb7d2-22d6-479a-a9ad-c7fcb1ba5c00.png)

- **Backup and Restore**:
    - Storage Gateway, Snowball, snapshots, AMIs
- **Pilot Light**:
    - Small application running, database active for critical core systems
- **Warm Standby**:
    - Duplicate at minimum requirements
- **Hot Site/Multi-Site**:
    - Full copy of production
- **AWS Multi-Region**:
    - Cross-region deployment
    

### Disaster recovery tips

1. **Backup**:
    - **EBS Snapshots** and **RDS automated backups**.
    - Regularly push data to **S3**, **S3 Infrequent Access (IA)**, **Glacier**, apply **Lifecycle Policies**, and enable **Cross Region Replication**.
    - For on-premise data, use **Snowball** or **Storage Gateway**.
2. **High Availability**:
    - Use **Route 53** to migrate DNS between regions.
    - Implement **RDS Multi-AZ**, **ElastiCache Multi-AZ**, **EFS**, and **S3** for redundancy.
    - Use **Site-to-Site VPN** as a recovery alternative to **Direct Connect**.
3. **Replication**:
    - Enable **RDS Cross-Region Replication** or **AWS Aurora Global Databases**.
    - Replicate databases from on-premises to RDS using **Storage Gateway**.
4. **Automation**:
    - Use **CloudFormation** or **Elastic Beanstalk** to rebuild environments.
    - **CloudWatch** can monitor EC2 instances and trigger recoveries/reboots upon failures.
    - **AWS Lambda** supports custom automation tasks.
5. **Chaos Engineering**:
    - Randomly terminate EC2 instances to test system resilience.
    - Overloading
    

### Database Migration Service (DMS)

- Quick and secure migration
- Source remains available during migration
- Supports homogeneous and heterogeneous migrations
- Continuous replication with CDC
- Requires EC2 instance for replication

### Schema Conversion Tool (SCT)

- Converts database schemas between engines
    - *Source DB (Oracle)  --->  DMS + SCT  --->  Target DB (MySQL)*
    - OLTP: SQL Server/Oracle to MySQL/PostgreSQL/Aurora
    - OLAP: Teradata/Oracle to Redshift
- **Instance Optimization**: Use compute-intensive instances to improve data conversion performance.
- **SCT Usage**:
    - SCT is not required if you’re migrating between the same database engine.
    - For example, migrating **On-Premise PostgreSQL** to **RDS PostgreSQL** doesn’t need SCT, as the database engine remains PostgreSQL (RDS is the platform).
    

### RDS/Aurora MySQL Migrations

- RDS MySQL to Aurora:
    - Database snapshots
    - Aurora Read Replica
- External MySQL to Aurora:
    - Percona XtraBackup to S3
    - mysqldump utility

### On-premises to AWS

- **Amazon Linux 2 AMI VM**:
    - Available as ISO
    - Compatible with major virtualization platforms
- **VM Import/Export**:
    - Migrate existing applications
    - Create DR strategies
    - Bi-directional migration
- **Application Discovery Service**:
    - Gathers on-premises server data
    - Server utilization metrics
    - Dependency mapping
- **Server Migration Service (SMS)**:
    - Incremental replication
    - Minimal disruption
    

### AWS Backup ****

- **Fully managed** backup service.
- **Automates & centralizes** backups across AWS services.
- **No need for custom scripts/manual processes**.
- **Supported services**: EC2, EBS, S3, RDS, Aurora, DynamoDB, DocumentDB, Neptune, EFS, FSx, Storage Gateway.
- **Cross-region & cross-account backups**.
- **Supports Point-In-Time Recovery (PITR)** for supported services.
- **On-Demand & Scheduled backups** with **Backup Plans**.
- **Backup policies**: Frequency, backup window, retention, cold storage transition.
- **AWS Backup Vault Lock**:
    - WORM enforcement (Write One Read Many)
    - Protection from deletion
    - Restrictive permissions

### **AWS Application Discovery Service**

- **Helps plan migrations** by gathering on-premises data center info.
- **Collects server utilization & dependency data** for migration planning.
- **Discovery Methods**:
    - **Agentless Discovery** (via AWS Connector) – VM inventory, config, CPU, memory, disk usage.
    - **Agent-based Discovery** – System performance, running processes, network connections.
- **Data is viewable in AWS Migration Hub**.

### **AWS Application Migration Service (MGN)**

- **Successor to CloudEndure Migration & replaces AWS SMS**.
- **Lift-and-shift (rehost) migration** of apps to AWS with minimal downtime.
- **Converts physical, virtual, and cloud servers** to run natively on AWS.
- **Supports various platforms, OS, and databases**.
- **Reduces migration costs & complexity**.

### VMware Cloud on AWS

- Manage on-premises data centers and AWS
- Migrate vSphere workloads
- Run production across private/public/hybrid cloud
- Implement DR strategy

# AWS Global Accelerator vs CloudFront Comparison

The image shows a comparison between two AWS services: AWS Global Accelerator and CloudFront.

**Similarities**

- Both services utilize the AWS global network and its edge locations around the world
- Both integrate with AWS Shield for DDoS protection

**CloudFront**

- Improves performance for cacheable content (such as images and videos)
- Handles dynamic content (such as API acceleration and dynamic site delivery)
- Content is served at the edge

**Global Accelerator**

- Improves performance for a wide range of applications over TCP or UDP
- Proxies packets at the edge to applications running in one or more AWS Regions
- Good fit for non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP
- Good for HTTP use cases that require static IP addresses
- Good for HTTP use cases that require deterministic, fast regional failover

# Data & Analytics

## **Amazon Athena**

- **Serverless** query service to analyze data stored in Amazon S3
- Uses standard SQL language to query the files (built on Presto)
- Supports CSV, JSON, ORC, Avro, and Parquet
- Pricing: $5.00 per TB of data scanned
- Commonly used with Amazon Quicksight for reporting/dashboards

**Use cases:** Business intelligence / analytics / reporting, analyze & query VPC Flow Logs, ELB Logs, **CloudTrail trails**, etc...

**Amazon Athena – Performance Improvement**

- **Use columnar data** for cost-savings (less scan)
    - Apache Parquet or ORC is recommended
    - Huge performance improvement
    - Use Glue to convert your data to Parquet or ORC
- **Compress data** for smaller retrievals (bzip2, gzip, lz4, snappy, zlip, zstd...)
- **Partition datasets in S3** for easy querying on virtual columns
    - `s3://yourBucket/pathToTable/`
        - `<PARTITION_COLUMN_NAME>=<VALUE>`
        - `<PARTITION_COLUMN_NAME>=<VALUE>`
        - `<PARTITION_COLUMN_NAME>=<VALUE>`
        - etc...
    - Example: `s3://athena-examples/flight/parquet/year=1991/month=1/day=1/`
- **Use larger files** (> 128 MB) to minimize overhead

## **Redshift**

- **OLAP system** based on PostgreSQL (not for OLTP), optimized for analytics.
- **Fast & scalable** (10x better performance, scales to PBs).
- **Columnar storage** with a parallel query engine.
- **Pay-as-you-go** pricing, SQL query support, and BI tool integration.
- **vs Athena:** Faster queries, joins, and aggregations using indexes.

Snapshots and Backups

- **Multi-AZ mode** is available for some Redshift clusters.
- **Snapshots** are incremental, point-in-time backups stored in S3.
- **Automated snapshots** occur every 8 hours, every 5 GB, or on a schedule (retention: 1–35 days).
- **Manual snapshots** persist until manually deleted.
- Snapshots can be **automatically copied** to another AWS Region.

Data Ingestion

1. Kinesis Data Firehose through s3 Copy
2. with or without VPC
3. EC2 Instance JDBC Driver (Large Batches)

> Redshift Spectrum: Multiple Compute nodes from Redshift cluster can query data from S3 without loading it in Redshift
> 

## Amazon OpenSearch / ElasticSearch

- Supports **full-text search** on any field, unlike DynamoDB.
- Used as a **complement** to other databases.
- Available in **managed** or **serverless** modes.
- **No native SQL support** (can be enabled via a plugin).
- Ingests data from **Kinesis, IoT, and CloudWatch Logs**.
- **Security** via Cognito, IAM, KMS encryption, and TLS.
- Includes **OpenSearch Dashboards** for visualization.

Search Pattern: Using Lambda > Kinesis Data Streams > Kinesis Firehose 

## **EMR (Elastic MapReduce)**

- simplifies Hadoop cluster creation for big data processing.
- Supports **hundreds of EC2 instances** with auto-scaling and Spot instance integration.
- Comes with **Apache Spark, HBase, Presto, Flink**, and more.
- Handles **provisioning and configuration** automatically.
- **Use cases:** data processing, machine learning, web indexing, big data analytics.

Node Types

- **Master Node:** Manages the cluster, coordinates tasks, and monitors health (long-running).
- **Core Node:** Runs tasks and stores data (long-running).
- **Task Node (optional):** Runs tasks only, often using **Spot Instances**.
- **Purchasing Options:**
    - **On-demand:** Reliable, won’t be terminated.
    - **Reserved:** 1+ year commitment, cost savings.
    - **Spot:** Cheaper but can be terminated anytime.

## Amazon QuickSight

- **Serverless BI service** for interactive dashboards with ML capabilities.
- **Fast, scalable, embeddable**, and offers **per-session pricing**.
- **Use cases:** business analytics, visualizations, ad-hoc analysis, and insights.
- **Integrates with** RDS, Aurora, Athena, Redshift, S3.
- Uses **SPICE engine** for in-memory computation.
- **Enterprise edition** supports **Column-Level Security (CLS)**.

**QuickSight Integrations**

- **AWS Data Sources**: RDS, Aurora, Redshift, Athena, S3, OpenSearch, Timestream.
- **SaaS Integrations**: Salesforce, Jira.
- **On-Premises Databases**: Supports JDBC connections (e.g., Teradata).
- **File Imports**: XLSX, CSV, JSON, TSV, ELF & CLF (log formats).

## AWS Glue

- Managed extract, transform, and load (ETL) service
- Useful to prepare and transform data for analytics
- Fully serverless service

- Glue Job Bookmarks: prevent re-processing old data
- Glue Elastic Views:
    - Combine and replicate data across multiple data stores using SQL
    - No custom code, Glue monitors for changes in the source data, serverless
    - Leverages a “virtual table” (materialized view)
- Glue DataBrew: clean and normalize data using pre-built transformation
- Glue Studio: new GUI to create, run and monitor ETL jobs in Glue
- Glue Streaming ETL (built on Apache Spark Structured Streaming): compatible with Kinesis Data Streaming, Kafka, MSK (managed Kafka)

## **AWS Lake Formation**

- **Centralized Data Lake**: Store structured & unstructured data for analytics.
- **Fully Managed**: Automates setup, data ingestion, and transformation.
- **ML-Powered**: Cleansing, deduplication, and cataloging.
- **Integrations**: Supports S3, RDS, relational & NoSQL databases.
- **Fine-Grained Access**: Row- and column-level security.
- **Built on AWS Glue** for data processing.

## **Kinesis Data Analytics (SQL Application)**

- **Real-time analytics** on Kinesis Data Streams & Firehose using SQL.
- **Enrich data** with reference data from S3.
- **Fully managed**, auto-scaling, and pay-per-use.
- **Outputs**: Streams (Kinesis Data Streams) or stored results (Firehose).
- **Use cases**: Time-series analytics, real-time dashboards, metrics.

**Kinesis Data Analytics for Apache Flink** 

- **Process streaming data** using Flink (Java, Scala, SQL) on AWS.
- **Fully managed**: automatic scaling, parallel computation, and backups.
- **Supports all Flink features** but **does not read from Firehose** (use Kinesis Analytics for SQL instead).

## **Amazon MSK (**Amazon Managed Streaming for Apache Kafka**)**

- **Fully managed Apache Kafka** on AWS, an alternative to Kinesis.
- **Handles Kafka brokers, Zookeeper, and failures** with automatic recovery.
- **Deploy in VPC with multi-AZ support** (up to 3 for high availability).
- **Stores data on EBS** with customizable retention.
    - **MSK Serverless**: Auto-scales compute & storage without capacity management.

**Kinesis Data Streams**

- **1 MB message size limit**
- **Uses Shards**, supports **Shard Splitting & Merging**
- **TLS in-flight & KMS at-rest encryption**
- **Default throughput: 1MB/s, configurable up to 10MB/s**

**Amazon MSK**

- 1 MB Default
- **Uses Kafka Topics with Partitions** (partitions can only be added)
- **Supports PLAINTEXT or TLS in-flight encryption**
- **KMS at-rest encryption**

# Security

### AWS KMS (Key Management Service)

- Managed encryption key storage and access control.
- Integrated with IAM for authorization.
- Logs key usage via CloudTrail.
- Supported across AWS services (EBS, S3, RDS, etc.).
- Enables API-based encryption via SDK/CLI.
- Ensures secrets are not stored in plaintext.

AWS KMS Key Types

- **KMS Keys** (formerly Customer Master Keys).
- **Symmetric Keys (AES-256)**: Single key for encryption & decryption, used by AWS services, requires KMS API.
- **Asymmetric Keys (RSA & ECC)**: Public key (encrypt) & private key (decrypt), used for encryption/signing, public key is downloadable, private key remains secure.
- **Use Case**: Asymmetric keys enable encryption outside AWS without KMS API access.

### SSM Parameter Store

- Secure storage for **configuration & secrets**.
- **Optional encryption** using **KMS**.
- **Serverless, scalable, durable** with **SDK support**.
- **Version tracking** for parameters.
- **IAM-based security** for access control.
- **Event notifications** via **Amazon EventBridge**.
- **Integration with CloudFormation** for automation.

### AWS Certificate Manager (ACM)

- **Provision, manage, and deploy** TLS certificates easily.
- Provides **in-flight encryption** for websites (**HTTPS**).
- Supports **public (free)** and **private** TLS certificates.
- **Automatic certificate renewal**.
- **Integrates with** ELB (CLB, ALB, NLB), **CloudFront**, and **API Gateway**.
- **Cannot be used directly** with **EC2** (certificates can't be extracted).

### AWS Secrets Manager

- **Securely store and manage secrets** (e.g., DB credentials, API keys).
- **Automatic secret rotation** (every X days, uses **Lambda**).
- **Integrated with RDS** (MySQL, PostgreSQL, Aurora).
- **Secrets are encrypted with KMS**.
- **Multi-Region Secrets**: Replicate secrets across AWS Regions.
- **Keeps read replicas in sync** with the primary secret.
- **Ability to promote a read replica to a standalone secret**.
- **Use cases**: Multi-region apps, disaster recovery, multi-region databases.

### AWS WAF – Web Application Firewall

- **Protects web apps from Layer 7 attacks** (SQL injection, XSS, DDoS).
- **Deploy on**: ALB, API Gateway, CloudFront, AppSync, Cognito.
- **Web ACL Rules**:
    - **IP Set**: Up to 10,000 IPs.
    - **Filter**: HTTP headers, body, URI strings.
    - **Geo-blocking**: Restrict by country.
    - **Rate-based rules**: Prevent excessive requests (DDoS protection).
- **Regional Web ACLs**, except CloudFront (global).
- **Rule groups**: Reusable sets of rules for Web ACLs.

WAF – Fixed IP with Load Balancer

- **WAF does not support NLB** (Layer 4).
- **Use Global Accelerator** for fixed IP while applying WAF on ALB.

### AWS Shield – DDoS Protection

- **DDoS**: Large-scale attack overwhelming services with traffic.
- **Shield Standard**: Free, automatic for all AWS users, protects against Layer 3/4 attacks (SYN/UDP floods, reflection attacks).
- **Shield Advanced** ($3,000/month): Protects EC2, ELB, CloudFront, Global Accelerator, Route 53; includes AWS DDoS Response Team (DRP), fee protection, and automatic WAF rules for Layer 7 attacks.

### AWS Firewall Manager

- **Centralized security rule management** across AWS Organization accounts.
- **Security policies**: WAF rules (ALB, API Gateway, CloudFront), Shield Advanced, Security Groups, Network Firewall, Route 53 DNS Firewall.
- **Policies are region-based** and apply automatically to new resources, ensuring compliance.

### **WAF vs. Firewall Manager vs. Shield**

- **AWS WAF**: Protects web apps from Layer 7 attacks with Web ACL rules (granular protection).
- **AWS Firewall Manager**: Centralized security management for AWS WAF across multiple accounts, automates protection for new resources.
- **AWS Shield Advanced**: Adds advanced DDoS protection, 24/7 support, and attack cost mitigation.
- **Best Practice**: Use all three together for comprehensive security.

![image.png](image%2029.png)

### **Amazon GuardDuty**

- **Threat Detection** using **ML, anomaly detection, and 3rd party data**.
- **One-click enablement** with a **30-day trial** (no software installation).
- **Monitors:**
    - **CloudTrail Events** (API calls, deployments).
    - **VPC Flow Logs** (unusual traffic, IPs).
    - **DNS Logs** (malicious queries).
    - **Optional Features:** EKS, RDS, Aurora, EBS, Lambda, S3.
- **Automated Response:**
    - **Integrates with EventBridge** for alerts & automation.
    - **Mitigates cryptocurrency mining attacks** (dedicated finding).

### **Amazon Inspector**

- **Automated Security Assessments** for **EC2, ECR, and Lambda**.
- **EC2:** Checks **network exposure** & OS vulnerabilities (via **SSM Agent**).
- **ECR:** Scans **container images** upon push for vulnerabilities.
- **Lambda:** Assesses **function code & dependencies** for security issues.
- **Findings Integration:** **Security Hub, EventBridge** for alerts & automation.

### **AWS Macie**

- **Fully managed data security & privacy service** using **ML & pattern matching**.
- **Identifies & alerts** on **sensitive data** (e.g., **PII, financial data**) in AWS.
- **Monitors Amazon S3** for **data discovery, classification & protection**.
- **Automates security policies** and integrates with **EventBridge & Security Hub**.

# Other Services

## **AWS CloudFormation**

- **Declarative Infrastructure as Code (IaC)** to define AWS resources.
- **Automatically provisions & configures resources** in the correct order.
- **Tracks costs by tagging resources** within a stack.
- **Supports almost all AWS resources**, with custom resources for unsupported ones.
- **Boosts productivity** by enabling infrastructure recreation, automation, and diagram generation.
- **Leverages pre-built templates** for quick deployments.

## **AWS Batch**

- **Fully managed** service for batch processing at scale.
- **Dynamically provisions EC2 & Spot Instances** based on workload needs.
- **Runs batch jobs** (start & end jobs, not continuous processes).
- **Uses Docker containers on ECS** to execute jobs.
- **Optimizes costs** by scaling compute resources automatically.
- **Handles job submission, scheduling, and execution** without manual intervention.

## **Amazon AppFlow**

- **Fully managed integration service** for secure data transfer between **SaaS applications** and **AWS services**.
- **Supports sources** like **Salesforce, SAP, Zendesk, Slack, and ServiceNow**.
- **Destinations** include **Amazon S3, Redshift, Snowflake, and Salesforce**.
- **Flexible scheduling**: runs **on demand, on a schedule, or event-driven**.
- **Data transformation** features like **filtering and validation**.
- **Transfers are encrypted** over **public internet** or **AWS PrivateLink**.
- **Reduces integration complexity** by leveraging **prebuilt API connections**.

## **AWS Trusted Advisor**

- **High-level AWS account assessment** with **no installation required**.
- Provides **recommendations** in **six categories**:
    - **Cost Optimization**
    - **Performance**
    - **Security**
    - **Fault Tolerance**
    - **Service Limits**
    - **Operational Excellence**
- **Full set of checks** available for **Business & Enterprise Support Plans**.
- **Programmatic access** via **AWS Support API** for automation.

## Well Architected Framework 6 Pillars

- Operational Excellence
- Security
- Reliability
- Performance Efficiency
- Cost Optimization
- Sustainability