## Kampot: "A New Port"

### Problem

A Python app serving simulated bank data runs as root and listens on port 20280. The app is managed by supervisor and cannot be stopped or reconfigured to use a different port.

An internal legacy monitoring system expects the service to be available on port 80, but the app is hardcoded to 20280 for security and legacy reasons. Your task is to make the service accessible on port 80 locally.

### Solution

#### Idea

Set an reverse proxy listening on port 80 and forwarding requests to 20280.

#### 1. Install nginx

```
admin@i-0441d2531eee9cb77:/etc$ sudo apt install nginx
```

#### 2. Configure nginx

Open `/etc/nginx/sites-available/default` and modify it to look like this:

```
server {
        listen 80 default_server;
        listen [::]:80 default_server;


        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name _;

    location / {
        proxy_pass http://localhost:20280;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

#### 3. Restart nginx

```
admin@i-0441d2531eee9cb77:/etc/nginx/sites-available$ sudo systemctl reload nginx
admin@i-0441d2531eee9cb77:/etc/nginx/sites-available$ systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Tue 2025-09-30 14:45:10 UTC; 3min 3s ago
 Invocation: b0410349f44b4666834bfcf679663002
       Docs: man:nginx(8)
    Process: 1823 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1828 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1962 ExecReload=/usr/sbin/nginx -g daemon on; master_process on; -s reload (code=exited, status=0/SUCCESS)
   Main PID: 1880 (nginx)
      Tasks: 3 (limit: 503)
     Memory: 2.9M (peak: 6.8M)
        CPU: 89ms
     CGroup: /system.slice/nginx.service
             ├─1880 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─1963 "nginx: worker process"
             └─1965 "nginx: worker process"

Sep 30 14:45:10 i-0441d2531eee9cb77 systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Sep 30 14:45:10 i-0441d2531eee9cb77 systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
Sep 30 14:47:59 i-0441d2531eee9cb77 systemd[1]: Reloading nginx.service - A high performance web server and a reverse proxy server...
Sep 30 14:47:59 i-0441d2531eee9cb77 nginx[1962]: 2025/09/30 14:47:59 [notice] 1962#1962: signal process started
Sep 30 14:47:59 i-0441d2531eee9cb77 systemd[1]: Reloaded nginx.service - A high performance web server and a reverse proxy server.
```

#### 4. Test

```
admin@i-0441d2531eee9cb77:/etc/nginx/sites-available$ curl localhost:80/accounts
[{"id":1,"name":"Alice","type":"Checking"},{"id":2,"name":"Bob","type":"Savings"},{"id":3,"name":"Charlie","type":"Business"}]
```
