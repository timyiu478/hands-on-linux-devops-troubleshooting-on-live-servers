## "Annapurna": High privileges

### Problem

Description: You are logged in as the user admin .

You have been tasked with auditing the admin user privileges in this server; "admin" should not have sudo (root) access.

Exploit this server so you as the admin user can read the file /root/secret.txt
Save the content of /root/secret.txt to the file /home/admin/mysolution.txt , for example: echo "secret" > ~/mysolution.txt

Root (sudo) Access: False

Test: Running md5sum /home/admin/mysolution.txt returns . (We also accept the md5sum of the same file without a newline at the end).

### Solution


