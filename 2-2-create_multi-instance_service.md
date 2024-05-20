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

### 2. create multi-instance service file

```
