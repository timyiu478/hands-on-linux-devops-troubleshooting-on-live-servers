## "Nerdearla Buenos Aires": Troubleshoot "A" no se conecta con "B"

### Problem

Description: There is a web server (Caddy) on HTTP port 80, but `curl http://127.0.0.1` is not working. Find out what's wrong and make the necessary fixes so the web server can return a URL.

Note: As a limitation, the file `/home/admin/db_connector.py` must not be modified for the problem to be considered resolved.

The web server must respond at the IP address 127.0.0.1, not just "localhost".

Root (sudo) Access: True

Test: The command `curl http://127.0.0.1` returns a URL.

The "Check My Solution" button runs the script `/home/admin/agent/check.sh`, which you can view and execute.

### Solution

#### 1. read the source code of db_connector.py

* the server listens on port 5050

```
admin@ip-10-1-11-24:~$ cat db_connector.py 
# script that retrieves secret from postgres database
import socket
import psycopg2
import logging
import sys
import os
from dotenv import load_dotenv, find_dotenv


# load secrets env var file .env
load_dotenv(find_dotenv())

# debugging
logging.basicConfig(level=logging.DEBUG,
                    handlers=[
                        logging.StreamHandler(sys.stdout),
                    ])

logging.debug("Starting db_connector...")


# Database connection parameters
dbname = os.getenv('DBNAME')
user = os.getenv('DBUSER')
password = os.getenv('DBPASSWORD')
host = os.getenv('DBHOST')
port = os.getenv('DBPORT')

db_params = {
    'dbname': dbname,
    'user': user,
    'password': password,
    'host': host,
    'port': port
}

# Create a socket and listen on port 5050
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(('0.0.0.0', 5050))
server_socket.listen(1)

logging.debug("Listening on port 5050...")

while True:
    client_socket, addr = server_socket.accept()
    logging.debug(f"Connection from {addr}")

    try:
        # Read the HTTP request
        request = client_socket.recv(1024).decode()
        logging.debug(f"Received request: {request}")

        # Connect to the PostgreSQL database
        conn = psycopg2.connect(**db_params)
        cursor = conn.cursor()

        # Execute the query
        cursor.execute("SELECT secret FROM secrets WHERE id=1;")
        result = cursor.fetchone()

        # Prepare the HTTP response
        if result:
            secret = result[0]
            response_body = secret.encode()
            response = f"HTTP/1.1 200 OK\r\nContent-Length: {len(response_body)}\r\n\r\n".encode() + response_body
        else:
            response = b"HTTP/1.1 404 Not Found\r\nContent-Length: 18\r\n\r\nNo secret found."

        # Send the HTTP response
        client_socket.sendall(response)

    except Exception as e:
        error_message = f"An error occurred: {e}"
        logging.error(error_message)
        response = f"HTTP/1.1 500 Internal Server Error\r\nContent-Length: {len(error_message)}\r\n\r\n{error_message}".encode()
        client_socket.sendall(response)

    finally:
        # Clean up
        if 'cursor' in locals():
            cursor.close()
        if 'conn' in locals():
            conn.close()
        client_socket.close()
```

#### 2. try to curl 127.0.0.1:5050

We can get the error response of the server.

The possible reasons why the server return error:

* the database parameters are wrong 
* the database is not running

```
admin@ip-10-1-11-24:~$ curl 127.0.0.1:5050
An error occurred: could not connect to server: Connection refused
        Is the server running on host "127.0.0.1" and accepting
        TCP/IP connections on port 5433?
```

#### 4. check the Caddy config and the status of caddy server

key log message: Caddyfile input is not formatted; run 'caddy fmt --overwrite' to fix inconsistencies

```
admin@ip-10-1-11-24:/etc/caddy$ cat Caddyfile 
# The Caddyfile is an easy way to configure your Caddy web server.
#
# Unless the file starts with a global options block, the first
# uncommented line is always the address of your site.
#
# To use your own domain name (with automatic HTTPS), first make
# sure your domain's A/AAAA DNS records are properly pointed to
# this machine's public IP, then replace ":80" below with your
# domain name.

:80 {
        # Set this path to your site's directory.
# root * /usr/share/caddy

        # Enable the static file server.
    reverse_proxy localhost:5050

        # Another common task is to set up a reverse proxy:
        # reverse_proxy localhost:8080

        # Or serve a PHP site through php-fpm:
        # php_fastcgi localhost:9000
}

# Refer to the Caddy docs for more information:
# https://caddyserver.com/docs/caddyfile
admin@ip-10-1-11-24:/etc/caddy$ systemctl status caddy
● caddy.service - Caddy
     Loaded: loaded (/lib/systemd/system/caddy.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2026-02-20 09:58:12 UTC; 22min ago
       Docs: https://caddyserver.com/docs/
   Main PID: 609 (caddy)
      Tasks: 8 (limit: 521)
     Memory: 38.2M
        CPU: 189ms
     CGroup: /system.slice/caddy.service
             └─609 /usr/bin/caddy run --environ --config /etc/caddy/Caddyfile

Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"warn","ts":1771581492.5906177,"msg":"Caddyfile input is not formatted; run 'caddy fmt --overwrite' to fix inconsistencies",">
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"info","ts":1771581492.5919764,"logger":"admin","msg":"admin endpoint started","address":"localhost:2019","enforce_origin":fa>
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"warn","ts":1771581492.5934756,"logger":"http.auto_https","msg":"server is listening only on the HTTP port, so no automatic H>
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"info","ts":1771581492.593845,"logger":"tls.cache.maintenance","msg":"started background certificate maintenance","cache":"0x>
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"info","ts":1771581492.5990047,"logger":"http.log","msg":"server running","name":"srv0","protocols":["h1","h2","h3"]}
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"info","ts":1771581492.6034136,"msg":"autosaved config (load with --resume flag)","file":"/var/lib/caddy/.config/caddy/autosa>
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"info","ts":1771581492.6036892,"msg":"serving initial configuration"}
Feb 20 09:58:12 ip-10-1-11-24 systemd[1]: Started Caddy.
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"info","ts":1771581492.6157632,"logger":"tls","msg":"cleaning storage unit","storage":"FileStorage:/var/lib/caddy/.local/shar>
Feb 20 09:58:12 ip-10-1-11-24 caddy[609]: {"level":"info","ts":1771581492.61647,"logger":"tls","msg":"finished cleaning storage units"}
```

#### 5. format the Caddyfile

key log message: adapted config to JSON

```
admin@ip-10-1-11-17:~$ sudo caddy fmt --overwrite /etc/caddy/Caddyfile 
admin@ip-10-1-11-17:~$ sudo systemctl restart caddy
admin@ip-10-1-11-17:~$ systemctl status caddy
● caddy.service - Caddy
     Loaded: loaded (/lib/systemd/system/caddy.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2026-03-30 09:40:45 UTC; 7s ago
       Docs: https://caddyserver.com/docs/
   Main PID: 910 (caddy)
      Tasks: 7 (limit: 521)
     Memory: 10.7M
        CPU: 76ms
     CGroup: /system.slice/caddy.service
             └─910 /usr/bin/caddy run --environ --config /etc/caddy/Caddyfile

Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"info","ts":1774863645.1231368,"msg":"adapted config to JSON","adapter":"caddyfile"}
Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"info","ts":1774863645.126954,"logger":"admin","msg":"admin endpoint started","address":"localhost:2019","enforce_origin":fal>
Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"warn","ts":1774863645.1272924,"logger":"http.auto_https","msg":"server is listening only on the HTTP port, so no automatic H>
Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"info","ts":1774863645.127502,"logger":"tls.cache.maintenance","msg":"started background certificate maintenance","cache":"0x>
Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"info","ts":1774863645.127689,"logger":"http.log","msg":"server running","name":"srv0","protocols":["h1","h2","h3"]}
Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"info","ts":1774863645.1279268,"msg":"autosaved config (load with --resume flag)","file":"/var/lib/caddy/.config/caddy/autosa>
Mar 30 09:40:45 ip-10-1-11-17 systemd[1]: Started Caddy.
Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"info","ts":1774863645.1289387,"msg":"serving initial configuration"}
Mar 30 09:40:45 ip-10-1-11-17 caddy[910]: {"level":"info","ts":1774863645.1320727,"logger":"tls","msg":"storage cleaning happened too rec
```

#### 6. check if postgresql is running

Nope.

```
admin@ip-10-1-11-17:/var/lib/postgresql/13$ systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
```

#### 7. run the postgresql

```
admin@ip-10-1-13-118:/etc/postgresql/13/main$ sudo cat /usr/lib/systemd/system/postgresql.service
[Unit]
Description=PostgreSQL database server
After=network-online.target

[Service]
Type=notify
User=postgres
Group=postgres
ExecStart=/usr/lib/postgresql/13/bin/postgres -D /etc/postgresql/13/main
KillMode=mixed
KillSignal=SIGINT
TimeoutSec=infinity

[Install]
WantedBy=multi-user.target
admin@ip-10-1-13-51:/var/lib/postgresql/13/main$ sudo systemctl start postgresql.service 
```

### 8. check if secrets table exists in nerdearla database

Yes.

```
admin@ip-10-1-13-51:/var/lib/postgresql/13/main$ sudo -u postgres psql
psql (13.16 (Debian 13.16-0+deb11u1))
Type "help" for help.

postgres-# \c nerdearla 
You are now connected to database "nerdearla" as user "postgres".
nerdearla-# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   
-----------+----------+----------+---------+---------+-----------------------
 nerdearla | usuario  | UTF8     | C.UTF-8 | C.UTF-8 | 
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 | 
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
(4 rows)

nerdearla-# select * from secrets
nerdearla-# \dt+
                            List of relations
 Schema |  Name   | Type  |  Owner   | Persistence | Size  | Description 
--------+---------+-------+----------+-------------+-------+-------------
 public | secrets | table | postgres | permanent   | 16 kB | 
(1 row)
```

#### 9. check the .env

DBPORT should be 5432 instead of 5433

```
admin@ip-10-1-11-17:~$ cat .env
DBNAME="nerdearla"
DBUSER="usuario"
DBPASSWORD="password"
DBHOST="127.0.0.1"
DBPORT="5433
```

change it to 5432 and then restart db_connector


#### 9. check if db_connector can get secret from postgres

Yes.

```
admin@ip-10-1-13-51:~$ sudo systemctl restart db_connector.service
admin@ip-10-1-13-51:~$ curl 127.0.0.1:5050
https://sadservers.com/nerdearla2024
```

#### 9. test caddy server

curl cant reach caddy server:

```
admin@ip-10-1-11-173:~$ curl -vvv http://127.0.0.1
*   Trying 127.0.0.1:80...
^C
admin@ip-10-1-11-173:~$ sudo ss -tupln | grep 80
udp   UNCONN 0      0      [fe80::1a:4aff:feff:db7f]%ens5:546          [::]:*    users:(("dhclient",pid=512,fd=8))
tcp   LISTEN 0      4096                                *:8080            *:*    users:(("gotty",pid=608,fd=6))   
tcp   LISTEN 0      4096                                *:80              *:*    users:(("caddy",pid=883,fd=7))   
```

#### 9. delete iptable rule that block input tcp packet to port 80

```
admin@ip-10-1-11-173:~$ sudo iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
DROP       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:80

Chain FORWARD (policy DROP)
target     prot opt source               destination         
DOCKER-USER  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-1  all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain DOCKER (1 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination         
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
target     prot opt source               destination         
DROP       all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0 
...
admin@ip-10-1-11-173:~$ sudo iptables -D INPUT 1
```
