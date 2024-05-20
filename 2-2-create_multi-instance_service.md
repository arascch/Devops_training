## Create multi-instance Service in Linux

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
> after creating .py file we need to grant access to execute for the run app.

```
root@server1:~/scripts# chmod +x app.py
```
### 2. create multi-instance service file

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
> in WorkingDirectory and Exec start you have to change define your locations.

### 3.Reload and start instances 

systemd neet reload to know about your created service so you need reload systems.
```
root@server1:/etc/systemd/system# sudo systemctl daemon-reload

```
then we need to start the service with like instance. in this part I'm run service with two different instance

#### - instance 1
```
root@server1:/etc/systemd/system# systemctl start myflaskapp@5001
root@server1:/etc/systemd/system# systemctl status myflaskapp@5001
```

