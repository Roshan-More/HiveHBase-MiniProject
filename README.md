********HiveHBaseMiniProject********

Problems in RDBMS:
When we had a Telecom client where the client used to provide telecom related services 
The whole data was stored in mysql Database. There are around 8 types of tables storing different types of values
Names of tables : Customers Tables , Complaints table etc
The data was huge and for getting relevant and up to date status of the customers was not happening at that pace
As the database was getting big day by day.
The query used to take around 48 hours even after the best optimizations.
Whenever the query used to run a lot of database resources were allocated to the ongoing task.

To solve this problem , it was decided to migrate the whole data to distributed system  which was on campus (hdfs)

This whole Process was done with the following steps:

Step 01 : We will import the tables from mysql to hdfs qith the help of ingestion tool Sqoop

The following two sqoop commands We have used also to automate the sqoop command we have made a sqoop job with incremental load

sqoop import \
-Dhadoop.security.credential.provider.path=jceks://hdfs/user/cloudera/mysql.password.jceks \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba \
--password-alias roshan.password \
--table orders \
--autoreset-to-one-mapper \
--warehouse-dir /user/cloudera/HiveHBaseMiniProject


sqoop import \
-Dhadoop.security.credential.provider.path=jceks://hdfs/user/cloudera/mysql.password.jceks \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba \
--password-alias roshan.password \
--table customers \
--autoreset-to-one-mapper \
--warehouse-dir /user/cloudera/HiveHBaseMiniProject


Step 02: We have a made a hive table schema based on the table which is in mysql database

sqoop create-hive-table \
-Dhadoop.security.credential.provider.path=jceks://hdfs/user/cloudera/mysql.password.jceks \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba \
--password-alias roshan.password \
--table orders \
--hive-table hive_orders \
--fields-terminated-by ','

sqoop create-hive-table \
-Dhadoop.security.credential.provider.path=jceks://hdfs/user/cloudera/mysql.password.jceks \
--connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username retail_dba \
--password-alias roshan.password \
--table customers \
--hive-table hive_customers \
--fields-terminated-by ','

Step 03: We have insert the data into hive tables

load data inpath '/user/cloudera/HiveHBaseMiniProject/orders in table hive_orders

load data inpath '/user/cloudera/HiveHBaseMiniProject/customers in table hive_customers

Step 04 : Now we can do all the analysis work of the data on the hive table with the help of various queries 

Step 05 : We will create a Hbase tables managed by hive

create table roshan_hbase
(customer_id int, customer_fname string,customer_lname string,order_id int,order_date string)
STORED BY 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
WITH SERDEPROPERTIES
('hbase.columns.mapping'=':key,personal:customer_fname,personal:customer_lname,personal:order_id,personal:order_date');

Step 06 : We will insert the desired input which we got by analysis into the HBase table managed by hive

Insert overwrite table roshan_hbase select c.customer_id,c.customer_fname,c.customer_lname,o.order_id,o.order_date 
from customers c 
join 
orders o 
on (c.customer_id=o.order_customer_id);
