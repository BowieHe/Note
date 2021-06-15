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

