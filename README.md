## Orginal repo difference:

There are three shard by one replicas to discover cluster with
 [Distributed-tables](https://clickhouse.yandex/docs/en/operations/table_engines/distributed).

## How to start

Install Docker Engine and Docker Compose.

Call `docker-compose up -d --remove-orphans`.

In result you will have:

* haproxy
* [Bulk](https://github.com/nikepan/clickhouse-bulk)
* 3 Zookeeper
* 3 Clickhouse

Next connect to any clickhouse instance by `clickhouse-client --host=127.0.0.1 --port=9000 --multiline`
 and create regular table at first:
```
CREATE TABLE IF NOT EXISTS default.test_real ON CLUSTER my_cluster (
  date DateTime DEFAULT now(),
  uuid String,
  data String,
)
ENGINE = MergeTree()
TTL date + INTERVAL 90 DAY
PARTITION BY toYYYYMMDD(date)
ORDER BY (uuid);
```

And distributed next:
```
CREATE TABLE IF NOT EXISTS default.test ON CLUSTER my_cluster AS default.test_real
ENGINE = Distributed(my_cluster, default, test_real, halfMD5(uuid));
```

## Explanation/Usage

So now you have two main ports:

- http port `8124` for inserting rows into `test_real`-table
- tcp port `9000` for selecting rows from `test`-table

You may insert by single rows, `bulk` will automatically aggregate it and write bulk into instance
 with round-robin sharding.

Select query will be distribute on all cluster.
