---

# **Buckets**
Link : https://cloud.google.com/storage/docs/buckets
Buckets are the basic containers that hold your data. Everything that you store in Cloud Storage must be contained in a bucket
    - The buckets will have objects(data)
    - The buckets will hold videos, images, yaml, json etc. GCS is BLOB (Binary Large Object).
    - These are immutable, we cannot edit the objects in the bucket; we will have to replace it
    - These are infinite in size, but each object in bucket will be under 5TB
    - Bucket name should be unique.
    - These cannot be used as disks in vm creation.
    - Usually used to store DB backups, application files

Structured data (key values) No then  GCS
If structured data & relational & if scalability option is needed then Spanner
If structured data & relational & if scalability option is not needed then Cloud SQL
If structured data & not relational then No SQL and svc is called Cloud Firestore

## **Storage Class**
The storage class set for an object affects the object's availability and pricing model.
Standard: For frequent retrieval of data
Nearline: For retrieval every month
ColdLine: For retrieval every quarter
Archieve: For retrieval every year

##**Data Encryption:**
i) Google Managed Encryption Key (GMEK) default
ii) Customer Managed Encryption Key (CMEK)
iii) Customer Supplied Encryption Key (CSEK)

