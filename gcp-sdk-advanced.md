## For Beginner: Understand the config for SDK and CLI

```
# Init by following the wizard step by step
gcloud init

# Create shortcut aliases
alias gc='gcloud config'
alias gconf='gcloud config configurations'

# List all config profiles you have
gcloud config configurations list

# Create a new profile
gcloud config configurations create dev

# Activate the new profile
gcloud config configurations activate dev

# Set default values: project, region
gcloud config set account user.name@domain.com 
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# Auth
gcloud auth login
```

## Advanced: Config cloud SDK

```
# Check the current default config
gcloud info
echo $CLOUDSDK_PYTHON
export CLOUDSDK_PYTHON=/usr/local/my-custom-python-install/python
echo $CLOUDSDK_ACTIVE_CONFIG_NAME
echo $CLOUDSDK_SECTION_PROPERTY
gcloud topic startup

# override this location by setting the environment variable
gcloud auth login

```

## DEMO Use Case: Keep ephemeral auth config to persistent volume
Use Case #1: When you use VDI in client environment, you need install and config and auth SDK every time because by default it's not installed on the persistent volume. How to install and config it on a persistent volume?

Use Case #2: When you use GCP in a new container at your own computer, you need config and auth SDK every time. Instead, how to use the pre-configured auth from your local computer?

## Solution for Use Case #2

### Step #2.1: Demystify gcloud SDK config

```
gcloud info

Cloud SDK on PATH: [False]
Kubectl on PATH: [/usr/bin/kubectl]

Installation Properties: [/usr/lib/google-cloud-sdk/properties]
User Config Directory: [/root/.config/gcloud]
Active Configuration Name: [default]
Active Configuration Path: [/root/.config/gcloud/configurations/config_default]

Cloud SDK on PATH: [True]
Kubectl on PATH: [/usr/local/bin/kubectl]

Installation Properties: [<Users-PATH>/google-cloud-sdk/properties]
User Config Directory: [<Users-PATH>/.config/gcloud]
Active Configuration Name: [conf-name]
Active Configuration Path: [<Users-PATH>/.config/gcloud/configurations/config_conf-name]

Account: [user.name@domain.com]
Project: [jmy-test]

Current Properties:
  [compute]
    zone: [us-central1-a]
    region: [us-central1]
  [core]
    account: [user.name@domain.com]
    disable_usage_reporting: [True]
    project: [test-proj-name]
```
### Step #2.2: Dig into the SDK auth config in a container
```
docker rm gcloud-config
docker run -it --name gcloud-config google/cloud-sdk gcloud auth login

docker start --name gcloud-config google/cloud-sdk

docker run --rm -ti --volumes-from gcloud-config google/cloud-sdk gcloud compute instances list --project test-proj-name
```

### Step #2.3: Save the SDK credential folder to local volume for the authenticated container
```
mkdir -p $HOME/.gcp-sdk-cred
docker cp gcloud-config:/root/.config $HOME/.gcp-sdk-cred/
```

### Step #2.4: Use local volume for any environment
```

# Check and revok default auth from your local computer
gcloud auth list
gcloud auth revoke user.name@domain.com

# Below will fail without auth
gcloud compute instances list --project test-proj-name

# Use pre-configured auth folder for your local terminal
export CLOUDSDK_CONFIG=$HOME/.gcp-sdk-cred/.config/gcloud
gcloud compute instances list --project test-proj-name

# Use pre-configured auth folder for any container via volume mount
docker run -ti -e CLOUDSDK_CONFIG=/config/mygcloud \
                 -v $HOME/.gcp-sdk-cred/.config/gcloud:/config/mygcloud \
                 google/cloud-sdk /bin/bash

docker run -ti -e CLOUDSDK_CONFIG=/config/mygcloud \
                 -v $HOME/.gcp-sdk-cred/.config/gcloud:/config/mygcloud \
                 google/cloud-sdk \
                 gcloud compute instances list --project test-proj-name                 
```

## Authenticate SDK for applications 

Auth python 
```
gcloud auth application-default revoke

# python should work after below auth
gcloud auth application-default login 

# python should fail after rename below cred file
mv $HOME/.config/gcloud/application_default_credentials.json $HOME/.config/gcloud/bkp-application_default_credentials.json

# python should work after specifying below cred file
export GOOGLE_APPLICATION_CREDENTIALS=$HOME/.config/gcloud/bkp-application_default_credentials.json

```


## Use Case: Auth in Github Actions via github Secret (json content)

```
# Add below secrets via github console => setting 
# Then dd below code to github action yaml file
        - name: Set up Cloud SDK
          uses: google-github-actions/setup-gcloud@master
          with:
            project_id: ${{ secrets.GCP_PROJECT_ID }}
            service_account_key: ${{ secrets.GCP_SA_KEY }}
            export_default_credentials: true
```
