# Configure DNS traffic steering using Geolocation Routing Policy
Geolocation policy helps to ensure that client request is routed to a closest server.
- Create VMs in separate regions
- Lunch servers in each reagion exept one (asia)
- Create a Geolocation routing policy
- Test configuration

<Image 1>

### Enable APIs
    gcloud services enable compute.googleapis.com
    gcloud services enable dns.googleapis.com

Veryfi that API is enabled:

    gcloud services list | grep -E 'compute|dns'

### Configure Firewall
For SSH:

    gcloud compute firewall-rules create fw-default-iapproxy \
    --direction=INGRESS \
    --priority=1000 \
    --network=default \
    --action=ALLOW \
    --rules=tcp:22,icmp \
    --source-ranges=35.235.240.0/20

For HTTP:
See that we operate on TAGS (VM servers later on will need to have such a tag): 

    gcloud compute firewall-rules create allow-http-traffic --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server

### Launch Client VMs
    gcloud compute instances create us-client-vm --machine-type=e2-micro --zone "Zone 1"

    gcloud compute instances create europe-client-vm --machine-type=e2-micro --zone ""Zone 2""

    gcloud compute instances create asia-client-vm --machine-type=e2-micro --zone ""Zone 3""

### Lauch Servers
    gcloud compute instances create us-web-vm \
    --machine-type=e2-micro \
    --zone="Zone 1" \
    --network=default \
    --subnet=default \
    --tags=http-server \
    --metadata=startup-script='#! /bin/bash
    apt-get update
    apt-get install apache2 -y
    echo "Page served from: "Region 1"" | \
    tee /var/www/html/index.html
    systemctl restart apache2'

    gcloud compute instances create europe-web-vm \
    --machine-type=e2-micro \
    --zone="Zone 2" \
    --network=default \
    --subnet=default \
    --tags=http-server \
    --metadata=startup-script='#! /bin/bash
    apt-get update
    apt-get install apache2 -y
    echo "Page served from: "Zone 2"" | \
    tee /var/www/html/index.html
    systemctl restart apache2'

### Collect IP addresses of the web servers
You can check and write IP addresses from GCP GUI or from CLI by:

    export US_WEB_IP=$(gcloud compute instances describe us-web-vm --zone="Zone 1" --format="value(networkInterfaces.networkIP)")

    export EUROPE_WEB_IP=$(gcloud compute instances describe europe-web-vm --zone="Zone 2" --format="value(networkInterfaces.networkIP)")

### Configure DNS
Before creating DNS records, DNS Private Zone needs to be created.

    gcloud dns managed-zones create example --description=test --dns-name=example.com --networks=default --visibility=private

### Create DNS Routing Policy and create a DNS record

    gcloud dns record-sets create geo.example.com \
    --ttl=5 --type=A --zone=example \
    --routing-policy-type=GEO \
    --routing-policy-data=""Region 1"=$US_WEB_IP;"Region 2"=$EUROPE_WEB_IP"

It creates a record with TTL 5 seconds. The policy type is GEO. Format of routing policy data = ${region}:${rrdata},${rrdata}.

Verify

    gcloud dns record-sets list --zone=example

### Testing
SSH to clinet VM

    gcloud compute ssh europe-client-vm --zone "Zone 2" --tunnel-through-iap

Connection Test. A loop that repeats the part between do and done. It literally sends a GET request to geo.example.com.

    for i in {1..10}; do echo $i; curl geo.example.com; sleep 6; done

Check the "Zone" from returned values. 

Connect to another client VM

    gcloud compute ssh us-client-vm --zone "Zone 1" --tunnel-through-iap

    for i in {1..10}; do echo $i; curl geo.example.com; sleep 6; done

Now check which server responded to the request. Responce should be sent from clients reagion.

Now, test Asia client

    gcloud compute ssh asia-client-vm --zone <SELECTED-ZONE> --tunnel-through-iap

    for i in {1..10}; do echo $i; curl geo.example.com; sleep 6; done

Notice the responding server. As there is no policy in Asia region, cloud DNS will direct the client to the nearest server.

### Clean all after tests:

    #delete VMS
    gcloud compute instances delete -q us-client-vm --zone "ZONE"

    gcloud compute instances delete -q us-web-vm --zone "ZONE"

    gcloud compute instances delete -q europe-client-vm --zone "Zone 2"

    gcloud compute instances delete -q europe-web-vm --zone "Zone 2"

    gcloud compute instances delete -q asia-client-vm --zone SELECTED-ZONE

    #delete FW rules
    gcloud compute firewall-rules delete -q allow-http-traffic

    gcloud compute firewall-rules delete fw-default-iapproxy

    #delete record set
    gcloud dns record-sets delete geo.example.com --type=A --zone=example

    #delete private zone
    gcloud dns managed-zones delete example