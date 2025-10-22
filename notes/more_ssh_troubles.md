## "Nuuk": More SSH Troubles

### Problem

SSH seems broken in this server. The user admin has an id_ed25519 SSH key pair in their ~/.ssh directory with the public key in ~/.ssh/authorized_keys but ssh 127.0.0.1 won't work.

Test: You can ssh locally, i.e. `ssh admin@127.0.0.1` works.

### Solution

#### 1. try `ssh admin@127.0.0.1`

`hostkeys_find_by_key_hostfile: hostkeys_foreach failed for /home/admin/.ssh/known_hosts: Permission denied`

```
$ ssh admin@127.0.0.1
hostkeys_find_by_key_hostfile: hostkeys_foreach failed for /home/admin/.ssh/known_hosts: Permission denied
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:TwKkpzk6C7ePQF8UrmP/72hBGxmj4/0b6iNUSMrGP78.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

#### 2. check `~/.known_hosts`

No permission to access `~/.known_hosts`.

```
admin@i-03177b79cc48247ad:~$ ls -alt ~/.ssh
ls: cannot open directory '/home/admin/.ssh': Permission denied
admin@i-03177b79cc48247ad:~$ ls -alt ~/
total 32
drwxr-xr-x 2 admin root  4096 Oct 21 17:27 agent
d--------- 2 admin admin 4096 Oct 21 17:27 .ssh
```

#### 3. `chmod 755 ~/.ssh`

```
admin@i-03177b79cc48247ad:~$ sudo chmod 755 ~/.ssh
admin@i-03177b79cc48247ad:~$ cd ~/.ssh
admin@i-03177b79cc48247ad:~/.ssh$ ls
authorized_keys  id_ed25519  id_ed25519.pub
```

#### 4. Try ssh

Success.

```
admin@i-03177b79cc48247ad:~/.ssh$ ssh admin@127.0.0.1
The authenticity of host '127.0.0.1 (127.0.0.1)' can't be established.
ED25519 key fingerprint is SHA256:TwKkpzk6C7ePQF8UrmP/72hBGxmj4/0b6iNUSMrGP78.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '127.0.0.1' (ED25519) to the list of known hosts.
Linux i-03177b79cc48247ad 6.12.41+deb13-cloud-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.41-1 (2025-08-12) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Oct 21 17:27:50 2025 from 24.52.248.51
```
