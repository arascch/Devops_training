## Create Simple Systemd Services

### Create a script file
Firstly you need to create your Script everywhere you need to be. In this example, I'm creating a script in ```/root/scripts/myscript.sh```.
```
root@server1:~# mkdir scripts
root@server1:~# cd scripts/
root@server1:~/scripts# nano myscript.sh
```
after you create your script, add your stuff to your file.

>myscript.sh
```
#!/bin/bash
echo "Hello, this is my first script in ubuntu" >> /tmp/myservice_output.log

```
after that, you need to grant permission to execute your script.
```
chmod +x myscript.sh
```
### Create a Service file

to start your script as a service you need to create a service file into the ```/etc/systemd/system/```.

if you do ``` ls ``` into this location you can see a bunch of .service files.

so we create our .service file beside these files and add our settings in our file like below.

>myservice.service
```
[Unit]
Description=My first custom systemd service  #in this part you can write about your script
After=network.target

[Service]
ExecStart=/root/scripts/myscript.sh #in this part you need to write exact path to your script
Restart=always

[Install]
WantedBy=default.target
```
so you create your service and need to start it with systemctl, but firstly must restart systemd with this command.
``` sudo systemctl daemon-reload ```
and then start your service :
``` sudo systemctl start myservice ```
if you need to check your service use this command:
```
sudo systemctl status myservice #myservice = write your service name you created

```

That's It ;)

