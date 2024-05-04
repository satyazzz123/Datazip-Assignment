# Helm chart to deploy ClickHouse with Hot-cold configuration

### How to run the Chart
```
$ git clone
$ cd clickhouse-assignment
$ helm install initial . --values ./values.yaml
```
## After a while the Deployment along with service will be up
![image](https://github.com/satyazzz123/clickhouse-deloyment/assets/105061492/e47800a0-9609-4d6b-ade1-e0b428c14b6d)

## To test the Hot-cold Configuration
To run commands interactively inside a pod
```
kubectl exec -it <POD NAME> -- /bin/bash
```
Then run this to access clickhouse client

```
clickhouse-client
```
We can see the disks being created also through this

```
SELECT *
FROM system.disks
```
![image](https://github.com/satyazzz123/clickhouse-deloyment/assets/105061492/25a587e6-9b33-4cad-bb32-f5be752cb3c9)

Then once inside clickhouse-client we create a Table
```
CREATE TABLE dz_test
(
    `B` Int64,
    `T` String,
    `D` Date
)
ENGINE = MergeTree
PARTITION BY D
ORDER BY B
SETTINGS storage_policy = 'hot_cold_policy';
```
Then we insert data
```
insert into dz_test select number, number, '2023-01-01' from numbers(1e11)
```
#### once we start inserting data we can observe the data moving into mnt/clickhouse/hot and once we reach the mark of 80% it will start moving into mnt/clickhouse/cold we can verify this
## We see hot storage getting data 

![image](https://github.com/satyazzz123/clickhouse-deloyment/assets/105061492/cfc2281b-3cb5-457f-8b96-6eebcc6461eb)

## While cold storage is empty

![image](https://github.com/satyazzz123/clickhouse-deloyment/assets/105061492/2981bebe-06aa-4e5f-af2a-f651efdd4a88)

## With time we see hot storage is filling up and due to migration factor we see cold storage also sees insetion of data

![image](https://github.com/satyazzz123/clickhouse-deloyment/assets/105061492/04776f05-517b-4974-8b3d-bf401dd34b57)

## Also with time we see cold getting more and more insertions as Hot has 5GB data part size 
![image](https://github.com/satyazzz123/clickhouse-deloyment/assets/105061492/02e341b2-6aff-49a9-8539-104c047268be)



## We can also test the hot cold configuration through this
```
SELECT
    disk_name,
    formatReadableSize(sum(bytes_on_disk)) AS total_bytes
FROM system.parts
WHERE active
GROUP BY disk_name;

```
![image](https://github.com/satyazzz123/clickhouse-deloyment/assets/105061492/10b1eb54-e59f-454d-aad6-c3f5054e8920)








