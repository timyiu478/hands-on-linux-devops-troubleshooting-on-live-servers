## "Bologna": counting ELB 5xx errors

### Problem

Description: Operations handed you a classic AWS Elastic Load Balancer access
log at /home/admin/elb.log. Each line is one request. Fields are
space-separated; the quoted HTTP request starts at field 12, so the numeric
fields before it are fixed-width columns.

Field 8 is the ELB status code and field 9 is the backend status code returned
by the target instance. Count how many log lines have a backend status code in
the 5xx range (500 through 599). Write that integer — digits only — to
/home/admin/solution.txt. For example: echo 42 > ~/solution.txt

The log mixes successful responses, redirects, client errors, and server
errors; only backend 5xx responses count toward your answer.

Root (sudo) Access: True

Test: The MD5 checksum of your answer file md5sum /home/admin/solution.txt is
b73ce398c39f506af761d2277d853a92 (we also accept the correct count with
a trailing newline in the file).

### Solution

```
admin@i-0a4599ea5e6681334:~$ cat /home/admin/elb.log | awk '{print $9}' | grep -E "^5[0-9]{2}$" | wc -w > /home/admin/solution.txt
admin@i-0a4599ea5e6681334:~$ ./agent/check.sh 
OKadmin@i-0a4599ea5e6681334:~$
```
