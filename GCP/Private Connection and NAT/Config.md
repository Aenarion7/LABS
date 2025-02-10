# 1 Configure Private Connection
Private connections are connections made to a VM for example, over its internal IP address.
- Create VPC network and firewall rules
- Create VM without external address
- Enable Private Google Access
# 2 Configure Cloud NAT gateway


# 1 Configure Private Connection

- Create VPC Network
    - Subnet creation mode, click Custom
    - Enable private google access
- Create Firewall Rule
    - Source IPv4 ranges	35.235.240.0/20
    - Source filter 
    - tcp port 22
- Create VM instance with no public IP address
    - Network (name of VPC netwok)
    - External IPv4 address None

- Activate Cloud Shell
- Connect to the VM trough interlan connection
        gcloud compute ssh vm-internal --zone europe-west4-a --tunnel-through-iap

If you ping google.com it will not work. There is no External IP address.

When instances do not have external IP addresses, they can only be reached by other instances on the network via a managed VPN gateway or via a Cloud IAP tunnel. Cloud IAP enables context-aware access to VMs via SSH and RDP without bastion hosts

# 2 Configure Cloud NAT gateway
- In Cloud Shell try to connect to the Internet:

        sudo apt-get update

It should work becouse Cloud Shell has a external IP address.

- connect to the VM
        
        gcloud compute ssh vm-internal --zone europe-west4-a --tunnel-through-iap

        sudo apt-get update

    Only Google packegess will be updated, as they have connection over API. Other packegess will not be updated.

Cloud NAT is a regional resource. You can configure it to allow traffic from all ranges of all subnets in a region, from specific subnets in the region only

- Network services
- Cloud NAT  (in Products & Page)
- Get started
    Gateway name	nat-config
    Network	        name-vpc
    Region          europe-west4
    For Cloud Router, select Create new router.
    For Name, type nat-router
    - click create
- go back to the VM and performe update once more, it should work