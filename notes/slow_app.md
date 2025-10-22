## "Bizerte": The Slow Application

### Problem

A Python web application running on port 5000 from the /opt directory is experiencing severe performance issues; every request takes more than 5 seconds to complete.
The application is supposed to use the redis-server cache service for speed.

Your mission is to diagnose the performance bottleneck and restore the application to its normal, fast response time.

Do not change the Python application file slow_app.py.

src: https://sadservers.com/scenario/bizerte

### Solution

#### 1. check the app source code

The code looks correct. Probably the issue is the redis connection or redis server itself.

```python
admin@i-08d4fa4f8dd17287b:/opt$ cat slow_app.py 
#!/usr/bin/python3
from flask import Flask
import redis
import time
import os

app = Flask(__name__)
redis_host = os.getenv('REDIS_HOST', '127.0.0.1')
r = redis.Redis(host=redis_host, port=6379, socket_connect_timeout=1)

@app.route('/')
def get_data():
    try:
        r.ping()
        return "Data from FAST cache!"
    except redis.exceptions.ConnectionError:
        time.sleep(5)
        return "Data from SLOW database!"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### 2. check the redis whether listening on the 6379 port

Yes.

```
admin@i-08d4fa4f8dd17287b:/opt$ sudo netstat -tupln 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:5000            0.0.0.0:*               LISTEN      814/python3         
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      318/systemd-resolve 
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      695/redis-server 12
```

#### 3. ping the redis server using redis-cli

redis server is working fine.

```
admin@i-08d4fa4f8dd17287b:/opt$ redis-cli -h 127.0.0.1 -p 6379 PING
PONG
```

#### 4. Found the slow app is managed by systemd and it is using the wrong redis host IP address

Wrong IP address: `127.0.0.2`.

```
admin@i-08d4fa4f8dd17287b:/etc/systemd/system$ cat slow-app.service 
[Unit]
Description=Slow Flask Application
After=network.target redis-server.service

[Service]
Environment="REDIS_HOST=127.0.0.2"
ExecStart=/usr/bin/python3 /opt/slow_app.py
Restart=always
User=nobody
Group=nogroup

[Install]
WantedBy=multi-user.target
```

#### 5. Fix the redis host IP address in the systemd unit file and restart the slow app service

```
admin@i-08d4fa4f8dd17287b:/etc/systemd/system$ sudo systemctl daemon-reload
admin@i-08d4fa4f8dd17287b:/etc/systemd/system$ sudo systemctl restart slow-app
admin@i-08d4fa4f8dd17287b:/etc/systemd/system$ curl localhost:5000
Data from FAST cache!admin@i-08d4fa4f8dd17287b:/etc/systemd/system$ 
```


