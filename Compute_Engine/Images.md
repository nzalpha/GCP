# Images
To create boot disks for your virtual machine (VM) instances, you need to obtain and use operating system (OS) images.. There are 2 types of OS Image Types:

A disk created in one zone then that disk can be attach@ed to vm that is present in that zone only.
Image created in region can be used in different region altogether.Images are shared accorss Projects and Organization. Provided the user in other project access this image should have **Compute Image User** role.

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

## Step 1: Creating a VM with a Startup Script

Use the following command to create a VM instance named `first-vm-cli` in the **us-central1-a** zone with an **e2-medium** machine type. This command also assigns the **http-server** tag to allow HTTP traffic.

    ```bash
    gcloud compute instances create disk-example-vm \
        --zone=us-central1-a \
        --machine-type=e2-micro \
        --tags=http-server \
        --metadata-from-file startup-script=startup-script.sh
    ```

## Step 2: Creating an Image out of the above VM(make sure the vm is stopped)

```bash
gcloud compute images create img1 --source-disk=vm-1 --family=web --source-disk-zone=us-central1-a --storage-location=us-central1-a
```

## Step 3: Creating VM out of the Image Created in Step 2
gcloud compute instances create imgfromvm --zone=us-central1-a --image=img1 --subnet=i27-global-lb-us


-----

Here’s the complete guide, including setting up two projects, sharing an image between them, and verifying Java and web server availability on a VM created from the shared image.

---

# **Guide: Sharing Images Across Projects and Verifying Configurations**

### **Objective**

This guide will cover:
1. Setting up two projects, `project1` and `project2`.
2. Giving user `siva@gcpbatch22.in` access only to `project2`.
3. Sharing an image (`image2`) from `project1` to `siva@gcpbatch22.in`.
4. Creating a VM in `project2` using the shared image from `project1`.
5. Verifying Java and web server installation on the VM.

---

### **Step 1: Create Two Projects in Google Cloud Console**

1. Go to **Google Cloud Console** > **IAM & Admin** > **Manage resources**.
2. **Create Project**:
   - Click **Create Project**.
   - **Project Name**: Set to `project1`.
   - Click **Create**.
3. Repeat the steps above to create a second project:
   - **Project Name**: Set to `project2`.

---

### **Step 2: Configure IAM Permissions**

1. **Assign Owner Access to `project2`**:
   - Go to **IAM & Admin** > **IAM** for `project2`.
   - Click **Add**.
   - **New Principal**: Enter `siva@gcpbatch22.in`.
   - **Role**: Select **Owner**.
   - Click **Save**.

2. **Verify No Access to `project1`**:
   - Do not assign any roles to `siva@gcpbatch22.in` in `project1`. This ensures that `siva` only has access to `project2`.

---

### **Step 3: Share Image `image2` from `project1` with `siva@gcpbatch22.in`**

1. **Share Image `image2`**:
   - In `project1`, go to **Compute Engine** > **Images**.
   - Select the image called `image2`.
   - Click **Show Info Panel**.
   - In the **Permissions** section, click **Add Principal**.
   - **Principal**: Enter `siva@gcpbatch22.in`.
   - **Role**: Assign **Compute Image User**.
   - Click **Save**.

---

### **Step 4: Verify Access as `siva` in `project2`**

1. **Login as `siva@gcpbatch22.in`**:
   - Ensure you’re logged into `siva@gcpbatch22.in` and switch to **project2** in the Google Cloud Console.

2. **Attempt to Use the Shared Image in Console**:
   - Go to **Compute Engine** > **VM instances** > **Create Instance**.
   - In the **Boot Disk** section, choose **Custom Images**.
   - The shared image `image2` will not appear here because cross-project images do not automatically populate in the console’s custom images section.

---

### **Step 5: Use `gcloud` Command to Create a VM from Shared Image**

To create a VM in `project2` using the shared image `image2` from `project1`, follow these steps:

1. **Open Cloud Shell as `siva` in `project2`**.
2. **Run the Command to Create a VM Using the Shared Image**:
   - Use the `gcloud` command, specifying `project1` as the image project and `image2` as the image name:

   ```bash
   gcloud compute instances create vm-from-shared-image \
       --project=project2 \
       --zone=us-central1-a \
       --machine-type=e2-medium \
       --image=image2 \
       --image-project=<project1_project_id>
   ```

   **Explanation**:
   - `--project=project2`: Specifies the project where the VM will be created.
   - `--image=image2`: Refers to the shared image from `project1`.
   - `--image-project=<project1_project_id>`: Replace `<project1_project_id>` with the actual Project ID of `project1`.

---

### **Step 6: Verify Java and Web Server Availability on the VM**

1. **Access the VM’s Public IP**:
   - After creating the VM in `project2`, find its public IP from the **Google Cloud Console**:
     - Go to **Compute Engine** > **VM instances**.
     - Locate `vm-from-shared-image` and note the **External IP**.

2. **Check Web Server Access**:
   - Open a web browser and enter the VM’s public IP.
   - If the web server was installed and configured to serve content, you should see a response (such as a default web page).

3. **SSH into the VM and Verify Java and Web Server Installation**:
   - SSH into the VM to verify installation:
     ```bash
     gcloud compute ssh vm-from-shared-image --zone=us-central1-a --project=project2
     ```
   - **Verify Java Installation**:
     ```bash
     java -version
     ```
   - **Confirm Local Web Server Access**:
     ```bash
     curl http://localhost
     ```

