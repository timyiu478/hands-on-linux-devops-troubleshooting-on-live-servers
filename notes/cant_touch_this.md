## "Kortenberg": Can't touch this!

### Problem

Description: Is "All I want for Christmas is you" already everywhere?. A bit unrelated, someone messed up the permissions in this server, the admin user can't list new directories and can't write into new files. Fix the issue.
NOTE: Besides solving the problem in your current admin shell session, you need to fix it permanently, as in a new login shell for user "admin" (like the one initiated by the scenario checker) should have the problem fixed as well.

Root (sudo) Access: True

Test: The admin user in a separate Bash login session should be able to create a new directory in your /home/admin directory, as well as being able to create a file into this new directory and add text into the new file.

---

### Solution

#### 1. try to create a new directory `abc`

No one has permission on newly created directory.

```
admin@i-038be5ca7a3896dec:~$ mkdir abc
admin@i-038be5ca7a3896dec:~$ ls -alt
total 36
drwx------ 6 admin admin 4096 Dec  6 00:40 .
d--------- 2 admin admin 4096 Dec  6 00:40 abc
drwxrwxrwx 2 admin admin 4096 Dec  1 00:31 agent
-rw-r--r-- 1 admin admin  796 Dec  1 00:31 .profile
-rw-r--r-- 1 admin admin    0 Sep  7 16:31 .sudo_as_admin_successful
drwx------ 3 admin admin 4096 Sep  7 16:31 .ansible
drwx------ 2 admin admin 4096 Sep  7 16:29 .ssh
drwxr-xr-x 3 root  root  4096 Sep  7 16:29 ..
-rw-r--r-- 1 admin admin  220 Jul 30 19:28 .bash_logout
-rw-r--r-- 1 admin admin 3526 Jul 30 19:28 .bashrc
```

#### 2. check umask

> umask is a shell command that reports or sets the mask value that limits the file permissions for newly created files in many Unix and Unix-like file systems. https://en.wikipedia.org/wiki/Umask

umask is the culprit.

```
admin@i-038be5ca7a3896dec:~$ umask
0777
admin@i-038be5ca7a3896dec:~$ umask -S
u=,g=,o=
```

#### 3. update umask

> .profile: executed by the command interpreter for login shells.

```
admin@i-038be5ca7a3896dec:~$ echo "umask 0022" >> ~/.profile
admin@i-038be5ca7a3896dec:~$ echo "umask 0022" >> ~/.bashrc
```

Check:

The newly creatd directory `123` has "0755 permission" now.

```
admin@i-038be5ca7a3896dec:~$ source ~/.bashrc
admin@i-038be5ca7a3896dec:~$ umask -S
u=rwx,g=rx,o=rx
admin@i-038be5ca7a3896dec:~$ mkdir 123
admin@i-038be5ca7a3896dec:~$ ls -alt
total 36
drwx------ 6 admin admin 4096 Dec  6 00:56 .
drwxr-xr-x 2 admin admin 4096 Dec  6 00:56 123
-rw-r--r-- 1 admin admin 3537 Dec  6 00:56 .bashrc
-rw-r--r-- 1 admin admin  807 Dec  6 00:56 .profile
drwxrwxrwx 2 admin admin 4096 Dec  1 00:31 agent
-rw-r--r-- 1 admin admin    0 Sep  7 16:31 .sudo_as_admin_successful
drwx------ 3 admin admin 4096 Sep  7 16:31 .ansible
drwx------ 2 admin admin 4096 Sep  7 16:29 .ssh
drwxr-xr-x 3 root  root  4096 Sep  7 16:29 ..
-rw-r--r-- 1 admin admin  220 Jul 30 19:28 .bash_logout
```
