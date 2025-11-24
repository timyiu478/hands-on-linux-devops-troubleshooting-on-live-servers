## "Solanea": ClickHouse mad house

### Problem

You have a ClickHouse installation CHI running on a Kubernetes cluster and a set of requests (located at ~/data/requests.csv) that you must populate into the http_requests table in the monitoring database (table may not exist in all pod instances).

Do this insert in all pod instances of the database.

The user and password to connect to the database are default.

The keeper pods provide clickhouse replication services.

### Solution

TODO
