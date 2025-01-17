# Stream ingestion

Apache Pinot allows user to consume data from streams and push it directly to pinot database. This process is known as Stream Ingestion. Stream Ingestion allows user to query data within seconds of publishing.

Stream Ingestion provides support for checkpoints out of the box for preventing data loss.

Stream ingestion requires the following steps -&#x20;

1. Create schema configuration
2. Create table configuration
3. Upload table and schema spec

Let's take a look at each of the following steps in a bit more detail. Let us assume the data to be ingested is in the following format -&#x20;

```bash
{"studentID":205,"firstName":"Natalie","lastName":"Jones","gender":"Female","subject":"Maths","score":3.8,"timestamp":1571900400000}
{"studentID":205,"firstName":"Natalie","lastName":"Jones","gender":"Female","subject":"History","score":3.5,"timestamp":1571900400000}
{"studentID":207,"firstName":"Bob","lastName":"Lewis","gender":"Male","subject":"Maths","score":3.2,"timestamp":1571900400000}
{"studentID":207,"firstName":"Bob","lastName":"Lewis","gender":"Male","subject":"Chemistry","score":3.6,"timestamp":1572418800000}
{"studentID":209,"firstName":"Jane","lastName":"Doe","gender":"Female","subject":"Geography","score":3.8,"timestamp":1572505200000}
{"studentID":209,"firstName":"Jane","lastName":"Doe","gender":"Female","subject":"English","score":3.5,"timestamp":1572505200000}
{"studentID":209,"firstName":"Jane","lastName":"Doe","gender":"Female","subject":"Maths","score":3.2,"timestamp":1572678000000}
{"studentID":209,"firstName":"Jane","lastName":"Doe","gender":"Female","subject":"Physics","score":3.6,"timestamp":1572678000000}
{"studentID":211,"firstName":"John","lastName":"Doe","gender":"Male","subject":"Maths","score":3.8,"timestamp":1572678000000}
{"studentID":211,"firstName":"John","lastName":"Doe","gender":"Male","subject":"English","score":3.5,"timestamp":1572678000000}
{"studentID":211,"firstName":"John","lastName":"Doe","gender":"Male","subject":"History","score":3.2,"timestamp":1572854400000}
{"studentID":212,"firstName":"Nick","lastName":"Young","gender":"Male","subject":"History","score":3.6,"timestamp":1572854400000}
```

### Create Schema Configuration

Schema defines the fields along with their data types which are available in the datasource. Schema also defines the fields which serve as `dimensions` , `metrics` and `timestamp` respectively.

Follow [creating a schema](../../getting-started/pushing-your-data-to-pinot.md#creating-a-schema) for more details on schema configuration. For our sample data, the schema configuration should look as follows&#x20;

{% code title="/tmp/pinot-quick-start/transcript-schema.json" %}
```bash
{
  "schemaName": "transcript",
  "dimensionFieldSpecs": [
    {
      "name": "studentID",
      "dataType": "INT"
    },
    {
      "name": "firstName",
      "dataType": "STRING"
    },
    {
      "name": "lastName",
      "dataType": "STRING"
    },
    {
      "name": "gender",
      "dataType": "STRING"
    },
    {
      "name": "subject",
      "dataType": "STRING"
    }
  ],
  "metricFieldSpecs": [
    {
      "name": "score",
      "dataType": "FLOAT"
    }
  ],
  "dateTimeFieldSpecs": [{
    "name": "timestamp",
    "dataType": "LONG",
    "format" : "1:MILLISECONDS:EPOCH",
    "granularity": "1:MILLISECONDS"
  }]
}
```
{% endcode %}

### Create Table Configuration

The next step is to create a table where all the ingested data will flow and can be queried. Unlike batch ingestion, table configuration for realtime ingestion also triggers the data ingestion job.For a more detailed overview about tables, check out the [table](../../components/table.md) reference.

The realtime table configuration consists of the the following fields -&#x20;

* **tableName** - The name of the table where the data should flow
* **tableType** - The internal type for the table. Should always be set to `REALTIME` for realtime ingestion
* **segmentsConfig** -&#x20;
* **tableIndexConfig** -  defines which column to use for indexing along with the type of index. You can refer \[Indexing Configs] for full configuration. It consists of the following _**required**_  fields -&#x20;
  *   **loadMode** - specifies how the segments should be loaded. Should be one of `heap` or `mmap`.  Here's the difference between both the configs

      ```
      heap: Segments are loaded on direct-memory. Note, 'heap' here is a legacy misnomer, and it does not
            imply JVM heap. This mode should only be used when we want faster performance than memory-mapped files,
             and are also sure that we will never run into OOM.
      mmap: Segments are loaded on memory-mapped file. This is the default mode.
      ```


  * **streamConfig** -  specifies the datasource along with the necessary configs to start consuming the realtime data. The streamConfig can be thought of as equivalent of job spec in case of batch ingestion. The following options are supported in this config - \


| Config key                                           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Supported values                                  |
| ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| streamType                                           | the streaming platform from  which to consume the data                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | `kafka`                                           |
| stream.\[streamType].consumer.type                   | whether to use per partition low-level consumer or high-level stream consumer                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | `lowLevel` or `highLevel`                         |
| stream.\[streamType].topic.name                      | the datasource (e.g. topic, data stream) from which to consume the data                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | String                                            |
| stream.\[streamType].decoder.class.name              | name of the class to be used for parsing the data. The class should implement `org.apache.pinot.spi.stream.StreamMessageDecoder` interface                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | String                                            |
| stream.\[streamType].consumer.factory.class.name     | name of the factory class to be used to provide the appropriate implementation of low level and high level consumer as well as the metadata                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | String                                            |
| stream.\[streamType].consumer.prop.auto.offset.reset | determines the offset from which to start the ingestion                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | `smallest` `largest` or timestamp in milliseconds |
| realtime.segment.flush.threshold.time                | Time threshold that will keep the realtime segment open for before we complete the segment                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |                                                   |
| realtime.segment.flush.threshold.size                | <p></p><p>Row count flush threshold for realtime segments. This behaves in a similar way for HLC and LLC. For HLC,</p><p>since there is only one consumer per server, this size is used as the size of the consumption buffer and determines after how many rows we flush to disk. For example, if this threshold is set to two million rows,</p><p>then a high level consumer would have a buffer size of two million.</p><p></p><p>If this value is set to 0, then the consumers adjust the number of rows consumed by a partition such that the size of the completed segment is the desired size (unless</p><p>   threshold.time is reached first)</p> |                                                   |
| realtime.segment.flush.desired.size                  | The desired size of a completed realtime segment.This config is used only if `threshold.size` is set to 0.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |                                                   |

You can also specify additional configs for the consumer by prefixing the key with stream.\[streamType] where `streamType` is the name of the streaming platform.  For our sample data and schema, the table config will look like -&#x20;

```javascript
{
  "tableName": "transcript",
  "tableType": "REALTIME",
  "segmentsConfig": {
    "timeColumnName": "timestamp",
    "timeType": "MILLISECONDS",
    "schemaName": "transcript",
    "replicasPerPartition": "1"
  },
  "tenants": {},
  "tableIndexConfig": {
    "loadMode": "MMAP",
    "streamConfigs": {
      "streamType": "kafka",
      "stream.kafka.consumer.type": "lowlevel",
      "stream.kafka.topic.name": "transcript-topic",
      "stream.kafka.decoder.class.name": "org.apache.pinot.plugin.stream.kafka.KafkaJSONMessageDecoder",
      "stream.kafka.consumer.factory.class.name": "org.apache.pinot.plugin.stream.kafka20.KafkaConsumerFactory",
      "stream.kafka.broker.list": "localhost:9876",
      "realtime.segment.flush.threshold.time": "3600000",
      "realtime.segment.flush.threshold.size": "50000",
      "stream.kafka.consumer.prop.auto.offset.reset": "smallest"
    }
  },
  "metadata": {
    "customConfigs": {}
  }
}
```

### Upload schema and table config

Now that we have our table and schema configurations, let's upload them to the Pinot cluster. As soon as the configs are uploaded, pinot will start ingesting available records from the topic.

{% tabs %}
{% tab title="Docker" %}
```
docker run \
    --network=pinot-demo \
    -v /tmp/pinot-quick-start:/tmp/pinot-quick-start \
    --name pinot-streaming-table-creation \
    apachepinot/pinot:latest AddTable \
    -schemaFile /tmp/pinot-quick-start/transcript-schema.json \
    -tableConfigFile /tmp/pinot-quick-start/transcript-table-realtime.json \
    -controllerHost pinot-quickstart \
    -controllerPort 9000 \
    -exec
```
{% endtab %}

{% tab title="Launcher Script" %}
```bash
bin/pinot-admin.sh AddTable \
    -schemaFile /path/to/transcript-schema.json \
    -tableConfigFile /path/to/transcript-table-realtime.json \
    -exec
```
{% endtab %}
{% endtabs %}

### Custom Ingestion Support&#x20;

We are working on adding more integrations such as Kinesis out of the box. You can easily write your on ingestion plugin in case it is not supported out of the box. Follow [Stream Ingestion Plugin](../../../developers/plugin-architecture/write-custom-plugins/write-your-stream.md) for a walkthrough.



