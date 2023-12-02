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

## S3 Sink

- **FieldPartitioner:**
  ```
  curl -X POST   -H "Content-Type: application/json"   --data '{
   "name": "s3-sink",
    "config": {
      "connector.class": "io.confluent.connect.s3.S3SinkConnector",
      "tasks.max": "6",
      "rotate.schedule.interval.ms": 3600000,
      "topics": "test",
      "s3.bucket.name": "test",
      "s3.region": "us-east-1",
      "flush.size": "50",
      "timezone" : "UTC", 
      "storage.class": "io.confluent.connect.s3.storage.S3Storage",
      "format.class": "io.confluent.connect.s3.format.json.JsonFormat",
      "partitioner.class": "io.confluent.connect.storage.partitioner.FieldPartitioner",
      "partition.field.name": "MSISDN",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",	
      "key.converter.schemas.enable": "false",
      "value.converter.schemas.enable": "true",	
      "aws.access.key.id": "*******************",
      "aws.secret.access.key": "***************",
      "store.url": "#################",
      "name": "s3-sink"
    }}' http://localhost:8083/connectors
  ```
  
  **Maximum number of records:** The connector’s `flush.size` configuration property specifies the maximum number of records that should be written to a single S3 object. There is no default for this setting.
  
  **Maximum span of record time:** The connector’s `rotate.interval.ms` specifies the maximum timespan in milliseconds a file can remain open and ready for additional records. The timestamp for each file starts with the record timestamp of the first record written to the file, as determined by the partitioner’s timestamp.extractor. As long as the next record’s timestamp fits within the timespan specified by the rotate.interval.ms, the record will be written to the file. If a record’s timestamp does not fit within the timespan of the file, the connector will flush the file, uploaded it to S3, commit the offsets of the records in that file, and then create a new file with a timespan that starts with the first record and writes the first record to the file.
  
  **Scheduled rotation:** The connector’s `rotate.schedule.interval.ms` specifies the maximum timespan in milliseconds a file can remain open and ready for additional records. Unlike with rotate.interval.ms, with scheduled rotation the timestamp for each file starts with the system time that the first record is written to the file. As long as a record is processed within the timespan specified by rotate.schedule.interval.ms, the record will be written to the file. As soon as a record is processed after the timespan for the current file, the file is flushed, uploaded to S3, and the offset of the records in the file are committed. A new file is created with a timespan that starts with the current system time, and the record is written to the file. The commit will be performed at the scheduled time, regardless of the previous commit time or number of messages. This configuration is useful when you have to commit your data based on current server time, for example at the beginning of every hour. The default value -1 means that this feature is disabled.

  > **Important:**
  > 
  > Be sure to set the timezone configuration property before setting rotate.schedule.interval.ms, otherwise the connector will throw an exception.

  ![image](https://github.com/arezvani/kafka-connect/assets/20871524/188f9eeb-b190-4cc6-909b-e4266267a711)

  ```json
  # test+0+0000000006.json
  
  {"MSISDN":"100095650650","CALL_PARTNER":"100096709436","DURATION":"4424","IMSI":"123456792"}
  {"MSISDN":"100095650650","CALL_PARTNER":"100096709436","DURATION":"34714","IMSI":"113456792"}
  {"MSISDN":"100095650650","CALL_PARTNER":"100096709436","DURATION":"44714","IMSI":"123456792"}
  {"MSISDN":"100095650650","CALL_PARTNER":"100396709436","DURATION":"41614","IMSI":"120456792"}
  ```

- **TimeBasedPartitioner:**
  To guarantee exactly-once semantics with the `TimeBasedPartitioner`, the connector must be configured to use a deterministic implementation of TimestampExtractor and a deterministic rotation strategy. The deterministic timestamp extractors are Kafka records (timestamp.extractor=Record) or record fields (timestamp.extractor=RecordField). The deterministic rotation strategy configuration is rotate.interval.ms (setting rotate.schedule.interval.ms is nondeterministic and will invalidate exactly-once guarantees).

- **[FieldAndTimeBasedPartitioner](https://github.com/canelmas/kafka-connect-field-and-time-partitioner):**
  ```
   curl -X POST   -H "Content-Type: application/json"   --data '{
   "name": "s3-sink",
    "config": {
      "connector.class": "io.confluent.connect.s3.S3SinkConnector",
      "tasks.max": "1",
      "topics": "test",
      "s3.bucket.name": "test",
      "s3.region": "us-east-1",
      "flush.size": "50",
      "storage.class": "io.confluent.connect.s3.storage.S3Storage",
      "format.class": "io.confluent.connect.s3.format.parquet.ParquetFormat",
      "partitioner.class": "com.canelmas.kafka.connect.FieldAndTimeBasedPartitioner",
      "partition.duration.ms" : 300000,
      "path.format": "'\'year\''=YYYY/'\'month\''=MM/'\'day\''=dd",
      "locale" : "US",
      "timezone" : "UTC",        
      "partition.field.name": "MSISDN",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "value.converter": "org.apache.kafka.connect.json.JsonConverter",	
      "key.converter.schemas.enable": "false",
      "value.converter.schemas.enable": "true",	
      "aws.access.key.id": "*******************",
      "aws.secret.access.key": "*******************",
      "store.url": "##############",
      "name": "s3-sink"
    }}' http://localhost:8083/connectors
  ```

- **[AivenKafkaConnectS3SinkConnector](https://github.com/Aiven-Open/s3-connector-for-apache-kafka) (Partition with key):**
  ```
   curl -X POST   -H "Content-Type: application/json"   --data '{
   "name": "s3-sink",
    "config": {
      "connector.class": "io.aiven.kafka.connect.s3.AivenKafkaConnectS3SinkConnector",
      "topics": "test",
      "aws.s3.bucket.name": "backup-redis",
      "aws.s3.region": "us-east-1",
      "format.output.type": "jsonl",
      "file.name.template": "k{{key}}",
      "format.output.fields": "key,value,offset,timestamp",
      "file.compression.type": "gzip",
      "format.output.envelope": "true",
      "aws.access.key.id": "CEGLR7TCEN471OHFPKOE",
      "aws.secret.access.key": "qaSvzHlkkdzEiFRKeYu1NehTnzP1PY5FYxDOmq6e",
      "aws.s3.endpoint": "http://192.168.96.108:10049",
      "key.converter": "org.apache.kafka.connect.storage.StringConverter",
      "name": "s3-sink"
    }}' http://localhost:8083/connectors
  ```

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
