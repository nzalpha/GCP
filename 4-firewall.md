# VPC

In On-Prem, routers, switches, Firewall and load balancers and other network appliances are physically present in Data Centers. 

In Cloud, the above are provided through Software Defined Network

VPC is Global or Regional Scoped
Subnet is Regional Scope
Vm is Zonal Scope.

*When creating VPC you need to create a Subnet. There are two different types of Subnets.*
i) Automode: Google will create subnet in each region. There are 40 regions and so there will 40 subnets created.If you want to create network quickly you can use this.
Con: Every Company will get the same CIDR range for every subnet that is create.
Flipkart                        Myntra
VPC (automode)                  VPC (automode)
  Subnet1 Region1(10.1.0.0/20).  Subnet1 Region1(10.1.0.0/20)

When Flipkart creates a vm in subnet1, it might get assigned ip as 10.1.1.4
Myntra creates a vm in subnet 1, it might get assigned same ip as above.
When both these company got acquired and want the vm to communicate the vm will not communicate as the private ip adrs are same.

```bash
gcloud compute networks create my-auto-vpc \
    --subnet-mode=auto
```

ii) Custom Mode: You will create subnet in a particular regions.
```bash
gcloud compute networks create my-custom-vpc \
    --subnet-mode=custom
```


### 3. Create a Subnet in Custom Mode VPC

Specify the region and IP range for each subnet.

```bash
gcloud compute networks subnets create my-custom-subnet \
    --network=my-custom-vpc \
    --region=us-central1 \
    --range=10.0.0.0/24
```

## 4. Create Virtual Machines (VMs) in a Specific VPC and Subnet

Specify the VPC network and subnet when creating the VM instance. You can also set a specific zone for the VM.

```bash
gcloud compute instances create my-vm-auto \
    --zone=us-central1-a \
    --network=my-auto-vpc

gcloud compute instances create my-vm-custom \
    --zone=us-central1-a \
    --network=my-custom-vpc \
    --subnet=my-custom-subnet
```


# firewall
Firewall helps control network traffic to and from instances (virtual machines, containers, etc.) based on specified rules.
skip Default N/W Creation: Goto Org >IAM > Organization Policies(user having Org Policy admin) default network will not be created.

*Ingress: Uber Eats coming to my gated community, I need to tell Security Uber is coming to my house, Security allows him inside. This is Ingress*
*Egress: I dont need to tell security when uber eats going out. During Covid when we need to go out we need to tell Security that I am Doctor and I am going out*
*If I give Ingress Rule, I dont need to tell Egress rule.

# Network Tags
Firewall implemented at VPC level, if we need to implement for a particular Instance we use network-tags

# Usecase
I need to create 1 VPC and 2 Subnets: Subnet-a (10.2.1.0/24) & Subnet-b (10.2.2.0/24).
Create 3 VM Instance-1a,Instance-1b & Instance-1c in Subnet-a; and now create Instance-2 and Instance-3 in Subnet-b

Requirement: Instance-1a and Intance-1b can ping instance 2 but Intance-1c cannot ping instance 2.
Instance-1a,1b,1c cannot ping Instance 3

Script for Infra:
---
*Create a custom-vpc*
gcloud compute networks create custom-network --subnet-mode=custom
echo "Network Created Succefsully........"

# Create Subnet-a in custom-vpc
# N/w name, cidr range, region
gcloud compute networks subnets create subnet-a --network custom-network --region us-central1 --range 10.2.1.0/24
echo "Subnet-a Created Succesfully......."

# Create subnet-b in custom-vpc
gcloud compute networks subnets create subnet-b --network custom-network --region us-central1 --range 10.2.2.0/24
echo "Subnet-b Created Succesfully......."

# Create instance-1a in subnet-a
gcloud compute instances create instance-1a --zone us-central1-a --subnet=subnet-a --machine-type=e2-small

# Create instance-1b in subnet-a
gcloud compute instances create instance-1b --zone us-central1-a --subnet=subnet-a --machine-type=e2-small --no-address

# Create instance-1c in subnet-a
gcloud compute instances create instance-1c --zone us-central1-a --subnet=subnet-a --machine-type=e2-small --tags=deny-ping

# Create instance-2 in subnet-b
gcloud compute instances create instance-2 --zone us-central1-a --subnet=subnet-b --machine-type=e2-small --tags=allow-ping

# Create instance-3 in subnet-b
gcloud compute instances create instance-3 --zone us-central1-a --subnet=subnet-b --machine-type=e2-small --no-address

---
Solution:

1) echo "Creating a firewall for SSH"
gcloud compute firewall-rules create allow-ssh-custom-nw --direction=INGRESS --priority=1000 --network=custom-network --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0

2)echo "Creating a firewall to ping using internal subnets"
This will make sure that VM in Subnet-a can ping Instance-2. 
gcloud compute  firewall-rules create allow-icmp-custom-internal --direction=INGRESS --priority=1000 --network=custom-network --action=ALLOW --rules=icmp --source-ranges=10.2.1.0/24

3) echo "Instance 2 cannot communicate to instance-1a so add subnet-b cidr range as well for icmp"
gcloud compute  firewall-rules create allow-icmp-custom-internal --direction=INGRESS --priority=1000 --network=custom-network --action=ALLOW --rules=icmp --source-ranges=10.2.2.0/24

3) Here instance 1-c is communicating to instance2 and we need to prevent this. So add n.w tag for Instance 1c: deny ping and instance 2: allow-ping
 echo "Create firewall to deny instance1c to ping instance 2"
gcloud compute  firewall-rules create deny-instance-1c --direction=INGRESS --priority=1000 --network=custom-network --action=DENY --rules=icmp --source-tags=deny-ping --target-tags=allow-ping


#test
gcloud compute  firewall-rules create deny-instance-1c --direction=INGRESS --priority=1000 --network=custom-network --action=DENY --rules=icmp --source-tags=deny-ping --target-tags=allow-ping

----

# Egress Destination Filter & Priority
* Our End Goal: VM1 ping google should work, VM2 when ping google should not work.


---

### Step 1: Create Two VMs

We will create two VMs in the same subnet. One VM (`vm-allow-google`) will be allowed to ping Google's DNS server (`8.8.8.8`), while the other VM (`vm-deny-google`) will be blocked.

```bash
# Create VM1 (allowed to ping Google)
gcloud compute instances create vm-allow-google \
    --zone us-central1-a \
    --subnet=subnet-b \
    --machine-type=e2-medium \
    --tags=allow-ping-google

# Create VM2 (denied from pinging Google)
gcloud compute instances create vm-deny-google \
    --zone us-central1-a \
    --subnet=subnet-b \
    --machine-type=e2-medium
```

#### Explanation:
- **vm-allow-google** has the tag `allow-ping-google` and will be allowed to ping Google.
- **vm-deny-google** will not be tagged and will be blocked from pinging Google.
- ** In general Eggress everything is enabled for all by default. If we login to VM1 and VM2 and ping 8.8.8.8 the ping works.We need to block everything and allow whatever we want**
---

### Step 2: Set Up Egress Firewall Rules with Destination Filters

We will now create two egress rules:
1. **Allow ICMP traffic to Google (8.8.8.8) for `vm-allow-google`.**
2. **Block all ICMP traffic to Google (8.8.8.8) for all other VMs** using priority.

---

#### Rule 1: Allow Egress ICMP to Google for `vm-allow-google`

```bash
gcloud compute firewall-rules create allow-ping-google \
    --direction=EGRESS \
    --priority=900 \
    --network=custom-network \
    --action=ALLOW \
    --rules=icmp \
    --destination-ranges=8.8.8.8/32 \
    --target-tags=allow-ping-google
```

#### Explanation:
- **Priority**: `900` (lower priority number means higher precedence).
- **Action**: Allows ICMP traffic to the destination range `8.8.8.8/32` (Google's DNS).
- **Target Tags**: Applies only to VMs with the tag `allow-ping-google`, i.e., `vm-allow-google`.

---

#### Rule 2: Deny Egress ICMP to Google for All Other VMs

```bash
gcloud compute firewall-rules create deny-ping-google \
    --direction=EGRESS \
    --priority=1000 \
    --network=custom-network \
    --action=DENY \
    --rules=icmp \
    --destination-ranges=8.8.8.8/32
```

#### Explanation:
- **Priority**: `1000` (a higher number than `900`, so it only applies if the allow rule doesn’t match).
- **Action**: Denies ICMP traffic to the destination range `8.8.8.8/32`.
- This applies to all VMs that do **not** have the `allow-ping-google` tag.

---

### Step 3: Test Egress Traffic

1. **SSH into `vm-allow-google`**:

   ```bash
   gcloud compute ssh --zone us-central1-a vm-allow-google
   ```

   Once inside the VM, try to ping Google's DNS server (`8.8.8.8`):

   ```bash
   ping 8.8.8.8
   ```

   **Expected Result**: The ping should be successful, as the egress rule with a priority of `900` allows this VM to send ICMP traffic to `8.8.8.8`.

2. **SSH into `vm-deny-google`**:

   ```bash
   gcloud compute ssh --zone us-central1-a vm-deny-google
   ```

   Try to ping Google's DNS server (`8.8.8.8`):

   ```bash
   ping 8.8.8.8
   ```

   **Expected Result**: The ping should fail because the deny rule with a priority of `1000` blocks ICMP traffic to `8.8.8.8` for all VMs without the `allow-ping-google` tag.

---

### Step 4: Clean Up Resources

After testing, clean up the VMs and firewall rules:

```bash
# Delete the VMs
gcloud compute instances delete vm-allow-google --zone us-central1-a --quiet

gcloud compute instances delete vm-deny-google --zone us-central1-a --quiet




# Delete the firewall rules
gcloud compute firewall-rules delete allow-ping-google deny-ping-google --quiet

# Delete subnet-a
gcloud compute networks subnets delete subnet-a \
    --region=us-central1 --quiet

# Delete subnet-b
gcloud compute networks subnets delete subnet-b \
    --region=us-central1 --quiet

# Delete the custom VPC
gcloud compute networks delete custom-network --quiet

```

---

## Key Concepts Highlighted in This Example

- **Destination Filters**: The `destination-ranges` field limits the egress traffic to Google's DNS server IP `8.8.8.8/32`.
- **Network Tags**: The firewall rules use network tags (`allow-ping-google`) to selectively allow traffic from specific VMs.
- **Priority**: The priority of the firewall rules determines which rule is applied first. In this case, the allow rule (priority `900`) takes precedence over the deny rule (priority `1000`) for the tagged VM.

---

## Summary:
- **VM1 (`vm-allow-google`)** is allowed to send ICMP (ping) traffic to Google's DNS server (`8.8.8.8`) based on a firewall rule with a destination filter and higher priority.
- **VM2 (`vm-deny-google`)** is denied from sending ICMP traffic to Google's DNS server based on a lower-priority deny rule.
- The use of **egress rules**, **destination filters**, and **priority** ensures that traffic is allowed or blocked as needed.

