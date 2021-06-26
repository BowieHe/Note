<https://programming.vip/docs/actual-write-to-hudi-using-spark-streaming.html>
#### file management

Hudi table exists in DFS system's base path directory, where hudi table is divided inyo different partitions. Each partition is uniquely identified by partition path and organized in the same way as Hive

Within each partition, files are divided into FileGroup filegroups by a unique FileId file id.Each FileGroup contains multiple FileSlice file slices, each containing a base file base file (parquet file) formed by a commit or compaction operation, and a log file (log file) containing an inserts/update operation on the base file

#### index

Hudi provides efficient upsert operations by mapping the Hoodie key (**record key + partition path**) to the file id.When the first version of a record is written to a file, the mapping relationship between the record key value and the file does not change.In other words, a mapped filegroup always contains all versions of a set of records.

#### table type & feature 

|feature | Coty on Write| Merge On Read|
|----|------|------|
|query| Snapshot Query + Incremental Query|Snapshot Query+Incremental Query+Read Optimization|
|Data delay| high| low|
|Update cost| high(overrides the entire parquet)| low(append to delta record)|
|parquet file size| Small (high update (I/O) overhead)| Large (low update overhead)|
|write frequency| high| low (depending on merge strategy)|

#### query type

- Snapshot query: The query will see the latest table snapshots of subsequent commit and merge operations.For the merge on read table, the latest base and delta files are merged to see near real-time data (a few minutes delay).For copy on write tables, existing parquet tables are directly replaced by update/delete operations or other write operations.
-  Incremental Query: Queries only see newly written data after a given submit/merge operation.This effectively provides a flow of change, enabling incremental data pipelines.
- Read Optimized Query: The query will see the latest snapshot of the table after a given commit/merge operation.Only the underlying/column storage files in the latest file slices are viewed, and query efficiency is guaranteed to be the same as non-hudi column storage tables.

| | snapshot| read optimization|
|--|----|----|
|data delay| low| high|
|query delay| high(merge base/column storage files + row storage delta/log files)|(merge base/column storage files + row storage delta/log files)|


#### hudi action type
1. commits:A commit denotes an atomic write of a batch of records into a dataset.
2. cleans:Background activity that gets rid of older versions of files in the dataset, that are no longer needed.
3. delta_commit:A delta commit refers to an atomic write of a batch of records into a MergeOnRead storage type of dataset
4. compaction:Background activity to reconcile differential data structures within Hudi e.g: moving updates from row based log files to columnar formats
5. rollback
6. savepoint:Marks certain file groups as “saved”, such that cleaner will not delete them. It helps restore the dataset to a point on the timeline

#### storage
- copy on wrote 
File slices in Copy-On-Write storage only contain the base/columnar file and each commit produces new versions of base files.

- merge on read
is a superset of copy on write, in the sense it still provides a read optimized view of the dataset via the Read Optmized table. Additionally, it stores incoming upserts for each file group, onto a row based delta log, that enables providing near real-time data to the queries by applying the delta log, onto the latest version of each file id on-the-fly during query time.
The most significant change here, would be to the compactor, which now carefully chooses which delta logs need to be compacted onto their columnar base file, to keep the query performance in check (larger delta logs would incur longer merge times with merge data on query side)

	- The semantics around when data from a commit is available to a query changes in a subtle way for the RO table. Note, that such a query running at 10:10, wont see data after 10:05 above, while a query on the RT table always sees the freshest data.

**MoR is better suited for write- or change-heavy workloads with fewer reads. CoW is better suited for read-heavy workloads on data that changes less frequently.**

##### hudi async compaction deployment model
<https://hudi.apache.org/blog/async-compaction-deployment-model/>


# question
- ~~写入的TPS和kudu 的比较，只能笔kudu好，不能差于kudu(和spark类似，远好于parquet)
- ~~延迟的考虑(可以通过compaction的时间间隔控制)~~
- merge on read的compact 时间间隔(如果是实时数据，structeredStreaming只查到支持通过文件大小和commit数量，不支持定时同步<https://github.com/Shopify/hoodie/blob/master/hoodie-client/src/main/java/com/uber/hoodie/config/HoodieCompactionConfig.java>，能不能考虑外部手动维护一个定时器，去执行script来手动compact或者通过hudi CLI来执行)
- 热数据还是通过kudu进行读写，冷数据通过hudi进行保存和处理？
- ~~hudi表的创建(只查询到spark sql，或者同步数据的时候自动创建)~~
- ~~每个表异步的compact都会有一个线程在后台常驻(可以只开始一个spark-application，但是针对每个不同的表，读取里面的数据)
- ~~能不能只compact(通过CLI或者hudi自己提供的org.apache.hudi.utilities.HoodieCompactor)~~
- ~~并发的写，从文件连接器和API进来的数据会不会有一边被阻塞(MOR表在inline写的时候会造成阻塞，但是异步写会自动分配到不同的spark partition和 task，可以设置每个task的线程数，不会造成阻塞)~~
- ~~如果需要实时的接受实时文件数据，每个表都需要有一个spark的常驻线程在后台保持开启(测试可以一个stream开启写多个parquet文件，但是针对不同的partition都需要有一系列的代码（hudi的三个KEY），因为toic和内容不同，有点不好写)
- 相比kudu 整体方案的区别
- ~~一个kafka topic里面数据包含去往多个数据表的数据，能不能支持(不建议，可以通过`HoodieMultiTableDeltaStreamer`实现，但是只支持COW表)
- 并发问题，两个程序同时操作一个hudi表，实时和离线数据同时写(测试过两个streaming关联到一张hudi表，如果两个topic同时写，会报错`Concurrent update to the log. Multiple streaming jobs detected for 3`)(<https://cwiki.apache.org/confluence/display/HUDI/RFC+-+22+%3A+Snapshot+Isolation+using+Optimistic+Concurrency+Control+for+multi-writers>)**但是支持同时写两个不同的文件**
- kafka数据从一个topic中拆分,不同的hudi_table在写数据的时候，需要指定写入表的3个key，表名，base_path等等信息，即一个dataset需要根据目标表的数量拆分(尝试从redis读meta信息？)
- 实时数据读取外部数据，比如表meta和mapping映射关系
- ~~hudi表创建(查到0.9版本中添加了可以通过sparkSQL创建表，或者新写一个空的dataFrame来创建,impala直接创建一张外表，hudi写的时候再创建内表)

- 为什么ods也要保存为hudi
- 如果hudi和ods同时存在的时候，

### hudi两种表的区别：
#### copy on write(COW)
COW表写数据，直接写入到basefile(\*.parquet)，而不写入到log文件。所以，COW表的文件片只包含basefile（一个parquet构成一个文件片)
对于Update：该文件ID的最新版本都将被重写一次，并对所有已经更改的记录使用新值
对于insert：记录首先打包到每个分区路径中的最小文件中，直到达到配置的最大大小
仅使用列式存储，例如parquet，仅更新版本号，通过写入过程中执行同步合并来重写文件。

因此对于COW表，只要有一个数据的更新，会直接更新整个parquet。
数据的写入可以认为是实时的，但是写入的IO消耗和性能都很差(只要有更新，就会重写整个partition)。**适用于读很多，但是写很少的场景**
#### merge on read(MOR)
MOR表写数据时，记录首先会被快速写进日志文件，稍后会使用时间轴上的压缩操作将其与基本文件合并。根据查询是读取日志中的合并快照流，还是变更流，还是仅读取未合并的基础文件，MOR支持多种查询类型。
基于列式存储(parquet)和行式存储(avro)结合的文件进行存储，更新记录到增量文件
基于列式存储(parquet)和行式存储(avro)结合的文件进行存储，更新记录到增量文件，压缩同步和异步生成新版本的文件。

在数据读取到的时候，只会写在delta log中，只有commit到了一定的数量(可以设置最小最大commit)，或者到了一定的时间间隔出发了compact的时候，才会吧delta log中的数据写入到对应的分区中
MOR表异步写的时候是默认开启每一次commit之后都会compact(**官方目前针对sparkDataset的写方式不支持定时实时压缩，推荐设置commit=1来实现实时同步**)

相比COW表模式，MOR表模式更加适合很多零散数据的写入，更新，更加的适合我们使用的场景

#### MOR vs COW
|feature | Coty on Write| Merge On Read|
|----|------|------|
|query| Snapshot Query + Incremental Query|Snapshot Query+Incremental Query+Read Optimization|
|Data delay| high| low|
|Update cost| high(overrides the entire parquet)| low(append to delta record)|
|parquet file size| Small (high update (I/O) overhead)| Large (low update overhead)|
|write frequency| high| low (depending on merge strategy)|

#### impala 支持
hudi table有三种数据的查询：snapshot，incremental，read_optimized(只支持MOR)
- snapshot：查询操作将查询最新快照的表数据。如果是MOR类型的表，它将动态合并最新文件版本的基本数据和增量数据用于查询；是COW类型的表，则直接查询parquet表，同时提供upsert/delete操作。
- incremental:查询只能看到写入表的新数据（需要提供一个查询的时间戳）
- read_optimized:查询将查看给定提交/压缩操作的最新快照

impala可以支持两种表类型的查询，
对于第一中copy on write表，只支持snapshot的查询；对于MOR的表，只支持read_optimized查询


#### 数据插入
hudi表在插入的时候，有以下几个参数必须要指定：
- RECORDKEY_FIELD_OPT_KEY:这个的值会作为HoodieKey的一部分，转换成String格式，用于标记改记录是否唯一
- PARTITIONPATH_FIELD_OPT_KEY：会被用于HoodieKey的partitionPath部分，配合上面的recordKey记录该条记录是否唯一
- PRECOMBINE_FIELD_OPT_KEY：在合并之前会找到两个记录有相同的key，这时候会取那个有更大的值的
下面几个选项是可选项，和主键以及compact相关：
- INLINE_COMPACT_NUM_DELTA_COMMITS_PROP: 在收到多少个commit之后，数据会被compact
- ASYNC_COMPACT_ENABLE_OPT_KEY: MOR模式默认开启

**Note**:
- hoodie表在创建的时候会自动加上几个字段，`hoodie_commit_time，_hoodie_commit_seqno, _hoodie_record_key,_hoodie_partition_path, _hoodie_file_name`,其中commit_time是用来维护hudi所有的commit的时间顺序的；record_key会指定key和值(`name:name0,last_updated_batch_id:test1`)；file_name就是该数据存储的parquet文件名
- hudi在建表的时候可以指定primaryKey，但是这个字段和hudi的primaryKey不同。hudi的primaryKey是一个recordKey+partitionKey的组合，而建表指定的primary更像是impala外表的primaryKey；
同时record_key会用于去除重复数据，如果没有指定正确的recordKey，会导致数据的缺失，而且没有任何预警和错误提示(但是会根据PRECOMBINE_FIELD_OPT_KEY的值比较，取最大值)
- hudi在数据插入的时候，如果表类型是COW，则每一个数据的导入都会重写整个partition(和hive没有区别)，因此数据都是直接读到的；但是MOR的话，没有实现compact的话，数据是无法被impala读到的(impala只支持读compact之后的mor表)

#### compaction
hudi提供了几种不同的压缩策略，但都是基于不用的场合和文件的大小来区分
> compaction strategy
<https://cwiki.apache.org/confluence/display/HUDI/def~compaction>
delta.commits=1 if want to optimized data-freshness and use for read-optimized queries
<https://github.com/apache/hudi/issues/2151>


### 性能

#### 写入性能

|type| performance|
|--|------|
|bulk_insert|和vanilla spark write相似，多了一个额外的文件大小排序|
|insert|和上面的批量插入性能相似。更多的取决于文件插入的并行|
|upsert(insert&de-duplicate)|性能会受到index的设置影响。相比spark直接join表重写，如果index设置合理，7-10X快于spark；相比重写整个partition，hudi的速度取决于文件的数量：一个partition一共有1000个文件，其中100个需要更新，那么就是10X快于重新|

Hudi默认upsert/insert/delete的并发度是1500，对于演示的小规模数据集可设置更小的并发度。
同时如果设置的record_key是一个保持单调递增的话，可以对写入的速度进行优化。比如官方推荐使用时间戳或者uuid

>Like with many typical system that manage time-series data, Hudi performs much better if your keys have a timestamp prefix or monotonically increasing/decreasing. Even if you have UUID keys, you can follow tricks like this to get keys that are ordered.

#### 读取性能
- read_optimized:和spark/hive读取标准的parquet文件相同
- incremental:取决于更新的文件数量。如果partition中有1000个文件，但是只更新了100个文件。那查询的速度会比扫描整个partition快10X
- read time：和avro表在hive/spark中查询性能相同

write hudi dataset
数据源如果是kafka，DFS，可以使用流式，比如推荐使用delta streaming来写，同时也支持通过spark datasource API，或者使用hudi datasource来写程序


#### 测试结果

写入读写做了一个简单的测试
1. 写了20个parquet文件，每个文件一个分区，一个parquet文件里有300w数据：写入数据到一个hudiTable大约花40s时间写入和compact。
同时由于写入的时候会针对record_key进行去重和比较，写入300w record_key完全不同和完全相同的数据耗时没有什么区别
2. 写了2个parquet文件，每个parquet中也是300w数据：写入到一个表中：第一次把300w数据分成30w分区，每个分区10条数据，写入前15w分区耗时23min，23w分区耗时1h


读数据：由于数据目前都是在commit提交之后直接compact，因此对于impala来说只需要refresh table之后读取即可，数据的读取没有时间的延迟，主要是花费在compact


### index
<http://hudi.apache.org/blog/hudi-indexing-mechanisms/>
hudi使用index来标记文件的file group，用于数据的快速的插入和更新

通过给定一个hoodie key(**index**)(一个Record Key + 一个partition path（可选，非必须）`通过DataSourceWriteOptions的RECORDKEY_FIELD_OPT_KEY，PARTITIONPATH_FIELD_OPT_KEY控制,两个都可以是多个列`)。
一个index记录了记录到hudi dataset文件的映射关系，因此这个index一旦写入之后，不会允许修改。被index标记的文件会保留所有的组记录。
当一条记录需要被更新和删除时，Hudi就是利用Index迅速定位到该记录所在的文件进行操作（如果是COW表，会以针拷贝此文件中未变更的数据再与变更数据合并形成新文件，如果是MOR表，会围绕此文件追加日志）
在数据真正的写入的时候，如果发现两条记录有相同的record key，可以通过设定`PRECOMBINE_FIELD_OPT_KEY`来保留值更大的记录。**只适用于upsert，不适用insert和bulk_insert**

从类型上分，Hudi目前有三种Index：Bloom Index (default)，Simple Index，HBase Index，而从索引的全局性上，则分为Global Index和Non Global Index，其区别就是Global Index的索引是全局唯一的，而Non Global Index的索引只在分区内唯一，联系刚才提到的Index解构就不难推断，对于Global Index来说，它不包含partition path，所以是全局唯一的，而Non Global Index包含了partition path，所以当同一条数据换了分区列重新写入，还是以插入方式写进去的，而不是更新原有数据。

不过需要注意的是：使用Global类型的索引是有代价的，每次更新和删除操作都需要影响到全表的所有文件，所以它比较适合数据量较小的表，不适合大表


### 数据整合

#### 整合类型
- 同步：在每一次数据的提交之后就同步到parquet文件中，而且在整合结束之前，下一次的数据写操作不能执行
- 异步：不会阻塞，近似实时的写入。可以通过Hudi DeltaStreamer，Spark Structured Streaming来实现(spark只能支持文件打到一定大小或者有了一定的提交次数之后才能对数据进行整合。deltaStreaming可以通过设定时间段来对数据进行整合)