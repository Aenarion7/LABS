# Docker Network Types

Docker offers several built-in network drivers that allow for various methods of communication between containers and between containers and the external world. Below is a brief overview of each:

## Bridge
<dl>
  <dt>Description:<dt> 
    <dd>This is the default network type for Docker containers.<dd>
  <dt>Functionality:<dt> 
    <dd>Creates an internal, private network on the Docker host, allowing containers to communicate with each other.<dd>
  <dt>Use Case:<dt>
    <dd>Ideal for applications running on a single host that need to interact with one another.<dd>
<dl>

## Host
<dl>
  <dt>Description:<dt>
    <dd>The container shares the network stack with the Docker host.<dd>
  <dt>Functionality:<dt>
    <dd>The container directly uses the host's network interfaces without network isolation.<dd>
  <dt>Use Case:<dt>
    <dd>Useful when the containerized application requires high network performance or needs to listen on the same ports as the host<dd>
<dl>