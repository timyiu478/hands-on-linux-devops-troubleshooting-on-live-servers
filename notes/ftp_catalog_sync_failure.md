## "Edinburgh": FTP catalog sync failure

### Problem

Description: The warehouse API on this host should answer on http://127.0.0.1:9178/ with a body containing OK.

It depends on a catalog file pulled from the internal FTP mirror at 127.0.0.1 running vsftpd (user edinburgh).

The sync job at /home/admin/edinburgh-sync.sh is failing; edinburgh-sync is in a failed state and the API never comes up healthy.

Fix the sync so the catalog is downloaded and the API works again.

Root (sudo) Access: True

Test: /var/lib/edinburgh/catalog.txt exists with the correct inventory data, the service edinburgh-api is active, and curl http://127.0.0.1:9178/ returns a response whose body contains OK.

### Solution

1. run the script

Problems:

* (1) the ftp command uses active mode if passive mode is explicitly disabled via passive off.
    * https://www.jscape.com/blog/active-v-s-passive-ftp-simplified
    * Fix: switch the FTP connection to passive mode so that your client establishes all data connections outbound, which safely avoids firewall blocks on localhost.
* (2) No `/var/lib/edinburgh/catalog.txt.tmp` file

```
dmin@i-03837a9a2cb983d45:~$ ./edinburgh-sync.sh 
Connected to 127.0.0.1.
220 Edinburgh FTP mirror (passive only)
331 Password required
230 Login successful
Remote system type is UNIX.
Using binary mode to transfer files.
200 Type set to I
Passive mode: off; fallback to active mode: off.
local: /var/lib/edinburgh/catalog.txt.tmp remote: catalog.txt
500 PORT/EPRT not allowed; use PASV
ftp: Can't bind for data connection: Address already in use
mv: cannot stat '/var/lib/edinburgh/catalog.txt.tmp': No such file or directory
```

2. fix the script

```
admin@i-03837a9a2cb983d45:~$ cat edinburgh-sync.sh 
#!/bin/bash
# Sync inventory catalog from the internal FTP mirror.
set -euo pipefail

HOST="127.0.0.1"
USER="edinburgh"
PASS="edinburgh-catalog"
REMOTE="catalog.txt"
DEST="/var/lib/edinburgh/catalog.txt"


# Note the '-p' flag added here for passive mode
sudo ftp -invp "$HOST" <<EOF
user ${USER} ${PASS}
binary
get ${REMOTE} ${DEST}
bye
EOF
```

3. restart api service

```
admin@i-03837a9a2cb983d45:~$ sudo systemctl status edinburgh-api.service 
○ edinburgh-api.service - Edinburgh warehouse API
     Loaded: loaded (/etc/systemd/system/edinburgh-api.service; enabled; preset: enabled)
     Active: inactive (dead)

May 19 12:18:31 i-03837a9a2cb983d45 systemd[1]: Dependency failed for edinburgh-api.service - Edinburgh warehouse API.
May 19 12:18:31 i-03837a9a2cb983d45 systemd[1]: edinburgh-api.service: Job edinburgh-api.service/start failed with result 'dependency'.
admin@i-03837a9a2cb983d45:~$ sudo systemctl restart edinburgh-api.service 
admin@i-03837a9a2cb983d45:~$ sudo systemctl status edinburgh-api.service 
● edinburgh-api.service - Edinburgh warehouse API
     Loaded: loaded (/etc/systemd/system/edinburgh-api.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-05-19 12:27:38 UTC; 2s ago
 Invocation: 6ca23d2eb1a2499ca74478d715aad744
   Main PID: 1351 (python3)
      Tasks: 1 (limit: 503)
     Memory: 8.8M (peak: 8.9M)
        CPU: 72ms
     CGroup: /system.slice/edinburgh-api.service
             └─1351 python3 /usr/local/bin/edinburgh-api.py

May 19 12:27:38 i-03837a9a2cb983d45 systemd[1]: Started edinburgh-api.service - Edinburgh warehouse API.
admin@i-03837a9a2cb983d45:~$ ~/agent/check.sh 
```
