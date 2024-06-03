## Running HAproxy on Ubuntu 24.04

In this practice, I'm running HAproxy beside nginx webservers to increase reliability and accessibility to our servers and reduce traffic on servers with load balancing.

### requirements:
> -Ubuntu 24.04
> 
> -nginx
> 
> -HAproxy

firstly I run nginx as a webserver on two servers and deploy simple websites on them, then I run and configure two servers for HAproxy as a load balancer and in the end, run and deploy a Keepalived for High availability and load balancing.

### 1. Server 1 Configure
