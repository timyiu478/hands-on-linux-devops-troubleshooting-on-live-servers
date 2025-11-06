## "Cairo": Time for a Timer

### Problem

A critical health check script at /opt/scripts/health.sh is supposed to run every 10 seconds. This check is triggered by a systemd timer.
The script's job is to check the local Nginx server and write its status (e.g., "STATUS: OK") to the log file at /var/log/health.log.
The log file is not being updated, and it appears the health check is failing.

Find out why the health check system is broken and fix it. The check will pass once the /var/log/health.log file is being correctly updated by the timer with a STATUS: OK message.

### Solution

1. check the `health.sh`

The script looks good.

```
admin@i-05cbf0a11c21286f8:~$ cat /opt/scripts/health.sh 
#!/bin/bash
# This script logs status and exits with 0 on success, 1 on failure
if curl -s --max-time 2 http://localhost | grep -q "Welcome to nginx"; then
  echo "$(date): STATUS: OK" >> /var/log/health.log
  exit 0
else
  echo "$(date): STATUS: FAILED" >> /var/log/health.log
  exit 1
fi
```

2. check if the `nginx` servier is running and listen to port 80

Yes.

```
admin@i-05cbf0a11c21286f8:~$ systemctl status nginx.service 
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Thu 2025-11-06 05:30:40 UTC; 2min 51s ago
 Invocation: ce0fd50f402f4e489d0183ffac6b18aa
       Docs: man:nginx(8)
    Process: 782 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 808 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 838 (nginx)
      Tasks: 3 (limit: 503)
     Memory: 4.1M (peak: 4.6M)
        CPU: 49ms
     CGroup: /system.slice/nginx.service
             ├─838 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─843 "nginx: worker process"
             └─844 "nginx: worker process"

Nov 06 05:30:40 i-05cbf0a11c21286f8 systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Nov 06 05:30:40 i-05cbf0a11c21286f8 systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
admin@i-05cbf0a11c21286f8:~$ sudo netstat -tupln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      820/sshd: /usr/sbin 
tcp        0      0 127.0.0.54:53           0.0.0.0:*               LISTEN      319/systemd-resolve 
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      319/systemd-resolve 
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      838/nginx: master p 
tcp        0      0 0.0.0.0:5355            0.0.0.0:*               LISTEN      319/systemd-resolve 
tcp6       0      0 :::22                   :::*                    LISTEN      820/sshd: /usr/sbin 
tcp6       0      0 :::6767                 :::*                    LISTEN      773/sadagent        
tcp6       0      0 :::80                   :::*                    LISTEN      838/nginx: master p 
tcp6       0      0 :::8080                 :::*                    LISTEN      772/gotty           
tcp6       0      0 :::5355                 :::*                    LISTEN      319/systemd-resolve 
udp        0      0 0.0.0.0:5355            0.0.0.0:*                           319/systemd-resolve 
udp        0      0 127.0.0.54:53           0.0.0.0:*                           319/systemd-resolve 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           319/systemd-resolve 
udp        0      0 10.1.13.167:68          0.0.0.0:*                           724/systemd-network 
udp6       0      0 :::5355                 :::*                                319/systemd-resolve
```

3. check if the health timer is active

No.

```
admin@i-05cbf0a11c21286f8:~$ systemctl cat health.timer
# /etc/systemd/system/health.timer
[Unit]
Description=Run health check every 10 seconds

[Timer]
OnBootSec=10
OnUnitActiveSec=10
Unit=health.service

[Install]
WantedBy=timers.target
admin@i-05cbf0a11c21286f8:~$ systemctl cat health.service 
# /etc/systemd/system/health.service
[Unit]
Description=Health Check Service

[Service]
Type=oneshot
ExecStart=/opt/scripts/health.sh

[Install]
WantedBy=multi-user.target
```

4. enable timer

```
admin@i-05cbf0a11c21286f8:~$ sudo systemctl enable health.timer 
Created symlink '/etc/systemd/system/timers.target.wants/health.timer' → '/etc/systemd/system/health.timer'.
admin@i-05cbf0a11c21286f8:~$ sudo systemctl start health.timer 
admin@i-05cbf0a11c21286f8:~$ sudo systemctl status health.timer
● health.timer - Run health check every 10 seconds
     Loaded: loaded (/etc/systemd/system/health.timer; enabled; preset: enabled)
     Active: active (running) since Thu 2025-11-06 05:39:49 UTC; 16s ago
 Invocation: 0905b172dd8540f0877a930c471c3cef
    Trigger: n/a
   Triggers: ● health.service

Nov 06 05:39:49 i-05cbf0a11c21286f8 systemd[1]: Started health.timer - Run health check every 10 seconds.
```

5. Try to curl and ping manually

* curl: can't establish connection
* ping: OK

```
admin@i-05cbf0a11c21286f8:~$ curl -vvv localhost
05:48:37.639152 [0-x] == Info: [READ] client_reset, clear readers
05:48:37.639400 [0-0] == Info: Host localhost:80 was resolved.
05:48:37.639691 [0-0] == Info: IPv6: ::1
05:48:37.639812 [0-0] == Info: IPv4: 127.0.0.1
05:48:37.639983 [0-0] == Info: [SETUP] added
05:48:37.640198 [0-0] == Info:   Trying [::1]:80...
05:48:37.640480 [0-0] == Info: [SETUP] Curl_conn_connect(block=0) -> 0, done=0
05:48:37.842018 [0-0] == Info:   Trying 127.0.0.1:80...
05:48:37.842273 [0-0] == Info: [SETUP] Curl_conn_connect(block=0) -> 0, done=0
05:48:38.843628 [0-0] == Info: [SETUP] Curl_conn_connect(block=0) -> 0, done=0
05:48:39.845031 [0-0] == Info: [SETUP] Curl_conn_connect(block=0) -> 0, done=0
05:48:40.846432 [0-0] == Info: [SETUP] Curl_conn_connect(block=0) -> 0, done=0
admin@i-05cbf0a11c21286f8:~$ ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
^C
--- 1.1.1.1 ping statistics ---
39 packets transmitted, 0 received, 100% packet loss, time 38893ms
admin@i-05cbf0a11c21286f8:~$ ping 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.042 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.042 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.046 ms
^C
--- 127.0.0.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2045ms
rtt min/avg/max/mdev = 0.042/0.043/0.046/0.002 ms
```

6. check if any iptables rules drop the TCP packets

Yes.

```
admin@i-05cbf0a11c21286f8:~$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy DROP)
target     prot opt source               destination         
DOCKER-USER  all  --  anywhere             anywhere            
DOCKER-FORWARD  all  --  anywhere             anywhere            

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  anywhere             localhost            tcp dpt:http /* The hidden problem (IPv4) */

Chain DOCKER (1 references)
target     prot opt source               destination         
DROP       all  --  anywhere             anywhere            

Chain DOCKER-BRIDGE (1 references)
target     prot opt source               destination         
DOCKER     all  --  anywhere             anywhere            

Chain DOCKER-CT (1 references)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             ctstate RELATED,ESTABLISHED
```

7. delete the iptables rule that drop TCP packets

```
admin@i-05cbf0a11c21286f8:~$ sudo iptables -D OUTPUT 1
admin@i-05cbf0a11c21286f8:~$ curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
