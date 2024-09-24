# Docker Network Types

Docker offers several built-in network drivers that allow for various methods of communication between containers and between containers and the external world. Below is a brief overview of each.

## Bridge

- Description:  
    This is the default network type for Docker containers.
- Functionality:  
    Creates an internal, private network on the Docker host, allowing containers to communicate with each other.
- Use Case:  
    Ideal for applications running on a single host that need to interact with one another.

![image](https://github.com/user-attachments/assets/67d25114-48cc-4130-a919-28adc6814263)


## BRIDGE (User Defined)

- Description:
    Exactly this same as previous but it is user created. It should be used instead of Default. Allows network ISOLATION and DNS in the network, so one can ping containers by names
- Functionality:  
    Creates an internal, private network on the Docker host, allowing containers to communicate with each other.
- Use Case:  
    Ideal for applications running on a single host that need to interact with one another.
    








## Host


- Description:  
      The container shares the network stack with the Docker host.  
- Functionality:  
      The container directly uses the host's network interfaces without network isolation.   
- Use Case:   
      Useful when the containerized application requires high network performance or needs to listen on the same ports as the host

