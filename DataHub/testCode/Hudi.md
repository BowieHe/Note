### TestCode
#### test spark shell start
```shell
spark-shell \
  --jars /opt/test/data-spark-jobs/lib/data-spark-jobs-cdh6.3.1.jar \
  --packages org.apache.hudi:hudi-spark-bundle_2.11:0.8.0,org.apache.spark:spark-avro_2.11:2.4.3,org.apache.spark:spark-streaming-kafka-0-10_2.11:2.4.3,org.apache.spark:spark-sql-kafka-0-10_2.11:2.4.3 \
  --conf 'spark.serializer=org.apache.spark.serializer.KryoSerializer'
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
import java.time.LocalDateTime
import org.apache.hudi.QuickstartUtils._
import scala.collection.JavaConversions._
import org.apache.spark.sql.SaveMode._
import org.apache.hudi.DataSourceReadOptions._
import org.apache.hudi.DataSourceWriteOptions._
import org.apache.hudi.config.HoodieWriteConfig._

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
    batchDF.write.format("hudi")
        .option(TABLE_TYPE_OPT_KEY, "COPY_ON_WRITE")
        .option(RECORDKEY_FIELD_OPT_KEY, "last_updated")
        // 以当前日期作为分区
        .option(PRECOMBINE_FIELD_OPT_KEY, "date_created")
        .option(PARTITIONPATH_FIELD_OPT_KEY, "batch_id")
        .option(TABLE_NAME, s"test_hudi_cow")
        .option(HIVE_STYLE_PARTITIONING_OPT_KEY, true)
        .option(HIVE_TABLE_OPT_KEY, "test_hudi_cow")
        .option(HIVE_SYNC_ENABLED_OPT_KEY, true)
        .option(HIVE_DATABASE_OPT_KEY, "data_tip873")
        .option(HIVE_TABLE_OPT_KEY, "ods_aaaa")
        .mode(SaveMode.Append)
        .save("/tmp/default/test_hudi_cow")

    println(LocalDateTime.now() + "start writing mor table")
    // batchDF.write.format("hudi")
    //     .option(TABLE_TYPE_OPT_KEY, "MERGE_ON_READ")
    //     .option(RECORDKEY_FIELD_OPT_KEY, "last_updated")
    //     .option(PRECOMBINE_FIELD_OPT_KEY, "date_created")
    //     .option(PARTITIONPATH_FIELD_OPT_KEY, "batch_id")
    //     .option(HIVE_TABLE_OPT_KEY, "test_hudi_mor")
    //     .option(TABLE_NAME, s"test_hudi_cow")
    //     .option(HIVE_STYLE_PARTITIONING_OPT_KEY, true)
    //     .opyion(HoodieWriteConfig.parquetBlockSize, "120MB")
    //     .mode(SaveMode.Append)
    //     .save("/tmp/default/test_hudi_mor")
        
    println(LocalDateTime.now() + "finish")
    batchDF.unpersist()
    }
    }
    .option("checkpointLocation", "/tmp/sparkHudi/checkpoint/")
    .start()

query.awaitTermination()

```