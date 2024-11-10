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
