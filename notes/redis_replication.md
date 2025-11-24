## "Tunis": Redis Replication Problem

### Problem

A Redis master-replica setup is running on this server, with the master on port 6379 and the replica on port 6380. Both instances show as "connected" when you check their status, but data synchronization has silently broken.

Recent writes to the master don't appear on the replica, even though there are no obvious errors in the logs and both Redis instances appear healthy.

Fix the replication issues so that data written to the master (port 6379) immediately appears on the replica (port 6380) without data loss.

Master: localhost:6379
Replica: localhost:6380
Password: masterpass123

A helper test script is available at /home/admin/test_replication.sh

### Solution

#### 1. check the replica's config

* The authentication password and the master IP are wrong

```bash
admin@i-02e6c7f99546a49e9:~$ sudo cat /etc/redis/replica/redis.conf
# Redis Replica Configuration (Port 6380)
port 6380
bind 127.0.0.1
dir /var/lib/redis/replica

replicaof 127.0.0.2 6379
masterauth "wrongpassword456"
replica-read-only no

# Persistence
save 900 1
save 300 10
save 60 10000

# Logging
logfile /var/log/redis/replica.log
loglevel notice
```

Fix: change password to `masterpass123` and ip to `127.0.0.1`

```
admin@i-02e6c7f99546a49e9:~$ sudo vim /etc/redis/replica/redis.conf
admin@i-02e6c7f99546a49e9:~$ sudo systemctl restart redis-replica.service 
admin@i-02e6c7f99546a49e9:~$ sudo systemctl status redis-replica.service 
● redis-replica.service - Redis Replica Instance
     Loaded: loaded (/etc/systemd/system/redis-replica.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-11-24 08:40:04 UTC; 8s ago
 Invocation: 0b629e6f75dc4293a7f3c0ce59781a7c
   Main PID: 1481 (redis-server)
      Tasks: 6 (limit: 503)
     Memory: 3.3M (peak: 3.5M)
        CPU: 51ms
     CGroup: /system.slice/redis-replica.service
             └─1481 "/usr/bin/redis-server 127.0.0.1:6380"

Nov 24 08:40:04 i-02e6c7f99546a49e9 systemd[1]: Started redis-replica.service - Redis Replica Instance.
```

#### 2. check the replica logs

Synced successfully.

```
1621:S 24 Nov 2025 08:47:02.144 * MASTER <-> REPLICA sync: Flushing old data
1621:S 24 Nov 2025 08:47:02.144 * Loading RDB produced by version 8.0.2
1621:S 24 Nov 2025 08:47:02.144 * RDB age 0 seconds
1621:S 24 Nov 2025 08:47:02.144 * RDB memory usage when created 0.85 Mb
1621:S 24 Nov 2025 08:47:02.144 * Done loading RDB, keys loaded: 3, keys expired: 0.
1621:S 24 Nov 2025 08:47:02.144 * MASTER <-> REPLICA sync: Finished with success
1621:S 24 Nov 2025 08:47:02.145 * MASTER <-> REPLICA sync: Starting to stream replication buffer into the db (0 bytes).
1621:S 24 Nov 2025 08:47:02.145 * MASTER <-> REPLICA sync: Successfully streamed replication buffer into the db (0 bytes in total)
admin@i-02e6c7f99546a49e9:~$ ./agent/check.sh 
OKadmin@i-02e6c7f99546a49e9:~$ 
```
