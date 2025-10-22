## "Bondo": Split my pile!

### Problem

A developer wants to run a program that splits their pile of their data into compressed parts for efficient transport across their network. Unfortunately when the tool runs it never completes.

The application binary in question is called `bondo` located in `/home/admin/bondo`.

Run it, then debug and help the developer find the issue.

Root (sudo) Access: True

Test: Executing `/home/admin/bondo` as admin returns part files generation completed!.

The file `/etc/fstab` has not been modified and the solution would work on reboot.

### Solution

#### 1. Try to run the program

```bash
admin@i-0188124afe9197503:~$ ./bondo 
panic: open /srv/bondo/part-7de9eb669d8303f0b145815f69e3360.bin: no space left on device

goroutine 1 [running]:
main.generate({0x51d89e, 0xb})
        bondo/main.go:84 +0x4f1
main.main()
        bondo/main.go:44 +0xfd
```

#### 2. Check file system disk space

The filesystem has plenty of space (only 1% used, ~458MB available out of 496MB total)

```bash
admin@i-0188124afe9197503:~$ df -h /srv/bondo
Filesystem                        Size  Used Avail Use% Mounted on
/dev/mapper/vg--backup-lv--bondo  496M  2.2M  458M   1% /srv/bondo
```

#### 3. Check inode usage

inode exhaustion:

```
admin@i-0188124afe9197503:~$ df -i /srv/bondo
Filesystem                       Inodes IUsed IFree IUse% Mounted on
/dev/mapper/vg--backup-lv--bondo    512   512     0  100% /srv/bondo
```

502 files are created in the `/src/bondo` directory where each part file consumes one inode:

```
admin@i-0412422a5e16409be:~$ sudo find /srv/bondo -xdev | uniq -c | wc -l
502
```

- `-xdev`: Prevents `find` from descending into directories on other filesystems.

#### 4. expand the number of inodes to 500,000

```
admin@i-0412422a5e16409be:~$ sudo umount /srv/bondo
admin@i-0412422a5e16409be:~$ sudo mkfs.ext4 -N 500000 /dev/mapper/vg--backup-lv--bondo
mke2fs 1.47.2 (1-Jan-2025)
/dev/mapper/vg--backup-lv--bondo contains a ext4 file system
        last mounted on /srv/bondo on Thu Oct  2 07:52:13 2025
Proceed anyway? (y,N) y
Creating filesystem with 131072 4k blocks and 500224 inodes
Filesystem UUID: 99a9635c-e199-4689-8e14-e8f00268ce85
Superblock backups stored on blocks: 
        8600, 25800, 43000, 60200, 77400

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 

admin@i-0412422a5e16409be:~$ ./bondo 
part files generation completed!
```

#### 5. change the owner of the `~/bondo` binary to `root`

```
admin@i-0412422a5e16409be:~$ sudo chown root:root bondo
admin@i-0412422a5e16409be:~$ sudo chmod u+s bondo // setuid bit to run as file owner (root)
```

This is because the `sudo mount -a` command in the `agent/check.sh` script will change the ownership of the `/srv/bondo` directory to `root:root` on each run.

```
admin@i-0412422a5e16409be:~$ ls -alt /srv | grep bondo
drwxr-xr-x  3 root root 45056 Oct  2 08:14 bondo
admin@i-0412422a5e16409be:~$ cat agent/check.sh 
#!/bin/bash
# DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

sudo mount -a
expected="1149fdd0fbca15404ea0974e7c3d60bf"
actual=$(tail -1 /etc/fstab|md5sum 2>/dev/null | awk '{print $1}')

if [[ "$actual" != "$expected" ]]; then
  echo -n "NO"
  exit 0
fi


/home/admin/bondo  > /dev/null 2>&1
if [ $? -ne 0 ]; then
  echo -n "NO"
else
  echo -n "OK"
fi
```
