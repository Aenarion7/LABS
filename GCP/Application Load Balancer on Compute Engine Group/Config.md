### Application Load Balancer on Compute Engine Group

## Set default Project Region and Zone [GCP Console Variables]
    export REGION=us-central1
    export ZONE=us-central1-b
    gcloud config set compute/region $REGION
    gcloud config set compute/zone $ZONE

## Create Compute Engine template
    gcloud compute instance-templates create gg-template-nginx \
        --network=default \
        --subnet=default \
        --tags=nginxtag \
        --machine-type=e2-medium \
        --image-family=debian-11 \
        --image-project=debian-cloud \
        --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    service nginx start
    sed -i -- "s/nginx/Google Cloud Platform - \$HOSTNAME/" /var/www/html/index.nginx-debian.html'


### Create Managed Instance Group based on the template:
    gcloud compute instance-groups managed create gg-nginx-group  \
    --template=gg-template-nginx \
    --size=2 \
    --zone=$ZONE \

### Create a Firewall Rule to allow external traffic to the VM instances:

    gcloud compute firewall-rules create grant-tcp-rule-803 \
        --target-tags nginxtag --allow tcp:80

### Set named ports (it works as mappings)
    gcloud compute instance-groups managed set-named-ports gg-nginx-group \
        --named-ports http:80 \
        --zone=$ZONE \

###  Reserve IP address for Load Balancer
    gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global

    gcloud compute addresses describe lb-ipv4-1 \
    --format="get(address)" \
    --global


### Create Health Checks for the Load Balancer:

        gcloud compute health-checks create http http-basic-check \
        --port 80

### Create Backend Service 

    gcloud compute backend-services create gg-web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global


### Attach Instance Group to Backend Service

    gcloud compute backend-services add-backend gg-web-backend-service \
    --instance-group=gg-nginx-group \
    --instance-group-zone=$ZONE \
    --global

## Create URL map to route incoming requests to the default backend 

    gcloud compute url-maps create gg-web-map-http \
    --default-service gg-web-backend-service

### Create a target HTTP proxy to route requests to your URL map:


    gcloud compute target-http-proxies create http-lb-proxy \
    --url-map gg-web-map-http

### Create a global forwarding rule to route incoming requests to the proxy:

    gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1\
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80