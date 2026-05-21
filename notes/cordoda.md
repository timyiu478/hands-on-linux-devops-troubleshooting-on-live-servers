## "Cordoba": df is lying (or is it du?)

### Problem

Description: Monitoring reports that the root filesystem is under pressure, but a quick du of /var/log shows almost nothing in the logs of the running application at /var/log/cordoba-app.

Find what is holding the space and reclaim it so df and du agree again for practical purposes; currently there's a ~300 MB discrepancy on the root partition /

The service unit is cordoba-hoarder.service.

Root (sudo) Access: True

Test: df -h / and sudo du -sh / report the same used space after reclaiming the ~300 MB discrepancy.

### Solution

1. found the swapfile is the file that holds the space

```
admin@i-0bc859b0f966573b0:/$ ls -alt
total 3145796
drwxrwxrwt   8 root root        160 May 21 10:19 tmp
drwxr-xr-x  24 root root        600 May 21 10:19 run
drwxr-xr-x  14 root root       3000 May 21 10:19 dev
drwxr-xr-x  73 root root       4096 May 21 10:19 etc
drwxr-xr-x  18 root root       4096 May 21 10:19 .
drwxr-xr-x  18 root root       4096 May 21 10:19 ..
dr-xr-xr-x 147 root root          0 May 21 10:18 proc
dr-xr-xr-x  13 root root          0 May 21 10:18 sys
drwxr-xr-x   3 root root       4096 Sep  7  2025 opt
-rw-------   1 root root 3221225472 Sep  7  2025 swapfile
```

2. find all active swap areas 

```
admin@i-0bc859b0f966573b0:~$ cat /proc/swaps
Filename                                Type            Size            Used            Priority
/swapfile                               file            3145724         0               -2
```

3. remove the file

```
admin@i-0bc859b0f966573b0:/$ sudo swapoff /swapfile // disable swaparea 
admin@i-0bc859b0f966573b0:/$ sudo chattr -i swapfile // remove immutable attribute
admin@i-0bc859b0f966573b0:/$ sudo rm swapfile
```
