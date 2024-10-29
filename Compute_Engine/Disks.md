---

# ** Disks **
Each Compute Engine VM has a single boot disk that contains the operating system. If we need to store any other application related data like logs we will be using Data disk.

The following are the storage options:
i) Block Storage || ii) Blob/Object Storage. || iii) File Storage

Block Storage consists of Persistent Disck and Local SSD.

### **Persistent Disk** : These are not directly attached to the VM, these are attached to NEtwork
Persistent Disk volumes are located independently from your VM, so you can detach or move Persistent Disk volumes to keep your data even after you delete your VMs

