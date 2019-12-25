---
title: Deployment Notes - Must Know
date: 2019-12-24 21:06:00 +0530
categories: [Software, Deployment]
tags: [Tech notes, deployment, Linux]
seo:
  date_modified: 2019-12-25 22:06:00 +0530
---

## Frequently Used Commands

```

# Running a process in background in remote machine even after exiting the shell.
# "&" runs the process in background while nohup lets the process continue even after exiting the shell. 
# nohup command_name arguments > output.log &
nohup python hello.py > hello.log &

# Find and replace in a file
sed -i -e 's/text_to_be_replaced/value_to_be_replaced_with/g' file_name


```

## Remote login and file transfer

```

# Remote login command
# ssh -i pem_file_path username@ip_address
ssh -i $HOME/AWS/test.pem ubuntu@198.162.1.99

# If pem file doesn't have proper permissions, set using following command
chmod 400 pem_file_path

# Copy file from local machine to remote machince
#scp -i pem_file_path file_path_to_be_transferred_to_remote_machine username@138.91.140.61:~/
scp -i $HOME/AWS/test.pem /home/ubuntu/Desktop/dump.cql ubuntu@198.162.1.99:~/

# Copy file from remote machince to local machine
#scp -i pem_file_path username@138.91.140.61:~/file_path_to_be_transferred destination_directory
scp -i $HOME/AWS/test.pem /home/ubuntu/Desktop/dump.cql ubuntu@198.162.1.99:~/filename.zip ~/Server

```

## MySQL Database

Default Port : 3306

```
# Accessing MySQL through client
mysql -h 10.0.1.5 -u root -p'root' database_name

# To execute queries even after exiting the remote shell
# nohup mysql -h server_ip -u user_name -p'password' database_name -e 'source mysql_commands.sql' &
nohup mysql -h 10.0.1.5 -u root -p'root' platform_data -e 'source /home/ubuntu/test.sql' &

# Backup creation of a database
mysqldump -h 10.0.1.5 -port=3306 -u root -p'root' database_name | gzip > database_name.sql.gz

# Backup of Schema(Without data)
mysqldump -u root -p --no-data database_name > database_name_schema.sql

# Backup creation of a table
mysqldump -h 10.0.1.5 -port=3306 -u root -p'root' database_name table_name --where='' | gzip > table_name.sql.gz

# Importing a Backup
zcat table_name.sql.gz | mysql -uroot -p'root' database_name;

# To insert huge amount of insert/update queries in local environment(Not for production usage)
set global innodb_flush_log_at_trx_commit=2;
# Execute mysql statements
# Revert to the initial flag using the below.
set global innodb_flush_log_at_trx_commit=1;

```

## Cassandra

Default Port : 9042

```
# Accessing Cassandra through client
cqlsh -u user_name -p  'password' 10.0.1.5 -k keyspace_name

# To execute queries even after exiting the remote shell
nohup cqlsh -u user_name -p  'password' 10.0.1.5 -k keyspace_name -e "select * from table_name limit 1;" > cass_output.log&

# Backup creation of a table

nodetool -h localhost -p 7199 snapshot keyspace_name table_name

/* 
   Snapshot is created in data_directory_location/keyspace_name/table_name-UUID/snapshots/snapshot_name directory
   data_directory_location ->  /var/lib/cassandra/data/.
   
   Create a new database, and table schema and 
   copy all files from snapshot folder of each table from old db to 
   respective table folder of new db in the target machine.
*/

// Run following command for all tables
nodetool refresh db_name table_name

# Backup creation for a subset of a table using queries

## Prepare the queries and add to dump.cql
paging off;use keyspace_name;
select * from table_name where partition_key_column1='' and partition_key_column2='';
Query2
Query3...

## Take dump
nohup cqlsh -u user_name -p  'password' 10.0.1.5 -f dump.cql > dump.csv &
zip dump.zip dump.csv

# Importing the dump in target machine
unzip ~/Server/dump.zip -d ~/Server
sed -r 's/(\".*\")|\s*/\1/g' ~/Server/dump.csv > ~/Server/data_without_spaces.csv
cqlsh -e "copy cass_business_data.icm_data from '~/Server/data_without_spaces.csv' with delimiter = '|';"

```

## Elastic Search

Database Default Port: 9200

```
ssh -i Elastic-Search.pem  ubuntu@ip_address

# To check available indexes
curl -X GET "localhost:9200/_cat/indices?v&pretty"

# To delete an index
curl -X DELETE 'localhost:9200/index_name'

# To submit a java mvc application which serves search requests
nohup sudo java -jar es-ms-1.0.0.jar > es.log &

```

## Kafka & Zookeeper

Kafka Default Port: 9092
Zookeeper Default Port: 2181

```
mkdir -p ~/deploy/tools

sudo kill -9 `ps -ef | grep java | awk '{print $2}'`
sudo rm -rf /tmp/zookeeper/
sudo rm -rf /tmp/kafka-logs/

cd $KAFKA_HOME 
nohup sudo  bin/zookeeper-server-start.sh config/zookeeper.properties > zookeeper_dev.log  &
tail -f zookeeper_dev.log 
nohup sudo bin/kafka-server-start.sh config/server.properties > kafka_dev.log &
tail -f kafka_dev.log 

bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 5 --topic KAFKA_TOPIC_NAME

```

## Storm

```
sudo kill -9 `ps -ef | grep java | awk '{print $2}'`

cd $STORM_HOME 
nohup sudo bin/storm nimbus > nimbus.log  &
nohup sudo bin/storm ui > storm_ui.log &
nohup sudo bin/storm supervisor > storm_supervisor.log &
tail -f nimbus_supervisor.log

# To Kill storm topology
localhost:8080

# To submit a Storm topology
#bin/storm jar topology-0.0.1-SNAPSHOT-jar-with-dependencies.jar main_class_name
bin/storm jar topology-0.0.1-SNAPSHOT-jar-with-dependencies.jar topology.CreateTopology


```

## Play

Default Port: 9000

```

git checkout dev
git pull --rebase origin dev
git checkout master
git pull --rebase origin master
git checkout dev
git merge master
git push origin dev
git checkout master
git merge dev
git push origin master
git apply --ignore-whitespace deployment.stash

# Apply the following command, if there is any maven dependencies need to be packaged
mvn clean install -DskipTests=true;


cd play_application_path;
# To create an executable at target/universal/vlplayservices-1.0-SNAPSHOT.zip 
# use the following command
activator clean compile dist

# To kill an existing application
sudo netstat -tulpn | grep java
sudo kill -9 <<processid>>

# rm -rf vlplayservices-1.0-SNAPSHOT/RUNNING_PID;
# To run the application
sudo nohup vlplayservices-1.0-SNAPSHOT/bin/vlplayservices -Dhttp.port=9000 -J-Xms2g -J-Xmx6g -J-XX:+PrintGCDetails -J-XX:+PrintGCDateStamps -J-XX:NewRatio=1 -J-Xloggc:gc.log -J-verbose:gc -J-XX:+UseGCLogFileRotation -J-XX:NumberOfGCLogFiles=5 -J-XX:GCLogFileSize=2M > play.log  &
# used /dev/null instead of play.log to avoid the log file

```

## UI

```
# Generate the zip file

cd angular_project_directory
npm install
ng build --prod
tar -czvf ui.tar.gz dist/ package.json server.js

# Run the UI server
tar -xzvf ui.tar.gz 

# Ignore the following command, if server is already running
nohup sudo node server.js > ui.log &

# To kill the existing server
#sudo netstat -tulpn | grep node
#sudo kill -9 <<processid>>

```