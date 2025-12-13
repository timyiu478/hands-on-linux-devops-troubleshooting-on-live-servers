## "Sumé": Tied in a Knot

### Problem

Description: A DNS server running Knot DNS is serving the zone sadservers.internal (see ls /var/lib/knot/zones/), but users are reporting that they cannot access blog.sadservers.internal neither api.sadservers.internal. Your task is to diagnose and fix the DNS issues so the services become accessible.
You can manage Knot DNS with sudo knotc commands.

Note: the 203.0.113.0/24 range is part of TEST-NET-3, a block reserved by RFC 5737 for documentation and examples, making it a Bogon IP range.

IMPORTANT. Do not change the Nginx configurations under /opt/services/ for the solution to work.

Root (sudo) Access: True

Test: You are able to access the blog and the API services: curl blog.sadservers.internal returns Welcome to blog.sadservers.internal
curl api.sadservers.internal returns {"status": "ok", "service": "api.sadservers.internal"}

### Solution

#### 1. try to resolve the domain names

```
admin@ip-172-31-29-190:~$ nslookup blog.sadservers.internal
;; Got SERVFAIL reply from 127.0.0.1
Server:         127.0.0.1
Address:        127.0.0.1#53

** server can't find blog.sadservers.internal: SERVFAIL

admin@ip-172-31-29-190:~$ nslookup api.sadservers.internal
;; Got SERVFAIL reply from 127.0.0.1
Server:         127.0.0.1
Address:        127.0.0.1#53

** server can't find api.sadservers.internal: SERVFAIL
```

#### 2. check the server IPs

```
admin@ip-172-31-29-190:/var/lib/knot/zones$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                     NAMES
a7e52edb2b12   nginx:alpine   "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes   203.0.113.10:80->80/tcp   www-server
036d5515639d   nginx:alpine   "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes   203.0.113.20:80->80/tcp   api-server
```

#### 3. check the zone file

* No A record for api-server
* blog.sadservers.internal is an alias domain name of www-server but the A record for www-server is incorrect and there are some typos on this record e.g `CNAM` should be `CNAME` and `www.sadservers.internal` should be `www.sadservers.internal.`

```
admin@ip-172-31-29-190:/var/lib/knot/zones$ cat sadservers.internal.zone 
$ORIGIN sadservers.internal.
$TTL 3600

@       IN  SOA ns1.sadservers.internal. admin.sadservers.internal. (
            2024120901  ; Serial
            3600        ; Refresh
            1800        ; Retry
            604800      ; Expire
            86400 )     ; Minimum TTL

; Name servers
@       IN  NS  ns1.sadservers.internal.
ns1.     IN  A   10.1.11.22

www     IN  A   198.51.100.99
blog    IN  CNAM www.sadservers.internal
```

Updated zone file:

* Note we need to INCREASE the serial number of this file for the server reflects the changes
* How to reload the zone: `sudo knotc zone-reload sadservers.internal`

```
admin@ip-172-31-29-190:/var/lib/knot/zones$ cat sadservers.internal.zone 
$ORIGIN sadservers.internal.
$TTL 3600

@       IN  SOA ns1.sadservers.internal. admin.sadservers.internal. (
            2024120903  ; Serial
            3600        ; Refresh
            1800        ; Retry
            604800      ; Expire
            86400 )     ; Minimum TTL

; Name servers
@       IN  NS  ns1.sadservers.internal.
ns1.     IN  A   10.1.11.22

www     IN  A   203.0.113.10
api     IN  A   203.0.113.20
blog    IN  CNAME www.sadservers.internal.
```

Check:

```
admin@ip-172-31-29-190:/var/lib/knot/zones$ curl blog.sadservers.internal
Welcome to blog.sadservers.internal
admin@ip-172-31-29-190:/var/lib/knot/zones$ curl api.sadservers.internal
{"status": "ok", "service": "api.sadservers.internal"}
```
