 LAB 1 Install Compute engine, Variables, SSH conectrion and some basics
    View Active Account and Project
        gcloud auth list
        gcloud config list project

    Set default Project Region and Zone [GCP Console]
        export REGION=us-central1         
        export ZONE=us-central1-a
        gcloud config set compute/region $REGION
        gcloud config set compute/zone $ZONE

        To make variables consistent over network terminal sesions:
            #they need to be addet to a file ~/.bashrc or ~/.bash_profile.     
            echo "export REGION=us-central1" >> ~/.bashrc
            echo "export ZONE=us-central1-a" >> ~/.bashrc
            source ~/.bashrc

    To check lisk of available zone and regions
        gcloud compute regions list
        gcloud compute zones list

    Creating a Compute Engine instance - gcloud

        gcloud compute instances create INSTANCE_NAME --machine-type e2-medium --zone=$ZONE

    Connecting from gcloud to SSH of Compute Engine
        gcloud compute ssh Instance_Name --zone=$ZONE
        Y - to continue
        ENTER - to leave passphrase empty 
        exit - to exite SSH sesion

        

Lab 2 Install Compute engine with Windows image and Remote Desktop to it
    Deploy Windows Image:
        gcloud compute instances create INSTANCE_NAME \
        --image-family=windows-2022 \
        --image-project=windows-cloud \             -your project name
        --zone=$ZONE \
        --machine-type=e2-medium
        --tags=rdp-server                           -adding tag
    Configure RD and connect:
        gcloud compute instances get-serial-port-output INSTANCE_NAME --zone=ZONE           -check if server is ready for RD. There should be "Instance setup finished. Instance is ready to use.
        gcloud compute reset-windows-password [INSTANCE_NAME] --zone ZONE --user [ADMIN]      -set password
        Connect using RD client to external IP
