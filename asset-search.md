# Tips for asset search via gcloud command


### get default project
```
export MY_PROJ=$(gcloud config list --format 'value(core.project)')

```
### List all private IPs for a project
```
gcloud asset search-all-resources \
  --project=$MY_PROJ \
  --asset-types='compute.googleapis.com/Instance' \
  --format='get(additionalAttributes.internalIPs)'

```
### List all Terminated/Active VMs for a project
```
gcloud asset search-all-resources \
 --query='state=TERMINATED' \
 --project=$MY_PROJ \
 --asset-types='compute.googleapis.com/Instance' \
 --format='table(name, assetType, location, versionedResources)'

```
### List all private IPs for a subnet within a project by filter
export MY_SUB_NET=default

```
gcloud asset search-all-resources \
 --filter="additionalAttributes.networkInterfaceNetworks[0]:projects/$MY_PROJ/global/networks/$MY_SUB_NET" \
 --project=$MY_PROJ \
 --asset-types='compute.googleapis.com/Instance' \
 --format='get(additionalAttributes.internalIPs)' 

```
### List all VMs for a subnet within a project by query, order by createTime
```
gcloud asset search-all-resources \
  --project=$MY_PROJ \
  --query="networks/$MY_SUB_NET" \
  --asset-types='compute.googleapis.com/Instance' \
  --order-by='createTime'

```
### Find the VM by private IP
```
gcloud asset search-all-resources \
  --project=$MY_PROJ \
  --query='10.128.0.2' \
  --asset-types='compute.googleapis.com/Instance' \
  --order-by='createTime'  

```
