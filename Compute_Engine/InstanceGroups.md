# Instance Groups
An instance group is a collection of virtual machine (VM) instances that you can manage as a single entity.If we want to have multiple machines, we create instance templates & then create group of instances.
   - Managed instance groups (MIGs) let you operate apps on multiple identical VMs. You can make your workloads scalable and highly available by taking advantage of automated MIG services, including: autoscaling, autohealing, regional (multiple zone) deployment, and automatic updating.
        - Stateless(we dont care if a vm is deleted and new vm is created) serving workloads, such as a website frontend.Stateless batch, high-performance, or high-throughput compute workloads, such as image processing from a queue
        - Stateful applications, such as databases, legacy applications, and long-running batch computations with checkpointing

   - Unnmanaged instance groups let you load balance across a fleet of VMs that you manage yourself. Here the zone should be same as the zone of the vm.We cannot do auto-caling nor we can do self healing. It is collection of varied configuration vm.
   We dont need Instance Templates for this. If we delete the group, the vm wont get deleted.

## You can create two types of MIGs:
A zonal MIG, which deploys instances to a single zone.
A regional MIG, which deploys instances to multiple zones across the same region.

## Benefits of MIG:

## Auto Scaling
MIGs support autoscaling that dynamically adds or removes VM instances from the group in response to increases or decreases in load. In your autoscaling policy, you can set one or more signals to scale the group based on CPU utilization, load balancing capacity.
In case i have 2 vm and the traffic is more and we need to scale the vm, we add more VM's this is called Scaling Up or Scaling Out.If the number of vm's need to reduce this is called Scaling Down or Scaling In.

## Autohealing:
You can also set up an application-based health check, which periodically verifies that your application responds as expected on each of the MIG's instances. If an application is not responding on a VM, the MIG automatically recreates that VM for you.
There will be endpoints like ipofvm/monitor.html or just ipofvm/ and if the response is 200; it is healthy else not healthy
  - Creating Health Check:
     Name: is7-mig-health check, Scope: Regional
     Protocal: http, port:80 , Request path: ip/ or ip/monitor.html
     Check Interval: 6(check 5 times if we getting response), timeout: 5 sec(give gap of 5 sec)
     Healthy Thresshold: If the above is done twice & if 2 times we get response it is healthy
     Unhealthy Thresshold: Consective thrice if no response, vm is unhealthy.

---
### **Complete Setup Steps for the Unmanaged Instance Group**

---

### **1. Create Startup Scripts for Blue and Green VMs**

Save these scripts as `script-green.sh` and `script-blue.sh`.

- **`script-green.sh`** (Green VM Startup Script):
  ```bash
  #!/bin/bash
  # Install necessary packages
  sudo apt install -y telnet
  sudo apt install -y nginx

  # Enable and start Nginx
  sudo systemctl enable nginx
  sudo systemctl start nginx

  # Set permissions
  sudo chmod -R 755 /var/www/html

  # Get hostname and IP address
  HOSTNAME=$(hostname)
  IP_ADDRESS=$(hostname -I | awk '{print $1}')

  # Create the HTML file with dynamic hostname and IP address
  sudo echo "<!DOCTYPE html>
  <html lang=\"en\">
  <head>
      <meta charset=\"UTF-8\">
      <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\">
      <title>i27Academy GCP UnManaged Instance Group Demo</title>
      <style>
          body {
              font-family: Arial, sans-serif;
              background-color: #f0f0f0;
              display: flex;
              justify-content: center;
              align-items: center;
              height: 100vh;
              margin: 0;
          }
          .container {
              text-align: center;
              background-color: #ffffff;
              padding: 20px;
              border-radius: 8px;
              box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
          }
          .title {
              font-size: 24px;
              font-weight: bold;
              margin-bottom: 20px;
          }
          .message {
              font-size: 18px;
              font-weight: bold;
              color: #008000; /* Green color for Green-VM */
              margin-top: 10px;
          }
          .details {
              font-size: 16px;
              color: #555555;
              margin-top: 5px;
              font-style: italic;
          }
      </style>
  </head>
  <body>
      <div class="container">
          <div class="title">i27Academy GCP UnManaged Instance Group Example</div>
          <div class="message">Request Coming from ${HOSTNAME}</div>
          <div class="details">Server IP Address: ${IP_ADDRESS}</div>
      </div>
  </body>
  </html>
  " | sudo tee /var/www/html/index.html > /dev/null
  ```

- **`script-blue.sh`** (Blue VM Startup Script):
  ```bash
  #!/bin/bash
  # Install necessary packages
  sudo apt install -y telnet
  sudo apt install -y nginx

  # Enable and start Nginx
  sudo systemctl enable nginx
  sudo systemctl start nginx

  # Set permissions
  sudo chmod -R 755 /var/www/html

  # Get hostname and IP address
  HOSTNAME=$(hostname)
  IP_ADDRESS=$(hostname -I | awk '{print $1}')

  # Create the HTML file with dynamic hostname and IP address, with blue color for Blue-VM
  sudo echo "<!DOCTYPE html>
  <html lang=\"en\">
  <head>
      <meta charset=\"UTF-8\">
      <meta name=\"viewport\" content=\"width=device-width, initial-scale=1.0\">
      <title>i27Academy GCP UnManaged Instance Group Demo</title>
      <style>
          body {
              font-family: Arial, sans-serif;
              background-color: #f0f0f0;
              display: flex;
              justify-content: center;
              align-items: center;
              height: 100vh;
              margin: 0;
          }
          .container {
              text-align: center;
              background-color: #ffffff;
              padding: 20px;
              border-radius: 8px;
              box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.2);
          }
          .title {
              font-size: 24px;
              font-weight: bold;
              margin-bottom: 20px;
          }
          .message {
              font-size: 18px;
              font-weight: bold;
              color: #0000ff; /* Blue color for Blue-VM */
              margin-top: 10px;
          }
          .details {
              font-size: 16px;
              color: #555555;
              margin-top: 5px;
              font-style: italic;
          }
      </style>
  </head>
  <body>
      <div class="container">
          <div class="title">i27Academy GCP UnManaged Instance Group Example</div>
          <div class="message">Request Coming from ${HOSTNAME}</div>
          <div class="details">Server IP Address: ${IP_ADDRESS}</div>
      </div>
  </body>
  </html>
  " | sudo tee /var/www/html/index.html > /dev/null
  ```

---

### **2. Create Blue and Green VMs Using the Startup Scripts**

- **Green VM**:
  ```bash
  gcloud compute instances create green-vm \
      --zone=us-central1-a \
      --machine-type=e2-medium \
      --metadata-from-file=startup-script=script-green.sh
  ```

- **Blue VM**:
  ```bash
  gcloud compute instances create blue-vm \
      --zone=us-central1-a \
      --machine-type=e2-medium \
      --metadata-from-file=startup-script=script-blue.sh
  ```

---

### **3. Create Firewall Rule to Allow HTTP Traffic on Port 80**

- **Firewall Rule for HTTP Access**:
  ```bash
  gcloud compute firewall-rules create allow-http-unmanaged \
      --direction=INGRESS \
      --priority=1000 \
      --network=default \
      --action=ALLOW \
      --rules=tcp:80 \
      --source-ranges=0.0.0.0/0 \
      --description="Allow incoming HTTP traffic on port 80 from all sources for unmanaged instance group"
  ```

---

### **4. Create Unmanaged Instance Group with Blue and Green VMs**

1. **Create the Unmanaged Instance Group**:
   ```bash
   gcloud compute instance-groups unmanaged create i27-unmig \
       --project=criticalproject \
       --description="i27 Un Managed Group Example" \
       --zone=us-central1-a
   ```

2. **Set the Named Port for the Group**:
   ```bash
   gcloud compute instance-groups unmanaged set-named-ports i27-unmig \
       --project=criticalproject \
       --zone=us-central1-a \
       --named-ports=i27-web-port:80
   ```

3. **Add Instances to the Unmanaged Group**:
   ```bash
   gcloud compute instance-groups unmanaged add-instances i27-unmig \
       --project=criticalproject \
       --zone=us-central1-a \
       --instances=blue-vm,green-vm
   ```

---

### **Note on Health Checks**

For unmanaged instance groups, **health checks** aren’t automatically created but can be configured separately to monitor the health and availability of instances within the group. This helps in determining if an instance is healthy and capable of serving traffic.

---

### **Commands to Delete Resources Created**

1. **Delete the Unmanaged Instance Group**:
   ```bash
   gcloud compute instance-groups unmanaged delete i27-unmig \
       --project=criticalproject \
       --zone=us-central1-a \
       --quiet
   ```

2. **Remove Instances from the Unmanaged Instance Group**:
   ```bash
   gcloud compute instance-groups unmanaged remove-instances i27-unmig \
       --project=criticalproject \
       --zone=us-central1-a \
       --instances=blue-vm,green-vm \
       --quiet
   ```

3. **Delete the Blue and Green VMs**:
   - **Green VM**:
     ```bash
     gcloud compute instances delete green-vm \
         --zone=us-central1-a \
         --quiet
     ```
   - **Blue VM**:
    ```bash
    gcloud compute instances delete blue-vm \
        --zone=us-central1-a \
        --quiet
    ```

4. **Delete the Firewall Rule**:
   ```bash
   gcloud compute firewall-rules delete allow-http-unmanaged \
       --project=criticalproject \
       --quiet
   ```

---

### **Summary**

- **Instance Group Name**: `i27-unmig`
- **Description**: `i27 Un Managed Group Example`
- **Zone**: `us-central1-a`
- **Network**: `default`
- **Named Port**: `i27-web-port` mapped to port `80`
- **VM Instances**: `green-vm`, `blue-vm`

