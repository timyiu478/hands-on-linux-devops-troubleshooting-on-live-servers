## "Auderghem": containers miscommunication

### Problem

Description: There is an nginx Docker container that listens on port 80, the purpose of which is to redirect the traffic to two other containers statichtml1 and statichtml2 but this redirection is not working.
Fix the problem.

IMPORTANT. You can restart all containers, but don't stop or remove them.

Root (sudo) Access: True

Test: The nginx container must redirect the traffic to the statichtml1 and statichtml2 containers:

curl http://localhost returns the Welcome to nginx default page
curl http://localhost/1 returns HelloWorld;1
curl http://localhost/2 returns HelloWorld;2

### Solution

#### 1. test the behaviours of the nginx

Both `curl http://localhost/1` and `curl http://localhost/2` return HTTP status code 504.

```
admin@i-03fc55a1924a48445:~$ curl http://localhost
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
admin@i-03fc55a1924a48445:~$ curl http://localhost/1
<html>
<head><title>504 Gateway Time-out</title></head>
<body>
<center><h1>504 Gateway Time-out</h1></center>
<hr><center>nginx/1.29.3</center>
</body>
</html>
admin@i-03fc55a1924a48445:~$ curl http://localhost/2
<html>
<head><title>504 Gateway Time-out</title></head>
<body>
<center><h1>504 Gateway Time-out</h1></center>
<hr><center>nginx/1.29.3</center>
</body>
</html>
```

#### 2. check the container statuses

statichtml1 and statichtml2 containers:

* The container exposes port 3000/tcp inside the container
* No host port is mapped (no arrow → and no host IP/port shown)
* => can only access port 3000 from inside the Docker network or from other containers, but not from the host machine or externally

```
admin@i-03fc55a1924a48445:~$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS         PORTS                                 NAMES
89bf0e394bb9   statichtml:2   "busybox httpd -f -v…"   11 hours ago   Up 2 minutes   3000/tcp                              statichtml2
1f96c1876662   statichtml:1   "busybox httpd -f -v…"   11 hours ago   Up 2 minutes   3000/tcp                              statichtml1
7440094fc321   nginx          "/docker-entrypoint.…"   11 hours ago   Up 2 minutes   0.0.0.0:80->80/tcp, [::]:80->80/tcp   nginx
```

#### 3. check the docker container network settings

Settings:

* nginx - bridge network
* statichtml1 - static-net network, domain name: statichtml1.sadservers.local
* statichtml2 - static-net network, domain name: statichtml2.sadservers.local

Problem: nginx and the upstream servers are not in the same network

```bash
admin@i-03fc55a1924a48445:~$ docker inspect -f '{{.Name}}: {{json .NetworkSettings.Networks}}' nginx statichtml1 statichtml2
/nginx: {"bridge":{"IPAMConfig":null,"Links":null,"Aliases":null,"MacAddress":"a6:29:19:87:0a:a2","DriverOpts":null,"GwPriority":0,"NetworkID":"a9171848a0c8bd84c48f883472b8473e5401b1b4e075231cf972facf20d0b7fd","EndpointID":"8ff666c3e1ed70562257d9c1eb7fd9fda4f9f0c4a0ed6cdbd00747771d26495d","Gateway":"172.17.0.1","IPAddress":"172.17.0.2","IPPrefixLen":16,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":null},"static-net":{"IPAMConfig":{},"Links":null,"Aliases":[],"MacAddress":"56:21:f1:63:a1:21","DriverOpts":{},"GwPriority":0,"NetworkID":"cc3e04c023f16d42f4c716f21a147860434baae049e21ae49db39e54ed02707f","EndpointID":"a420149e27f8abe1d83d3b668abe0644af1e7ca49e5fa3ae738aa626a27aabf8","Gateway":"172.172.0.1","IPAddress":"172.172.0.2","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["nginx","7440094fc321"]}}
/statichtml1: {"static-net":{"IPAMConfig":{"IPv4Address":"172.172.0.11"},"Links":null,"Aliases":null,"MacAddress":"7e:41:56:02:49:a1","DriverOpts":null,"GwPriority":0,"NetworkID":"cc3e04c023f16d42f4c716f21a147860434baae049e21ae49db39e54ed02707f","EndpointID":"53f15659b7066574e5e4d199145e16a316aa97b0392388494b3d5ae38a1b4e50","Gateway":"172.172.0.1","IPAddress":"172.172.0.11","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["statichtml1","1f96c1876662","statichtml1.sadservers.local"]}}
/statichtml2: {"static-net":{"IPAMConfig":{"IPv4Address":"172.172.0.12"},"Links":null,"Aliases":null,"MacAddress":"26:b5:ce:b1:5d:6c","DriverOpts":null,"GwPriority":0,"NetworkID":"cc3e04c023f16d42f4c716f21a147860434baae049e21ae49db39e54ed02707f","EndpointID":"741145ddfc509642e35e7c6a1163a5fe367ed7eca65ba6bb1a82816ec00a3305","Gateway":"172.172.0.1","IPAddress":"172.172.0.12","IPPrefixLen":24,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"DNSNames":["statichtml2","89bf0e394bb9","statichtml2.sadservers.local"]}}
```

#### 4. change nginx network to static-net

```
admin@i-03fc55a1924a48445:~$ docker network disconnect bridge nginx
admin@i-03fc55a1924a48445:~$ docker network connect static-net nginx
```

Test:

Now, both `curl http://localhost/1` and `curl http://localhost/2` return HTTP status code 502.

```
admin@i-03fc55a1924a48445:~$ curl http://localhost
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
admin@i-03fc55a1924a48445:~$ curl http://localhost/1
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.29.3</center>
</body>
</html>
admin@i-03fc55a1924a48445:~$ curl http://localhost/2
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.29.3</center>
</body>
</html>
```

#### 5. check the nginx config

The nginx config is proxying to:

```
proxy_pass http://statichtml1.sadservers.local; // missing port 3000
proxy_pass http://statichtml2.sadservers.local; // missing port 3000
```

```
admin@i-03fc55a1924a48445:~/app$ cat default.conf 
    server {
        listen 80;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        location /1 {
            rewrite ^ / break;
            proxy_pass http://statichtml1.sadservers.local;
            proxy_connect_timeout   2s;
            proxy_send_timeout      2s;
            proxy_read_timeout      2s;
        }
        location /2 {
            rewrite ^ / break;
            proxy_pass http://statichtml2.sadservers.local;
            proxy_connect_timeout   2s;
            proxy_send_timeout      2s;
            proxy_read_timeout      2s;
        }
    }
```

#### 6. update and reload nginx config

```
admin@i-03fc55a1924a48445:~/app$ cat default.conf 
    server {
        listen 80;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }
        location /1 {
            rewrite ^ / break;
            proxy_pass http://statichtml1.sadservers.local:3000;
            proxy_connect_timeout   2s;
            proxy_send_timeout      2s;
            proxy_read_timeout      2s;
        }
        location /2 {
            rewrite ^ / break;
            proxy_pass http://statichtml2.sadservers.local:3000;
            proxy_connect_timeout   2s;
            proxy_send_timeout      2s;
            proxy_read_timeout      2s;
        }
    }
admin@i-03fc55a1924a48445:~/app$ docker exec nginx nginx -s reload
2025/12/02 03:43:20 [notice] 30#30: signal process started
```

Test:

```
admin@i-03fc55a1924a48445:~/app$ curl http://localhost/2
HelloWorld;2
admin@i-03fc55a1924a48445:~/app$ curl http://localhost/1
HelloWorld;1
```

---

## Notes

HTTP status codes:

* 502: Upstream server responded, but the response was invalid or unexpected.
* 504: Upstream server did not respond within the expected timeframe.
