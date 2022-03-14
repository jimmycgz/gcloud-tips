# Resolve python3.9 issue with error: AttributeError: module 'importlib' has no attribute 'util'
# solution: user python 3.8 

# make sure python3 uses 3.8
which python3
# should be /usr/bin/python3

# remove the incorerct soft link of 3.9
rm /usr/local/bin/python3

export CLOUDSDK_PYTHON=python3 

# Get folder id/Hierarchy of parent or Ancestry for a project, run in cloud cli if denied in local terminal
curl -X POST -H "Authorization: Bearer \"$(gcloud auth application-default print-access-token)\"" \
          -H "Content-Type: application/json; charset=utf-8" \
             https://cloudresourcemanager.googleapis.com/v1/projects/indigo-lotus-318921:getAncestry

gcloud resource-manager folders create \
   --display-name=[DISPLAY_NAME] \
   --organization=[ORGANIZATION_ID]

# Create folder fldr-jmy-test <id: 1234567899> under folder /GCP/Internal <id: 2234567899> 
gcloud resource-manager folders create \
   --display-name=fldr-jmy-test \
   --folder=2234567899

# Create new proj under my folder sada-jmy-test
export MY_PROJ=proj-jmy-anthos
gcloud projects create $MY_PROJ \
   --folder=1234567899  

curl -X GET -H "Authorization: Bearer \"$(gcloud auth application-default print-access-token)\"" \
-H "Content-Type: application/json; charset=utf-8" \
https://cloudresourcemanager.googleapis.com/v3/projects

curl -X POST -H "Authorization: Bearer \"$(gcloud auth application-default print-access-token)\"" \
          -H "Content-Type: application/json; charset=utf-8" \
             https://cloudresourcemanager.googleapis.com/v1/projects/sada-jmy-test-2021:getAncestry   


gcloud resource-manager folders describe 1234567899
gcloud projects describe proj-jmy-anthos

gcloud config list
gcloud config configurations list
gcloud config configurations create jmy-test
gcloud config configurations activate jmy-test
gcloud config set project $MY_PROJ
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# Setup billing account for the project
gcloud beta billing accounts list
gcloud beta billing projects link $MY_PROJ --billing-account=xx303-12345-054321

# Create test instance

gcloud compute instances create lamp-1-vm2 
gcloud compute instances delete lamp-1-vm 


# create gce with local ssd
gcloud compute instances create instance-ssd \
    --local-ssd interface=nvme \
    --project centralized-monitoring-jmy \
    --zone us-central1-a