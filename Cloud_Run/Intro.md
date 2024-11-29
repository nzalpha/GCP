# *Cloud Run*
Cloud Run is a managed compute platform that lets you run containers directly on top of Google's scalable infrastructure.

You can deploy code written in any programming language on Cloud Run if you can build a container image from it. You dont have to worry of servers, scaling, HA, certificates etc.

This is serverless tech to deploy containerized application. This is built on top of KNative(K8 serverless platform)

---

## HandsOn Deploy Container : https://cloud.google.com/run/docs/quickstarts/deploy-container
1) To use Cloud Run to deploy application you need Container Images in GAR (Google Archieve Registry)
2) Create a GAR in shell
```bash
gcloud artifacts repositories create repo1 --repository-format=docker --location=us-central1
```
This will be the GAR : us-central1-docker.pkg.dev/host-project-439520/repo1

3) Now lets get some application with the docker file and create docker image
In the GCP shell 
```bash
git clone https://github.com/devopswithcloud/gcp_training_images.git
cd gcp_training_images/blue/
/* building an image*/
docker build -t us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage:v1 .
/* pushing the image to the repository
docker push us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage:v1 
```
4) Now we will create a cloud run service for this container image.
gcloud run deploy svc-cli --image=us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage:v1 --region=us-central1 --allow-unauthenticated

After running the above you will get the service url and also you will see the 
# Service [svc-cli] revision [svc-cli-00001-6lx] has been deployed and is serving 100 percent of traffic.
# Service URL: https://svc-cli-526842492784.us-central1.run.app

5) Now Suppose you have a new feature which you want to rollout with background color as Green. 
  First we will get the image for green color and then push it to the GAR, 
  Second, we will edit & deploy the new version and choose the image with tag as green.
 
 ```bash
 ----- Creating image for new feature of green background
docker build -t us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage:green .
---- Pushing to GAR
docker push us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage:green
---- Edit & Deploy new version in Cloud Run
gcloud run deploy svc-cli --region=us-central1 --allow-unauthenticated --image=us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage:green

# Service [svc-cli] revision [svc-cli-00002-pcx] has been deployed and is serving 100 percent of traffic.
# Service URL: https://svc-cli-526842492784.us-central1.run.app
```
6) Lets say you have a buggy image purple and you have it deployed. In order to make sure there is no traffic going to it you use "--no-traffic"
    First we will get the image for green color and then push it to the GAR, 
    Second, we will edit & deploy the new version and choose the image with tag as green.
    Third, you found its buggy, you do step second and go back to image with tag as green. 
    Fourth, if you want testing team to do testing and still have it in prod you just make sure that "--no-traffic" is used.


7) We want to change the name of the different revisions
gcloud run deploy svc-cli --region=us-central1 --allow-unauthenticated --image=us-central1-docker.pkg.dev/host-project-439520/repo1/firstimage:purple --revision-suffix=white