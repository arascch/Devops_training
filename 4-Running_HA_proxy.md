# Running HAproxy on Ubuntu 24.04

In this practice, I'm running HAproxy beside nginx webservers to increase reliability and accessibility to our servers and reduce traffic on servers with load balancing.

### requirements:
> -Ubuntu 24.04 - running minimum 4 Server
>
> 2 of them for Nginx web servers and 2 for haproxy and keepalived
> 
> -nginx
> 
> -HAproxy

firstly I run nginx as a webserver on two servers and deploy simple websites on them, then I run and configure two servers for HAproxy as a load balancer and in the end, run and deploy a Keepalived for High availability and load balancing.
> [!note]
>  Server IP address :
> 
> Nginx 1: 192.168.1.17
> 
> Nginx 2: 192.168.1.107
> 
> HAproxy and keepalived 1 : 192.168.1.110
>
> HAproxy and keepalived 2 : 192.168.1.108

## Practice Diagram:

![diagram](/img/haf.jpg)

## 1. Webserver's Configuration

firstly install nginx on both servers 
```
root@server1:~# apt install nginx
root@server2:~# apt install nginx
```
then as you see in the below pictures, I config both webservers as the same and copied index files in ```/var/www/raman.net/index.html```.

after that go to this root ```/etc/nginx/sites-available/``` and create a config file ```/etc/nginx/sites-available/raman.net.conf``` for the website you want Server on servers.

> raman.net.conf

>Webserver 1 
![raman.net.conf file](img/wsrv1.png)

>Webserver 2 
![raman.net.conf file](/img/wsrv2.png)

> [!important]
> YOU HAVE TO CREATE INDEX FILE AND CONFIG FILE ON BOTH OF SERVERS !!!!

If you have done the right config, have to see this page (in server 2 I changed the title to see a difference in servers):
![server1 curl](img/rmn.png)

## 2. Install and config HAproxy
because we want to run SSL termination beside the Load balancing firstly have to create Certificate files.
```
root@server3:/etc/haproxy/cert# openssl req -new -key private.key -out csr.csr
root@server3:/etc/haproxy/cert# openssl x509 -req -days 365 -in csr.csr -signkey private.key -out certificate.pem
```
>[!important]
> In a few distro when you reload HAproxy, you catch an error for your private.key, because the private key isn't recognised in .pem file and you have to do this :
>
> ```root@server3:/etc/haproxy/cert# cat certificate.pem private.key >raman.net.pem ```

### HAproxy Config file 
config your HAproxy config file as you can see:
> ### HAproxy Server 1
```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend raman.net #add your FQDN and replace with raman.net
bind *:80
bind *:443 ssl crt /etc/haproxy/cert/raman.net.pem #replace your certificate file location 
acl raman_front hdr(host) -i raman.net 
use_backend raman_backend if raman_front

backend raman_backend
server raman_srv1 192.168.1.13:80 check
server raman_srv2 192.168.1.107:80 check
```
> ### HAproxy Server 2
```
  GNU nano 7.2                                                                                                                        haproxy.cfg                                                                                                                                 
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend raman.net
bind *:80
bind *:443 ssl crt /etc/haproxy/cert/raman.net.pem
acl raman_front hdr(host) -i raman.net
use_backend raman_backend if raman_front

backend raman_backend
server raman_srv1 192.168.1.13:80 check
server raman_srv2 192.168.1.107:80 check
```
so now, if you do trace root, can see that our IP address has change to the HAproxy server IP.
but this is not enough. to run the right high-availability infrastructure, have to create VRRP with keepalive.
with keepalived we have Virtual IP.

## config Keepalived
I config keepalive on Server 3 and Server 4 because don't have enough resources to create new servers :D

> ### Install and config Keepalive on servers

Server 1 ```/etc/keepalived/keepalived.conf ```
```
#master Keepalived server 192.168.1.201
global_defs {
        router_id haproxy_ha;
}
vrrp_script chk_haproxy {
        script "pgrep haproxy"
        }
vrrp_instance VI_1 {
        state MASTER # Change to BACKUP on the other server
        interface ens33 # Replace with your network interface name
        virtual_router_id 51
        priority 100 # Set to 50 on the other server
        advert_int 1
        unicast_src_ip    192.168.1.110

        unicast_peer {
                192.168.1.108
        }
        virtual_ipaddress {
                192.168.1.201 # Replace with your desired virtual IP address
        }
        track_script {
                chk_haproxy;
        }
}

#Slave Keepalived server 192.168.1.200
global_defs {
        router_id haproxy_ha;
}
vrrp_script chk_haproxy {
        script "pgrep haproxy"
        }
vrrp_instance VI_2 {
        state BACKUP; # Change to BACKUP on the other server
        interface ens33 # Replace with your network interface name
        virtual_router_id 52
        priority 50 # Set to 50 on the other server
        advert_int 2
        unicast_src_ip    192.168.1.110

        unicast_peer {
                192.168.1.108
        }
        virtual_ipaddress {
                192.168.1.200 # Replace with your desired virtual IP address
        }
        track_script {
                chk_haproxy;
        }
}


```
Server 2 ```/etc/keepalived/keepalived.conf ```
```
#maser keepalived server 192.168.1.200
global_defs {
        router_id haproxy_ha;
}
vrrp_script chk_haproxy {
        script "pgrep haproxy"
        }
vrrp_instance VI_1 {
        state MASTER # Change to BACKUP on the other server
        interface ens33 # Replace with your network interface name
        virtual_router_id 51
        priority 100 # Set to 50 on the other server
        advert_int 1
        unicast_src_ip    192.168.1.108

        unicast_peer {
                192.168.1.110
        }
        virtual_ipaddress {
                192.168.1.200 # Replace with your desired virtual IP address
        }
        track_script {
                chk_haproxy;
        }
}

#Slave keepalived server 192.168.1.201

global_defs {
        router_id haproxy_ha;
}
vrrp_script chk_haproxy {
        script "pgrep haproxy"
        }
vrrp_instance VI_2 {
        state BACKUP; # Change to BACKUP on the other server
        interface ens33 # Replace with your network interface name
        virtual_router_id 52
        priority 50 # Set to 50 on the other server
        advert_int 2
        unicast_src_ip    192.168.1.108

        unicast_peer {
                192.168.1.110
        }
        virtual_ipaddress {
                192.168.1.201 # Replace with your desired virtual IP address
        }
        track_script {
                chk_haproxy;
        }
}

```
Now if you write ``` root@server3:/etc/keepalived# ip a ``` have to see :
>keepalived 1
![keepalived1](/img/keep1.png)
>keepalived2
![keepalived2](/img/keep2.png)


