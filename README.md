# cicd-system-demo-01
Demo using Cloud Workstations, GitHub, Cloud Build, and Cloud Deploy (and other stuff)

### set up 2 clusters

```
export PROJECT=cicd-system-demo-01
export CLUSTER_DEV=cluster-dev-01
export CLUSTER_PROD=cluster-prod-01
export REGION=us-central1
export VPC=default

# set up VPC
gcloud compute networks create ${VPC} --project=${PROJECT} --subnet-mode=auto --mtu=1460 --bgp-routing-mode=regional && gcloud compute firewall-rules create default-allow-custom --project=${PROJECT} --network=projects/${PROJECT}/global/networks/${VPC} --description=Allows\ connection\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ custom\ protocols. --direction=INGRESS --priority=65534 --source-ranges=10.128.0.0/9 --action=ALLOW --rules=all && gcloud compute firewall-rules create default-allow-ssh --project=${PROJECT} --network=projects/${PROJECT}/global/networks/${VPC} --description=Allows\ TCP\ connections\ from\ any\ source\ to\ any\ instance\ on\ the\ network\ using\ port\ 22. --direction=INGRESS --priority=65534 --source-ranges=0.0.0.0/0 --action=ALLOW --rules=tcp:22


# set up clusters
gcloud container --project ${PROJECT} clusters create-auto ${CLUSTER_DEV} --region ${REGION} --release-channel "regular" --network "projects/${PROJECT}/global/networks/${VPC}" --subnetwork "projects/${PROJECT}/regions/${REGION}/subnetworks/${VPC}" --cluster-ipv4-cidr "/17" --services-ipv4-cidr "/22"

gcloud container --project ${PROJECT} clusters create-auto ${CLUSTER_PROD} --region ${REGION} --release-channel "regular" --network "projects/${PROJECT}/global/networks/${VPC}" --subnetwork "projects/${PROJECT}/regions/${REGION}/subnetworks/${VPC}" --cluster-ipv4-cidr "/17" --services-ipv4-cidr "/22"
```



