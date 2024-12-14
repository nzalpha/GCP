
### **Guide for Setting up  Integration between CloudRun & Cloud SQL**
We never keep any data in the containers because containers are epehemeral, hence we save all the data in Cloud SQL

*

#### **In this guide, you will have an application, build the image push to GAR. 2) Have Cloud SQL create DB and create integration to CloudRun:**

- **Pull the Code**: The java application is present in "https://github.com/devopswithcloud/cloudrun-cloudsql-integration-java-app.git"; do a git clone

- **Step 1: Create CloudSQL**: Also make sure you have chose Private IP and associated network. in Console under Connection> Networking after running below command and creating CloudSQL
```bash
 gcloud sql instances create cloudrun-db-instance \
--database-version=MYSQL_8_0 \
--cpu=1 \
--memory=4GB \
--region=us-central1 \
--authorized-networks 0.0.0.0/0 \
--root-password=Gcp@2022
```

- **Step 2: Create a database**: gcloud sql databases create compare_db --instance=cloudrun-db-instance

- **Step 3: Run the gcloud sql users create command to create the user**: 
```bash
gcloud sql users create cloud-user \
--instance=cloudrun-db-instance \
--password=Gcp@2022
```

- **Step 4: Build Container Image and push it to GCR**: 
**Create a rep : us-central1-docker.pkg.dev/host-project-439520/repo2**
```bash
gcloud artifacts repositories create repo2  --repository-format=docker --location=us-central1
```
Run the below where pom.xml file is present
----
```bash
mvn clean package com.google.cloud.tools:jib-maven-plugin:2.8.0:build \
 -Dimage=us-central1-docker.pkg.dev/host-project-439520/repo2/cloudsqlrun:v1 -DskipTests
```

**Step 5: Serverless VPC Access**:
Serverless VPC access makes it possible for you to connect directly to your VPC network from serverless environments such as Cloud Run, App Engine.

![alt text](<WhatsApp Image 2024-12-01 at 11.02.08 PM.jpeg>)

As you can see your cloud sql and other vm are in VPC where as Cloud Run is not on VPC. to make it possible for to connect cloud sql through private IP you need to have VPC Access Connector which attaches to the VPC. You connect Cloud Run to this connector which in turns connects to Cloud SQL

GOTO VPC Network> Serverless VPC access
name: vpc-connector, region us-central1
network:host-network(this should be same network as cloudsql) and subnet : custom ip range, and give some range Scale settings: min : 2 and max 3

**Step 6: Create a Cloud Run Service by passing the required Jdbc parameters needed for application(refer to the env.yaml file in the source code).**

These details should be sent through CloudRun

gcloud run deploy run-sql --image us-central1-docker.pkg.dev/host-project-439520/repo2/cloudsqlrun:v1  \
  --region us-central1 \
  --allow-unauthenticated \
  --add-cloudsql-instances host-project-439520:us-central1:cloudrun-db-instance \
  --set-env-vars INSTANCE_HOST="10.135.224.5" \
  --set-env-vars DB_PORT="3306" \
  --set-env-vars DB_NAME="compare_db" \
  --set-env-vars DB_USER="cloud-user" \
  --set-env-vars DB_PASS="Gcp@2022"

**Step 7:
  Once service is deployed in the Cloud Run UI goto> network> Connect to VPC for outbound traffic > Use Serverless VPC Access Connector and add the connector we created in Step 5

  Now when we access the url we see the application.

  Also you can conenct to db : mysql -h pub-ipadrs -u cloud-user -p
  
  show databases;
+--------------------+
| Database           |
+--------------------+
| compare_db         |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.04 sec)

mysql> use compare_db
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables
    -> ;
+----------------------+
| Tables_in_compare_db |
+----------------------+
| votes                |
+----------------------+
1 row in set (0.04 sec)

mysql> select * from votes;
+---------+---------------------+-----------+
| vote_id | time_cast           | candidate |
+---------+---------------------+-----------+
|       1 | 2024-12-02 07:46:11 | TABS      |
|       2 | 2024-12-02 07:46:14 | TABS      |
+---------+---------------------+-----------+
2 rows in set (0.03 sec)
