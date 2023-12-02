# kafka-connect
Kafka Connect examples
- DataGen Source
- JDBC Source & Sink
- S3 Sink
- Elasticsearch Sink

## DataGen Source
**Transactions:**

```bash
curl -X POST   -H "Content-Type: application/json"   --data '{
  "name": "datagen-transactions",
  "config": {
    "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
    "kafka.topic": "transactions",
    "quickstart": "transactions",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false",
    "max.interval": 1000,
    "tasks.max": "1"
  }
}' http://my-cp-kafka-connect:8083/connectors
```

Kafka:
```json
{"transaction_id":546,"card_id":4,"user_id":"User_5","purchase_id":545,"store_id":2}
```

**Credit_Cards:**

```bash
curl -X POST   -H "Content-Type: application/json"   --data '{
  "name": "datagen-credit_cards",
  "config": {
    "connector.class": "io.confluent.kafka.connect.datagen.DatagenConnector",
    "kafka.topic": "credit_cards",
    "quickstart": "credit_cards",
    "schema.keyfield": "card_id",
    "key.converter": "org.apache.kafka.connect.storage.StringConverter",
    "value.converter": "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable": "false",
    "max.interval": 250,
    "tasks.max": "1"
  }
}' http://localhost:8083/connectors
```

Kafka:
![photo1701452795](https://github.com/arezvani/kafka-connect/assets/20871524/1934c622-e42b-47f4-b214-cd6341837b77)

## JDBC Source & Sink
### Source

```bash
curl -X POST   -H "Content-Type: application/json"   --data '{  "name": "postgres-source",
            "config": {
            "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
            "connection.url": "jdbc:postgresql://dbaas.abriment.com:31811/persons",
            "connection.user": "admin",
            "connection.password": "SAra@131064",
            "mode":"bulk",
            "topic.prefix":"postgres_table_",
            "table.whitelist":"persons_information",
            "incrementing.column.name" : "uuid",
            "key.converter": "org.apache.kafka.connect.storage.StringConverter",
            "value.converter": "org.apache.kafka.connect.json.JsonConverter",
            "poll.interval.ms" : 500
        }}' http://localhost:8083/connectors
```

For changing config you can use `PUT` method:

```bash
curl -X PUT    -H "Content-Type: application/json"   --data '{
            "connector.class": "io.confluent.connect.jdbc.JdbcSourceConnector",
            "connection.url": "jdbc:postgresql://dbaas.abriment.com:31811/persons",
            "connection.user": "admin",
            "connection.password": "SAra@131064",
            "mode":"bulk",
            "topic.prefix":"postgres_table_",
            "table.whitelist":"persons_information",
            "incrementing.column.name" : "uuid",
            "catalog.pattern": "",
            "key.converter": "org.apache.kafka.connect.storage.StringConverter",
            "value.converter": "org.apache.kafka.connect.json.JsonConverter",
            "poll.interval.ms" : 500
        }' http://localhost:8083/connectors/postgres-source/config
```

It will create topic automatically and produce records from Postgres table to Kafka.

### Sink
```bash
curl -X POST   -H "Content-Type: application/json"   --data '{  "name": "postgres-sink",
            "config": {
    "connector.class" : "io.confluent.connect.jdbc.JdbcSinkConnector",
    "connection.url" : "jdbc:postgresql://dbaas.abriment.com:31811/persons",
    "topics" : "topic-sink",
    "key.converter" : "org.apache.kafka.connect.storage.StringConverter",
    "value.converter" : "org.apache.kafka.connect.json.JsonConverter",
    "value.converter.schemas.enable" : "true",
    "connection.user" : "admin",
    "connection.password" : "SAra@131064",
    "auto.create" : true,
    "auto.evolve"  : true,
    "insert.mode" : "insert"
        }}' http://localhost:8083/connectors
```

We can produce data with kafka-console-producer like that:
```
{ "schema": { "type": "struct", "optional": false, "version": 1, "fields": [ { "field": "ID", "type": "string", "optional": true }, { "field": "Artist", "type": "string", "optional": true }, { "field": "Song", "type": "string", "optional": true } ] }, "payload": { "ID": "1", "Artist": "Rick Astley", "Song": "Never Gonna Give You Up" } }
```

It will crate table automatically and insert consumed records from kafka to database.

You can also use [Kafka Connect Transformations](https://docs.confluent.io/platform/current/connect/transforms/overview.html) for drop, cast, replace or change records or fields before insert or produce to kafka or database.

> **Note**
> You can use this configs for print or parse key and value in kafka (kafka-console-producer & kafka-console-consumer):
> 
> Producer:
> ```
> --property "parse.key=true" --property "key.separator=:"
> ```
>
> Consumer:
> ```
> --property print.key=true --property key.separator=":"
> ```


## Command and Rest API

### List Plugins
```bash
curl localhost:8083/connector-plugins | jq

[
  {
    "class": "io.confluent.connect.replicator.ReplicatorSourceConnector"
  },
  {
    "class": "org.apache.kafka.connect.file.FileStreamSourceConnector"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSinkConnector"
  },
  {
    "class": "io.confluent.connect.jdbc.JdbcSourceConnector"
  },
  {
    "class": "io.confluent.connect.hdfs.HdfsSinkConnector"
  },
  {
    "class": "org.apache.kafka.connect.file.FileStreamSinkConnector"
  }
]

```

### Listing active connectors on a worker
```
curl localhost:8083/connectors

["local-file-sink"]
```

### Restarting a connector or task
```
curl -X POST localhost:8083/connectors/local-file-sink/restart

(no response printed if success)

curl -X POST localhost:8083/connectors/local-file-sink/tasks/0/restart
```

### Getting tasks for a connector
```
curl localhost:8083/connectors/local-file-sink/tasks | jq

[
  {
    "id": {
      "connector": "local-file-sink",
      "task": 0
    },
    "config": {
      "task.class": "org.apache.kafka.connect.file.FileStreamSinkTask",
      "topics": "connect-test",
      "file": "test.sink.txt"
    }
  }
]
```

### Pause a connector
```
curl -X PUT localhost:8083/connectors/local-file-sink/pause

(no response printed if success)
```

### Resuming a connector
```
curl -X PUT localhost:8083/connectors/local-file-sink/resume

(no response printed if success)
```

### Updating connector configuration
```
curl -X PUT -H "Content-Type: application/json" --data '{"connector.class":"FileStreamSinkConnector","file":"test.sink.txt","tasks.max":"2","topics":"connect-test","name":"local-file-sink"}' localhost:8083/connectors/local-file-sink/config

{
  "name": "local-file-sink",
  "config": {
    "connector.class": "FileStreamSinkConnector",
    "file": "test.sink.txt",
    "tasks.max": "2",
    "topics": "connect-test",
    "name": "local-file-sink"
  },
  "tasks": [
    {
      "connector": "local-file-sink",
      "task": 0
    },
    {
      "connector": "local-file-sink",
      "task": 1
    }
  ]
}
```

### Getting connector status
```
  curl localhost:8083/connectors/local-file-sink/status | jq

{
  "name": "local-file-sink",
  "connector": {
    "state": "RUNNING",
    "worker_id": "192.168.86.101:8083"
  },
  "tasks": [
    {
      "state": "RUNNING",
      "id": 0,
      "worker_id": "192.168.86.101:8083"
    },
    {
      "state": "RUNNING",
      "id": 1,
      "worker_id": "192.168.86.101:8083"
    }
  ]
}
```

### Getting connector configuration
```
curl localhost:8083/connectors/local-file-sink | jq

{
  "name": "local-file-sink",
  "config": {
    "connector.class": "FileStreamSinkConnector",
    "file": "test.sink.txt",
    "tasks.max": "2",
    "topics": "connect-test",
    "name": "local-file-sink"
  },
  "tasks": [
    {
      "connector": "local-file-sink",
      "task": 0
    },
    {
      "connector": "local-file-sink",
      "task": 1
    }
  ]
}
```

### Deleting a connector 
```
curl -X DELETE localhost:8083/connectors/local-file-sink

(no response printed if success)
```
