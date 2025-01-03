# VPC Peering: 
In Org we will have multiple VPC's & we will need instances in different VPC to communicate & establish communication.

boutique -org
    boutique-dev project
        botique-dev-vpc
            dev-subnet
                vm1
                db1
        boutiquest -tst-vpc
            tst-subnet
                vm2

In the above instead of creating a db in tst vpc, vm2 can store data in DB1 which is in different vpc.

# Conditions:
Every thing should be in same gcp, you cannot have 1 vpc in gcp and another vpc in aws and have it peered.
If VPC1 and VPC2 are in same project we can peer vpc
ii)if VPC are in different project or org then also vpc peering can happen.

# Restriction:

i) CIDR range on 1 n/w should not overlap with other n/w
Also consider 
VPC1                                         vpc2                                   vpc3
 subnet 1a- 10.0.1.0/24                       subnet2-a : 10.0.2.0/24.                subnet3a: 10.0.3.0/24
 subnet 1b- 10.1.1.0/24                       subnet 2b: 10.1.2.0/24                  subnet3b: 10.1.3.0/24
                                             conflict subnet: 10.0.1.0/24.            conflict: 10.0.2.0/24

VPC1 - VPC2 peering will fail bczo conflict subnet in vpc2 is same, so we need to delete it
VPC1 -VPC3 peering will fail even though now common subnet but bcoz vpc1 is peered with vpc2 and there is common subnet so this also fails.

ii) Should be in GCP vpc only

---

#echo "Enabling Compute API"
# gcloud services enable compute.googleapis.com
#echo Delete default VPC

#gcloud compute firewall-rules delete default-allow-icmp default-allow-internal default-allow-rdp default-allow-ssh --quiet

# gcloud compute networks delete default --quiet

#echo Create 3 VPC networks, with purposely overlapping subnets


gcloud compute networks create network-1 --subnet-mode=custom 

gcloud compute networks subnets create subnet-1a --network=network-1 --region=us-central1 --range=10.0.1.0/24

gcloud compute networks subnets create subnet-1b --network=network-1 --region=us-central1 --range=10.1.1.0/24


gcloud compute networks create network-2 --subnet-mode=custom

gcloud compute networks subnets create subnet-2a --network=network-2 --region=us-central1 --range=10.0.2.0/24

gcloud compute networks subnets create subnet-2b --network=network-2 --region=us-central1 --range=10.1.2.0/24

gcloud compute networks subnets create conflict-with-network-1-subnet --network=network-2 --region=us-central1 --range=10.0.1.0/24

gcloud compute networks create network-3 --subnet-mode=custom

gcloud compute networks subnets create subnet-3a --network=network-3 --region=us-central1 --range=10.0.3.0/24

gcloud compute networks subnets create subnet-3b --network=network-3 --region=us-central1 --range=10.1.3.0/24

gcloud compute networks subnets create conflict-with-network-2-subnet --network=network-3 --region=us-central1 --range=10.0.2.0/24

echo Create firewall rules to allow port 22 , icmp access for all network resources

gcloud compute firewall-rules create ssh-allow-network-1 --direction=INGRESS --priority=1000 --network=network-1 --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules create icmp-allow-network-1 --direction=INGRESS --priority=1000 --network=network-1 --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0


gcloud compute firewall-rules create ssh-allow-network-2 --direction=INGRESS --priority=1000 --network=network-2 --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules create icmp-allow-network-2 --direction=INGRESS --priority=1000 --network=network-2 --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0

gcloud compute firewall-rules create ssh-allow-network-3 --direction=INGRESS --priority=1000 --network=network-3 --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules create icmp-allow-network-3 --direction=INGRESS --priority=1000 --network=network-3 --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0

echo Create instance in each created subnet

gcloud compute instances create instance-1 --zone=us-central1-a --machine-type=f1-micro --subnet=subnet-1a

gcloud compute instances create instance-2 --zone=us-central1-a --machine-type=f1-micro --subnet=subnet-2a

gcloud compute instances create instance-3 --zone=us-central1-a --machine-type=f1-micro --subnet=subnet-3a


echo Setup complete, proceed to establish a VPC peering


## *VPC Peering*

1) Goto VPC Network & choose VPC network peering and create connection.
2) Set up peering for nw1 to nw2 by giving project id and network name. 

The status will be inactive and will turn to active when we do peering from nw2 to nw1




---


# SHARED VPC:

In big companies there will be vpc present in one project and these vpc is shared by other projects. Any firewalls rules are also created on the project where vpc is created and restrictions on the child projects is done through network filters.

**Host Project**: If I am hosting vpc and share it to other teams.
**Service Project*: If i am application team and consuming someone else projects or vpc.

*Restrictions*: Only withing Single GCP Organization

Shared VPC =XPN (CRoss Platform Network)

Permissions Needed: 
For Host Projects:At org level, the user should have Compute Shared VPC administrator Role
For Svc Projects: User needs to have Compute Network User Role

---
#Create 3 projects dev-shared-project, prod-shared-project, host-shared-project

#To create a host-network vpc
gcloud compute networks create host-network --subnet-mode=custom
echo "********* Host-network created succesfully *******"

# To Create a dev Subnet
gcloud compute networks subnets create dev-subnet --range=10.0.2.0/24 \
--network=host-network --region=us-central1
echo "********* Dev Subnet Created Succesfully ********"

#To Create a Private subnet
gcloud compute networks subnets create private-subnet --range=10.0.3.0/24 \
--network=host-network --region=us-central1
echo "********* Private Subnet Created Succesfully ********"


#To Create a Prod-subnet
gcloud compute networks subnets create prod-subnet --range=10.0.1.0/24 \
--network=host-network --region=us-central1
echo "********* Prod Subnet Created Succesfully ********"

---

## Usecase:

In Dev-Project we need to create a vm. For Zain we gave compute instance admin and service account user.
Now if zain logins & createa VM in dev-project it fails becoz there is no network and zain does not have persmission to create VPC and hence we need shared vPC.

1) As the admin goto host-project> VPC> Shared VPC > Set up Shared VPC> Enable host project(make sure compute shared vpc admon  at org level is added to user accessing host project)
2) Attach service projects & select Principals, uncheck the owner and editor and other roles here.
3) Grant Access: Choose accessmode as the dev and prod subnets and save
4)Now in the tab "Attached Projects"> add svc projects dev and prod.
5) In Indidividual Subnet Access, you see that the subnets are not assigned to any users. Click on 0 users and add the users and give Compute Network user role.










boutique -org
    boutique-dev project
        botique-dev-vpc
            dev-subnet
                vm1
                db1
        boutiquest -tst-vpc
            tst-subnet
                vm2

In the above instead of creating a db in tst vpc, vm2 can store data in DB1 which is in different vpc.

# Conditions:
Every thing should be in same gcp, you cannot have 1 vpc in gcp and another vpc in aws and have it peered.
If VPC1 and VPC2 are in same project we can peer vpc
ii)if VPC are in different project or org then also vpc peering can happen.

# Restriction:

i) CIDR range on 1 n/w should not overlap with other n/w
Also consider 
VPC1                                         vpc2                                   vpc3
 subnet 1a- 10.0.1.0/24                       subnet2-a : 10.0.2.0/24.                subnet3a: 10.0.3.0/24
 subnet 1b- 10.1.1.0/24                       subnet 2b: 10.1.2.0/24                  subnet3b: 10.1.3.0/24
                                             conflict subnet: 10.0.1.0/24.            conflict: 10.0.2.0/24

VPC1 - VPC2 peering will fail bczo conflict subnet in vpc2 is same, so we need to delete it
VPC1 -VPC3 peering will fail even though now common subnet but bcoz vpc1 is peered with vpc2 and there is common subnet so this also fails.

ii) Should be in GCP vpc only

---

#echo "Enabling Compute API"
# gcloud services enable compute.googleapis.com
#echo Delete default VPC

#gcloud compute firewall-rules delete default-allow-icmp default-allow-internal default-allow-rdp default-allow-ssh --quiet

# gcloud compute networks delete default --quiet

#echo Create 3 VPC networks, with purposely overlapping subnets


gcloud compute networks create network-1 --subnet-mode=custom 

gcloud compute networks subnets create subnet-1a --network=network-1 --region=us-central1 --range=10.0.1.0/24

gcloud compute networks subnets create subnet-1b --network=network-1 --region=us-central1 --range=10.1.1.0/24


gcloud compute networks create network-2 --subnet-mode=custom

gcloud compute networks subnets create subnet-2a --network=network-2 --region=us-central1 --range=10.0.2.0/24

gcloud compute networks subnets create subnet-2b --network=network-2 --region=us-central1 --range=10.1.2.0/24

gcloud compute networks subnets create conflict-with-network-1-subnet --network=network-2 --region=us-central1 --range=10.0.1.0/24

gcloud compute networks create network-3 --subnet-mode=custom

gcloud compute networks subnets create subnet-3a --network=network-3 --region=us-central1 --range=10.0.3.0/24

gcloud compute networks subnets create subnet-3b --network=network-3 --region=us-central1 --range=10.1.3.0/24

gcloud compute networks subnets create conflict-with-network-2-subnet --network=network-3 --region=us-central1 --range=10.0.2.0/24

echo Create firewall rules to allow port 22 , icmp access for all network resources

gcloud compute firewall-rules create ssh-allow-network-1 --direction=INGRESS --priority=1000 --network=network-1 --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules create icmp-allow-network-1 --direction=INGRESS --priority=1000 --network=network-1 --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0


gcloud compute firewall-rules create ssh-allow-network-2 --direction=INGRESS --priority=1000 --network=network-2 --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules create icmp-allow-network-2 --direction=INGRESS --priority=1000 --network=network-2 --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0

gcloud compute firewall-rules create ssh-allow-network-3 --direction=INGRESS --priority=1000 --network=network-3 --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules create icmp-allow-network-3 --direction=INGRESS --priority=1000 --network=network-3 --action=ALLOW --rules=icmp --source-ranges=0.0.0.0/0

echo Create instance in each created subnet

gcloud compute instances create instance-1 --zone=us-central1-a --machine-type=f1-micro --subnet=subnet-1a

gcloud compute instances create instance-2 --zone=us-central1-a --machine-type=f1-micro --subnet=subnet-2a

gcloud compute instances create instance-3 --zone=us-central1-a --machine-type=f1-micro --subnet=subnet-3a


echo Setup complete, proceed to establish a VPC peering


## *VPC Peering*

1) Goto VPC Network & choose VPC network peering and create connection.
2) Set up peering for nw1 to nw2 by giving project id and network name. 

The status will be inactive and will turn to active when we do peering from nw2 to nw1




---


# SHARED VPC:

In big companies there will be vpc present in one project and these vpc is shared by other projects. Any firewalls rules are also created on the project where vpc is created and restrictions on the child projects is done through network filters.

**Host Project**: If I am hosting vpc and share it to other teams.
**Service Project*: If i am application team and consuming someone else projects or vpc.

*Restrictions*: Only withing Single GCP Organization

Shared VPC =XPN (CRoss Platform Network)

Permissions Needed: 
For Host Projects:At org level, the user should have Compute Shared VPC administrator Role
For Svc Projects: User needs to have Compute Network User Role

---
#Create 3 projects dev-shared-project, prod-shared-project, host-shared-project

#To create a host-network vpc
gcloud compute networks create host-network --subnet-mode=custom
echo "********* Host-network created succesfully *******"

# To Create a dev Subnet
gcloud compute networks subnets create dev-subnet --range=10.0.2.0/24 \
--network=host-network --region=us-central1
echo "********* Dev Subnet Created Succesfully ********"

#To Create a Private subnet
gcloud compute networks subnets create private-subnet --range=10.0.3.0/24 \
--network=host-network --region=us-central1
echo "********* Private Subnet Created Succesfully ********"


#To Create a Prod-subnet
gcloud compute networks subnets create prod-subnet --range=10.0.1.0/24 \
--network=host-network --region=us-central1
echo "********* Prod Subnet Created Succesfully ********"

---

## Usecase:

In Dev-Project we need to create a vm. For Zain we gave compute instance admin and service account user.
Now if zain logins & createa VM in dev-project it fails becoz there is no network and zain does not have persmission to create VPC and hence we need shared vPC.

1) As the admin goto host-project> VPC> Shared VPC > Set up Shared VPC> Enable host project(make sure compute shared vpc admon  at org level is added to user accessing host project![Image Alt](https://github.com/nzalpha/GCP/blob/1a44fdcd3015fde0f95cca7ff20ed4503ede434a/image.png).
2) Attach service projects & select Principals, uncheck the owner and editor and other roles here.
3) Grant Access: Choose accessmode as the dev and prod subnets and save
4)Now in the tab "Attached Projects"> add svc projects dev and prod.
5) In Indidividual Subnet Access(make sure you enable "Only show subnets that have individual IAM policies"), you see that the subnets are not assigned to any users. Click on 0 users and add the users and give Compute Network user role.

6) <img width="1395" alt="image" src="https://github.com/user-attachments/assets/80200b0f-577d-485e-8eca-bd63369f614f">

7) Now when zain logins in the dev-project and tries to create vm, under network > network interfaces, you need to select the 
Networks shared with me (from host project: "host-project-439520")

<img width="931" alt="image" src="https://github.com/user-attachments/assets/5978817f-2a7e-4407-8385-2760671d11db">












