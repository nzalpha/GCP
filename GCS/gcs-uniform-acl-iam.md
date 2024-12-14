
1. First list item
   - Create a SVC Account
   gcloud iam service-accounts create instancestorage --display-
    name="Instances to storage"
    The above creates svc account: instancestorage@host-project-439520.iam.gserviceaccount.com

   - Create Keys to this svc account (before creating key make sure you enable Disable service account key creation)
    gcloud iam service-accounts keys create key.json --iam-account=instancestorage@host-project-439520.iam.gserviceaccount.com
   This generates the key.json, goto the vm where you want to copy file to bucket; create key.json and copy the content of the genearate content.

   - Give access to buckets to this svc account
   in Console goto iam> grant access and give storage admin role to this svc account

   - Config Svc Account inside the vm
   If you goto vm and run gcloud config list you see its default svc account.
   gcloud config set account instancestorage@host-project-439520.iam.gserviceaccount.com

   - Authorize & authenticate the svc account in vm
   gcloud auth activate-service-account instancestorage@host-project-439520.iam.gserviceaccount.com --key-file key.json

   - Copy file from vm to bucket:
  gsutil cp log.txt gs://host-project-439520-bucket1
---
Versioning:

Versioning is enabled at buckey level

**Check Versioning Enabled or not**
gsutil versioning get gs://host-project-439520-bucket1/


1) Create a file index.html in the bucket
2) Now edit this index.html file and copy in the bucket
3) In the bucket you will see only 1 file index.html with the latest information at step 2

**Enable Versioning**
gsutil versioning set on gs://host-project-439520-bucket1/

After you enable Version and step 1 and 2. Now if you see the content of bucket in cli: if you do gsutil ls bucket you will see only index.html with latest data but

gsutil ls -la gs://host-project-439520-bucket1/ you will see two files
gs://host-project-439520-bucket1/index.html#123
gs://host-project-439520-bucket1/index.html#124

You can see content by: gsutil cat gs://host-project-439520-bucket1/index1.html#123

**IF you want to get the content mentioned in first time index.html as the main content and not the latest changes of index.html
gsutil cp  gs://host-project-439520-bucket1/index.html#123 gs://host-project-439520-bucket1/index.html**

**LifeCycle Rules**
Applied at bucket level
You created backup through versioning for 1gb file so you have say 10 versions which mean 10gb; hence you use lifecyle option. Here you set conditions like after 365 days perform action: delete; object after 2 weeks of no use action should should be sent to Nearline

In Console> bucket > Lifecycle: yo will see an action and conditions

**Access Control**
Unifrom: ensure uniform access to all objects in the bucket by using bucket-level ermission (IAM)

Fine Grained: Specify access to individual objects by using object-level permissions(ACL)

## Making Bucket Public

**Gcloud Command for bucket creation**
gcloud storage bucket create gs://i27example

**Describe bucket**
gcloud storage bucket describe gs://i27example

**Provide Public Read access**
gcloud storage bucket update gs://i27example --predefined-default-object-acl=publicRead

## Making Specific Object Public through ACL

**Gcloud Command for bucket creation**
gcloud storage bucket create gs://i27example-object

**Copy some files to this bucket**
gcloud cp log1.txt gs://i27example-object
gcloud cp log2.txt gs://i27example-object

**Provide Public Read access to log1.txt only**
gcloud storage object update gs://i27example-object/log1.txt --predefined-acl=publicRead

## Removing Bucket:
gsutil rm -r gs://i27example-object (recursively remove everything in object)
for objects: gsutil rm gs://i27example-object/log1.txt

## Making Specific Object Public through ACL

**Gcloud Command for bucket creation**
gcloud storage bucket create gs://i27example-iam

**Copy some files to this bucket**
gcloud cp log1.txt gs://i27example-iam
gcloud cp log2.txt gs://i27example-iam

**List the objects in bucket**
gsutil ls gs://i27example-iam

**Get IAM policy binding for Bucket**
gcloud storage buckets get-iam-policy gs://i27example-iam
Gives list of members and roles

**Add IAM policy binding to make the bucket public**
gcloud storage buckets add-iam-policy-binding gs://i27example-iam --role roles/storage.objectViewer --member 

**Create a lifecycle Rule through json file**
gcloud storage buckets update gs://host-project-439520-bucket1 --lifecycle-fi
le=life.json
