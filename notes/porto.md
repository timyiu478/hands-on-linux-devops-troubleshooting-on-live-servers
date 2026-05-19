## "Porto": Port audit without net tools

### Problem

Description: The security team removed common network recon utilities from this host. Your job is to determine which TCP ports on localhost (127.0.0.1) are accepting connections.

The ports to check are listed in /home/admin/ports-to-scan.txt (one port per line).

Write your results to /home/admin/port-audit.txt with one line per port, sorted by port number (ascending), using this format:


PORT STATUS
where STATUS is exactly open or closed (lowercase).

A template file /home/admin/port-audit.txt is available with values per port "open|closed", delete the separator and the incorrect value per port or re-create the file.

The following are not available on this system (removed or restricted): ss, netstat, nmap, nc, telnet, curl, lsof, tcpdump.

NOTE: you don't have root (superuser) access.
Root (sudo) Access: False

Test: The file /home/admin/port-audit.txt exists and correctly reports whether each port in /home/admin/ports-to-scan.txt is open or closed on 127.0.0.1.

### Solution

Coplitot: gemini 3.1 flash mode

```python
admin@i-09fa9b4c1d13fdb89:~$ python3 -c '
import socket
import sys

input_path = "/home/admin/ports-to-scan.txt"
output_path = "/home/admin/port-audit.txt"

results = []

try:
    with open(input_path, "r") as f:
        ports = [int(line.strip()) for line in f if line.strip()]
except Exception as e:
    print(f"Error reading input file: {e}")
    sys.exit(1)

for port in ports:
    # Create a standard TCP socket
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(1.0) # 1 second timeout
    
    # connect_ex returns 0 if the connection was successful
    result = s.connect_ex(("127.0.0.1", port))
    s.close()
    
    status = "open" if result == 0 else "closed"
    results.append((port, status))

# Sort by port number ascending
results.sort(key=lambda x: x[0])

# Write to the file
with open(output_path, "w") as f:
    for port, status in results:
        f.write(f"{port} {status}\n")

print("Scan complete via Python!")
'
```
