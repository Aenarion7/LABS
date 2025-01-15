# Configure multiple VPC with VM-s and one VM with multiple interfacess
The purpose of this lab is to provide an understanding of how communication occurs between VPCs.
- Create custom mode VPC networks with firewall rules
- Create VM instances using Compute Engine
- Explore the connectivity for VM instances across VPC networks
- Create a VM instance with multiple network interfaces


![image](https://github.com/user-attachments/assets/b37803ca-fca7-434d-8814-752f2cc4efb3)


## Create custom mode VPC networks
Create VPC network "managementnet" and new subnet managementsubnet-us:

You can go with GUI VPC network > VPC networks > Create VPC Network or CLI:

        gcloud compute networks create managementnet --project=qwiklabs-gcp-04-d0caaae828cc --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional --bgp-best-path-selection-mode=legacy && gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-04-d0caaae828cc --range=10.130.0.0/20 --stack-type=IPV4_ONLY --network=managementnet --region=us-central1

Create VPC network "privatenet" and new subnets: privatesubnet-us and privatesubnet-notus:

        gcloud compute networks create privatenet --subnet-mode=custom

        gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-central1 --range=172.16.0.0/24

        gcloud compute networks subnets create privatesubnet-notus --network=privatenet --region=europe-west4 --range=172.20.0.0/20

Check available VPC networks:

        gcloud compute networks list

Note that auto mode networks like default and mynetwork have autocreated subnets in each region. managementnet and privatenet are custom networks and thats why there are no autocreated subnets.

To check list of avaible subnets sorted by network:

        gcloud compute networks subnets list --sort-by=NETWORK

## Create firewall rules for just created VPC

Create two firewall rules to allow SSH and ICMP for managementnet

You can go with GUI VPC network > Firewall > Create Firewall Rule or CLI:

        gcloud compute --project=qwiklabs-gcp-04-d0caaae828cc firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

Create two firewall rules to allow SSH and ICMP for privatenet

        gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0

To view all firewall rules by network:

        gcloud compute firewall-rules list --sort-by=NETWORK

## Create VM-s

managementnet-us-vm in managementsubnet-us
privatenet-us-vm in privatesubnet-us

You can go with GUI VCompute Engine > VM Instances > Create Instance or CLI

Instance managementnet-us-vm:

        gcloud compute instances create managementnet-us-vm --project=qwiklabs-gcp-04-d0caaae828cc --zone=us-central1-f --machine-type=e2-medium --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=managementsubnet-us --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=968517507339-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=managementnet-us-vm,image=projects/debian-cloud/global/images/debian-12-bookworm-v20250113,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any

Instance privatenet-us-vm:

        gcloud compute instances create privatenet-us-vm --zone=us-central1-f --machine-type=e2-medium --subnet=privatesubnet-us

To view list of VM-s by zone:

        gcloud compute instances list --sort-by=ZONE

## Create VPC network:mynetwork, with firewall rules and two VM-s: mynet-notus-vm and mynet-us-vm
In similiar way created earlier with subnets names and rangess from image1.

## Test external IP connectivity

Connect over SSH to mynet-us-vm

        ping -c 3 <mynet-notus-vm external IP>

it should work, same for privatenet-us-vm and managementnet-us-vm

        ping -c 3 <managementnet-us-vm external IP>
        ping -c 3 <privatenet-us-vm external IP>

Ping is success, even though they are in either a different zone or VPC network. This confirms that public access to those instances is only controlled by the ICMP firewall rules that was created.

## Test internal IP connectivity

        ping -c 3 <privatenet-us-vm inernal IP>
        ping -c 3 <managementnet-us-vm inernal IP>
        ping -c 3 <mynet-notus-vm inernal IP>

Only mynet-notus-vm you shoud be able to ping. You can ping the internal IP address of mynet-notus-vm because it is on the same VPC network as the source of the ping (mynet-us-vm), even though both VM instances are in separate zones, regions.

Rest will not work. You cannot ping the internal IP address of managementnet-us-vm and privatenet-us-vm because they are in a separate VPC networks from the source of the ping (mynet-us-vm), even though they are all in the same zone. VPC are by default isolated from other VPC in private space IP. You can configure VPC peering or VPN to connect them

## Create VM with multiple network interfaces

You can go with GUI VCompute Engine > VM Instances > Create Instance or CLI

        gcloud compute instances create vm-appliance --project=qwiklabs-gcp-04-d0caaae828cc --zone=us-central1-f --machine-type=e2-standard-4 --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=privatesubnet-us --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=managementsubnet-us --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=mynetwork --metadata=enable-oslogin=true --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=968517507339-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/trace.append --create-disk=auto-delete=yes,boot=yes,device-name=vm-appliance,image=projects/debian-cloud/global/images/debian-12-bookworm-v20250113,mode=rw,size=10,type=pd-balanced --no-shielded-secure-boot --shielded-vtpm --shielded-integrity-monitoring --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any

Each network interface has its own internal IP address so that the VM instance can communicate with those networks.

SSH to new vm-appliance

Run command to lists a Linux VM's network interfaces with the internal IP addresses for each interface.

        sudo ifconfig

## Test connectivity over internal IP

        ping -c 3 <privatenet-us-vm inernal IP>
        ping -c 3 <managementnet-us-vm inernal IP>  
        ping -c 3 <mynet-notus-vm inernal IP>
        ping -c 3 <mynetus-us-vm inernal IP>

Ping to thouse mynet-us-vm, privatenet-us-vm, managementnet-us-vm should work

You can ping privatenet-us-vm by its name because VPC networks have an internal DNS service that allows you to address instances by their DNS names instead of their internal IP addresses. When an internal DNS query is made with the instance hostname, it resolves to the primary interface (nic0) of the instance. Therefore, this only works for privatenet-us-vm in this case.

Ping to mynet-notus-vm should not work. In a multiple interface instance, every interface gets a route for the subnet that it is in. In addition, the instance gets a single default route that is associated with the primary interface ens4. Unless manually configured otherwise, any traffic leaving an instance for any destination other than a directly connected subnet will leave the instance via the default route on ens4.

list routes:

        ip route

The primary interface ens4 gets the default route (default via 172.16.0.1 dev ens4), and all three interfaces, ens4, ens5, and ens6, get routes for their respective subnets. Because the subnet of mynet-notus-vm (10.132.0.0/20) is not included in this routing table, the ping to that instance leaves vm-appliance on ens4 (which is on a different VPC network).
