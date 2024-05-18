## Why regular users cannot access ports below 1024 in Linux
### In this article we know about "why regular users cannot access ports below 1024 in Linux"

By default, ports less than 1024 are reserved for root users for security reasons and rootless users are not allowed to use them.
but if you need to access them, you can change the file ```/etc/sysctl.conf```.

Use the command ```sysctl net.ipv4.ip_unprivileged_port_start=xxx``` as root or privileged user to access non-root users to those ports.

> [!note]
> you can manually add the below command in ```/etc/sysctl.conf```.

```
net.ipv4.ip_unprivileged_port_start=xxx
```


