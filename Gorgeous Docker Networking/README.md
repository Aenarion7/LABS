# Links

Dockerdocs: https://docs.docker.com/build/building/multi-stage/

# Docker Network Types

Docker offers several built-in network drivers that allow for various methods of communication between containers and between containers and the external world. Below is a brief overview of each

## Bridge

- Description:  
    This is the default network type for Docker containers
- Functionality:  
    Creates an internal, private network on the Docker host, allowing containers to communicate with each other
- Use Case:  
    Ideal for applications running on a single host that need to interact with one another

![image](https://github.com/user-attachments/assets/67d25114-48cc-4130-a919-28adc6814263)


## Bridge User Defined

- Description:
    Exactly this same as previous but it is user created. It should be used instead of Default. Allows network ISOLATION and DNS in the network, so one can ping containers by names
- Functionality:  
    Creates an internal, private network on the Docker host, allowing containers to communicate with each other
- Use Case:  
    Ideal for applications running on a single host that need to interact with one another
    
![User Created Bridge](https://github.com/user-attachments/assets/e5490bfc-d3c2-469b-9446-045ffc177b10)


## Host

- Description:  
      The container shares the network stack with the Docker host
- Functionality:  
      The container directly uses the host's network interfaces without network isolation.   
- Use Case:   
      Useful when the containerized application requires high network performance or needs to listen on the same ports as the host

![Host](https://github.com/user-attachments/assets/b5847482-513d-4f65-96fa-b029aca51711)

## MACVLAN Bridge

- Description:  
    Assigns a unique MAC address to each container, allowing it to appear as a physical device on the network. So like virtual machines on the server
- Functionality:  
    Provides direct network access with individual IPs
- Use Case:
    Scenarios requiring containers to be directly accessible on the physical network
- It has some drawbacks: NoDHCP, you need to set address and if you dont, there could be a problem with two DHCP severs. And Promiscous mode is needed on all devicess

![MacVLAN](https://github.com/user-attachments/assets/72e5531e-e7f8-4b69-a429-61046d4f8717)


## MACVLAN 802.1q
- Description:
    Same as Macvlan Bridge, but it adds trunked VLAN-s.
  
![MACVLAN 802.1q](https://github.com/user-attachments/assets/35b8cddd-7e5a-4cdd-b36e-93bdad056210)


## IPVLAN L2
- Description:
    L2 IPVLAN gives all which directly connected to the network conteiners does in Macvlan. Containers share MAC with the host, so there is no problem with promiscous mode. Conteiners has their own IP addresses. BUT one MAC could be a problem (security, architecture)

![IPVLAN L2](https://github.com/user-attachments/assets/1b48860a-42e8-4408-87cb-19f458637d45)

## IPVLAN L3
- Description:
    In that network type, one can think about Host as a Router betwean switch and docker containers. The one drowback of this, there is routing needed, static routes/dynamic. But it gives our best isolation.

  ![IPVLAN L3](https://github.com/user-attachments/assets/5c9c3ea6-7fff-4733-8c17-e44b6bccd570)

## Overlay networks - docker swarm - not covered
## The none network - loopback network - not covered 