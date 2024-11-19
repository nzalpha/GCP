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





