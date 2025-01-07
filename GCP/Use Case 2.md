# Overview:
- Set up Network Load Balancer
- Set up Application Load Balancer
- Create VM template
- Create VM pools
- Test connectivity and observe load balancers work
- Add firewall Rules to allow some external traffic

# 1. Set Network Load Balancer
## Set the default region and zone
    gcloud config set compute/region Region
    gcloud config set compute/zone Zone

## Create multiple web server instances

Create two Compute Engine VM-s and install Apache on them. 

    gcloud compute instances create gg1 \
        --zone=Zone \
        --tags=network-lb-tag \
        --machine-type=e2-small \
        --image-family=debian-11 \
        --image-project=debian-cloud \
        --metadata=startup-script='#!/bin/bash
        apt-get update
        apt-get install apache2 -y
        service apache2 restart
        echo "
    <h3>Web Server: gg1</h3>" | tee /var/www/html/index.html'

Secound instance:

    gcloud compute instances create gg2 \
        --zone=Zone \
        --tags=network-lb-tag \
        --machine-type=e2-small \
        --image-family=debian-11 \
        --image-project=debian-cloud \
        --metadata=startup-script='#!/bin/bash
        apt-get update
        apt-get install apache2 -y
        service apache2 restart
        echo "
    <h3>Web Server: gg2</h3>" | tee /var/www/html/index.html'

### Create a firewall rule to allow external traffic to the VM instances

    gcloud compute firewall-rules create www-firewall-network-lb \
        --target-tags network-lb-tag --allow tcp:80

### Check IP addresses of instances and verify if they are running
Check EXTERNAL_IP column:

    gcloud compute instances list

Verify instance status by its IP address for both instances:

    curl http://[IP_ADDRESS]

##  Configure Network Load Balancer Service
Create a static external IP address for your load balancer (Reserve IP address and give it a name (network-lb-ip-1)):

    gcloud compute addresses create network-lb-ip-1 \
    --region Region

Create a health check for compute instances and give it a name (basic-check):

    gcloud compute http-health-checks create basic-check

Create a VM pool (right now only a container for VM-s) and assign health check to it

    gcloud compute target-pools create www-pool \
    --region Region --http-health-check basic-check

Add the instances to the pool:

    gcloud compute target-pools add-instances www-pool \
    --instances gg1, gg2

Add forwarding rule which will redirect target traffic to specyfic pool:

    gcloud compute forwarding-rules create www-rule \
    --region  Region \
    --ports 80 \
    --address network-lb-ip-1 \
    --target-pool www-pool

Now Network Balancer should work. 

### Test traffic to VM-s and observe Load Balanced traffic:
Enter the following command to view the external IP address of the www-rule forwarding rule used by the load balancer.
This command shows detailed information about "www-rule" forwarding rule

    gcloud compute forwarding-rules describe www-rule --region Region

Create environment variable:

    IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region Region --format="json" | jq -r .IPAddress)

Show external IP address just populated to variable:

    echo $IPADDRESS

Use curl command to access the external IP address, replacing IP_ADDRESS with an external IP address from the previous command:

    while true; do curl -m1 $IPADDRESS; done

    Ctrl+C to stop



# 2. Set Application Load Balancer
Application Load Balancer in this lab will load traffic to Backend Service which is Managed Instance Group. 

## Setting Environment
### Create Compute Engine Template

This command will create VM template (instance template). This tempate can be used to create new instance in borders of one region. There is also a script which runs after every new vm start. 

    gcloud compute instance-templates create lb-backend-template \
    --region=Region \
    --network=default \
    --subnet=default \
    --tags=allow-health-check \
    --machine-type=e2-medium \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
        apt-get update
        apt-get install apache2 -y
        a2ensite default-ssl
        a2enmod ssl
        vm_hostname="$(curl -H "Metadata-Flavor:Google" \
        http://169.254.169.254/computeMetadata/v1/instance/name)"
        echo "Page served from: $vm_hostname" | \
        tee /var/www/html/index.html
        systemctl restart apache2'

### Create Managed Instance Group based on the template:

    gcloud compute instance-groups managed create lb-backend-group \
   --template=lb-backend-template --size=2 --zone=Zone

### Create Firewall Rule which allows health checks

    gcloud compute firewall-rules create fw-allow-health-check \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \          # This is Google Health checking system
    --target-tags=allow-health-check \                      # Tag used before for VM template
    --rules=tcp:80

Servers with Appache should be up and running

## Setting Application Load Balancer Service

### Set up a global static external IP address that customers can use to reach load balancer

This command creates global static IP address. 

    gcloud compute addresses create lb-ipv4-1 \
    --ip-version=IPV4 \
    --global

Note the IP address:

    gcloud compute addresses describe lb-ipv4-1 \
    --format="get(address)" \
    --global

### Create Health Checks for the Load Balancer:

    gcloud compute health-checks create http http-basic-check \
    --port 80

### Create Backend Service 

    gcloud compute backend-services create web-backend-service \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

### Attache Instance Group to Backend Service

    gcloud compute backend-services add-backend web-backend-service \
    --instance-group=lb-backend-group \
    --instance-group-zone=Zone \
    --global

### Create URL map to route incoming requests to the default backend 

URL map is a Google Cloud configuration resource used to route requests to backend services or backend buckets. For example, with an external Application Load Balancer, you can use a single URL map to route requests to different destinations based on the rules configured in the URL map:

Requests for https://example.com/video go to one backend service.
Requests for https://example.com/audio go to a different backend service.
Requests for https://example.com/images go to a Cloud Storage backend bucket.
Requests for any other host and path combination go to a default backend service.

This command bellow creates map which directs entire HTTP traffic to specyfic backend service:

    gcloud compute url-maps create web-map-http \
    --default-service web-backend-service

### Create a target HTTP proxy to route requests to your URL map:


    gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http

### Create a global forwarding rule to route incoming requests to the proxy:

    gcloud compute forwarding-rules create http-content-rule \
    --address=lb-ipv4-1\
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80


## Test traffic

In the Google Cloud console, from the Navigation menu, go to Network services > Load balancing.

Click on the load balancer that you just created (web-map-http).

In the Backend section, click on the name of the backend and confirm that the VMs are Healthy. If they are not healthy, wait a few moments and try reloading the page.

When the VMs are healthy, test the load balancer using a web browser, going to http://IP_ADDRESS/, replacing IP_ADDRESS with the load balancer's IP address.

This may take three to five minutes. If you do not connect, wait a minute, and then reload the browser.

Your browser should render a page with content showing the name of the instance that served the page, along with its zone (for example, Page served from: lb-backend-group-xxxx).

