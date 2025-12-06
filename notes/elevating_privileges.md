## "La Rinconada": Elevating privileges

### Problem

Description: You are logged in as the user "admin" without general "sudo" privileges.
The system administrator has granted you limited "sudo" access; this was intended to allow you to read log files.

Your mission is to find a way to exploit this limited sudo permission to gain a full root shell and read the secret file at /root/secret.txt
Copy the content of /root/secret.txt into the /home/admin/solution.txt file, for example: cat /root/secret.txt > /home/admin/solution.txt (the "admin" user must be able to read the file).

Root (sudo) Access: False

Test: As the user "admin", md5sum /home/admin/solution.txt returns 52a55258e4d530489ffe0cc4cf02030c (we also accept the hash of the same secret string without an ending newline).

### Solution

