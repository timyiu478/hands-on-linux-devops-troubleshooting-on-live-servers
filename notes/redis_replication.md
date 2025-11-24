## "Tunis": Redis Replication Problem

### Problem

A Redis master-replica setup is running on this server, with the master on port 6379 and the replica on port 6380. Both instances show as "connected" when you check their status, but data synchronization has silently broken.

Recent writes to the master don't appear on the replica, even though there are no obvious errors in the logs and both Redis instances appear healthy.

Fix the replication issues so that data written to the master (port 6379) immediately appears on the replica (port 6380) without data loss.

Master: localhost:6379
Replica: localhost:6380
Password: masterpass123

A helper test script is available at /home/admin/test_replication.sh

### Solution

todo
