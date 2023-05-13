# cicd-system-demo-01
Demo using Cloud Workstations, GitHub, Cloud Build, and Cloud Deploy (and other stuff)

### set up 2 clusters

```
export PROJECT=cicd-system-demo-01
export CLUSTER_DEV=cluster-dev-01
export CLUSTER_PROD=cluster-prod-01
export REGION=us-central1
export VPC=default
export GKE_URI_DEV=https://container.googleapis.com/v1/projects/${PROJECT}/locations/${REGION}/clusters/${CLUSTER_DEV}
export GKE_URI_PROD=https://container.googleapis.com/v1/projects/${PROJECT}/locations/${REGION}/clusters/${CLUSTER_PROD}
export PROJECT_NUMBER=$(gcloud projects list --filter=${PROJECT} --format="value(PROJECT_NUMBER)")

# had to move project via https://console.cloud.google.com/cloud-resource-manager?_ga=2.44535143.2104396338.1683284829-1620217996.1643748390 to `default` folder

# set up VPC
gcloud compute networks create ${VPC} --project=${PROJECT} --subnet-mode=auto --mtu=1460 --bgp-routing-mode=regional && gcloud compute firewall-rules create default-allow-custom --project=${PROJECT} --network=projects/${PROJECT}/global/networks/${VPC} --description=Allows\ connection\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ custom\ protocols. --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all && gcloud compute firewall-rules create default-allow-ssh --project=${PROJECT} --network=projects/${PROJECT}/global/networks/${VPC} --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22

# not sure but might need egress
gcloud compute --project=cicd-system-demo-01 firewall-rules create allow-all-egress --direction=EGRESS --priority=1000 --network=default --action=ALLOW --rules=all --destination-ranges=0.0.0.0/0
# also added artifact registry reader role to compute SA # this did it 

# set up clusters
gcloud container --project ${PROJECT} clusters create-auto ${CLUSTER_DEV} --region ${REGION} --release-channel "regular" --network "projects/${PROJECT}/global/networks/${VPC}" --subnetwork "projects/${PROJECT}/regions/${REGION}/subnetworks/${VPC}" --cluster-ipv4-cidr "/17" --services-ipv4-cidr "/22"

gcloud container --project ${PROJECT} clusters create-auto ${CLUSTER_PROD} --region ${REGION} --release-channel "regular" --network "projects/${PROJECT}/global/networks/${VPC}" --subnetwork "projects/${PROJECT}/regions/${REGION}/subnetworks/${VPC}" --cluster-ipv4-cidr "/17" --services-ipv4-cidr "/22"

# set up fleet
# this doesn't represent a real environment where prod and dev would be in different fleets
gcloud services enable mesh.googleapis.com \
  --project=${PROJECT}

gcloud container fleet mesh enable --project ${PROJECT}
gcloud container fleet memberships register ${CLUSTER_DEV} \
  --gke-uri=${GKE_URI_DEV} \
  --enable-workload-identity \
  --project ${PROJECT}
gcloud container fleet memberships register ${CLUSTER_PROD} \
  --gke-uri=${GKE_URI_PROD} \
  --enable-workload-identity \
  --project ${PROJECT}

gcloud container fleet memberships list --project ${PROJECT}
gcloud services enable mesh.googleapis.com \
  --project=${PROJECT}

gcloud container clusters update ${CLUSTER_PROD} --project ${PROJECT} \
  --region ${REGION} --update-labels mesh_id=proj-${PROJECT_NUMBER}

gcloud container clusters update ${CLUSTER_DEV} --project ${PROJECT} \
  --region ${REGION} --update-labels mesh_id=proj-${PROJECT_NUMBER}

gcloud container fleet mesh update \
    --management automatic \
    --memberships ${CLUSTER_DEV} \
    --project ${PROJECT} \
    --location ${REGION}

gcloud container fleet mesh update \
    --management automatic \
    --memberships ${CLUSTER_PROD} \
    --project ${PROJECT} \
    --location ${REGION}

# check MCP status
gcloud container fleet mesh describe --project ${PROJECT}

```

### enable github -> cloud build integration

```
# enable API for Cloud Build
# enable API for Artifact Registry
# enable API for Secret Manager
# created AR repo `github`
# image ID format us-central1-docker.pkg.dev/cicd-system-demo-01/github/whereami-go:$COMMIT_SHA
# turned on vuln scanning
```

