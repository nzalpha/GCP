---
#*Cloud SQL*
Link : 
Fully managed database service, it helps to maintain manage administer MYSQL, POSTGRES,SQL Server.
→ Its a PAAS Solution
→ Its regional scoped

*uptime.is* to tell, what is the SLA for 


The following are important concetps:
i) Primary : this is the main database. When creating an instance if you set zonal availability as Single zone.
ii) High Availability: If primary is down this comes into picture.When creating an instance if you set zonal availability as Multiple zone.
iii)Disaster Recovery: IF Primary and HA is down then DR comes into play.

##Networks:
Instance IP assignments:
1. Public IP: Assigns an external, internet accessible IP address. Requires using an authorized network or the Cloud SQL Proxy to connect to this instance.Not everyone can connect.We will have NAT IP's we will enter these NATIp in authorized network so that they can connect to the db. If you dont add these range, you wont be able to connect to mysql workbench.

In simple language the above can be understood as below
When a Cloud SQL instance has a public IP, it means the database can be accessed over the internet. However, for security reasons, not everyone can connect to it.
To control access, you need to define Authorized Networks — a list of IP addresses that are allowed to connect. In this case, you would add the NAT IPs (the public IPs of your network) to this list. If you don’t add these NAT IPs, you won’t be able to connect to the Cloud SQL instance using tools like MySQL Workbench.
Alternatively, you can use the Cloud SQL Proxy as a more secure option to avoid exposing the database to the internet.


2. Private IP: Assigns an internal, Google hosted VPC IP address.
To use private IP we need Private services access connection. So create PSA.

Private services access : PSA are per vpc network and can be used across all managed services. *These are b/w ur vpc network and network owned by Google using VPC peering, enabling ur instances & services to communicate exclusively by using internal ip adrs.*

 Once you create PSA you should see in the Network> your network> PSA section that it is enabled.

 Now you will create a vm in same network as the above private ip, open a firewall rule; install maria-db client (apt update-y &&  apt install mariadb-cleint -y) and use that vm to connect to the DB

 mysql -u siva -h (pvtipofdb) -p enter and give password

or



## Security:
Suppose we want our application to follow secure and encrypted path  meaning it should have certificate to connect to DB.

Goto Connections> Security > Manage Client Certificate and download the PEM files for CA, Certificate and Key.

Now choose "Require trusted client Certificates in Manage SSL Mode.

What the above does is: It ensures only encrypted connections are esatablisted to DB

## Read Replicas:
You use a read replica to offload work from a Cloud SQL instance. The read replica is an exact copy of the primary instance. Data and other changes on the primary instance are updated in almost real time on the read replica.

Read replicas are read-only; you cannot write to them. The read replica processes queries, read requests, and analytics traffic, thus reducing the load on the primary instance.

During Disaster recover, if HA all regions are failed along with primary and becase read replica can be in diff region we can promote read replica to primary


## Connecting to Cloud SQL

You can connect to cloud sql through public ip private IP and also through Cloud SQL Auth Proxy.

Cloud SQL Auth Proxy:
The Cloud SQL Auth Proxy is a utility for ensuring secure connections to your Cloud SQL instances. It provides IAM authorization, allowing you to control who can connect to your instance through IAM permissions, and TLS 1.3 encryption, without having to manage certificates.

We use this when application should not go outside, when it should be in same network and same infra

## How Cloud SQL Auth Proxy works:
This works similar to having a client application and cloud sql in same machine which is you can connect using localhost.

1.Create a vm which acts as client machine. In the client machine you install proxy client and the proxy client will be in running state.
2.The client machine should have mysql client installed
3.Download Cloud SQL Auth PRoxy using "curl -o cloud-sql-proxy https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.14.0/cloud-sql-proxy.linux.amd64" from doc: https://cloud.google.com/sql/docs/mysql/connect-auth-proxy
4.chmod +x cloud-sql-proxy
5. Now authenticate the cloud sql auth proxy .We need to execute the cloud proxy and provide the credential file. Use a service account to create and download the associated JSON file, and set the --credentials-file flag to the path of the file when you start the Cloud SQL Auth Proxy. The service account must have the required permissions for the Cloud SQL instance.
./cloud-sql-proxy --credentials-file PATH_TO_KEY_FILE \
INSTANCE_CONNECTION_NAME
6.now conntect to the database in client machine as  mysql -u USERNAME -p --host 127.0.0.1





