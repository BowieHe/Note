### TestCode
#### test spark shell start
```shell
spark-shell \
  --jars /opt/test/data-spark-jobs/lib/data-spark-jobs-cdh6.3.1.jar \
  --packages org.apache.hudi:hudi-spark-bundle_2.11:0.8.0,org.apache.spark:spark-avro_2.11:2.4.3,org.apache.spark:spark-streaming-kafka-0-10_2.11:2.4.3,org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.3 \
  --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer'
```

#### create table( spark SQL)
```
create table test_hudi_table (
  id int,
  name string,
  price double,
  ts long,
  dt string
) using hudi
 partitioned by (dt)
 options (
  primaryKey = 'id',
  type = 'mor'
 )
 location 'file:///tmp/test_hudi_table'
 
```

#### test code
```scala

import org.apache.spark.sql.types._
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._
import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.streaming.kafka010._
import org.apache.spark.streaming.kafka010.LocationStrategies.PreferConsistent
import org.apache.spark.streaming.kafka010.ConsumerStrategies.Subscribe
import org.apache.spark.sql.catalyst.encoders.RowEncoder
import org.apache.spark.sql.{DataFrame, Row, SaveMode}
import org.apache.spark.sql.functions._
import org.apache.spark.sql.types.{LongType, StringType, StructField, StructType}
import org.apache.hudi.QuickstartUtils._
import java.time.LocalDateTime
import scala.collection.JavaConversions._
import org.apache.spark.sql.SaveMode._
import org.apache.hudi.DataSourceReadOptions._
import org.apache.hudi.DataSourceWriteOptions._
import org.apache.hudi.config.HoodieWriteConfig
import org.apache.hudi.config.HoodieWriteConfig._
import org.apache.hudi.config.HoodieCompactionConfig
import org.apache.hudi.config.HoodieCompactionConfig._

val df = spark.read.format("org.apache.hudi").option(QUERY_TYPE_OPT_KEY, QUERY_TYPE_SNAPSHOT_OPT_VAL).load("/tmp/data_t1p873/test_mor/*")

val kafkaParams = Map[String, Object](
  "bootstrap.servers" -> "10.2.2.17:9092",
  "key.deserializer" -> classOf[StringDeserializer],
  "value.deserializer" -> classOf[StringDeserializer],
  "group.id" -> "data-t1-connector_sync_data_to_ods_groupId",
  "auto.offset.reset" -> "earliest",
  "enable.auto.commit" -> (false: java.lang.Boolean)
)

val ssc = new StreamingContext(sc, Seconds(1))

val topics = Array("data-t1-connector_sync_data_to_ods")
val stream = KafkaUtils.createDirectStream[String, String](
  ssc,
  PreferConsistent,
  Subscribe[String, String](topics, kafkaParams)
)

var index = 1 

val dataStreamReader = spark
      .readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "10.2.2.17:9092")
      .option("subscribe", "data-t1-connector_sync_data_to_ods")
      .option("startingOffsets", "earliest")
      .option("maxOffsetsPerTrigger", 100000)
      .option("failOnDataLoss", false)
      .load()

val kafkaData = dataStreamReader
  .withColumn("name", $"key".cast(StringType))
  .withColumn("topic", $"topic".cast(StringType))
  .withColumn("last_updater_id", $"offset".cast(LongType))
  .withColumn("creator_id", $"partition".cast(IntegerType))
  .withColumn("last_updated", $"timestamp".cast(TimestampType))
  .withColumn("id", $"value".cast(StringType))
  .withColumn("date_created", current_timestamp())
  .withColumn("last_updated_batch_id", lit("asd"))
  .withColumn("batch_id", lit("test124"))
  .select("batch_id","last_updated_batch_id", "name", "id", "last_updated", "date_created", "last_updater_id","creator_id")


val query = kafkaData
    .writeStream
    .queryName("demo")
    .foreachBatch { (batchDF: DataFrame, _: Long) => {
    batchDF.persist()

    batchDF.show()
        
    println(LocalDateTime.now() + "start writing cow table")
    // batchDF.write.format("hudi")
    //     .option(TABLE_TYPE_OPT_KEY, "COPY_ON_WRITE")
    //     .option(RECORDKEY_FIELD_OPT_KEY, "last_updated")
    //     // 以当前日期作为分区
    //     .option(PRECOMBINE_FIELD_OPT_KEY, "date_created")
    //     .option(PARTITIONPATH_FIELD_OPT_KEY, "batch_id")
    //     .option(TABLE_NAME, s"test_cow_1")
    //     .mode(SaveMode.Append)
    //     .save("/tmp/default/test_cow_1")

    println(LocalDateTime.now() + "start writing mor table")
    batchDF.write.format("hudi")
        .option(TABLE_TYPE_OPT_KEY, "MERGE_ON_READ")
        .option(OPERATION_OPT_KEY, "upsert")
		.option("hoodie.cleaner.policy.failed.writes", "LAZY")
       .option("hoodie.write.concurrency.mode", "optimistic_concurrency_control")
       .option("hoodie.write.lock.zookeeper.url", "10.2.2.17")
       .option("hoodie.write.lock.zookeeper.port", "2181")
       .option("hoodie.write.lock.zookeeper.lock_key", "test_table")
       .option("hoodie.write.lock.zookeeper.base_path", "/tmp")
        .option(RECORDKEY_FIELD_OPT_KEY, "last_updated")
        .option(PRECOMBINE_FIELD_OPT_KEY, "date_created")
        .option(PARTITIONPATH_FIELD_OPT_KEY, "batch_id")
        .option(ASYNC_COMPACT_ENABLE_OPT_KEY, "true")
        .option(HIVE_SUPPORT_TIMESTAMP, "true")
        .option(INLINE_COMPACT_NUM_DELTA_COMMITS_PROP, "1")
        .option(TABLE_NAME, s"test_mor_2")
        .mode(SaveMode.Append)
        .save("/tmp/data_t1p873/test_mor_2")
        
    println(LocalDateTime.now() + "finish")
    batchDF.unpersist()
    }
    }
    .option("checkpointLocation", "/tmp/sparkHudi/checkpoint/")
    .start()

query.awaitTermination()


// hdfs://HDFS44014

```

##### delete table record
```
 deleteDF // dataframe containing just records to be deleted
   .write().format("org.apache.hudi")
   .option(...) // Add HUDI options like record-key, partition-path and others as needed for your setup
   // specify record_key, partition_key, precombine_fieldkey & usual params
   .option(DataSourceWriteOptions.PAYLOAD_CLASS_OPT_KEY, "org.apache.hudi.EmptyHoodieRecordPayload")
 ```
 
 
 ```
 df.write.format("hudi").option(HoodieWriteConfig.TABLE_NAME, "test_hudi_mor").option(PAYLOAD_CLASS_OPT_KEY, "org.apache.hudi.EmptyHoodieRecordPayload").mode(Append).save("/tmp/default/test_hudi_mor/*/")
 
 ```
 
 val tripsSnapshotDF = spark.read.format("hudi").load("/tmp/default/test__cow/*/")
 
 val df = spark.read().format("org.apache.hudi").option(DataSourceReadOptions.QUERY_TYPE_OPT_KEY(), DataSourceReadOptions.QUERY_TYPE_SNAPSHOT_OPT_VAL()).load("/tmp/sparkHudi/checkpoint/*/*") 
 
 read parquet file saved by hudi
 ```
 var ds = spark.read.parquet("hdfs:///tmp/data_t1p873/test_mor/batch_id=test124/*/")
 ```
 
 ##### hudi conpactor script
 ```
 spark-submit --packages org.apache.hudi:hudi-utilities-bundle_2.11:0.6.0 \
--class org.apache.hudi.utilities.HoodieCompactor \
--instant-time 20190117010349 \
--base-path /tmp/data_t1p873/test_mor \
--table-name test_mor \
--schema-file 
--parallelism 1 \
--spark-memory 471859
```
```
spark-submit --packages org.apache.hudi:hudi-utilities-bundle_2.11:0.8.0 \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--table-type MERGE_ON_READ \
--target-base-path /tmp/data_t1p873/test_mor \
--target-table test_mor \
--source-class org.apache.hudi.utilities.sources.JsonDFSSource \
--source-ordering-field ts \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
--props /path/to/source.properties \
--continous
```

#### create impala table for query
```
CREATE EXTERNAL TABLE data_t1p873.test_mor_3 like parquet 'hdfs://HDFS44014/tmp/data_t1p873/test_mor_3/test124/b7815e9a-82cd-4636-b61f-ef56d1d64e5c-0_0-61-33987_20210618173300.parquet' stored as hudiparquet location 'hdfs://HDFS44014/tmp/data_t1p873/test_mor_3';
```


```
spark-submit --packages org.apache.hudi:hudi-utilities-bundle_2.11:0.6.0 \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
--table-type MERGE_ON_READ \
--target-base-path hdfs://HDFS44014/tmp/data_t1p873/test_mor \
--target-table test_mor \
--source-class org.apache.hudi.utilities.sources.JsonDFSSource \
--source-ordering-field ts \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
--props /path/to/source.properties \
--continous
```

submit quest to compact asynchronously
```
spark-submit --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
 --packages org.apache.hudi:hudi-utilities-bundle_2.11:0.8.0,org.apache.spark:spark-avro_2.11:2.4.3 \
 --master yarn --deploy-mode cluster \
--conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
--conf spark.sql.hive.convertMetastoreParquet=false \
/root/.ivy2/jars/org.apache.hudi_hudi-utilities-bundle_2.11-0.8.0.jar \
 --table-type MERGE_ON_READ \
 --source-ordering-field dms_received_ts \
 --props s3:///properties/dfs-source-retail-transactions-full.properties \
 --source-class org.apache.hudi.utilities.sources.ParquetDFSSource \
 --target-base-path s3:///hudi/retail_transactions --target-table hudiblogdb.retail_transactions \
 --transformer-class org.apache.hudi.utilities.transform.SqlQueryBasedTransformer \
 --payload-class org.apache.hudi.payload.AWSDmsAvroPayload \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
--enable-hive-sync
```



spark-submit \
--packages org.apache.hudi:hudi-spark-bundle_2.11:0.8.0,org.apache.hudi:hudi-utilities_2.11:0.8.0 \
--deploy-mode client \
--class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
/mnt/hudi/packaging/hudi-utilities-bundle/target/hudi-utilities-bundle_2.11-0.6.1-SNAPSHOT.jar \
--props /var/demo/config/kafka-source.properties \
--table-type COPY_ON_WRITE \
--source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
--source-ordering-field ts \
--target-base-path /user/hive/warehouse/stock_ticks_cow \
--target-table stock_ticks_cow \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider





spark-submit --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
 --packages org.apache.hudi:hudi-utilities-bundle_2.11:0.8.0,org.apache.spark:spark-avro_2.11:2.4.3 \
 --master yarn --deploy-mode cluster \
--conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
--conf spark.sql.hive.convertMetastoreParquet=false \
/root/.ivy2/jars/org.apache.hudi_hudi-utilities-bundle_2.11-0.8.0.jar \
 --table-type MERGE_ON_READ \
 --source-ordering-field dms_received_ts \
 --props s3:///properties/dfs-source-retail-transactions-full.properties \
 --source-class org.apache.hudi.utilities.sources.ParquetDFSSource \
 --target-base-path s3:///hudi/retail_transactions --target-table hudiblogdb.retail_transactions \
 --transformer-class org.apache.hudi.utilities.transform.SqlQueryBasedTransformer \
 --payload-class org.apache.hudi.payload.AWSDmsAvroPayload \
--schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
--enable-hive-sync



spark-submit --class org.apache.hudi.utilities.HoodieCompactor \
--packages org.apache.hudi:hudi-utilities-bundle_2.11:0.8.0,org.apache.spark:spark-avro_2.11:2.4.3 \
--conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
/root/.ivy2/jars/org.apache.hudi_hudi-utilities-bundle_2.11-0.8.0.jar \
--base-path hdfs://HDFS44014/tmp/data_t1p873/test_mor \
--table-name test_mor \
--instant-time 20190117010349 \
--parallelism 1 \
--spark-memory 471859200 \
--schema-file ./data/test.avsc



```
{
     "type": "record",
     "namespace": "com.example",
     "name": "test_mor",
     "fields": [
       { "name": "name", "type": "string" },
	   { "name": "topic", "type": "string" },
	   { "name": "last_updater_id", "type": "long" },
	   { "name": "creator_id", "type": "int" },
	   { "name": "last_updated", "type": "string" },
	   { "name": "id", "type": "string" },
	   { "name": "date_created", "type": "string" },
	   { "name": "last_updated_batch_id", "type": "string" },
       { "name": "batch_id", "type": "string" }
     ]
} 
```


spark-submit --packages org.apache.hudi:hudi-utilities-bundle_2.11:0.8.0,org.apache.spark:spark-avro_2.11:2.4.4 \
 --master yarn \
 --deploy-mode cluster \
 --num-executors 2 \
 --executor-memory 2g \
 --driver-memory 2g \
 --conf spark.driver.extraJavaOptions="-XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintGCTimeStamps -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/varadarb_ds_driver.hprof" \
 --conf spark.executor.extraJavaOptions="-XX:+PrintGCApplicationStoppedTime -XX:+PrintGCApplicationConcurrentTime -XX:+PrintGCTimeStamps -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/varadarb_ds_executor.hprof" \
 --queue hadoop-platform-queue \
 --conf spark.scheduler.mode=FAIR \
 --conf spark.task.cpus=1 \
 --conf spark.executor.cores=1 \
 --conf spark.task.maxFailures=10 \
 --conf spark.memory.fraction=0.4 \
 --conf spark.rdd.compress=true \
 --conf spark.kryoserializer.buffer.max=200m \
 --conf spark.serializer=org.apache.spark.serializer.KryoSerializer \
 --conf spark.memory.storageFraction=0.1 \
 --conf spark.shuffle.service.enabled=true \
 --conf spark.sql.hive.convertMetastoreParquet=false \
 --conf spark.ui.port=5555 \
 --conf spark.driver.maxResultSize=3g \
 --conf spark.executor.heartbeatInterval=120s \
 --conf spark.network.timeout=600s \
 --conf spark.eventLog.overwrite=true \
 --conf spark.eventLog.enabled=true \
 --conf spark.yarn.max.executor.failures=10 \
 --conf spark.sql.catalogImplementation=hive \
 --conf spark.sql.shuffle.partitions=100 \
 --class org.apache.hudi.utilities.deltastreamer.HoodieDeltaStreamer \
 /root/.ivy2/jars/org.apache.hudi_hudi-utilities-bundle_2.11-0.8.0.jar \
 --table-type MERGE_ON_READ \
 --source-class org.apache.hudi.utilities.sources.JsonKafkaSource \
 --source-ordering-field timestamp  \
 --target-base-path hdfs://HDFS44014/tmp/data_t1p873/test_mor_1 \
 --target-table test_mor_1 \
 --props /var/demo/config/kafka-source.properties \
 --schemaprovider-class org.apache.hudi.utilities.schema.FilebasedSchemaProvider \
 --continuous