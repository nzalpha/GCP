# *Cloud Build* : https://cloud.google.com/build/docs/overview
Cloud Build is a service that executes your builds on Google Cloud. This is fully managed CI Server

Cloud Build can import source code from a variety of repositories or cloud storage spaces, execute a build to your specifications and produce artifacts such as Docker containers or Java archives.

*Build configuration and build steps*
You can write a build config to provide instructions to Cloud Build on what tasks to perform. Cloud Build executes your build as a series of build steps, where each build step is run in a Docker container.

How to create a build configuration file: refer to this link- https://cloud.google.com/build/docs/configuring-builds/create-basic-configuration

*Cloud Builders*: https://github.com/GoogleCloudPlatform/cloud-builders-community
Cloud builders are container images with common languages and tools installed in them.

If you want to use Docker, as part of your Build steps you use any publicly available images. Use name(specify the image url) field and args (to specify the command) in your buildconfig.yaml file. The following is the example 
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', '...']

---

## HandsOn Writing a Build Config to build container image, push and deploy to Cloud Run  : https://cloud.google.com/build/docs/configuring-builds/create-basic-configuration

We need to give Service Account Logs Writer role and Storage Admin Role


*STEP 1* : Clone the folder which contains the python application Docker file
git clone https://github.com/devopswithcloud/gcp_cloud_build.git

*STEP 2* : Create file  cloudbuild.yaml
  # building the container image
  steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage/cloudbuild:v2', '.']
    # Push the container image to registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push','us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage/cloudbuild:v2']
    # Deploy the image to cloud Run
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['run', 'deploy', 'cloud-run-deploy', '--image', 'us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage/cloudbuild:v2', '--region', 'us-central1','--allow-unauthenticated' ]

*STEP 3*: Run 
gcloud builds submit --config cloudbuild.yaml
Note: Make sure to give Service Account Logs Writer role and Storage Admin Role
us-central1-docker.pkg.dev/host-project-439520/repo1