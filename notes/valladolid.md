## "Valladolid": Cleaner not cleaning 

### Problem

The systemd service log-cleaner.service is supposed to be run manually (not a timer or cron job) and delete log files older than 7 days in the /var/log/app directory.

The service runs successfully (exit code 0), but no logs are ever deleted.

Fix the service and/or the script so that old_data.log (older than 7 days) is deleted, but recent_data.log is preserved.

If you accidentally delete the wrong files while debugging, run ~/reset_logs.sh to restore them.

### Solution


1. check the log cleaner script

Problem: The LOG_DIR variable is defined but never used/the find command looks the files start from its current working directory => the find command never looks inside /var/log/app

```
admin@i-07d6b8298bf666f1f:~$ systemctl status log-cleaner
○ log-cleaner.service - Daily Log Cleaner
     Loaded: loaded (/etc/systemd/system/log-cleaner.service; static)
     Active: inactive (dead)
admin@i-07d6b8298bf666f1f:~$ systemctl cat log-cleaner
# /etc/systemd/system/log-cleaner.service
[Unit]
Description=Daily Log Cleaner

[Service]
Type=oneshot
ExecStart=/bin/bash /opt/scripts/log-cleaner.sh
WorkingDirectory=/root
admin@i-07d6b8298bf666f1f:~$ cat /opt/scripts/log-cleaner.sh 
#!/bin/bash

# Configuration
LOG_DIR="/var/log/app"
DAYS=7

echo "Starting Cleanup..."

find . -maxdepth 1 -name "*.log" -type f -mtime -7 -print -delete

echo "Cleanup finished."
```

2. fix the find command

```
find "${LOG_DIR}" -maxdepth 1 -name "*.log" -type f -mtime +${DAYS} -print -delete
```

3. test

```
admin@i-07d6b8298bf666f1f:~$ sudo systemctl restart log-cleaner
admin@i-07d6b8298bf666f1f:~$ ls /var/log/app
recent_data.log
```
