## "Kampala": Strange Script Error

### Problem

A developer has been working on Linux deployment scripts on their machine and then transferred the files to a Linux server. However, when they try to execute the scripts, they encounter the mysterious error:

```
-bash: cannot execute: required file not found
```

The scripts appear to be syntactically correct, but something is preventing them from executing properly. The developer needs your help to identify and fix the issue so the deployment can proceed.

There are several script files in `/home/admin/deploy/` that need to be fixed before the deployment process can work correctly.

### Solution

#### 1. Convert the scripts from a DOS text file to a Unix text file

The `file` command can be used to check the file type:

We can see that the `setup.sh` file has `CRLF line terminators`, which indicates it is in DOS format. We need to convert it to Unix format.

```
admin@i-0395ec9d4efc0aa51:~/deploy$ file setup.sh 
setup.sh: Bourne-Again shell script, ASCII text executable, with CRLF line terminators
```

The `dos2unix` command can be used to convert the file:

```
admin@i-0395ec9d4efc0aa51:~/deploy$ dos2unix -- setup.sh 
dos2unix: converting file setup.sh to Unix format...
admin@i-0395ec9d4efc0aa51:~/deploy$ dos2unix -- deploy.sh 
dos2unix: converting file deploy.sh to Unix format...
admin@i-0395ec9d4efc0aa51:~/deploy$ dos2unix -- backup.sh 
dos2unix: converting file backup.sh to Unix format...
```

Ref: https://unix.stackexchange.com/questions/721844/linux-bash-shell-script-error-cannot-execute-required-file-not-found

#### 2. Run `mkdir` with sudo to create directories that require elevated permissions.

Directories that need to be created are owned by root:

```
admin@i-0395ec9d4efc0aa51:~$ ls -alt /
total 3145800
drwxr-xr-x   4 root root       4096 Sep 30 15:19 backup
drwxr-xr-x  19 root root       4096 Sep 30 15:05 .
drwxr-xr-x  12 root root       4096 Sep 29 14:07 var
drwxr-xr-x   4 root root       4096 Sep 29 14:07 opt
```

Updated scripts:

```
admin@i-0395ec9d4efc0aa51:~$ cat -n deploy/setup.sh 
     1  #!/bin/bash
     2
     3  echo "Setting up application environment..."
     4
     5  # Create necessary directories
     6  sudo mkdir -p /opt/app/logs
     7  sudo mkdir -p /opt/app/config
     8
     9  echo "Environment setup completed!"
    10  exit 0
admin@i-0395ec9d4efc0aa51:~$ cat -n deploy/deploy.sh 
     1  #!/bin/bash
     2
     3  echo "Starting deployment process..."
     4  echo "Checking system requirements..."
     5
     6  # Check if required directories exist
     7  if [ ! -d "/var/www" ]; then
     8      echo "Creating /var/www directory..."
     9      sudo mkdir -p /var/www
    10  fi
    11
    12  echo "Deployment script executed successfully!"
    13  exit 0
admin@i-0395ec9d4efc0aa51:~$ cat -n deploy/backup.sh 
     1  #!/bin/bash
     2
     3  echo "Creating backup of current deployment..."
     4
     5  BACKUP_DIR="/backup/$(date +%Y%m%d_%H%M%S)"
     6  sudo mkdir -p "$BACKUP_DIR"
     7
     8  echo "Backup created at: $BACKUP_DIR"
     9  echo "Backup script completed successfully!"
    10  exit 0
```
