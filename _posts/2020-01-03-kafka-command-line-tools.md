---
layout: post
title: kafka command line tools
tags: [kafka, kafka-console-consumer, kafka-console-producer, kafka-topics]
---

First of all we gonna need kafka itself, e.g.:

```bash
docker run -it --rm --name=kafka -e SAMPLEDATA=0 -e RUNNING_SAMPLEDATA=0 -e RUNTESTS=0 -e FORWARDLOGS=0 -e ADV_HOST=127.0.0.1 -p 2181:2181 -p 3030:3030 -p 8081-8083:8081-8083 -p 9092:9092 -p 9581-9585:9581-9585 lensesio/fast-data-dev:2.3.0
```

or

```bash
wget https://raw.githubusercontent.com/confluentinc/examples/5.3.1-post/cp-all-in-one/docker-compose.yml
docker-compose up -d
```

or

run everything by hands on your own like described in [quickstart](https://kafka.apache.org/quickstart)

or

get [confluent.cloud](https://confluent.cloud)

# Kafka topics

The most basic and needed

## List topics

```bash
kafka-topics --bootstrap-server localhost:9092 --list
```

## Create topic

```bash
kafka-topics --bootstrap-server localhost:9092 --create --topic demo2 --partitions 3 --replication-factor 1
```

## Delete topic

```bash
kafka-topics --bootstrap-server localhost:9092 --delete --topic demo2
```

# Console producer and consumers

Here are examples for following use cases:

- simple without key
- simple with string key
- simple with integer key
- json without key
- json with string key
- json with ingeteger key
- json with json key
- avro without key
- avro with string key
- avro with integer key
- avro with avro key

By deafult in all following examples messages delimited by new line, e.g. start producer, type something, press enter.

All follogin examples are run agains

```bash
docker run -it --rm --name=kafka -e SAMPLEDATA=0 -e RUNNING_SAMPLEDATA=0 -e RUNTESTS=0 -e FORWARDLOGS=0 -e ADV_HOST=127.0.0.1 -p 2181:2181 -p 3030:3030 -p 8081-8083:8081-8083 -p 9092:9092 -p 9581-9585:9581-9585 lensesio/fast-data-dev:2.3.0
```

## String producer and consumer

### Simple without key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic SimpleWithoutKey --partitions 3 --replication-factor 1
```

Start `kafka-console-producer` which will produce simple string messages

```bash
docker exec -it kafka kafka-console-producer --broker-list localhost:9092 --topic SimpleWithoutKey
```

Start `kafka-console-consumer` to consume simple string messages

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic SimpleWithoutKey --from-beginning
```

Produce simple messages like:

```bash
hello
world
```

And you should see them in consumer as:

```bash
hello
world
```

### Simple with string key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic SimpleWithStringKey --partitions 3 --replication-factor 1
```

Start `kafka-console-producer` which will produce simple string messages with string key

```bash
docker exec -it kafka kafka-console-producer --broker-list localhost:9092 --topic SimpleWithStringKey --property parse.key=true --property key.separator=:
```

Notes:

- `--property parse.key=true` our consumer will expect us to enter key along side value
- `--property key.separator=:` is optional and by default is space

Start `kafka-console-consumer` to consume simple string messages with string key

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic SimpleWithStringKey --property print.key=true --from-beginning
```

Notes:

- `--property print.key=true` will print key

Produce messages like:

```bash
1:one
2:two
```

And you should see them in consumer as:

```bash
1	one
2	two
```

If you try to produce message without key you should see an error:

```
>message without key
org.apache.kafka.common.KafkaException: No key found on line 3: acme
	at kafka.tools.ConsoleProducer$LineMessageReader.readMessage(ConsoleProducer.scala:265)
	at kafka.tools.ConsoleProducer$.main(ConsoleProducer.scala:54)
	at kafka.tools.ConsoleProducer.main(ConsoleProducer.scala)
```

### Simple with integer key

Not fully possible at moment, here are some links:

- [kafka-console-producer ignores value serializer?](https://stackoverflow.com/questions/44803392/)
- [Console Producer / Consumer's serde config is not working](https://issues.apache.org/jira/browse/KAFKA-2526)
- [Console Producer sources](https://github.com/apache/kafka/blob/trunk/core/src/main/scala/kafka/tools/ConsoleProducer.scala#L96)

The problem is that no matter what you will pass to console producer it still will send bytes

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic SimpleWithIntKey --partitions 3 --replication-factor 1
```

Start `kafka-console-producer` which will produce simple string messages with integer key

```bash
docker exec -it kafka kafka-console-producer --broker-list localhost:9092 --topic SimpleWithIntKey --property parse.key=true --property key.serializer=org.apache.kafka.common.serialization.IntegerDeserializer --property value.serializer=org.apache.kafka.common.serialization.StringDeserializer --property key.separator=:
```

Notes:

- `--property key.serializer=org.apache.kafka.common.serialization.IntegerDeserializer` defines which deserializer should be used for key
- `--property value.serializer=org.apache.kafka.common.serialization.StringDeserializer` defines which deserializer should be user for value
- both not being applied

Start `kafka-console-consumer` to consume simple string messages with integer key

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic SimpleWithIntKey --property print.key=true --from-beginning --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property value.deserializer=org.apache.kafka.common.serialization.StringDeserializer --skip-message-on-error
```

Notes:

- `key.deserializer=org.apache.kafka.common.serialization.StringDeserializer` we are forced to use string instead of integer deserializer here, otherwise will receive an error `ERROR Error processing message, skipping this message: (kafka.tools.ConsoleConsumer$) org.apache.kafka.common.errors.SerializationException: Size of data received by IntegerDeserializer is not 4`
- `--skip-message-on-error` do not crash on bad message, just skip it

Produce messages like:

```bash
1:one
2:two
```

And you should see them in consumer as:

```bash
1	one
2	two
```

## Json producer and consumer

### Json without key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic JsonWithoutKey --partitions 3 --replication-factor 1
```

Start `kafka-console-producer` which will produce json messages

```bash
docker exec -it kafka kafka-console-producer --broker-list localhost:9092 --topic JsonWithoutKey
```

Note that like in previous example with integer key, `kafka-console-producer` does not respect given serializers so we will just put string which looks like json but still sent as bytes

Start `kafka-console-consumer` to consume json messages

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic JsonWithoutKey --property value.deserializer=org.apache.kafka.connect.json.JsonDeserializer --skip-message-on-error --from-beginning --property print.timestamp=true
```

Produce json messages like:

```
{"foo": "bar"}
{"acme": 42}
```

And you should see them like:

```
CreateTime:1578081298745        {"foo":"bar"}
CreateTime:1578081304001        {"acme":42}
```

There is not checks in producer but if you send something wrong you will see an error in consumer

```
CreateTime:1578081353956        [2020-01-03 19:55:54,970] ERROR Error processing message, skipping this message:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.SerializationException: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'foo': was expecting 'null', 'true', 'false' or NaN
 at [Source: (byte[])"foo"; line: 1, column: 7]
Caused by: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'foo': was expecting 'null', 'true', 'false' or NaN
 at [Source: (byte[])"foo"; line: 1, column: 7]
```

### Json with string key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic JsonWithStringKey --partitions 3 --replication-factor 1
```

Start `kafka-console-producer` which will produce json messages with string key

```bash
docker exec -it kafka kafka-console-producer --broker-list localhost:9092 --topic JsonWithStringKey --property parse.key=true --property key.separator=:
```

Start `kafka-console-consumer` to consume json messages with string key

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic JsonWithStringKey --property print.key=true --from-beginning --property value.deserializer=org.apache.kafka.connect.json.JsonDeserializer
```

Produce messages:

```
1:{"foo":"bar"}
2:{"acme":42}
```

And you should see:

```
1       {"foo":"bar"}
2       {"acme":42}
```

Note that there is the same problem with keys as in previous examples, and you can not force integer key.

### Json with json key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic JsonWithJsonKey --partitions 3 --replication-factor 1
```

Start `kafka-console-producer` which will produce json messages with json keys

```bash
docker exec -it kafka kafka-console-producer --broker-list localhost:9092 --topic JsonWithJsonKey --property parse.key=true --property key.separator="|"
```

Start `kafka-console-consumer` to consume json messages

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic JsonWithJsonKey  --property value.deserializer=org.apache.kafka.connect.json.JsonDeserializer --property key.deserializer=org.apache.kafka.connect.json.JsonDeserializer --skip-message-on-error --from-beginning --property print.key=true
```

Produce messages:

```
{"id":1}|{"foo":"bar"}
{"id":2}|{"acme":42}
```

And you should see:

```
{"id":1}        {"foo":"bar"}
{"id":2}        {"acme":42}
```

If you will produce bad key or value you will get:

```
ERROR Error processing message, skipping this message:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.SerializationException: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'foo': was expecting 'null', 'true', 'false' or NaN
 at [Source: (byte[])"foo"; line: 1, column: 7]
Caused by: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'foo': was expecting 'null', 'true', 'false' or NaN
 at [Source: (byte[])"foo"; line: 1, column: 7]
```

## Avro producer and consumer

### Avro without key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic AvroWithoutKey --partitions 3 --replication-factor 1
```

Start `kafka-avro-console-producer` to produce avro messages

```bash
docker exec -it kafka kafka-avro-console-producer --broker-list localhost:9092 --topic AvroWithoutKey --property value.schema='{"type":"record","name":"AvroWithoutKey","fields":[{"name":"foo","type":"string"}]}'
```

Note that from now on we are using `kafka-avro-console-producer` instead of `kafka-console-producer` which has few additional properties like `--property value.schema='{"type":"record","name":"AvroWithoutKey","fields":[{"name":"foo","type":"string"}]}'` messages published via this consumer will be validated against given schema. Also note that this producer does not show `>` symbol, so do not wait for it.

Start `kafka-avro-console-consumer` to consume avro messages

```bash
docker exec -it kafka kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic AvroWithoutKey --from-beginning
```

Try sending something like:

```bash
{"foo":"hello"}
{"foo":"world"}
```

and you should see exactly the same output in consumer.

If you will try send something wrong you will receive an error:

```
{"acme":42}
org.apache.kafka.common.errors.SerializationException: Error deserializing json {"acme":42} to Avro of schema {"type":"record","name":"AvroWithoutKey","fields":[{"name":"foo","type":"string"}]}
Caused by: org.apache.avro.AvroTypeException: Expected field name not found: foo
        at org.apache.avro.io.JsonDecoder.doAction(JsonDecoder.java:477)
        at org.apache.avro.io.parsing.Parser.advance(Parser.java:88)
```

but still if something you are sending is schema compatible everything should be ok, try sending `{"foo":"bar","acme":42}` and you will receive `{"foo":"bar"}` in your consumer

### Avro with string key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic AvroWithStringKey --partitions 3 --replication-factor 1
```

Start `kafka-avro-console-producer` to produce avro messages with primitive string key

```bash
docker exec -it kafka kafka-avro-console-producer --broker-list localhost:9092 --topic AvroWithStringKey --property value.schema='{"type":"record","name":"AvroWithStringKey","fields":[{"name":"foo","type":"string"}]}' --property parse.key=true --property key.schema='{"type":"string"}' --property key.separator=" "
```

Not that we have added `--property key.schema='{"type":"string"}'` which allow us to use primitives as key and they still will be validated.

Start `kafka-avro-console-consumer` to consume avro messages with string keys

```bash
docker exec -it kafka kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic AvroWithStringKey --from-beginning --property print.key=true --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer
```

Try send something like:

```
"one" {"foo":"1"}
"two" {"foo":"2"}
```

and you should get:

```
one     {"foo":"1"}
two     {"foo":"2"}
```

Do not forget to wrap key with double quotes otherwise you will get an error:

```
org.apache.kafka.common.errors.SerializationException: Error deserializing json one to Avro of schema "string"
Caused by: org.codehaus.jackson.JsonParseException: Unexpected character ('o' (code 111)): expected a valid value (number, String, array, object, 'true', 'false' or 'null')
 at [Source: java.io.StringReader@3feb2dda; line: 1, column: 2]
```

### Avro with int key

Does not work, in example below after trying to send `1|{"foo":"bar"}` receiving an error:

```
org.apache.kafka.common.errors.SerializationException: Error deserializing json 1|{"foo":"hello"} to Avro of schema {"type":"record","name":"AvroWithIntKey","fields":[{"name":"foo","type":"int"}]}
Caused by: org.apache.avro.AvroTypeException: Expected record-start. Got VALUE_NUMBER_INT
```

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic AvroWithIntKey --partitions 3 --replication-factor 1
```

Start `kafka-avro-console-producer` to produce avro messages with integer keys

```bash
docker exec -it kafka kafka-avro-console-producer --broker-list localhost:9092 --topic AvroWithIntKey --property value.schema='{"type":"record","name":"AvroWithIntKey","fields":[{"name":"foo","type":"int"}]}' --property key.separator="|"
```

Start `kafka-avro-console-consumer` to consume avro messages

```bash
docker exec -it kafka kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic AvroWithIntKey --from-beginning
```

### Avro with avro key

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic AvroWithAvroKey --partitions 3 --replication-factor 1
```

Start `kafka-avro-console-producer` which will produce avro messages with avro keys

```bash
docker exec -it kafka kafka-avro-console-producer --broker-list localhost:9092 --topic AvroWithAvroKey --property value.schema='{"type":"record", "name": "AvroWithAvroKey", "fields":[{"name":"foo","type":"string"}]}' --property parse.key=true --property key.schema='{"type":"record","name": "key", "fields":[{"name":"id","type":"int"}]}' --property key.separator=" "
```

Start `kafka-avro-console-consumer` to consume avro messages with avro keys

```bash
docker exec -it kafka kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic AvroWithAvroKey --from-beginning --property print.key=true
```

Try send

```
{"id":1} {"foo":"hello"}
{"id":2} {"foo":"world"}
```

and you should receive

```
{"id":1}        {"foo":"hello"}
{"id":2}        {"foo":"world"}
```

if you will try send wrong key like `{"id":"guid"}` you will receive an error

```
org.apache.kafka.common.errors.SerializationException: Error deserializing json {"id":"guid"} to Avro of schema {"type":"record","name":"key","fields":[{"name":"id","type":"int"}]}
Caused by: org.apache.avro.AvroTypeException: Expected int. Got VALUE_STRING
```

# Confluent Cloud

If you are using [confluent.cloud](https://confluent.cloud/) from [confluent.io](https://confluent.io/) you still able to do all this with few more params added for commands

More examples can be found [here](https://github.com/confluentinc/examples/tree/5.3.2-post/clients/cloud/kafka-commands)

## Topics

You gonna need properties file which you can retrieve from `https://confluent.cloud/environments/*****/clusters/***-*****/integrations/clients#java` by navigating cluster then "CLI & client configuration"

**cloud.properties**

```ini
bootstrap.servers=xxx-xxxxx.us-east1.gcp.confluent.cloud:9092
ssl.endpoint.identification.algorithm=https
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
```

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-kafka:5.3.2 kafka-topics \
  --bootstrap-server xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
  --command-config /cloud.properties \
  --list
```

all other commands will work as expected

## Produce & consume simple messages

If you are going to run simple producer without avro and schema registry then properties file from previous example should be enough

Create topic

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-kafka:5.3.2 kafka-topics \
  --bootstrap-server xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
  --command-config /cloud.properties \
   --create --topic simple1 --partitions 3 --replication-factor 3
```

Start producer

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-kafka:5.3.2 kafka-console-producer \
  --broker-list xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
  --producer.config /cloud.properties \
  --topic simple1
```

Start consumer

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-kafka:5.3.2 kafka-console-consumer \
  --bootstrap-server xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
  --consumer.config /cloud.properties \
  --topic simple1
```

Cleanup

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-kafka:5.3.2 kafka-topics \
  --bootstrap-server xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
  --command-config /cloud.properties \
   --delete --topic simple1
```

Note that you are not restricted to strings only, you can also use all previous examples with different keys and json

## Produce & consume AVRO messages in confluent.cloud

Create topic

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-kafka:5.3.2 kafka-topics \
  --bootstrap-server xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
  --command-config /cloud.properties \
   --create --topic avro1 --partitions 3 --replication-factor 3
```

Start producer

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-schema-registry:5.3.2 kafka-avro-console-producer \
    --broker-list xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
    --topic avro1 \
    --property value.schema='{"type":"record","name":"AvroWithoutKey","fields":[{"name":"foo","type":"string"}]}' \
    --producer.config /cloud.properties \
    --property schema.registry.url="https://xxxx-xxxxx.us-east1.gcp.confluent.cloud" \
    --property schema.registry.basic.auth.user.info="xxxxxxx:xxxxxxx" \
    --property basic.auth.credentials.source=USER_INFO
```

Start consumer

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-schema-registry:5.3.2 kafka-avro-console-consumer \
    --bootstrap-server xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
    --topic avro1 \
    --from-beginning \
    --value-deserializer io.confluent.kafka.serializers.KafkaAvroDeserializer \
    --key-deserializer org.apache.kafka.common.serialization.StringDeserializer \
    --consumer.config /cloud.properties \
    --property schema.registry.url="https://xxxx-xxxxx.us-east1.gcp.confluent.cloud" \
    --property schema.registry.basic.auth.user.info="xxxxxxx:xxxxxxx" \
    --property basic.auth.credentials.source=USER_INFO
```

Notes:

- we are using another docker image `confluentinc/cp-schema-registry:5.3.2` because of kafka avro console consumer and producer
- `cloud.properties` is still enough but schema registry settings should be passed via command line arguments

# Kafka connect

We are going to use kafka connect to:

- produce predefined messages from a file to replay some sequence of events
- produce generated messages to get millions of them for test purposes
- have sample sink connector to save messages to a file

All example will be made as standalone worker which should not be used in production and used here only because of its easy to use

At very end worker command looks liks like this: `connect-standalone worker.properties task1.properties task2.properties` where `worker.properties` contains configuration for worker itself and some defaults for tasks, `taskX.properties` is task configuration, you can have many of them, for example your worker might have few tasks which will produce messages from different files and one task to consume them into elasticsearch.

Tasks producing data into kafka called `source`, tasks consuming data from kafka called `sink`.

Be aware of advertised hosts and rest ports, if you are connecting to dockerized kafka which have localhost as advertised host from your worker which is also run in container nothing will work, use `--net=host` for such scenarios, but then you gonna need to change `rest.port` to avoid conflict with already taken port.

More links about worker properties:

- [common worker configs](https://docs.confluent.io/3.2.0/connect/userguide.html#common-worker-configs)
- [connect configurations](https://docs.confluent.io/current/installation/configuration/connect/index.html)

Also you can get samples like so:

```bash
docker run -it --rm confluentinc/cp-kafka-connect:5.3.2 cat /etc/schema-registry/connect-avro-standalone.properties
```

## Local kafka connect

Start your kafka

```bash
docker run -it --rm --name=kafka -e SAMPLEDATA=0 -e RUNNING_SAMPLEDATA=0 -e RUNTESTS=0 -e FORWARDLOGS=0 -e ADV_HOST=127.0.0.1 -p 2181:2181 -p 3030:3030 -p 8081-8082:8081-8082 -p 9092:9092 -p 9581-9585:9581-9585 lensesio/fast-data-dev:2.3.0
```

Note that I'm not exposing `8083` which is used by kafka connect rest api to avoid conflicts, otherwise do not forget to change `rest.port` in `worker.properties`

### Simple messages

**worker.properties**

```ini
bootstrap.servers=localhost:9092

# do not forget to change me to avoid conflicts
rest.port=8083

# required for standalone workers
offset.storage.file.filename=/tmp/standalone.offsets

# where to look for additional plugins
plugin.path=/usr/share/java,/usr/share/confluent-hub-components

# optional, defaults for tasks
key.converter=org.apache.kafka.connect.storage.StringConverter
value.converter=org.apache.kafka.connect.storage.StringConverter
```

Notes:

- key and value converters are optional and can be overriden in tasks
- most used converters are: `org.apache.kafka.connect.storage.StringConverter`, `org.apache.kafka.connect.json.JsonConverter`, `io.confluent.connect.avro.AvroConverter`
- avro converter requires schema registry
- for json converter do not forget to add `value.converter.schemas.enable=false` if you wish not to receive schema, e.g. by sending `{"foo":"bar"}` you will receive `{"schema":{"type":"string","optional":false},"payload":"{\"foo\": \"bar\"}"}`

### Kafka Connect Source Text File

Notes on task configuration properties:

- do not forget that each task should have unique `name` it will be used to watch for offsets and for distributed wrokers it will be used for topic names
- `connector.class` is a kind of plugin, you can choose from [hub.confluent.io](https://hub.confluent.io/)
- `tasks.max` control parallelism, for sink tasks can not be bigger that number of topic partitions

**source-text-file.properties**

```ini
name=source-text-file
connector.class=org.apache.kafka.connect.file.FileStreamSourceConnector
# optional, override worker defaults
value.converter=org.apache.kafka.connect.storage.StringConverter
topic=DemoTextFile
file=demo-text-file.txt
```

**demo-text-file.txt**

```
hello
world
mac
was
here
```

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic DemoTextFile --partitions 3 --replication-factor 1
```

Start consumer

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic DemoTextFile --from-beginning
```

Start worker

```bash
docker run -it --rm \
    --name=standalone \
    --net=host \
    -v $PWD:/data \
    -w /data \
    confluentinc/cp-kafka-connect:5.3.2 connect-standalone worker.properties source-text-file.properties
```

Note that we are bypassing our current directory into container so worker has access to all configuration files

If everything is ok after some while you will see your messages from a source file in your consumer

### Kafka Connect Source JSON File

This one will work same way as previous

**source-json-file.properties**

```ini
name=source-json-file
connector.class=org.apache.kafka.connect.file.FileStreamSourceConnector
# optional, override worker defaults
# value.converter=org.apache.kafka.connect.json.JsonConverter
# value.converter.schemas.enable=false
# if your will use JsonConverter here you will receive string with escaped json
value.converter=org.apache.kafka.connect.storage.StringConverter
topic=DemoJsonFile
file=demo-json-file.ndjson
```

**demo-json-file.ndjson**

```
{"foo": "hello"}
{"foo": "world"}
{"foo": "bar"}
{"acme": 42}
```

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic DemoJsonFile --partitions 3 --replication-factor 1
```

Start consumer

```bash
docker exec -it kafka kafka-console-consumer --bootstrap-server localhost:9092 --topic DemoJsonFile --property value.deserializer=org.apache.kafka.connect.json.JsonDeserializer --skip-message-on-error --from-beginning
```

Start worker

```bash
docker run -it --rm \
    --name=standalone \
    --net=host \
    -v $PWD:/data \
    -w /data \
    confluentinc/cp-kafka-connect:5.3.2 connect-standalone worker.properties source-json-file.properties
```

While everything running, try add more records to a source file and save it, you should immediatelly see them in consumer.

Also try to add non json line to a source file, you will get an error:

```
[2020-01-04 10:09:00,896] ERROR Error processing message, skipping this message:  (kafka.tools.ConsoleConsumer$)
org.apache.kafka.common.errors.SerializationException: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'non': was expecting 'null', 'true', 'false' or NaN
 at [Source: (byte[])"non json"; line: 1, column: 5]
Caused by: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'non': was expecting 'null', 'true', 'false' or NaN
 at [Source: (byte[])"non json"; line: 1, column: 5]
```

but because we are running consumer with a `--skip-message-on-error` flag it should not die and continue listening to new records

unfortunatelly there is no way to produce messages with keys from simple files, if you will look at [sources](https://github.com/apache/kafka/blob/trunk/connect/file/src/main/java/org/apache/kafka/connect/file/FileStreamSourceTask.java#L153) you will see that `null` is passed as key

If you wish to have keys you should run configured console producer and pipe file contents into it

### Replaying Avro Messages With Key Value

This particular example does not use Kafka Connect but still might be used to replay some sequence of messages

Lets suppose that our `source.txt` file will look like:

**source.txt**

```
{"id":1}|{"foo":"hello"}
{"id":2}|{"foo":"world"}
```

where each line is an message with key and value separated by pipe

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic AvroFromFile --partitions 3 --replication-factor 1
```

Start `kafka-avro-console-consumer` to consume avro messages from file

```bash
docker run -it --rm --net=host confluentinc/cp-schema-registry:5.3.2 kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic AvroFromFile --from-beginning --property print.key=true
```

Start `kafka-avro-console-producer` which will produce avro messages from a file

```bash
docker run -it --rm --net=host -v $PWD:/data -w /data confluentinc/cp-schema-registry:5.3.2 sh -c "kafka-avro-console-producer --broker-list localhost:9092 --topic AvroFromFile --property value.schema='{\"type\":\"record\", \"name\": \"AvroFromFile\", \"fields\":[{\"name\":\"foo\",\"type\":\"string\"}]}' --property parse.key=true --property key.schema='{\"type\":\"record\",\"name\": \"key\", \"fields\":[{\"name\":\"id\",\"type\":\"int\"}]}' --property key.separator=\"|\" < source.txt"
```

And you should see your desired messages in consumer:

```
{"id":1}	{"foo":"hello"}
{"id":2}	{"foo":"world"}
```

Note that I have used `sh -c "...."` here because of bash can not understand whether last `< source.txt` should be ran inside docker or not

### Kafka Connect Source DataGen Avro

In following example we are going to generate tousand of recods based on given avro schema

**source.properties**

```ini
name=source
connector.class=io.confluent.kafka.connect.datagen.DatagenConnector
kafka.topic=AvroDatagen
# override worker.properties
value.converter=io.confluent.connect.avro.AvroConverter
value.converter.schema.registry.url=http://localhost:8081
key.converter=io.confluent.connect.avro.AvroConverter
key.converter.schema.registry.url=http://localhost:8081
# number of messages to generate
iterations=1000
tasks.max=1
# avro schema
schema.filename=/data/AvroDatagen.avsc
```

Some additional properties can be found [here](https://docs.confluent.io/current/schema-registry/connect.html)

Note that by default `auto.register.schemas` is set to `true` so you do not need to register schemas upfront everything will be done automatically. Also note that both `key.subject.name.strategy` and `value.subject.name.strategy` are set to `io.confluent.kafka.serializers.subject.SubjectNameStrategy` so schema names will be `AvroDatagen-key` and `AvroDatagen-value` retrospectively.

**AvroDatagen.avsc**

```
{
  "type": "record",
  "name": "AvroDatagen",
  "namespace": "ua.rabota.topics",
  "fields": [
    {
      "name": "userId",
      "type": {
        "type": "int",
        "arg.properties": {
          "range": {
            "min": 1,
            "max": 100
          }
        }
      }
    },
    {
      "name": "vacancyId",
      "type": {
        "type": "long",
        "arg.properties": {
          "range": {
            "min": 7710732,
            "max": 7711732
          }
        }
      }
    },
    {
      "name": "platform",
      "type": ["null", {
        "type": "string",
        "arg.properties": {
          "options": ["desktop", "mobile", "ios", "android"]
        }
      }],
      "default": null
    }
  ]
}
```

Note that usually in avro schema you defining properties like `{"name": "foo", "type": "string"}` where `type` is usually primitive string with type name, for datagen we are describing type as object with additional `arg.properties`

- [avro schema examples](https://github.com/confluentinc/kafka-connect-datagen/tree/master/src/main/resources)
- [connector config examples](https://github.com/confluentinc/kafka-connect-datagen/tree/0.2.x/config)
- [avro generator args](https://github.com/confluentinc/avro-random-generator)

Crate topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic AvroDatagen --partitions 3 --replication-factor 1
```

Start consumer

```bash
docker exec -it kafka kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic AvroDatagen --from-beginning
```

Start avro datagen producer

```bash
docker run -it --rm \
    --name=standalone \
    --net=host \
    -v $PWD:/data \
    -w /data \
    confluentinc/cp-kafka-connect:5.3.2 bash -c "confluent-hub install --no-prompt confluentinc/kafka-connect-datagen:0.2.0 && connect-standalone worker.properties source.properties"
```

Note how we are installing `kafka-connect-datagen` before starting `connect-standalone` it does not shipped by deafult

After a while, when everything will boot up you should see incomming messages in consumer

When datagen will produce desired 1000 messages it will die and you will see something like:

```
[2020-01-04 11:22:37,984] ERROR WorkerSourceTask{id=source-0} Task threw an uncaught and unrecoverable exception (org.apache.kafka.connect.runtime.WorkerTask:179)
org.apache.kafka.connect.errors.ConnectException: Stopping connector: generated the configured 1000 number of messages
```

Unfortunatelly datagen is quite limited about keys only way you can have keys is to provide `schema.keyfield` which will use one of generated properties as message key, and according to [sources](https://github.com/confluentinc/kafka-connect-datagen/blob/0.2.x/src/main/java/io/confluent/kafka/connect/datagen/DatagenTask.java#L255) it still will be simple string key.

### Kafka Connect Simple Sink To Text File

This might be used for debug and log

**sink.properties**

```ini
name=sink
connector.class=org.apache.kafka.connect.file.FileStreamSinkConnector
tasks.max=1
topics=SinkDemo
file=/data/data.txt
```

Create topic

```bash
docker exec -it kafka kafka-topics --bootstrap-server localhost:9092 --create --topic SinkDemo --partitions 3 --replication-factor 1
```

Start Kafka Connect Sink to save messages to a text file

```bash
docker run -it --rm \
    --name=standalone \
    --net=host \
    -v $PWD:/data \
    -w /data \
    confluentinc/cp-kafka-connect:5.3.2 connect-standalone worker.properties sink.properties
```

Start console producer

```bash
docker exec -it kafka kafka-console-producer --broker-list localhost:9092 --topic SinkDemo
```

and start typing messages into it, you should immediatelly see them in text file

Do not forget that you can run some tricky setups like `connect-standalone worker.properties source.properties sink.properties` which might generate data into topic and immediatelly sink them into source

# Standalone connect worker with confluent.cloud

All previous examples should work well with confluent.cloud if you will provide required configuration options

What you gonna need

**cloud.properties**

```ini
bootstrap.servers=xxx-xxxxx.us-east1.gcp.confluent.cloud:9092
ssl.endpoint.identification.algorithm=https
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
```

this file will be used by `kafka-topics` to create topic

**worker.properties**

```ini
bootstrap.servers=xxx-xxxxx.us-east1.gcp.confluent.cloud:9092
plugin.path=/usr/share/java,/usr/share/confluent-hub-components

offset.storage.file.filename=/tmp/standalone.offsets

# TODO: check whether this is a deafults
# default 60000
offset.flush.interval.ms=10000
# default 40000
request.timeout.ms=20000
# 100
retry.backoff.ms=500
consumer.request.timeout.ms=20000
consumer.retry.backoff.ms=500
producer.request.timeout.ms=20000
producer.retry.backoff.ms=500

# deafult https
ssl.endpoint.identification.algorithm=https
# default PLAINTEXT
security.protocol=SASL_SSL
# default GSSAPI
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";

# Connect producer and consumer specific configuration
producer.ssl.endpoint.identification.algorithm=https
producer.confluent.monitoring.interceptor.ssl.endpoint.identification.algorithm=https
consumer.ssl.endpoint.identification.algorithm=https
consumer.confluent.monitoring.interceptor.ssl.endpoint.identification.algorithm=https
producer.security.protocol=SASL_SSL
producer.confluent.monitoring.interceptor.security.protocol=SASL_SSL
consumer.security.protocol=SASL_SSL
consumer.confluent.monitoring.interceptor.security.protocol=SASL_SSL
producer.sasl.mechanism=PLAIN
producer.confluent.monitoring.interceptor.sasl.mechanism=PLAIN
consumer.sasl.mechanism=PLAIN
consumer.confluent.monitoring.interceptor.sasl.mechanism=PLAIN
producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
producer.confluent.monitoring.interceptor.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
consumer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
consumer.confluent.monitoring.interceptor.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";

# Confluent Schema Registry for Kafka Connect
value.converter=io.confluent.connect.avro.AvroConverter
value.converter.basic.auth.credentials.source=USER_INFO
value.converter.schema.registry.basic.auth.user.info=xxxxxxx:xxxxxxx
value.converter.schema.registry.url=https://xxxx-xxxxx.us-east1.gcp.confluent.cloud

key.converter=io.confluent.connect.avro.AvroConverter
key.converter.basic.auth.credentials.source=USER_INFO
key.converter.schema.registry.basic.auth.user.info=xxxxxxx:xxxxxxx
key.converter.schema.registry.url=https://xxxx-xxxxx.us-east1.gcp.confluent.cloud


# additions - https://docs.confluent.io/current/cloud/connect/connect-cloud-config.html

confluent.topic.bootstrap.servers=xxx-xxxxx.us-east1.gcp.confluent.cloud:9092
confluent.topic.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
confluent.topic.security.protocol=SASL_SSL
confluent.topic.sasl.mechanism=PLAIN

reporter.admin.bootstrap.servers=xxx-xxxxx.us-east1.gcp.confluent.cloud:9092
reporter.admin.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
reporter.admin.security.protocol=SASL_SSL
reporter.admin.sasl.mechanism=PLAIN

reporter.producer.bootstrap.servers=xxx-xxxxx.us-east1.gcp.confluent.cloud:9092
reporter.producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username\="xxxxxxx" password\="xxxxxxx";
reporter.producer.security.protocol=SASL_SSL
reporter.producer.sasl.mechanism=PLAIN
```

this one is for worker to be able to comminicate with confluent cloud

**source.properties**

```ini
name=source
tasks.max=1

connector.class=io.confluent.kafka.connect.datagen.DatagenConnector
kafka.topic=demo1
iterations=1000
schema.filename=/data/demo1.avsc
```

this one will be used by datagen connector to generate random data into given topic

**sink.properties**

```ini
name=sink
connector.class=org.apache.kafka.connect.file.FileStreamSinkConnector
topics=demo1
file=/data/data.txt
```

sing generated messages back from cloud to local file

**demo1.avsc**

```
{
  "type": "record",
  "name": "demo1",
  "namespace": "ua.rabota.topics",
  "fields": [
    {
      "name": "userId",
      "type": {
        "type": "int",
        "arg.properties": {
          "range": {
            "min": 1,
            "max": 100
          }
        }
      }
    },
    {
      "name": "vacancyId",
      "type": {
        "type": "long",
        "arg.properties": {
          "range": {
            "min": 7710732,
            "max": 7711732
          }
        }
      }
    },
    {
      "name": "platform",
      "type": ["null", {
        "type": "string",
        "arg.properties": {
          "options": ["desktop", "mobile", "ios", "android"]
        }
      }],
      "default": null
    }
  ]
}
```

schema for messages to be generated

create topic

```bash
docker run -it --rm -v $PWD/cloud.properties:/cloud.properties confluentinc/cp-kafka:5.3.2 kafka-topics \
  --bootstrap-server xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
  --command-config /cloud.properties \
  --create --topic demo1  --partitions 3 --replication-factor 3
```

start worker

```bash
docker run -it --rm \
    --name=standalone \
    -v $PWD:/data \
    -w /data \
    confluentinc/cp-kafka-connect:5.3.2 bash -c "confluent-hub install --no-prompt confluentinc/kafka-connect-datagen:0.2.0 && connect-standalone worker.properties source.properties sink.properties"
```

after a while you will see that your data.txt file becomes full of random generated messages

So now you can quickly send batch of messages both generated and predefined not only to local kafka but also to your confluent cloud one - profit

# Distributed Worker

Confluent cloud not giving you distributed workers for some reasons. Seems like it is because they do not know how much of them you gonna need. To start your own connect cluster you will need `worker.properties` from previous example just remove `offset.storage.file.filename` from it and add

```
group.id=mac1
offset.storage.topic=mac1-offsets
config.storage.topic=mac1-configs
status.storage.topic=mac1-status

offset.storage.partitions=3
replication.factor=3
config.storage.replication.factor=3
offset.storage.replication.factor=3
status.storage.replication.factor=3
```

take a closer look to first four settings, make sure they are unique

The difference between standalone and distributed worker is that from now you going to add and remove your tasks via [rest api](https://docs.confluent.io/current/connect/references/restapi.html)

In most of the cases everything will look the same as in previous examples, except that now you are going to post json instead of property files like in example from docs:

```
POST /connectors HTTP/1.1
Host: connect.example.com
Content-Type: application/json
Accept: application/json

{
    "name": "hdfs-sink-connector",
    "config": {
        "connector.class": "io.confluent.connect.hdfs.HdfsSinkConnector",
        "tasks.max": "10",
        "topics": "test-topic",
        "hdfs.url": "hdfs://fakehost:9000",
        "hadoop.conf.dir": "/opt/hadoop/conf",
        "hadoop.home": "/opt/hadoop",
        "flush.size": "100",
        "rotate.interval.ms": "1000"
    }
}
```

Here is an example of docker run which is a good starting point to run your connect cluster in kubernetes

```bash
docker run -it --rm \
    --name=mac1 \
    -p 8083:8083 \
    -e CONNECT_BOOTSTRAP_SERVERS=xxx-xxxxx.us-east1.gcp.confluent.cloud:9092 \
    -e CONNECT_GROUP_ID=mac1 \
    -e CONNECT_OFFSET_STORAGE_TOPIC=mac1-offsets \
    -e CONNECT_CONFIG_STORAGE_TOPIC=mac1-configs \
    -e CONNECT_STATUS_STORAGE_TOPIC=mac1-status \
    -e CONNECT_OFFSET_STORAGE_PARTITIONS=3 \
    -e CONNECT_REPLICATION_FACTOR=3 \
    -e CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR=3 \
    -e CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR=3 \
    -e CONNECT_STATUS_STORAGE_REPLICATION_FACTOR=3 \
    -e CONNECT_OFFSET_FLUSH_INTERVAL_MS=10000 \
    -e CONNECT_REQUEST_TIMEOUT_MS=20000 \
    -e CONNECT_RETRY_BACKOFF_MS=500 \
    -e CONNECT_CONSUMER_REQUEST_TIMEOUT_MS=20000 \
    -e CONNECT_CONSUMER_RETRY_BACKOFF_MS=500 \
    -e CONNECT_PRODUCER_REQUEST_TIMEOUT_MS=20000 \
    -e CONNECT_PRODUCER_RETRY_BACKOFF_MS=500 \
    -e CONNECT_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM=https \
    -e CONNECT_SECURITY_PROTOCOL=SASL_SSL \
    -e CONNECT_SASL_MECHANISM=PLAIN \
    -e CONNECT_SASL_JAAS_CONFIG="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"xxxxxxx\" password=\"xxxxxxx\";" \
    -e CONNECT_PRODUCER_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM=https \
    -e CONNECT_PRODUCER_CONFLUENT_MONITORING_INTERCEPTOR_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM=https \
    -e CONNECT_CONSUMER_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM=https \
    -e CONNECT_CONSUMER_CONFLUENT_MONITORING_INTERCEPTOR_SSL_ENDPOINT_IDENTIFICATION_ALGORITHM=https \
    -e CONNECT_PRODUCER_SECURITY_PROTOCOL=SASL_SSL \
    -e CONNECT_PRODUCER_CONFLUENT_MONITORING_INTERCEPTOR_SECURITY_PROTOCOL=SASL_SSL \
    -e CONNECT_CONSUMER_SECURITY_PROTOCOL=SASL_SSL \
    -e CONNECT_CONSUMER_CONFLUENT_MONITORING_INTERCEPTOR_SECURITY_PROTOCOL=SASL_SSL \
    -e CONNECT_PRODUCER_SASL_MECHANISM=PLAIN \
    -e CONNECT_PRODUCER_CONFLUENT_MONITORING_INTERCEPTOR_SASL_MECHANISM=PLAIN \
    -e CONNECT_CONSUMER_SASL_MECHANISM=PLAIN \
    -e CONNECT_CONSUMER_CONFLUENT_MONITORING_INTERCEPTOR_SASL_MECHANISM=PLAIN \
    -e CONNECT_PRODUCER_SASL_JAAS_CONFIG="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"xxxxxxx\" password=\"xxxxxxx\";" \
    -e CONNECT_PRODUCER_CONFLUENT_MONITORING_INTERCEPTOR_SASL_JAAS_CONFIG="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"xxxxxxx\" password=\"xxxxxxx\";" \
    -e CONNECT_CONSUMER_SASL_JAAS_CONFIG="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"xxxxxxx\" password=\"xxxxxxx\";" \
    -e CONNECT_CONSUMER_CONFLUENT_MONITORING_INTERCEPTOR_SASL_JAAS_CONFIG="org.apache.kafka.common.security.plain.PlainLoginModule required username=\"xxxxxxx\" password=\"xxxxxxx\";" \
    -e CONNECT_VALUE_CONVERTER=io.confluent.connect.avro.AvroConverter \
    -e CONNECT_VALUE_CONVERTER_BASIC_AUTH_CREDENTIALS_SOURCE=USER_INFO \
    -e CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_BASIC_AUTH_USER_INFO=xxxxxxx:xxxxxxx \
    -e CONNECT_VALUE_CONVERTER_SCHEMA_REGISTRY_URL=https://xxxx-xxxxx.us-east1.gcp.confluent.cloud \
    -e CONNECT_KEY_CONVERTER=io.confluent.connect.avro.AvroConverter \
    -e CONNECT_KEY_CONVERTER_SCHEMAS_ENABLE=true \
    -e CONNECT_VALUE_CONVERTER_SCHEMAS_ENABLE=true \
    -e CONNECT_REST_POST=8083 \
    -e CONNECT_REST_ADVERTISED_HOST_NAME=localhost \
    -e CONNECT_KEY_CONVERTER_BASIC_AUTH_CREDENTIALS_SOURCE=USER_INFO \
    -e CONNECT_KEY_CONVERTER_SCHEMA_REGISTRY_BASIC_AUTH_USER_INFO=xxxxxxx:xxxxxxx \
    -e CONNECT_KEY_CONVERTER_SCHEMA_REGISTRY_URL=https://xxxx-xxxxx.us-east1.gcp.confluent.cloud
    confluentinc/cp-kafka-connect:5.3.2
```

# Bash aliases

Even simple operations like creating topic becomes not easy to remember especially if you will have local, dev, prod kafka clusters

If you are using [ccloud](https://docs.confluent.io/current/cloud/cli/command-reference/ccloud.html) command line tool you already should have `~/.ccloud/` which to me seems a good place to save my `cloud.properties` files in my case it will be `dev.properties` and `prod.peroperties`

Here are few starting point examples

## Local kafka bash aliases

```bash
alias local-topic="docker run -it --rm --net=host confluentinc/cp-kafka:5.3.2 kafka-topics --bootstrap-server localhost:9092"

alias local-topic-list="local-topic --list"

alias local-topic-delete="local-topic --delete --topic"

alias local-topic-describe="local-topic --describe --topic"

alias local-topic-create="local-topic --create --replication-factor 1 --topic"

alias local-topic-create1="local-topic --create --replication-factor 1 --partitions 1 --topic"

alias local-topic-create2="local-topic --create --replication-factor 1 --partitions 2 --topic"

alias local-console-consumer="docker run -it --rm --net=host confluentinc/cp-kafka:5.3.2 kafka-console-consumer --bootstrap-server localhost:9092 --from-beginning --topic"

alias local-console-producer="docker run -it --rm --net=host confluentinc/cp-kafka:5.3.2 kafka-console-producer --broker-list localhost:9092 --topic"
```

## Confluent cloud kafka bash aliases

```bash
alias dev-topic="docker run -it --rm -v /Users/mac/.ccloud/dev.properties:/dev.properties confluentinc/cp-kafka:5.3.2 kafka-topics --bootstrap-server $(grep bootstrap.server ~/.ccloud/dev.properties | tail -1 | cut -d'=' -f2) --command-config dev.properties"

alias dev-topic-list="dev-topic --list"

alias dev-topic-delete="dev-topic --delete --topic"

alias dev-topic-describe="dev-topic --describe --topic"

alias dev-topic-create="dev-topic --create  --replication-factor 3 --topic"

alias dev-console-consumer="docker run -it --rm -v /Users/mac/.ccloud/dev.properties:/dev.properties confluentinc/cp-kafka:5.3.2 kafka-console-consumer --bootstrap-server $(grep bootstrap.server ~/.ccloud/dev.properties | tail -1 | cut -d'=' -f2) --consumer.config dev.properties --topic"

alias dev-console-producer="docker run -it --rm -v /Users/mac/.ccloud/dev.properties:/dev.properties confluentinc/cp-kafka:5.3.2 kafka-console-producer --broker-list $(grep bootstrap.server ~/.ccloud/dev.properties | tail -1 | cut -d'=' -f2) --producer.config dev.properties --topic"
```
