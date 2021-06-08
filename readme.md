## Kafka Streams Demo

### Agenda
- Demo Overview
- How to run Demo On Prem
- Review Docker Compose file
- Explore Control Center
- Kafka Platform
- Application Development
  - Admin
  - Producer
  - Consumer
  - Avro schema
  - Streams Topology
  - Streams API 
- Review KStream Code
- Q & A

### Demo Overview
https://docs.confluent.io/platform/current/tutorials/cp-demo/docs/overview.html

### How to run Demo On Prem
https://docs.confluent.io/platform/current/tutorials/cp-demo/docs/on-prem.html

### Review Docker Compose file
This demo uses 15 docker images.
[Full version](./docker-compose.yml)
```yml
  zookeeper:
    image: ${REPOSITORY}/cp-zookeeper:${CONFLUENT_DOCKER_TAG}
  tools:
    image: cnfltraining/training-tools:5.5
  openldap:
    image: osixia/openldap:1.3.0
  kafka1:
    image: ${REPOSITORY}/cp-server:${CONFLUENT_DOCKER_TAG}
  kafka2:
    image: ${REPOSITORY}/cp-server:${CONFLUENT_DOCKER_TAG}
  connect:
    image: localbuild/connect:${CONFLUENT_DOCKER_TAG}-${CONNECTOR_VERSION}
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.6.0
  kibana:
    image: docker.elastic.co/kibana/kibana:5.5.2
  control-center:
    image: ${REPOSITORY}/cp-enterprise-control-center:${CONFLUENT_DOCKER_TAG}
  schemaregistry:
    image: ${REPOSITORY}/cp-schema-registry:${CONFLUENT_DOCKER_TAG}
  kafka-client:
    image: ${REPOSITORY}/cp-server:${CONFLUENT_DOCKER_TAG}
  ksqldb-server:
    image: ${REPOSITORY}/cp-ksqldb-server:${CONFLUENT_DOCKER_TAG}
  ksqldb-cli:
    image: ${REPOSITORY}/cp-ksqldb-cli:${CONFLUENT_DOCKER_TAG}
  restproxy:
    image: ${REPOSITORY}/cp-kafka-rest:${CONFLUENT_DOCKER_TAG}
  streams-demo:
    image: cnfldemos/cp-demo-kstreams:0.0.6
```

### Explore Control Center
**Control Center**: http://192.168.1.110:9021/
Log in as userName/password: superUser/superUser

**Kibana** http://192.168.1.110:5601/app/dashboards#/view/Overview


**Note:** This should be a localhost or your host IP.
#### Kafka Streaming Platform
**Distributed System**  
The following diagram illustrates how the Kafka cluster is organized. Kafka cluster is made up of
multiple hosts called as brokers in a distributed arrangement. Kafka stores the messages in the logical unit 
called Topics. The topics are typically divided into multiple partitions and replicated across several hosts.
The partitioning, replication and the distribution of messages make Kafka into a highly scalable and resilient 
system. 

```ditaa {cmd=true args=["-E"]}

                    +--------------+     
                    | Applications |
                    +-------+------+
                            ^  
                            | Communicate using Kafka API                           
   Kafka cluster            v
/---------------------------+-----------------------------------------\
| cGRE                                                                |
| Physical system made up of multiple machines                        |
| +----------+     +----------+                                       |
| + Broker 1 | ... + Broker n |                                       |
| +----------+     +----------+                                       |
+---------------------------------------------------------------------+
| cBLU                                                                |
| Distributed Logs                                                    |
| +-----------------------------------------------------------------+ |
| | Topic "topic_name_A"                                            | |
| | +---------------------------+     +---------------------------+ | |
| | | Partition 0               | ... | Partition n               | | |
| | | +---+---+---+---+---+---+ |     | +---+---+---+---+---+---+ | | |
| | | | 0 | 1 | 2 | 3 | . | n | |     | | 0 | 1 | 2 | 3 | . | n | | | |
| | | +---+---+---+---+---+---+ |     | +---+---+---+---+---+---+ | | |
| | +---------------------------+     +---------------------------+ | |
| +-----------------------------------------------------------------+ |
|                                  .                                  |
|                                  .                                  |
|                                  .                                  |
| +-----------------------------------------------------------------+ |
| | Topic "topic_name_Z"                                            | |
| | +---------------------------+     +---------------------------+ | |
| | | Partition 0               | ... | Partition n               | | |
| | | +---+---+---+---+---+---+ |     | +---+---+---+---+---+---+ | | |
| | | | 0 | 1 | 2 | 3 | . | n | |     | | 0 | 1 | 2 | 3 | . | n | | | |
| | | +---+---+---+---+---+---+ |     | +---+---+---+---+---+---+ | | |
| | +---------------------------+     +---------------------------+ | |
| +-----------------------------------------------------------------+ |
\---------------------------------------------------------------------/

```
**Figure 1. Kafka System Diagram**

## Client Operations

### Producer
The applications that write messages to Kafka topics are known as the producers. The topics are persistent logs and Kafka allows the write operations to only append the log as shown in the following diagram.


```ditaa {cmd=true args=["-E"]}

              Topic "topic_name"
              +---+---+---+---+---+---+---+---+---+---+---+---+---+-=--+
Partition 0   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 | 13 +<----+
              +---+---+---+---+---+---+---+---+---+---+---+---+---+----+     |
                                                                             |
              +---+---+---+---+---+---+---+---+---+---+-=-+                  |
Partition 1   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 +<-----------------+
              +---+---+---+---+---+---+---+---+---+---+---+                  | Writes are
                                                                             +-append only
              +---+---+---+---+---+---+---+---+-=-+                          |
Partition 2   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 +<-------------------------+
              +---+---+---+---+---+---+---+---+---+                          |
                                                                             |
              +---+---+---+---+---+---+---+---+---+---+---+-=-+              |
Partition 3   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 +<-------------+
              +---+---+---+---+---+---+---+---+---+---+---+---+


```
**Figure 2. Kafka Producer**

### Consumer
The applications that read messages from the Kafka topics are known as consumers. Kafka maintains the last read 
locations of the consumers and lets them read the next messages sequentially.
```ditaa {cmd=true args=["-E"]}
              Topic "topic_name"
              +---+---+---+---+---+---+---+---+---+---+---+---+---+-=--+
Partition 0   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 | 13 +
              +---+---+---+---+---+-+-+---+---+---+---+---+---+---+----+     /--------------\
                                    ^                                        |Consumer Group|
                                    |                                        | +----------+ |
                                    +----------------------------------------|-+Consumer 0| |
              +---+---+---+---+---+---+---+---+---+---+-=-+                  | +----------+ |
Partition 1   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |                  |              |
              +---+---+---+---+---+---+---+---+---+---+---+                  |              |
                                        ^                                    |              |
                                        |                                    | +----------+ |
                                +-------+------------------------------------|-+Consumer 1| |
                                |                                            | +----------+ |
                                v                                            |              |
              +---+---+---+---+-+-+---+---+---+-=-+                          |              |
Partition 2   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |                          |              |
              +---+---+---+---+---+---+---+---+---+                          | +----------+ |
                                                    +------------------------|-+Consumer 2| |
                                                    |                        | +----------+ |  
                                                    v                        \--------------/
              +---+---+---+---+---+---+---+---+---+-+-+---+-=-+
Partition 3   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |
              +---+---+---+---+---+---+---+---+---+---+---+---+

```
**Figure 3. Kafka Consumer**

## Messaging
For Kafka message is simply an array of bytes and producers can choose to represent messages in any data type that can be serialized to byte streams. Conversely consumers can read any message that it can de-serialize to its data type. To facilitate data exchange between diverse systems and programming languages, a standard serialization and de-serialization is required that all the systems can support. PuSH uses Apache Avro serialization system supported by Confluent Kafka to stream business domain objects.

### AVRO
Apache Avro uses schemas to map the domain structures and the serialized binary data format. While the binary format helps to efficiently exchange messages across systems, the schemas help applications to handle the messages as business domain objects.  

The following diagram shows this relationship between these components within the Avro serialization scheme. 
## Avro Serialization 
```ditaa {cmd=true args=["-E"]}
 +----------+      Binary Stream
 | Business |      +---+---+---+---+---+---+---+---+---+---+
 | Object   +<---->+ 0 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | . | . |
 +----------+      +---+---+---+---+---+---+---+---+---+---+
       ^
       | 
 +----------+
 |  Avro    |
 |  Schema  |
 |{d}       |
 +----------+
```

```ditaa {cmd=true args=["-E"]}
+--------------+
| Applications |
+--------------+
       |                               Binary Stream                             
  +----------+      +-----------+      +---+---+---+---+---+---+---+---+---+---+
  | Business +<---->+ Confluent |<---->+ 0 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | . | . |
  | Objects  |      | Kafka     |      +---+---+---+---+---+---+---+---+---+---+
  +----------+      +-----------+
                          ^
                          |
                    +------------+
                    | Schema     |
                    | Registry   |
                    |{s}         |
                    | +--------+ |
                    | | Avro   | |
                    | | Schemas| |
                    | |{d}     | |
                    | +--------+ |
                    +------------+
```

### Application Development

#### Admin
- kafka-consumer-groups
```sh
docker-compose exec kafka1 kafka-consumer-groups \
   --list \
   --bootstrap-server kafka1:9091 \
   --command-config /etc/kafka/secrets/client_sasl_plain.config
```

- kafka-topics
  - list topics
```sh
docker-compose exec kafka1 kafka-topics \
  --list \
  --zookeeper zookeeper:2181 \
  --command-config /etc/kafka/secrets/client_sasl_plain.config

```
- 
  - create topic
```sh
docker-compose exec kafka1 kafka-topics \
      --create --if-not-exists \
      --zookeeper zookeeper:2181 \
      --partitions 1 \
      --replication-factor 2 \
      --topic TEST_TOPIC

docker-compose exec kafka1 kafka-topics \
      --create --if-not-exists \
      --zookeeper zookeeper:2181 \
      --partitions 2 \
      --replication-factor 2 \
      --config retention.ms=2592000 \
      --topic DEMO_TOPIC
```

**Note** ERROR InvalidReplicationFactorException: Replication factor: 3 larger than available brokers: 2.

- 
  - delete topic
```sh
docker-compose exec kafka1 kafka-topics \
      --delete \
      --zookeeper zookeeper:2181 \
      --topic DEMO_TOPIC
```

#### Producer
- kafka-console-producer
```
docker-compose exec kafka1 sh

echo "test 1" | kafka-console-producer \
--broker-list kafka1:9091, kafka2:9092 \
--producer.config /etc/kafka/secrets/client_sasl_plain.config \
--topic TEST_TOPIC

```

#### Consumer
- kafka-console-consumer
```sh
docker-compose exec kafka1 kafka-console-consumer \
--bootstrap-server kafka1:9091, kafka2:9092 \
--from-beginning \
--consumer.config /etc/kafka/secrets/client_sasl_plain.config \
--topic TEST_TOPIC

```

```sh
docker-compose exec connect kafka-avro-console-consumer \
  --bootstrap-server kafka1:11091,kafka2:11092 \
  --consumer-property security.protocol=SSL \
  --consumer-property ssl.truststore.location=/etc/kafka/secrets/kafka.appSA.truststore.jks \
  --consumer-property ssl.truststore.password=confluent \
  --consumer-property ssl.keystore.location=/etc/kafka/secrets/kafka.appSA.keystore.jks \
  --consumer-property ssl.keystore.password=confluent \
  --consumer-property ssl.key.password=confluent \
  --property schema.registry.url=https://schemaregistry:8085 \
  --property schema.registry.ssl.truststore.location=/etc/kafka/secrets/kafka.appSA.truststore.jks \
  --property schema.registry.ssl.truststore.password=confluent \
  --property basic.auth.credentials.source=USER_INFO \
  --property basic.auth.user.info=appSA:appSA \
  --group wikipedia.test \
  --topic wikipedia.parsed \
  --from-beginning \
  --max-messages 5
```

#### Avro Schema
- Example Schema 1
```json
{
  "type": "record",
  "name": "WikiFeedMetric",
  "namespace": "io.confluent.demos.common.wiki",
  "fields": [
    {
      "name": "domain",
      "type": "string",
      "doc": "Associated domain"
    },
    {
      "name": "editCount",
      "type": "long",
      "doc": "The count of edits for the domain"
    }
  ]
}
```

- Example Message 1
```json
{
  "domain": "bn.wikipedia.org",
  "editCount": 31
}
```

- Example Schema 2
```json
{
  "type": "record",
  "name": "WikiEdit",
  "namespace": "io.confluent.demos.common.wiki",
  "fields": [
    {
      "default": null,
      "name": "bot",
      "type": [
        "null",
        "boolean"
      ]
    },
    {
      "default": null,
      "name": "comment",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "id",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "default": null,
      "name": "LOG_ACTION",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "LOG_ACTION_COMMENT",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "LOG_ID",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "default": null,
      "name": "LOG_TYPE",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "meta",
      "type": [
        "null",
        {
          "fields": [
            {
              "default": null,
              "name": "domain",
              "type": [
                "null",
                "string"
              ]
            },
            {
              "default": null,
              "name": "dt",
              "type": [
                "null",
                "long"
              ]
            },
            {
              "default": null,
              "name": "ID",
              "type": [
                "null",
                "string"
              ]
            },
            {
              "default": null,
              "name": "REQUEST_ID",
              "type": [
                "null",
                "string"
              ]
            },
            {
              "default": null,
              "name": "STREAM",
              "type": [
                "null",
                "string"
              ]
            },
            {
              "default": null,
              "name": "uri",
              "type": [
                "null",
                "string"
              ]
            }
          ],
          "name": "KsqlDataSourceSchema_META",
          "type": "record"
        }
      ]
    },
    {
      "default": null,
      "name": "minor",
      "type": [
        "null",
        "boolean"
      ]
    },
    {
      "default": null,
      "name": "NAMESPACE",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "default": null,
      "name": "PARSEDCOMMENT",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "patrolled",
      "type": [
        "null",
        "boolean"
      ]
    },
    {
      "default": null,
      "name": "REVISION",
      "type": [
        "null",
        {
          "fields": [
            {
              "default": null,
              "name": "NEW",
              "type": [
                "null",
                "long"
              ]
            },
            {
              "default": null,
              "name": "OLD",
              "type": [
                "null",
                "long"
              ]
            }
          ],
          "name": "KsqlDataSourceSchema_REVISION",
          "type": "record"
        }
      ]
    },
    {
      "default": null,
      "name": "SERVER_NAME",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "SERVER_SCRIPT_PATH",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "SERVER_URL",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "TIMESTAMP",
      "type": [
        "null",
        "long"
      ]
    },
    {
      "default": null,
      "name": "TITLE",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "TYPE",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "user",
      "type": [
        "null",
        "string"
      ]
    },
    {
      "default": null,
      "name": "WIKI",
      "type": [
        "null",
        "string"
      ]
    }
  ]
}

```

- Example Message 2
```json
{
  "bot": {
    "boolean": false
  },
  "comment": {
    "string": "Moving from [[Category:West Timor]] to [[Category:Maps of West Timor]]"
  },
  "id": {
    "long": 1702587869
  },
  "length": {
    "properties.length": {
      "new": {
        "long": 3309
      },
      "old": {
        "long": 3301
      }
    }
  },
  "log_action": null,
  "log_action_comment": null,
  "log_id": null,
  "log_type": null,
  "meta": {
    "domain": {
      "string": "commons.wikimedia.org"
    },
    "dt": 1623096942000,
    "id": "5e31d223-753d-49c4-a631-613b13ea6a8e",
    "request_id": {
      "string": "fa6bd694-a544-4084-9cb0-7f1f2e8bbdf4"
    },
    "stream": "mediawiki.recentchange",
    "uri": {
      "string": "https://commons.wikimedia.org/wiki/File:Kabupaten_Timor_Tengah_Utara_(Timor_Barat).png"
    }
  },
  "minor": {
    "boolean": true
  },
  "namespace": {
    "long": 6
  },
  "parsedcomment": {
    "string": "Moving from <a href=\"/wiki/Category:West_Timor\" title=\"Category:West Timor\">Category:West Timor</a> to <a href=\"/wiki/Category:Maps_of_West_Timor\" title=\"Category:Maps of West Timor\">Category:Maps of West Timor</a>"
  },
  "patrolled": {
    "boolean": true
  },
  "revision": {
    "properties.revision": {
      "new": {
        "long": 567815638
      },
      "old": {
        "long": 561173559
      }
    }
  },
  "server_name": {
    "string": "commons.wikimedia.org"
  },
  "server_script_path": {
    "string": "/w"
  },
  "server_url": {
    "string": "https://commons.wikimedia.org"
  },
  "timestamp": {
    "long": 1623096942
  },
  "title": {
    "string": "File:Kabupaten Timor Tengah Utara (Timor Barat).png"
  },
  "type": {
    "string": "edit"
  },
  "user": {
    "string": "Milenioscuro"
  },
  "wiki": {
    "string": "commonswiki"
  }
}
```

#### Topology
```ditaa {cmd=true args=["-E"]}
                +------------+ 
                | Wikipedia  | 
                +------+-----+ 
                       |
/--------------------------------------------------\
|                      |                           |
|                      v                           |
|               +------------+                     |
|  /------------| Connector  |                     |
|  |            +------------+                     |
|  |   wikipedia.parsed                            |
|  |   +-=-+---+---+---+---+---+---+---+---+---+   |  +-----------------------+
|  \-->+ . | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 +---|--+ KStreams Application  |
|      +---+---+---+---+---+---+---+---+---+---+   |  |{io}                   |
|                                                  |  +-----------+-----------+
|                                                  |              |
|  +-----------------------------------------------|--------------+
|  |                                               | 
|  |                                               |
|  |   wikipedia.parsed.count-by-domain            | 
|  |   +-=-+---+---+---+---+---+---+---+---+---+   | 
|  +---+ . | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |   | 
|      +---+---+---+---+---+---+---+---+---+---+   | 
| Kafka Cluster                                    |
| cEEE                                             |
\--------------------------------------------------/

```

### KStreams API
- DSL API
https://kafka.apache.org/28/documentation/streams/developer-guide/dsl-api.html

- Processor API
https://kafka.apache.org/documentation/streams/developer-guide/processor-api.html

#### DSL Example
```java
    builder.<String, GenericRecord>stream(INPUT_TOPIC)
       // INPUT_TOPIC has no key so use domain as the key
       .map((key, value) -> new KeyValue<>(((GenericRecord)value.get(META)).get(META_DOMAIN).toString(), value))
       .filter((key, value) -> !(boolean)value.get(BOT))
       .groupByKey()
       .count()
       .mapValues(WikiFeedMetric::new)
       .toStream()
       .peek((key, value) -> logger.debug("{}:{}", key, value.getEditCount()))
       .to(OUTPUT_TOPIC, Produced.with(Serdes.String(), metricSerde));
```


## References
1.  **Apache Kafka** https://kafka.apache.org/intro
2.  **Streams Concepts** https://docs.confluent.io/current/streams/concepts.html
3.  **Streaming 101** https://www.oreilly.com/radar/the-world-beyond-batch-streaming-101/
4.  **Apache Avro Specification** https://avro.apache.org/docs/current/spec.html
5.  **Avro4s** https://github.com/sksamuel/avro4s
6. **Mastering Kafka Streams and ksqlDB** https://www.confluent.io/resources/ebook/mastering-kafka-streams-and-ksqldb/
