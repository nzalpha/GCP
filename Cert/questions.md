1. **Google Deployment Manager** is a tool designed for infrastructure-as-code (IaC) on Google Cloud. It allows you to create and manage Google Cloud resources like Compute Engine VMs in a declarative way using configuration files written in YAML, Python, or Jinja2 templates.

2.BigTable:NoSQL Database for low-latency, high-throughput operations.Operational, real-time data (like time-series, IoT, telemetry)

BigQuery:Data Warehouse for large-scale data analytics.Analytical, batch processing data (like BI, dashboards)

3.Bigquery, cost is based on number of bytes read.

4. Web (HTTP/HTTPS): 80, 443, 8080, 8443
SSH / Admin: 22 (SSH), 3389 (RDP), 5900 (VNC)
Database: 3306 (MySQL), 5432 (PostgreSQL), 1433 (SQL Server)
Email: 25 (SMTP, blocked), 465 (SMTP SSL), 587 (SMTP TLS)

5.1️⃣ Cloud Pub/Sub:

Acts as the message ingestion system for streaming time-series data.
It can handle high-throughput, real-time streams of data from IoT devices, logs, or other event sources.
It decouples producers (data sources) and consumers (processing systems).

2️⃣ Cloud Dataflow:
Processes and transforms the data in real time.
It enables the application of filters, aggregations, and data enrichment in a streaming or batch processing model.
It seamlessly integrates with Pub/Sub for real-time streaming pipelines.

3️⃣ Cloud Bigtable:
Serves as a low-latency NoSQL database for storing and querying large amounts of time-series data.
It is optimized for time-series data, as it allows for fast lookups and range scans.
It is ideal for use cases like IoT data, stock prices, and telemetry.

4️⃣ BigQuery:
Used for data analysis and reporting.
Data from Cloud Bigtable (historical data) can be periodically exported to BigQuery for large-scale analytical queries.
It supports SQL queries, making it easy to integrate with BI tools like Looker, Data Studio, etc.

Cloud Storage is used for batch data (like files) but not for real-time lookup of time-series data. 

6.*Activity log:*
This is the primary location to view detailed information about user actions on your Google Cloud resources, including Cloud Storage buckets, making it the most efficient way to check for specific user activity and metadata changes.

Stackdriver (now known as Google Cloud Operations Suite) is a set of monitoring, logging, and diagnostics tools for applications running on Google Cloud Platform.
While Stackdriver logs can contain information about Cloud Storage activities, accessing data access logs specifically would require navigating to the correct log type, making it less efficient than directly filtering the Activity log.

7. A Project Billing Manager in GCP is a role responsible for managing billing accounts and project billing associations. This role allows users to link GCP projects to billing accounts and manage how the project is billed. It is crucial for organizations to control and monitor cloud costs.

8.App Engine is a fully managed Platform as a Service (PaaS) offering from Google Cloud Platform (GCP) that allows developers to build and deploy web applications and services without managing the underlying infrastructure.
Once an App Engine application is created in a specific region (e.g., us-central), it cannot be moved to another region. The region for an App Engine application is set permanently when the application is created and cannot be changed later.
Every project can have only 1 App Engine application

9.The jobUser role allows users to run queries in BigQuery. This role is specifically designed for those who need to create and execute jobs (queries) but not modify the dataset or access the raw data directly.
The dataViewer role only allows users to view dataset contents, which does not meet the requirement for users to be able to run queries.

10.Performing a rolling-action start-update with maxSurge set to 1 and maxUnavailable set to 0 refers to how updates are managed for a group of instances (like GKE pods or Compute Engine instances in a Managed Instance Group). This process ensures zero downtime while updating your instances.

Here's a breakdown of what each parameter means and how the process works.
Key Concepts
*Rolling Update:*
Instead of taking down all instances at once, a rolling update replaces instances one at a time (or in small batches) to ensure availability.
Rolling updates are used to apply changes like new application versions, patches, or configuration updates.

*maxSurge:*
Definition: The maximum number of additional (new) instances that can be created beyond the original number of instances during an update.
In This Case: maxSurge = 1, so only one extra instance can be added at a time. If you have 5 running instances, during the update, you can have up to 6 running instances (5 existing + 1 new) while updating.

*maxUnavailable:*
Definition: The maximum number of instances that can be unavailable at a time while the update is in progress.
In This Case: maxUnavailable = 0, which means all existing instances must remain available during the update. None of the old instances will be terminated until the new instance is verified to be healthy.

11. Connecting application in data center to a cloud storage in GCP.
**Create a Cloud VPN or Interconnect:**
Use Cloud VPN (if you need a simple IPsec-based connection) or Cloud Interconnect (for higher bandwidth and lower latency) to establish a private tunnel between your on-premises environment and a VPC in GCP.
This ensures secure, private connectivity between your data center and Google Cloud.

**Advertise Custom Route for Google APIs (199.36.153.4/30) via Cloud Router:**
Cloud Router is used to manage route advertisements dynamically.
You configure Cloud Router to advertise the route for 199.36.153.4/30, which is the IP range used by Google's Private Google Access for on-premises.
This IP range allows private access to Google APIs and services like Cloud Storage from within your on-premises environment via VPN or Interconnect.

**DNS Configuration to Use restricted.googleapis.com:**
In your on-premises network, configure your DNS server to resolve all *.googleapis.com to restricted.googleapis.com.
This ensures that instead of trying to resolve to a public IP, your on-premises servers resolve Google services (like storage.googleapis.com) to a private IP in the range 199.36.153.4/30.
Your on-premises servers will route requests for Google Cloud APIs (like Cloud Storage) over the private VPN or Interconnect tunnel instead of using the public internet.

12.The monitoring.viewer role in Google Cloud is a predefined Identity and Access Management (IAM) role that grants read-only access to Cloud Monitoring (formerly Stackdriver Monitoring).

13.The Storage Object Viewer IAM role (roles/storage.objectViewer) is a predefined role in Google Cloud Platform (GCP) that grants read-only access to objects (like files) in Cloud Storage buckets.
This role is commonly used when you want to allow users, applications, or services to view and download objects but not modify, upload, or delete them.

14.When you create a Windows VM on Google Compute Engine, you need to ensure you can log in via Remote Desktop Protocol (RDP). Google doesn't automatically assign Windows login credentials, so you must create a user account and password.
The gcloud compute reset-windows-password command is the recommended approach for securely generating a password for a user account on a Windows VM.

15.The service account key file is used for API access

16.The project Viewer role is a built-in IAM role that grants read-only access to all resources in a project. This includes the ability to view configurations and settings for various resources like Compute Engine, Cloud Storage, BigQuery, and more, but it does not allow modifications.

17.In Google Cloud, subnet IP ranges are defined at the time of subnet creation, and IP ranges cannot be changed directly after a subnet is created. However, you can expand the subnet range using gcloud or the Google Cloud Console.

18. If you need to search for table containing ssn column in data warehousing you can search in Data Catalog.

19.gVisor is a user-space kernel that provides stronger isolation for container workloads. By using gvisor, each customer's Pod will be isolated in a way that ensures security and prevents potential compromises from spreading to other Pods running in the same cluster. This provides an additional layer of protection by reducing the interaction between containers on the same node.

20.A Managed Instance Group (MIG) automatically creates, deletes, and maintains instances according to a specified instance template. If instance creation fails, it could be due to several issues related to invalid syntax in the template, disk name conflicts, or persistent disk reuse.

Key Issues and Their Solutions
**Instance Template Syntax**
If the syntax of the instance template is incorrect, instance creation will fail.
Solution: Verify that the instance template is properly configured with no syntax errors.

**Disk Name Conflict**"
If a persistent disk exists with the same name as an instance in the MIG, the system might not be able to create new instances due to the name collision.
Solution: Delete any persistent disks with the same name as instance names to avoid this conflict.

**Persistent Disk Reuse**:
When an instance in a managed instance group is deleted, its associated disk may not be deleted if disks.autoDelete is set to false.
If the MIG tries to recreate the instance, it might fail because the disk with the same name still exists.
Solution: Set the disks.autoDelete property to true in the instance template. This ensures that disks are deleted automatically when instances are deleted, preventing naming conflicts.

21.Google Cloud Spanner is a fully managed, distributed, and horizontally scalable relational database service offered by Google Cloud Platform (GCP). It combines the benefits of SQL-like relational databases (like MySQL, PostgreSQL) with the horizontal scalability and high availability of NoSQL databases.

Key Features of Cloud Spanner
Global Distribution: Data can be replicated across multiple regions while maintaining strong consistency.
Horizontal Scalability: Spanner can scale automatically to handle large amounts of traffic and storage.
Relational Database with SQL Support: It supports ANSI SQL and features like joins, indexes, and foreign keys.
High Availability and Resiliency: It offers 99.999% availability (five 9s) with automatic failover and replication.
Strong Consistency: Unlike most distributed databases, Cloud Spanner guarantees strong consistency across multiple regions.
Automatic Sharding and Replication: Data is partitioned and stored across multiple servers, and replicas are automatically maintained.
ACID Transactions: Cloud Spanner supports multi-row, multi-region ACID transactions, ensuring data integrity.
Serverless, Fully Managed: No need to manage servers, replication, backups, or maintenance.

22.Subnet expansion is a non-disruptive operation, and it does not affect existing resources using the subnet.

23. BigQuery Job User only allows your app to run queries, but it doesn't give access to read data from tables.

 BigQuery Data Viewer role (roles/bigquery.dataViewer).
This role allows your application to read tables and data in BigQuery but not create or run jobs (which the "Job User" role would allow).

24. To create a copy of a VM for scaling purposes, you need to follow these steps:

Create a Snapshot:
A snapshot is a point-in-time backup of a disk. It captures the current state of the disk but doesn't directly allow you to create new VMs.

Create a Custom Image from the Snapshot:
Once you have a snapshot, you can use it to create a custom image. This custom image contains all the configuration and software from the original VM and can be 
used to create new VM instances.

Create Instances from the Custom Image:
After creating the custom image, you can use it to launch new instances of the VM. These instances will be copies of the original VM and can handle increased traffic.

25.Set up application performance monitoring on Google Cloud projects A, B, and C as a single pane of glass:

In Google Cloud, Cloud Monitoring allows you to monitor metrics across multiple projects. By creating a workspace in one project (in this case, Project A), you can centralize monitoring for all the projects that need to be monitored. Here’s how it works:

Enable API: You need to enable the Cloud Monitoring API in each of the projects that you want to monitor (A, B, and C).
Create a Workspace: A workspace in Cloud Monitoring serves as a container for your monitoring data (metrics, dashboards, etc.). You can create this workspace in one project (like Project A).
Add Projects to the Workspace: You can then add other projects (B and C) to this workspace, allowing you to monitor resources (like CPU, memory, and disk) across multiple projects from a single pane of glass.

26.Cloud Source Repositories: Storing the Deployment Manager templates in Cloud Source Repositories allows for version control and collaboration with the team. Cloud Source Repositories is a fully managed Git repository that helps teams to store, version, and manage code and configuration files in a collaborative environment. But you cannot store the templates

27.When you want to automatically trigger code in response to an event, like a file upload to a Cloud Storage bucket, Cloud Functions is the best choice. Google Cloud Functions integrates directly with Cloud Storage and can be triggered by "finalize" (file upload), "delete" (file deletion), and other Cloud Storage events.

28.A. Use App Engine and configure Cloud Scheduler to trigger the application using Pub/Sub.
Why it's wrong: This is unnecessary complexity. Cloud Scheduler triggers tasks at specific intervals (not event-driven), and using Pub/Sub is an additional layer that isn't required here. You want to trigger the function immediately after a file is uploaded, not on a schedule.

29.You are storing sensitive information in a Cloud Storage bucket. For legal reasons, you need to be able to record all requests that read any of the stored data. You want to make sure you comply with these requirements.
**Enable Data Access audit logs for the Cloud Storage API.**

30. Command to create the Cloud function inside GCloud
gcloud functions deploy

31.To extract the IP address of a Kubernetes Service, you can use kubectl get svc with the -o jsonpath option. The jsonpath format allows you to specify a query path to extract specific information from the Kubernetes resource's JSON output.

Command Breakdown:
kubectl get svc -o jsonpath='{.items[*].status.loadBalancer.ingress[0].ip}'

kubectl get svc: Lists all services in the current namespace.

-o jsonpath: Outputs the result as specified by the JSON path query.

{.items[*].status.loadBalancer.ingress[0].ip}:
items[*]: Selects all services in the list.
status.loadBalancer.ingress[0].ip: Extracts the IP address from the LoadBalancer service's status.

32.. To Deploy App engine application: gcloud app deploy app.yaml

33.To create a Kubernetes cluster on Google Kubernetes Engine (GKE), you use the gcloud container clusters create command.

34.You’re trying to provide temporary access to some files in a Cloud Storage bucket. You want to Limit the time that the files are available to 10 minutes. With the fewest steps possible, what is The best way to generate a signed URL?
**Create a service account and JSON key. Use the gsutil signurl -d 10m command and pass in The JSON key and bucket.**

35.You have an App Engine application serving as your front-end. It’s going to publish messages to Pub/Sub. The Pub/Sub API hasn’t been enabled yet. What is the fastest way to enable the API? (two)
**Enable the API in the Console.**
**The API wil be enabled the first time the code attempts to access Pub/Sub.**

36.Use a Deployment Manager script to automate creating storage buckets in an appropriate region

37. You have a project for your App Engine application that serves a development environment. The required testing has succeeded and you want to create a new project to serve as your production environment. What should you do?
A. Use gcloud to create the new project, and then deploy your application to the new project.
B. Use gcloud to create the new project and to copy the deployed application to the new project.
**C. Create a Deployment Manager configuration file that copies the current App Engine deployment into a new project.**
D. Deploy your application again using gcloud and specify the project parameter with the new project name to create the new project.

38.The Project Viewer role in Google Cloud Platform (GCP) is a built-in IAM (Identity and Access Management) role that grants read-only access to most resources within a GCP project.

39.You have an application that receives SSL-encrypted TCP traffic on port 443. Clients for this application are located all over the world. You want to minimize latency for the clients. Which load balancing option should you use?
A. HTTPS Load Balancer
B. Network Load Balancer
C. SSL Proxy Load Balancer
D. Internal TCP/UDP Load Balancer. Add a firewall rule allowing ingress traffic from 0.0.0.0/0 on the target instances.

Ans: Is C
C. SSL Proxy Load Balancer

Here's why:
SSL Proxy Load Balancer is specifically designed for SSL-encrypted traffic (like HTTPS or SSL/TCP) and provides SSL termination at the load balancer. This means the SSL traffic is decrypted at the load balancer, and the traffic sent to backend instances is unencrypted (which reduces the load on your backend servers).

Minimizing Latency: The SSL Proxy Load Balancer operates at the network edge (Google's global points of presence), meaning that it can direct traffic to the closest available backend, reducing latency for clients around the world.