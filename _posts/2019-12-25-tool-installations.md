---
title: Software Installation - Must Know
date: 2019-12-25 21:06:00 +0530
categories: [Software, Tool Installations]
tags: [Tools, installation, Linux]
seo:
  date_modified: 2019-12-25 21:06:00 +0530
---

## Nodejs

Commands to install are as follows

```
sudo apt-get update
sudo apt-get install nodejs
sudo apt-get install npm
nodejs -v
sudo ln -s /usr/bin/nodejs /usr/bin/node
node -v

```

## Java

```
sudo apt-get install openjdk-8-jdk

```

## Kafka

```
Current Version : kafka_2.10-0.8.2.0.tgz
http://apachemirror.wuchna.com/kafka/2.3.0/kafka_2.12-2.3.0.tgz
https://www.apache.org/dyn/closer.cgi?path=/kafka/2.3.0/kafka_2.12-2.3.0.tgz

mkdir -p ~/deploy/tools
cd ~/deploy/tools;tar -xvzf kafka_2.10-0.8.2.0.tgz

```

## Storm

```
Current Version : apache-storm-1.1.1.tar.gz
http://mirrors.estointernet.in/apache/storm/apache-storm-1.2.3/apache-storm-1.2.3.tar.gz
https://www.apache.org/dyn/closer.lua/storm/apache-storm-1.2.3/apache-storm-1.2.3.tar.gz

mkdir -p ~/deploy/tools
cd ~/deploy/tools;tar -xvzf apache-storm-1.1.1.tar.gz

```

## Cassandra

```
java -version (Make sure java is installed first) 
http://cassandra.apache.org/download/

# To stop the service and disable it for future reboots
sudo service cassandra status
sudo service cassandra stop
sudo systemctl disable cassandra

# Authentication settings
https://docs.datastax.com/en/archived/cassandra/3.0/cassandra/configuration/secureConfigNativeAuth.html

# Configuration to access Cassandra remotely
start_rpc: true
rpc_address: 0.0.0.0
broadcast_rpc_address: 10.0.1.5
listen_address: 10.0.1.5
seed_provider:
  - class_name: ...
    - seeds: "10.0.1.5"

# Insertion batch size settings in cassandra.yaml
batch_size_warn_threshold_in_kb: 10240
batch_size_fail_threshold_in_kb: 102400

CREATE ROLE user1 WITH PASSWORD = 'password' AND SUPERUSER = true AND LOGIN = true;
cqlsh -u user1 -p  'password'
ALTER ROLE cassandra WITH PASSWORD='xyz' AND SUPERUSER=false;

# Schema Creation on Cassandra Cluster

CREATE KEYSPACE cass_business_data WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3'}  AND durable_writes = true;

CREATE TABLE cass_business_data.test_data (
    parent_entity_uuid uuid,
    entity_type text,
    component_type int,
    time_period text,
    event_value_type int,
    source_type text,
    sub_time_period bigint,
    entity_uuid uuid,
    data1 text,
    data2 text,
    PRIMARY KEY ((parent_entity_uuid, entity_type, component_type, time_period, event_value_type, source_type), sub_time_period, entity_uuid)
) WITH CLUSTERING ORDER BY (sub_time_period DESC, entity_uuid ASC)
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '64', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND crc_check_chance = 1.0
    AND dclocal_read_repair_chance = 0.1
    AND default_time_to_live = 0
    AND gc_grace_seconds = 864000
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair_chance = 0.0
    AND speculative_retry = '99PERCENTILE';

```

## MySQL

```
# Only Client
sudo apt install mysql-client-core-5.7

# Server & Client
sudo apt-get update
sudo apt-get install mysql-server
sudo mysql(update the credentials if required)
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
mysql_secure_installation

#To access MySQL Remotely comment "bind-address" in /etc/mysql/mysql.conf.d/mysqld.cnf
# Group Queries length increment
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
group_concat_max_len=102400

create database platform_data;
CREATE USER 'user1'@'%';
GRANT ALL PRIVILEGES ON platform_data.* To 'user1'@'%' IDENTIFIED BY 'password';
mysql -uuser1 -p'password'

# Uninstallation steps

sudo apt-get purge mysql-server mysql-client mysql-common mysql-server-core-* mysql-client-core-*
sudo rm -rf /etc/mysql /var/lib/mysql
sudo apt-get autoremove
sudo apt-get autoclean

```

## Postgres

```
#Java --- sudo apt-get install openjdk-8-jre 

# PostgreSQL and PostGIS
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib postgis
#/usr/lib/postgresql/10/bin/pg_ctl -D /var/lib/postgresql/10/main -l logfile start

# Create a user and database
sudo -u postgres createuser -P user_name
Password: password
sudo -u postgres createdb -O user_name database_name

# For geospatial features, postgis extension needs to be installed
sudo -u postgres psql -c "CREATE EXTENSION postgis; CREATE EXTENSION postgis_topology;" geoserver

# Set the following for remote connections
/etc/postgresql/10/main/postgresql.conf:
listen_addresses = '*'

/etc/postgresql/10/main/pg_hba.conf:
host all all 0.0.0.0/0 md5

sudo service postgresql restart

```

## PgAdmin4

```
#Source : https://wiki.postgresql.org/wiki/Apt
sudo apt-get install curl ca-certificates gnupg
curl https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
sudo apt-get update
# if key error happens, https://askubuntu.com/questions/13065/how-do-i-fix-the-gpg-error-no-pubkey
sudo apt-get install pgadmin4
#sudo apt-get install postgresql-11 pgadmin4

```

## GeoServer

```
#Download the platform-independent binary file
https://docs.geoserver.org/stable/en/user/installation/linux.html

sudo apt install tomcat8

download and add geoserver.war file to /var/lib/tomcat8/webapps/
Install vector layer extenstion jars in /var/lib/tomcat8/webapps/geoserver/WEB-INF/lib
for CORS https://stackoverflow.com/questions/22363192/cors-tomcat-geoserver

# Change tilecaching default storage in /var/lib/tomcat8/webapps/WEB-INF/web.xml
<context-param>
   <param-name>GEOWEBCACHE_CACHE_DIR</param-name>
   <param-value>/data/geoserver/geowebcache</param-value>
</context-param>

# Data directory specification
<context-param>
  <param-name>GEOSERVER_DATA_DIR</param-name>
  <param-value>/data/geoserver</param-value>
</context-param>

# Add the following to /etc/logrotate.d/tomcat for log rotation
/var/log/tomcat8/catalina.out {  
 copytruncate  
 daily  
 rotate 7  
 compress  
 missingok  
 size 5M  
}
# For manual rotation, use /usr/sbin/logrotate /etc/logrotate.conf
# Add the following cron job : sudo crontab -e
@daily root find /var/log/tomcat8/* -mtime +2 -type f -delete


sudo service tomcat8 restart

User name: admin
Password: geoserver

```

## Elastic Search

```
# Installation instructions page
https://www.elastic.co/guide/en/elasticsearch/reference/current/deb.html

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update && sudo apt-get install elasticsearch
curl -X GET "localhost:9200/_cat/indices?v&pretty"

```

## R

```
https://cran.r-project.org/bin/linux/ubuntu/README.html
#Other dependencies
sudo apt-get install aptitude
sudo aptitude install libgdal-dev
sudo aptitude install libproj-dev
# R packages
install.packages("sp",dep=TRUE)
install.packages("rgdal",dep=TRUE)
install.packages("rgeos",dep=TRUE)
install.packages("ggmap",dep=TRUE)
install.packages("gstat",dep=TRUE)
install.packages("deldir",dep=TRUE)

```

## Nginx

```
sudo apt-get update
sudo apt-get install nginx
sudo ufw app list
sudo ufw allow 'Nginx Full'
sudo ufw status
sudo service nginx status # To check status

# Update the configuration
sudo vi /etc/nginx/sites-available/default

# Reload the configuration
sudo service nginx reload;

# Production server configuration
server {
        listen 80;
        listen [::]:80;
        server_name sample-domain.com;

    location / {
        proxy_pass http://localhost:8000;
    }

    location /api {
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_pass http://localhost:9000;
    }

}

# Staging server configuration
server {
        listen 80;
        listen [::]:80;
        server_name sample-domain2.com; 

    location / {
        proxy_pass http://localhost:8001;
    }

    location /api {
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_pass http://localhost:9000;
    }

}

# Default server configuration
server {
        listen 80 default_server;
        listen [::]:80 default_server;

    location / {
        proxy_pass http://localhost:8000;
    }

    location /api {
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_pass http://localhost:9000;
    }


        # SSL configuration
        #
        # listen 443 ssl default_server;
        # listen [::]:443 ssl default_server;
        #
        # Note: You should disable gzip for SSL traffic.
        # See: https://bugs.debian.org/773332
        #
        # Read up on ssl_ciphers to ensure a secure configuration.
        # See: https://bugs.debian.org/765782
        #
        # Self signed certs generated by the ssl-cert package
        # Don't use them in a production server!
        #
        # include snippets/snakeoil.conf;

        #hp root /var/www/html;

        # Add index.php to the list if you are using PHP
        #hp index index.html index.htm index.nginx-debian.html;

        #hp server_name _;

        #hp location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
        #hp     try_files $uri $uri/ =404;
        #hp}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php7.0-cgi alone:
        #       fastcgi_pass 127.0.0.1:9000;
        #       # With php7.0-fpm:
        #       fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}

```
