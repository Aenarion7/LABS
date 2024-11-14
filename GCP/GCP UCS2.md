# Overview:
- Set up Network Load Balancer
- Set up Application Load Balancer
- Create VM template
- Create VM pools
- Test connectivity and observe load balancers work
- Add firewall Rules to allow some external traffic

# Lab 1. Set Network Load Balancer
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



# Lab 2. Set Application Load Balancer