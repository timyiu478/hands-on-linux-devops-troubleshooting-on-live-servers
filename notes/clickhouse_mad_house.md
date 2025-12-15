## "Solanea": ClickHouse mad house

### Problem

You have a ClickHouse installation CHI running on a Kubernetes cluster and a set of requests (located at ~/data/requests.csv) that you must populate into the http_requests table in the monitoring database (table may not exist in all pod instances).

Do this insert in all pod instances of the database.

The user and password to connect to the database are default.

The keeper pods provide clickhouse replication services.

### Solution

#### 1. check the data of the csv

```
admin@i-0761381d44f8e21cd:~/data$ head requests.csv 
timestamp,service_name,endpoint,method,status_code,latency_ms,region,user_id
2025-10-28 12:00:00.000,inventory-service,/items/987,DELETE,201,64.27,us-east,2c3a2182-7f26-457b-9ba4-7048eea3511f
2025-10-28 12:00:01.531,orders-service,/orders,GET,400,94.65,us-central,b01fd1ed-f83f-43ea-b7dd-54f734bd7298
2025-10-28 12:00:03.740,orders-service,/orders/123,PUT,403,44.85,us-east,d0745acc-bde5-44bd-bae9-be2d002d4b71
2025-10-28 12:00:05.202,orders-service,/orders/456,PUT,403,64.12,us-west,9a0205bc-92e0-4395-98e4-86a078f7cd68
2025-10-28 12:00:06.428,payment-service,/charge,PUT,404,43.32,eu-west,3d47102d-eb9d-422f-82ea-014b3c12db58
2025-10-28 12:00:08.110,inventory-service,/items,POST,200,131.44,us-east,ff059a3c-a167-4cfd-a68b-77abdf363798
2025-10-28 12:00:09.847,orders-service,/orders/456,POST,400,44.95,us-central,6c71d521-601a-485e-82d2-046ca0d1e428
2025-10-28 12:00:13.655,auth-service,/refresh,POST,204,56.29,us-east,a4d7289a-844a-448a-8699-a967acce3641
2025-10-28 12:00:10.277,orders-service,/orders/456,PUT,404,112.61,us-east,f9421b37-9b65-45f1-a3c6-90553fd5832d
```

#### 2. check the k8s cluster status

```
admin@i-0761381d44f8e21cd:~/data$ kubectl get pods -A
NAMESPACE             NAME                                                              READY   STATUS    RESTARTS        AGE
clickhouse-operator   clickhouse-operator-altinity-clickhouse-operator-5bd598bb9jz2w4   1/1     Running   1 (3m44s ago)   31d
clickhouse            chi-clickhouse-clickhouse-0-0-0                                   1/1     Running   1 (3m44s ago)   31d
clickhouse            chi-clickhouse-clickhouse-0-1-0                                   1/1     Running   1 (3m44s ago)   31d
clickhouse            chi-clickhouse-clickhouse-0-2-0                                   1/1     Running   1 (3m44s ago)   31d
clickhouse            keeper-clickhouse-0-0                                             1/1     Running   1 (3m44s ago)   31d
clickhouse            keeper-clickhouse-1-0                                             1/1     Running   1 (3m44s ago)   31d
clickhouse            keeper-clickhouse-2-0                                             1/1     Running   1 (3m44s ago)   31d
kube-system           coredns-6799fbcd5-jhx6z                                           1/1     Running   2 (3m44s ago)   92d
local-path-storage    local-path-provisioner-7599fd788b-84bxp                           1/1     Running   2 (3m ago)      31d
```

#### 3. learn more about the database table


```
root@chi-clickhouse-clickhouse-0-0-0:/# clickhouse-client -h 127.0.0.1 --user default --password default --query="SHOW TABLES FROM monitoring"
http_requests
root@chi-clickhouse-clickhouse-0-0-0:/# clickhouse-client -h 127.0.0.1 --user default --password default --query="DESCRIBE TABLE monitoring.http_requests"
timestamp       DateTime64(3, \'UTC\')
service_name    String
endpoint        String
method  Enum8(\'GET\' = 1, \'POST\' = 2, \'PUT\' = 3, \'DELETE\' = 4)
status_code     UInt16
latency_ms      Float32
region  LowCardinality(String)
user_id UUID
```

#### 4. Insert requests

co-pilot: Grok 4.1
ref: https://clickhouse.com/docs/integrations/data-formats/csv-tsv

```bash
IPS=$(kubectl get po -n clickhouse -o jsonpath='{.items[*].status.podIP}' -l clickhouse.altinity.com/chi=clickhouse 2>/dev/null)

# Step 1: Create the database and table on every pod
for ip in $IPS; do
  echo "=== Creating table on $ip ==="
  clickhouse-client -h "$ip" --user default --password default -q "
    CREATE DATABASE IF NOT EXISTS monitoring;
    CREATE TABLE IF NOT EXISTS monitoring.http_requests (
        timestamp DateTime64(3, 'UTC'),
        service_name String,
        endpoint String,
        method Enum8('GET' = 1, 'POST' = 2, 'PUT' = 3, 'DELETE' = 4),
        status_code UInt16,
        latency_ms Float32,
        region LowCardinality(String),
        user_id UUID
    ) ENGINE = MergeTree
    ORDER BY (timestamp, service_name, endpoint)"
done

# Step 2: Insert the full CSV (including header) using CSVWithNames
for ip in $IPS; do
  echo "=== Inserting data into $ip ==="
  clickhouse-client -h "$ip" --user default --password default \
    -q "INSERT INTO monitoring.http_requests FORMAT CSVWithNames" \
    < ~/data/requests.csv
done
```

Run:

```
admin@i-0761381d44f8e21cd:~$ vim pop.sh
admin@i-0761381d44f8e21cd:~$ chmod +x pop.sh
admin@i-0761381d44f8e21cd:~$ ./pop.sh 
=== Creating table on 10.42.0.19 ===
=== Creating table on 10.42.0.20 ===
=== Creating table on 10.42.0.26 ===
=== Inserting data into 10.42.0.19 ===
=== Inserting data into 10.42.0.20 ===
=== Inserting data into 10.42.0.26 ===
admin@i-0761381d44f8e21cd:~$ ./agent/check.sh 
OKadmin@i-0761381d44f8e21cd:~$
```
