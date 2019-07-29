# Hadoop

CREATE TABLE
•	create external table if not exists drivers 
(driverId INT, name STRING, ssn BIGINT, location STRING, certified STRING, wageplan STRING)   
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/maria_dev/temp' TBLPROPERTIES('skip.header.line.count'='1');
•	create external table if not exists 
timesheet (driverId INT, week INT, hours_logged INT , miles_logged INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/maria_dev/time' TBLPROPERTIES('skip.header.line.count'='1');
•	CREATE TABLE timesheet (driverId INT, week INT, hours_logged INT , miles_logged INT)
•	CREATE TABLE drivers (driverId INT, name STRING, ssn BIGINT, location STRING, certified STRING, wageplan STRING)

SQOOP to hive
•	sqoop import --connect jdbc:postgresql://127.0.0.1/ambari --username ambari -P --table artifact --hive-import --create-hive-table –direct

SQOOP to HDFS
•	sqoop import --connect jdbc:postgresql://127.0.0.1/ambari --username ambari -P --table artifact --target-dir /tmp/guests/ambari_table –direct

SQOOP to HDFS with specific columns
•	sqoop import --connect jdbc:postgresql://127.0.0.1/ambari --username ambari -password bigdata --table hoststate --target-dir /tmp/guests/ambari_table_hoststate --columns "current_state, health_status, host_id" –direct

Hive table with SQOOPed data of specific columns
•	create table hoststate(current_state string, health_status string, host_id int) 
row format delimited
fields terminated by ',' 
stored as Textfile;

Hive table with source data containing "," commas in the field
•	create table hoststate(current_state string, health_status string, host_id int) 
row format serde 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
stored as Textfile;

SQOOP to HDFS with query
•	sqoop import -Dorg.apache.sqoop.splitter.allow_text_splitter=true --connect jdbc:postgresql://127.0.0.1/ambari --username ambari -P --target-dir /tmp/guests/ambari_table_alert_current --query "select * from alert_current where alrt_id >50 and \$CONDITIONS" --split-by maintenance_state –direct

o	If you use --query, then you must also specify a --split-by column or the Sqoop command will fail to execute.
o	Here, split-by column is a text value, hence parameter -Dorg.apache.sqoop.splitter.allow_text_splitter=true is used. Its mandatory
o	There are just two possible values for maintenance_state, still 4 files are created through SQOOP--coz by default the no of mappers/reducers in SQOOP is 4. 2 files will be empty & other two might contain data..

SQOOP Incremental Import of Data 
•	sqoop import --connect jdbc:mysql://db.foo.com/somedb --table sometable 
--where "id > 100000" --target-dir /incremental_dataset –append

o	Incremental import arguments:
o	--check-column (col)	Specifies the column to be examined when determining which rows to import. (the column should not be of type CHAR/NCHAR/VARCHAR/VARNCHAR/ LONGVARCHAR/LONGNVARCHAR)
o	--incremental (mode)	Specifies how Sqoop determines which rows are new. Legal values for mode include append and lastmodified.
o	--last-value (value)	Specifies the maximum value of the check column from the previous import.

SQOOP Export
•	sqoop export --connect jdbc:mysql://127.0.0.1/mysql --username root -P --table salaries2 --export-dir /tmp/salarydata --driver com.mysql.jdbc.Drive

VALIDATE
•	sqoop import --connect jdbc:mysql://db.foo.com/corp --username ambari -P
--table EMPLOYEES --target-dir /tmp/guests/ambari_table --validate

o	A basic import of a table named EMPLOYEES in the corp database that uses validation to validate the row counts:

HIVE
•	Hive is a good choice when you want to query data that has a certain known structure to it.
o	$ hive -f myquery.hive
o	$ CREATE EXTERNAL TABLE salaries (gender string, age int, salary double, zip int )
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/train/salaries/';
•	Comment or Location cant be used as column names because those are reserved Hive keywords.
•	ORDER BY: A complete ordering of the data, which is accomplished by using a single reducer. Be aware that a single reducer with a large number of records can take an extremely long time to finish.
SORT BY: Data output is sorted per reducer.
•	In Hive, skew refers to one or more columns in a table that have values that appear very often. If you know a column is going to have heavy skew, you can specify this in the table’s schema. By specifying the values with heavy skew, Hive will split those out into separate files automatically and take this fact into account during queries so that it can skip whole files if possible.
o	CREATE TABLE Customers (
id int,
username string,
zip int)
SKEWED BY (zip) ON (57701, 57702)
STORED as DIRECTORIES;
•	The following command outputs the results of a query to a file in HDFS. For example:
o	INSERT OVERWRITE DIRECTORY '/user/train/ca_or_sd/' select
name, state from names where state = 'CA' or state = 'SD';
•	All rows with the same distribute by columns will go to the same reducer.  For example, suppose you have the following dataset with the schema (year, name, age) with year being the key:
o	from dataset select * 
distribute by age 
sort by age;
	OR
o	from dataset select *
cluster by age;
•	Hive has two virtual columns that gets created automatically for every table: INPUT__FILE__NAME and BLOCK__OFFSET__INSIDE__FILE.
o	select INPUT__FILE__NAME, lname, fname FROM wh_visits
WHERE lname LIKE 'Y%';
•	Storing Results to a File: 
o	INSERT OVERWRITE DIRECTORY '/user/root/ca_or_sd/' select
name, state from names where state = 'CA' or state = 'SD';
o	INSERT OVERWRITE LOCAL DIRECTORY '/tmp/myresults/' SELECT *
FROM bucketnames ORDER BY age;
•	MapReduce Properties:
o	more myscript.hive
SELECT * FROM names WHERE age = ${age}	
o	hive -f myscript.hive -hivevar age=33
•	Apache Tez replaces MapReduce as the default Hive execution engine in HDP 3.0. MapReduce is no longer supported.
•	Views in Hive are non--‐materialized, so you can use them without concern of creating more work for the resulting MapReduce job.
o	select * from 2010_visitors
where info_comment like "%CONGRESS%"
order by lname;
•	The index data for a table is stored in another table in the Hive metastore known as the index table. Indexes can be a good alternative to partitioning, especially when the partitioned column would result in a large number of small partitions. Index Handler can be COMPACT or BITMAP.
o	CREATE INDEX zipcode_index ON TABLE Customers (zipcode) AS
'COMPACT' WITH DEFERRED REBUILD;
•	The WITH DEFERRED REBUILD clause specifies that the index table not be built immediately. You need to run the REBUILD command to actually create the index table:
o	ALTER INDEX zipcode_index ON Customers REBUILD;
o	SHOW INDEX ON Customers;
o	DROP INDEX zipcode_index ON Customers;
•	Aggregate the username values using the collect_set function, but output only one of them
o	SELECT sum(ordertotal), userid, collect_set(username)[0]
FROM orders
GROUP BY userid;
	OR
SELECT userid, itemlist, sum(ordertotal)
OVER (PARTITION BY userid)
FROM orders;
•	ORC files have three main components: 
o	Stripe
o	Footer
o	Postscript
Here are the features of these components: 
o	An ORC file is broken down into sets of rows called stripes. 
o	The default stripe size is 250 MB. Large stripe sizes enable efficient reads of columns. 
o	An ORC file contains a footer that contains the list of stripe locations. 
o	The footer also contains column data like the count, min, max, and sum. 
o	At the end of the file, the postscript holds compression parameters and the size of the compressed footer.
o	create table orctest (
gender string,
age int,
salary double,
zip int
) stored as ORC;
o	describe formatted orctest;
•	The ANALYZE TABLE command gathers statistics for a table, partition and columns and writes them to the Hive metastore. The ANALYZE command runs a MapReduce job that processes the entire table.
o	ANALYZE TABLE tablename COMPUTE STATISTICS;
•	Vectorization is a new feature that allows Hive to process a batch of up to 1,024 rows together, instead of processing one row at a time. Each batch consists of a column vector, which is usually an array of primitive types. Operations are performed on the entire column vector, which improves the instruction pipelines and cache usage.
o	hive.vectorized.execution.enabled=true
•	Older versions of Hive used the hiveserver process, which can only process one request at a time. HDP 2.0 ships with HiveServer2, a Thrift--‐based implementation that allows multiple concurrent connections and also supports Kerberos authentication.
•	Hive Optimization Tips
o	Divide data amongst different files which can be pruned out by using partitions, buckets and skews
o	Sort data ahead of time to simplify joins
o	Use the ORC file format
o	Sort and Bucket on common join keys
o	Use map (broadcast) joins whenever possible
o	Increase the replication factor for hot data (which reduces latency)
o	Take advantage of Tez
•	Hive Query Tunings
o	mapreduce.input.fileinputformat.split.maxsize and mapreduce.input.fileinputformat.split.minsize: if the min is too  large, you will have too few mappers. If the max is too small, you will have too many mappers.
o	mapreduce.tasks.io.sort.mb: increase this value to avoid disk spills. 
o	Set the following properties all the time: 
hive.optimize.mapjoin.mapreduce=true; hive.optimize.bucketmapjoin=true;
hive.optimize.bucketmapjoin.sortedmerge=true;
hive.auto.convert.join=true;
hive.auto.convert.sortmerge.join=true;
hive.auto.convert.sortmerge.join.noconditionaltask=true;
o	When bucketing data, set the following properties:
hive.enforce.bucketing=true;
hive.enforce.sorting=true;

PIG
•	Pig is a good choice for ETL jobs, where unstructured data is reformatted so that it is easier to define a structure to it.
Modes in Pig:
Local Mode
$ pig -x local
... - Connecting to ...
grunt> 
Tez Local Mode
$ pig -x tez_local
... - Connecting to ...
grunt> 
Mapreduce Mode
$ pig -x mapreduce
... - Connecting to ...
grunt> 

or

$ pig 
... - Connecting to ...
grunt> 
Tez Mode
$ pig -x tez
... - Connecting to ...
grunt> 

Interactive Mode:
Copy /etc/passwd to hdfs path (/user/root/).. 
grunt> A = load 'passwd' using PigStorage(':'); 
grunt> B = foreach A generate $0 as id; 
grunt> dump B;

Batch Mode
Create pig script in local directory. “id.pig”
/* id.pig */

A = load 'passwd' using PigStorage(':');  -- load the passwd file 
B = foreach A generate $0 as id;  -- extract the user IDs 
store B into 'id.out';  -- write the results to a file name id.out

Tez Mode
$ pig -x tez id.pig
•	You can run HDFS commands as well from Grunt shell:
o	grunt> copyFromLocal /root/labs/demos/pigdemo.txt demos/
o	grunt> pwd
hdfs://sandbox:8020/user/root/demos

•	Create txt file:
o	grunt> cat pigdemo.txt
SD	Rich
NV	Barry
CO	George
CA	Ulf
IL	Danielle
OH	Tom
CA	manish
CA	Brian
CO	Mark
o	grunt> employees = LOAD 'pigdemo.txt' AS (state, name);
o	grunt> describe employees;
employees: {state: bytearray,name: bytearray}

•	Default datatype of a field is bytearray.
•	Filters:
o	grunt> ca_only = FILTER employees BY (state=='CA');
o	grunt> DUMP ca_only;
(CA,Ulf)
(CA,manish)
(CA,Brian)
o	grunt> emp_group = GROUP employees BY state;
o	grunt> DUMP emp_group;
(CA,{(CA,Ulf),(CA,manish),(CA,Brian)})
(CO,{(CO,George),(CO,Mark)})
(IL,{(IL,Danielle)})
(NV,{(NV,Barry)})
(OH,{(OH,Tom)})
(SD,{(SD,Rich)})
•	The DUMP command dumps the contents of a relation to the console. The STORE command sends the output to a folder in HDFS.
o	grunt> STORE emp_group INTO 'emp_group';
•	Tab character is the default delimiter in Pig. Use the PigStorage object to specify a different delimiter:
o	grunt> STORE emp_group INTO 'emp_group_csv' USING PigStorage(',');
o	grunt> aliases;
aliases: [ca_only, emp_group, employees]
•	Three commands trigger a logical plan to  be converted to a physical plan and execute as a MapReduce job: STORE, DUMP and ILLUSTRATE.
o	grunt> B = LOAD '/user/root/whitehouse/visits.txt' USING PigStorage(',') AS (
lname:chararray,
fname:chararray,
mname:chararray,
id:chararray,
status:chararray,
state:chararray,
arrival:chararray
);
o	grunt> store B into 'whouse_tab' using PigStorage('\t');
o	grunt> salaries = LOAD 'salaries.txt' USING PigStorage(',') AS (gender:chararray, age:int,salary:double,zip:int);
o	grunt> salariesbyage = GROUP salaries BY age;
o	grunt> mygroup = GROUP salaries BY (gender,age);
•	Relations without a Schema
o	grunt> salaries = LOAD 'salaries.txt' USING PigStorage(',');
o	grunt> salariesgroup = GROUP salaries BY $3;
o	grunt> A = FOREACH salaries GENERATE age, salary;
o	grunt> C = FOREACH salaries GENERATE age..zip;
o	grunt> describe C;
C: {age: int,salary: double,zip: int}
•	POTUS:
o	grunt> visits = LOAD ‘/user/root/whitehouse’ using PigStorage(‘,’);
o	grunt> potus = FILTER visits by $19 MATCHES ‘POTUS’;
o	grunt> potus_details = FOREACH potus generate (chararray) $0 as lname:chararray, (chararray) $1 as fname:chararray, (chararray) $6 as arrival_time:chararray, (chararray) $19 as visitee:chararray;
o	grunt> STORE potus_details INTO ‘potus’ USING PigStorage(‘,’);

HDFS

•	HDFS Federation	:	The new Hadoop infrastructure provides for multiple NameNodes that run independently and don’t require coordination of each other, providing: 
o	Scalability: NameNodes can now scale horizontally, allowing you to improve the performance of NameNode tasks by distributing reads and writes across a cluster of NameNodes.
o	Namespaces:  the ability to define multiple Namespaces allows for the organizing and separating of your big data.
•	The HDFS High Availability (HA) feature provides the option of running two redundant NameNodes in the same cluster in an Active/Passive configuration with a hot standby. This allows a fast failover to a new NameNode in the case that a machine crashes, or a graceful administrator--‐ initiated failover for the purpose of planned maintenance. You can now achieve NameNode HA by configuring your cluster to use the Quorum Journal Manager (QJM).

 


•	Automatic Failover:	A Quorum Journal Manager requires manual failover. If you want your HA NameNodes to failover automatically, you need to configure ZooKeeper:
o	ZooKeeper: An odd number of ZooKeeper daemons that monitor when a NameNode fails
o	ZKFailoverController (ZKFC): A new component that is a ZooKeeper client that monitors and manages the state of a NameNode. 
Each of the machines that runs a NameNode also runs a ZKFC. The ZKFC pings its local NameNode on a periodic basis with a health--‐check command. So long as the NameNode responds in a timely fashion with a healthy status, the ZKFC considers the node healthy. If the NameNode has crashed, frozen, or otherwise entered an unhealthy state, the ZKFC will mark it as unhealthy.
