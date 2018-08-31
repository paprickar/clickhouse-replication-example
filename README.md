# Clickhouse ReplicatedMergeTree Setup

I try to setup a Clickhouse cluster with replicated tables across the whole cluster. This currently does not work.

## Getting started
```
docker-compose up
```

## Create the table on all Nodes

Connect to each server and create the tables and databases
```
docker exec -it clickhouse-replication-example_ch-client_1 /usr/bin/clickhouse-client --host ch-server-1
docker exec -it clickhouse-replication-example_ch-client_1 /usr/bin/clickhouse-client --host ch-server-2
docker exec -it clickhouse-replication-example_ch-client_1 /usr/bin/clickhouse-client --host ch-server-3
```
```
CREATE TABLE mytable(some_date Date,some_int UInt64,some_str String,some_float Float32) ENGINE = MergeTree(some_date, (some_int, some_date), 8192);
CREATE DATABASE IF NOT EXISTS r0;

CREATE TABLE r0.mytable AS default.mytable
ENGINE = ReplicatedMergeTree(
    '/clickhouse/clusters/ontime_cluster/tables/{r0shard}/mytable',
    '{r0replica}',
    some_date,
    (some_int, some_date),
    8192
);
CREATE DATABASE IF NOT EXISTS r1;

CREATE TABLE IF NOT EXISTS r1.mytable AS default.mytable
ENGINE = ReplicatedMergeTree(
    '/clickhouse/clusters/ontime_cluster/tables/{r0shard}/mytable',
    '{r1replica}',
    some_date,
    (some_int, some_date),
    8192
);
```

## Create a Distributed view on the cluster across all nodes
Connect to the first database and create the Distributed view
```
docker exec -it clickhouse-replication-example_ch-client_1 /usr/bin/clickhouse-client --host ch-server-1
```
```
CREATE TABLE default.mytable AS default.mytable
ENGINE = Distributed(
    'ontime_cluster',
    'r0',
    mytable
);
```

## Insert data
Connect to the first database and insert test data to see if the replication work
```
docker exec -it clickhouse-replication-example_ch-client_1 /usr/bin/clickhouse-client --host ch-server-1
```
```
INSERT INTO mytable VALUES('2011-03-10', 28194901262, '8006-6129-3130-5580', 1169);
INSERT INTO mytable VALUES('2016-10-17', 67128904894, '4681-5453-2740-1617', 8109);
INSERT INTO mytable VALUES('2014-08-05', 79681799770, '6661-8986-3509-6991', 55);
```

# Credits
Thanks to abraithwaite/clickhouse-replication-example for getting me started
