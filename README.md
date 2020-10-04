## UserBehaviorAnalysis

### Project Overview 

Toys R US is known as a famous toy and juvenile-products retailer. We are providing sales analysis both for the local stores and our clients such as lego. The project is focusing on collecting data from the web, stores and amazon for sales reporting, visualization and updating the recommendation model. Our team provided data processing solutions including data ingestion, data cleansing, data transformation and data storage using Spark, Kafka and AWS services and delivered the structured data to other teams for further analysis.


### Summary
  
1. On a daily basis, the data came from thousands of local stores and stored in S3 as CSV files and text files. Used glue crawler to create tables for metadata and stored in the database. Implemented ETL process by writing scripts in pyspark or scala, such as changing schema or remove incomplete records. Then the data will pass to Athena in order to apply sales analysis by writing queries.  

 ![AWS](https://miro.medium.com/max/1050/1*nazxDhqIohJCYaAqXBNslw.png)


2. The streaming data came from the web server, including the logs and online sales data. We used flume and kafka to divide data into different topics for different consumer groups and then store the data in HDFS for further analysis.

Format: ![streaming](https://howtoprogram.xyz/wp-content/uploads/2016/08/Apache-Flume-Kafka-Source-And-HDFS-Sink.png)

3. The data scientist team updated the offline recommendation system in python. As a data engineer, my duty is to migrate the code to spark. 

### Enviroment Setup
Hardware:
EMR - I used a 3 node cluster with below Instance Types:

m5.xlarge
4 vCore, 16 GiB memory, EBS only storage
EBS Storage:64 GiB

### Senerio
1. Data Skew
- Case 1
  - improve the parallelism: separate different key in the same task
  - use groupbykey(12->48) / spark.default.parallelism
  - time decrease 75%

personalize partitioner
```java
.groupByKey(new Partitioner() {
@Override
public int numPartitions() {
return 12;
}

@Override
public int getPartition(Object key) {
int id = Integer.parseInt(key.toString());
if(id >= 9500000 && id <= 9500084 && ((id - 9500000) % 12) == 0) {
return (id - 9500000) / 12;
} else {
return id % 12;
}
}
})
```
- Case 2
   - broadcast small table
```sql
SET spark.sql.autoBroadcastJoinThreshold=104857600;
INSERT OVERWRITE TABLE test_join
SELECT test_new.id, test_new.name
FROM test
JOIN test_new
ON test.id = test_new.id;
```

2. Schedue
- Pipelines would be run on 12am daily. 
