## "Suzhou": MongoDB replicas!

### Problem

Description: A new MongoDB replica set has been setup in the development environment trough /home/admin/app/rs0.js, however, a variety or errors are showing up when trying to bring it up. You should bring up all the replica servers, get them communicating to each other and make sure the replica set is working as it should.

The status of the first replica can be checked via systemctl status mongo1 same for the replicas mongo2 and mongo3. The logs are also in a separate file for each replica under the directory /var/log/mongodb. To initilize the replica set again: mongosh --file app/rs0.js

Note: The default configuration file /etc/mongo.conf is not the problem.

Root (sudo) Access: True

Test: mongosh --eval "rs.status()" | grep health returns the status of all the replicas

       health: 1,       health: 1,       health: 1, 

### Solution

#### 1. check the /home/admin/app/rs0.js

```
admin@i-020e5e8cab0b48a49:~$ cat  /home/admin/app/rs0.js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017" },
    { _id: 1, host: "mongo2:27018" },
    { _id: 2, host: "mongo3:27019" }
  ]
})
```

#### 2. check the status of the replicas

Mogon1 is running. Mongo2 and Mongo3 are failed to start.

```
admin@i-020e5e8cab0b48a49:~$ systemctl status mongo1
● mongo1.service - MongoDB mongo1
     Loaded: loaded (/etc/systemd/system/mongo1.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-02-20 07:32:45 UTC; 1min 28s ago
 Invocation: 8e3e4bf1c16e4848b6ef8a898001dab4
   Main PID: 761 (mongod)
      Tasks: 62 (limit: 503)
     Memory: 94.8M (peak: 143.9M, swap: 64.2M, swap peak: 65.2M)
        CPU: 3.531s
     CGroup: /system.slice/mongo1.service
             └─761 /usr/bin/mongod --config /etc/mongo1.conf

Feb 20 07:32:45 i-020e5e8cab0b48a49 systemd[1]: Started mongo1.service - MongoDB mongo1.
admin@i-020e5e8cab0b48a49:~$ systemctl status mongo2
× mongo2.service - MongoDB mongo2
     Loaded: loaded (/etc/systemd/system/mongo2.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Fri 2026-02-20 07:32:50 UTC; 1min 40s ago
   Duration: 989ms
 Invocation: 37cafc601eb84ca18afc7b42fac9e9cc
    Process: 1163 ExecStart=/usr/bin/mongod --config /etc/mongo2.conf (code=exited, status=1/FAILURE)
   Main PID: 1163 (code=exited, status=1/FAILURE)

Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo2.service: Main process exited, code=exited, status=1/FAILURE
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo2.service: Failed with result 'exit-code'.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo2.service: Scheduled restart job, restart counter is at 5.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo2.service: Start request repeated too quickly.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo2.service: Failed with result 'exit-code'.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: Failed to start mongo2.service - MongoDB mongo2.
admin@i-020e5e8cab0b48a49:~$ systemctl status mongo3
× mongo3.service - MongoDB mongo3
     Loaded: loaded (/etc/systemd/system/mongo3.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Fri 2026-02-20 07:32:50 UTC; 1min 46s ago
   Duration: 980ms
 Invocation: 931ce0bd73aa4bd9aeb4c693e4b379af
    Process: 1164 ExecStart=/usr/bin/mongod --config /etc/mongo3.conf (code=exited, status=1/FAILURE)
   Main PID: 1164 (code=exited, status=1/FAILURE)

Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo3.service: Main process exited, code=exited, status=1/FAILURE
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo3.service: Failed with result 'exit-code'.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo3.service: Scheduled restart job, restart counter is at 5.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo3.service: Start request repeated too quickly.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: mongo3.service: Failed with result 'exit-code'.
Feb 20 07:32:50 i-020e5e8cab0b48a49 systemd[1]: Failed to start mongo3.service - MongoDB mongo3.
```

#### 3. compare the configs of Mongo1 and Mongo3

```
admin@i-020e5e8cab0b48a49:~$ diff /etc/mongo1.conf /etc/mongo3.conf
2c2
<   dbPath: /var/lib/mongo1
---
>   dbPath: /var/lib/mongo3
6c6
<   path: /var/log/mongodb/mongo1.log
---
>   path: /var/log/mongodb/mongo3.log
8c8
<   port: 27017
---
>   port: 27019
13c13
<   keyFile: /etc/mongo1-keyfile
---
>   keyFile: /etc/mongo3-keyfile
```

#### 4. check the error logs of Mongo3 and Mongo2

Mongo3:

```
{"t":{"$date":"2026-02-20T07:55:31.794+00:00"},"s":"I",  "c":"ACCESS",   "id":20254,   "ctx":"main","msg":"Read security file failed","attr":{"error":{"code":30,"codeName":"InvalidPath","errmsg":"permissions on /etc/mongo3-keyfile are too open"}}}
admin@i-020e5e8cab0b48a49:~$ sudo cat /var/log/mongodb/mongo3.log | grep Error
{"t":{"$date":"2026-02-18T15:58:11.993+00:00"},"s":"F",  "c":"CONTROL",  "id":20575,   "ctx":"main","msg":"Error creating service context","attr":{"error":"Location5579201: Unable to acquire security key[s]"}}

```

Mongo2:

```
{"t":{"$date":"2026-02-20T07:55:31.795+00:00"},"s":"I",  "c":"ACCESS",   "id":20254,   "ctx":"main","msg":"Read security file failed","attr":{"error":{"code":30,"codeName":"InvalidPath","errmsg":"permissions on /etc/mongo2-keyfile are too open"}}}
{"t":{"$date":"2026-02-20T07:32:50.072+00:00"},"s":"F",  "c":"CONTROL",  "id":20575,   "ctx":"main","msg":"Error creating service context","attr":{"error":"Location5579201: Unable to acquire security key[s]"}}
```

#### 5. check the key files permissions and ownerships

By looking at /etc/mongo3.conf, we know that the /etc/mongo3-keyfile is the key file of the mongo3 process.

```
admin@i-020e5e8cab0b48a49:~$ ls -alt /etc/ | grep mongo
-rw-r--r--  1 root    root      234 Feb 18 15:57 mongo3.conf
-rw-r--r--  1 root    root      234 Feb 18 15:57 mongo2.conf
-rw-r--r--  1 root    root      234 Feb 18 15:57 mongo1.conf
-rw-r--r--  1 root    root       14 Feb 18 15:57 mongo3-keyfile
-rw-r--r--  1 root    root       14 Feb 18 15:57 mongo2-keyfile
-rw-------  1 mongodb mongodb    14 Feb 18 15:57 mongo1-keyfile
-rw-r--r--  1 root    root      316 Feb 18 15:57 mongod.conf
-rw-------  1 mongodb mongodb    14 Feb 18 15:57 mongo-keyfile
```

#### 6. change the key files permissions and ownerships such that they follow mongo1-keyfile so that the key files will not too open

```
admin@i-020e5e8cab0b48a49:~$ sudo chown mongodb /etc/mongo2-keyfile && sudo chgrp mongodb /etc/mongo2-keyfile && sudo chmod 600 /etc/mongo2-keyfile
admin@i-020e5e8cab0b48a49:~$ sudo chown mongodb /etc/mongo3-keyfile && sudo chgrp mongodb /etc/mongo3-keyfile && sudo chmod 600 /etc/mongo3-keyfile
admin@i-020e5e8cab0b48a49:~$ ls -alt /etc/ | grep mongo
-rw-r--r--  1 root    root      234 Feb 18 15:57 mongo3.conf
-rw-r--r--  1 root    root      234 Feb 18 15:57 mongo2.conf
-rw-r--r--  1 root    root      234 Feb 18 15:57 mongo1.conf
-rw-------  1 root    root       14 Feb 18 15:57 mongo3-keyfile
-rw-------  1 mongodb mongodb    14 Feb 18 15:57 mongo2-keyfile
-rw-------  1 mongodb mongodb    14 Feb 18 15:57 mongo1-keyfile
-rw-r--r--  1 root    root      316 Feb 18 15:57 mongod.conf
-rw-------  1 mongodb mongodb    14 Feb 18 15:57 mongo-keyfile
```

#### 7. restart the replicas

```
admin@i-020e5e8cab0b48a49:~$ sudo systemctl restart mongo3
admin@i-020e5e8cab0b48a49:~$ sudo systemctl restart mongo2
admin@i-020e5e8cab0b48a49:~$ sudo systemctl status mongo3
● mongo3.service - MongoDB mongo3
     Loaded: loaded (/etc/systemd/system/mongo3.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-02-20 07:59:58 UTC; 9s ago
 Invocation: e8267a28cbbb4aef965513170c51f894
   Main PID: 1483 (mongod)
      Tasks: 64 (limit: 503)
     Memory: 51.9M (peak: 138.5M, swap: 40.4M, swap peak: 40.4M)
        CPU: 1.367s
     CGroup: /system.slice/mongo3.service
             └─1483 /usr/bin/mongod --config /etc/mongo3.conf

Feb 20 07:59:58 i-020e5e8cab0b48a49 systemd[1]: Started mongo3.service - MongoDB mongo3.
admin@i-020e5e8cab0b48a49:~$ sudo systemctl status mongo2
● mongo2.service - MongoDB mongo2
     Loaded: loaded (/etc/systemd/system/mongo2.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-02-20 08:00:00 UTC; 10s ago
 Invocation: 42da1cc94a704f2eb96493496018d60b
   Main PID: 1556 (mongod)
      Tasks: 64 (limit: 503)
     Memory: 107.7M (peak: 109.4M, swap: 8.8M, swap peak: 8.8M)
        CPU: 1.431s
     CGroup: /system.slice/mongo2.service
             └─1556 /usr/bin/mongod --config /etc/mongo2.conf

Feb 20 08:00:00 i-020e5e8cab0b48a49 systemd[1]: Started mongo2.service - MongoDB mongo2.
```

#### 8. change the hosts in rs0.js to use 127.0.0.1 consistently

```
admin@i-020e5e8cab0b48a49:~$ cat /home/admin/app/rs0.js
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "127.0.0.1:27017" },
    { _id: 1, host: "127.0.0.1:27018" },
    { _id: 2, host: "127.0.0.1:27019" }
  ]
})
```

#### 9. initialise the replica set

ref: https://www.mongodb.com/docs/manual/tutorial/deploy-replica-set/

```
admin@i-020e5e8cab0b48a49:~$ mongosh --port 27017 --file /home/admin/app/rs0.js
admin@i-020e5e8cab0b48a49:~$ mongosh --eval "rs.status()" | grep health
      health: 1,
      health: 1,
      health: 1,
```
