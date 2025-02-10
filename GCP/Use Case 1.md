 # Overview:
- Deploy Compute Eengine Iinstance
- SSH Connection from GCP to VM Instance
- Default Variables
- Some other basics
- List all Zones in a "Asia"

# 1. Set Variables, view Basic information
### View Active Account and Project
    gcloud auth list
    gcloud config list project

### To check lisk of available zones and regions
    gcloud compute regions list
    gcloud compute zones list


### Set default Project Region and Zone [GCP Console Variables]
    export REGION=us-central1         
    export ZONE=us-central1-a
    gcloud config set compute/region $REGION
    gcloud config set compute/zone $ZONE

To make variables consistent over network terminal sesions they need to be added to a file ~/.bashrc or ~/.bash_profile 

    echo "export REGION=us-central1" >> ~/.bashrc
    echo "export ZONE=us-central1-a" >> ~/.bashrc
    source ~/.bashrc

### Creating a Compute Engine instance - gcloud

    gcloud compute instances create INSTANCE_NAME --machine-type e2-medium --zone=$ZONE

### Connecting from gcloud to SSH of Compute Engine

    gcloud compute ssh Instance_Name --zone=$ZONE
    Y           - to continue
    ENTER       - to leave passphrase empty 
    exit        - to exite SSH sesion

# 2. Deploy COMPUTE ENGINE with Windows image and REMOTE DESKTOP
### Deploy Windows Image:
        gcloud compute instances create INSTANCE_NAME \
        --image-family=windows-2022 \
        --image-project=windows-cloud \             #your project name
        --zone=$ZONE \
        --machine-type=e2-medium
        --tags=rdp-server                           #adding tag

### Configure RD and connect:

        gcloud compute instances get-serial-port-output INSTANCE_NAME --zone=ZONE           
Check if server is ready for RD. There should be "Instance setup finished. Instance is ready to use".

        gcloud compute reset-windows-password [INSTANCE_NAME] --zone ZONE --user [ADMIN]       
        set password
        
Connect using RD client to external IP

# List all Zones in a "Asia"
    gcloud compute zones list | grep -iE 'NAME.*asia' | sed 's/NAME: //i' | sort
