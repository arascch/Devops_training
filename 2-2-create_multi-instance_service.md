## Create multi-instance Service in Ubuntu 24.04

as I describe in [systemd service](https://github.com/arascch/Devops_training/blob/main/2-0-systemdService.md) we can simply create a service in Linux but in this part, I want to create a multi_instance service with python and flask and config it as Multi-instance service. so ready to go ;)


### 1. Create a script that you want to run as a multi-instance service

you can write your script with any programming language but in this practice, I'm creating a script with the Flask framework.



>app.py
```
from flask import Flask , jsonify
import sys

app = Flask(__name__)

@app.route('/hello')
def hello():
        response = {'message':'Hello,this is your Flask API!'}
        return jsonify(response)

if __name__=='__main__':
        port = int(sys.argv[1]) if len(sys.argv)>1 else 5000
        app.run(host = "0.0.0.0" ,port=port , debug=True)
```
> [!note]
> After creating .py file we need to grant access 'execute' to run app.
> I set host address "0.0.0.0" for access every devices in the local network to my service  

```
root@server1:~/scripts# chmod +x app.py
```
### 2. Create multi-instance service file

```
root@server1: cd /etc/systemd/system
root@server1: nano myflaskapp@.service
```
>myflaskapp@.service
```
[Unit]
Description = My flask api service(Instance %i)
After = network.target

[Service]
User = root
WorkingDirectory = /root/scripts/
ExecStart = /usr/bin/python /root/scripts/app.py %i
Restart = always

[Install]
WantedBy =default.target
```
> [!note]
> In WorkingDirectory and Exec start you have to change and define your locations.

### 3.Reload and start instances 

systemd needs to reload to know about your created service so you need to reload systems.
```
root@server1:/etc/systemd/system# sudo systemctl daemon-reload

```
then we need to start the service with your instance. in this part, I'm running service with two different instance

#### - instance 1
```
root@server1:/etc/systemd/system# systemctl start myflaskapp@5001
root@server1:/etc/systemd/system# systemctl enable myflaskapp@5001
root@server1:/etc/systemd/system# systemctl status myflaskapp@5001
```
![instance 5001 status](/img/5001.jpg)

### - instance 2

```
root@server1:/etc/systemd/system# systemctl start myflaskapp@8010
root@server1:/etc/systemd/system# systemctl enable myflaskapp@8010
root@server1:/etc/systemd/system# systemctl status myflaskapp@8010
```
![instance 8010 status](/img/8010.png)

### 4. Enter the URL and check the result 

I go to 192.168.1.13:5001/hello and 192.168.1.13:8010 and that's the result :

>192.168.1.13:5001/hello

![first result](/img/port5001.png)

>192.168.1.13:8010/hello

![second result](/img/port8010.png)

That's It ;)
