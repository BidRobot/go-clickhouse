# go-clickhouse [![Travis status](https://img.shields.io/travis/roistat/go-clickhouse.svg)](https://travis-ci.org/roistat/go-clickhouse) [![Coverage Status](https://img.shields.io/coveralls/roistat/go-clickhouse.svg)](https://coveralls.io/github/roistat/go-clickhouse) ![](https://img.shields.io/github/license/roistat/go-clickhouse.svg)

Golang [Yandex ClickHouse](https://clickhouse.yandex/) connector

ClickHouse manages extremely large volumes of data in a stable and sustainable manner.
It currently powers Yandex.Metrica, world’s second largest web analytics platform,
with over 13 trillion database records and over 20 billion events a day, generating
customized reports on-the-fly, directly from non-aggregated data. This system was
successfully implemented at CERN’s LHCb experiment to store and process metadata on
10bn events with over 1000 attributes per event registered in 2011.

## Examples

#### Query rows

```go
conn := clickhouse.NewConn("localhost:8123", clickhouse.NewHttpTransport())
query := clickhouse.NewQuery("SELECT name, date FROM clicks")
iter := query.Iter(conn)
var (
    name string
    date string
)
for iter.Scan(&name, &date) {
    //
}
```

#### Single insert
```go
conn := clickhouse.NewConn("localhost:8123", clickhouse.NewHttpTransport())
query, err := clickhouse.BuildInsert("clicks",
    clickhouse.Columns{"name", "date"},
    clickhouse.Row{"Test name", "2016-01-01 21:01:01"},
)
if err == nil {
    err = query.Exec(conn)
    if err == nil {
        //
    }
}
```

## Cluster

Cluster is useful if you have several servers with same `Distributed` table (master). In this case you can send
requests to random master to balance load.

* `cluster.Check()` pings all connections and filters active ones
* `cluster.ActiveConn()` returns random active connection
* `cluster.OnCheckError()` is called when any connection fails

**Important**: You should call method `Check()` at least once after initialization, but we recommend
to call it continuously, so `ActiveConn()` will always return filtered active connection.

```go
http := clickhouse.NewHttpTransport()
conn1 := clickhouse.NewConn("host1", http)
conn2 := clickhouse.NewConn("host2", http)

cluster := clickhouse.NewCluster(conn1, conn2)

// Ping connections every second
go func() {
    for {
        cluster.Check()
        time.Sleep(time.Second)
    }
}()
```
