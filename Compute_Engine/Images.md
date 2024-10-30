# Images
To create boot disks for your virtual machine (VM) instances, you need to obtain and use operating system (OS) images.. There are 2 types of OS Image Types:

i) Public OS images are provided and maintained by Google, open source communities. The Cloud providers offer pre-built OS images, including popular distributions like Ubuntu, CentOS, Windows Server, and others

ii) Custom OS images are available only to your Google Cloud project.A custom OS image is a boot disk image that you own and control access to.
We use this to create an image from the boot disk of existing Compute Engine VM instances with new apps and softwares, then use that image to create a new boot disk for your vm. this process lets us to create new vm that are preconfigured with the apps that we dont need to reinstall from public OS image.

# Image Families:
Image families help you manage images in your project by grouping related images together, so that you can roll forward and roll back between specific image versions. An image family always points to the latest version of an OS image that is not deprecated.

Deprecated: Still works but gives warning
Obsolete: new users can't use it, error if attempt to use, existing links still work.
Deleted: all users cannot use it

Example: 
We create an custom image img1 on ubutu with java 11, webserver and python 3.6 and we have this image part of family called WebFamily.

After 1 year, org says we need java 17, webserver and python 3.7 and we create the image img2 part of family called WebFamily.

The img1 becomes obsolute.
---

### **Overview**

In this guide, we’ll walk through the following tasks:

1. **Creating a VM with a Startup Script**: Set up a virtual machine (VM) in Google Cloud, we install jdk,apache2 configure it to serve a custom web page, and set a startup script.
2. **Creating an Image out of the above VM**: Create image based on the boot disk of the above VM
3. **Creating VM out of the Image Created in Step 2**: Create a VM in different Region based on the image that is created.

## Step 1: Create a VM Instance with HTTP Access

Use the following command to create a VM instance named `first-vm-cli` in the **us-central1-a** zone with an **e2-medium** machine type. This command also assigns the **http-server** tag to allow HTTP traffic.