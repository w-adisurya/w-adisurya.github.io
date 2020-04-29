---
layout: post
title: "High Availability PostgreSQL 9.6 using Bucardo"
date: 2019-10-16 22:32:00 +0700
categories: [IT]
---

**Requirement:**

|          	| User Requirement 	| Installed                         	| How to check        	|
|----------	|------------------	|-----------------------------------	|---------------------	|
| OS       	| centos6.x        	| CentOS release 6.9 (Final)        	| /etc/centos-release 	|
| Postgres 	| 9.6              	| 9.6.15                            	| SELECT version ();  	|  

<head>						<!--- Memberikan border line pada table --->
<style>
table 

td, th {
  border: 1px solid #dddddd;
  text-align: left;
  padding: 8px;
}

tr:nth-child(even) {
  background-color: #dddddd;
}
</style>
</head>
<br/>							<!--- Memberikan blank line --->
Buat dua Virtual Machine dengan konfigurasi sbb:      
  
**Dummy Server:**

| IP            	| Hostname   	|               	|
|---------------	|------------	|---------------	|
| 192.168.10.6  	|            	| Floating IP   	|
| 192.168.10.10 	| node5.test 	| Master Server 	|
| 192.168.10.11 	| node6.test 	| Slave Server  	|

**Note**:
+	<em>Pastikan server master dan slave sama2 mempunyai eth0</em>
+	<em>Backup configuration file dengan nama .backup</em>

### Install Bucardo in Master and Slave
	wget https://bucardo.org/downloads/DBIx-Safe-1.2.5.tar.gz
	tar xvfz dbix_safe.tar.gz
	cd DBIx-Safe-1.2.5
	perl Makefile.PL
	make
	make test
	sudo make install
	wget https://bucardo.org/downloads/Bucardo-5.5.0.tar.gz
	tar xvfz Bucardo-5.5.0.tar.gz
	perl Makefile.PL
	make
	sudo make install
	sudo mkdir /var/run/bucardo
	sudo chmod 777 /var/run/bucardo
	su postgres
	psql
	create user bucardo superuser;
	create database bucardo;
	alter database bucardo owner to bucardo;
	\q
	exit
	sudo bucardo install
	bucardo show all

### Create Database and tables in Master and Slave  
Bucardo mandatory that every table should have primary key for replication purpose  

	su postgres
	psql
	create database mus;
	\c mus
	create table tab1( id integer primary key,num integer);
	create table tab2( id integer primary key,num integer);
	create table tab3( id integer primary key,num integer);
	\dt
	\q

### Make Replication with Bucardo, setting only in Master  
bucardo add database serv1 dbname=mus
bucardo add database serv2 dbname=mus host=192.168.10.11 dbuser=postgres dbpass=postgres  
nano /var/lib/pgsql/9.6/data/pg_hba.conf  

	# "local" is for Unix domain socket connections only
	local   all             bucardo                                 trust
	local   all             postgres                                trust
	local   all             all                                     trust
	# IPv4 local connections:
	host    all             all             127.0.0.1/32            trust
	host    all             all             127.0.0.1/32            md5
	host    all             all             192.168.10.4/24         md5
	host    all             all             192.168.10.11/24        trust

bucardo add table % db=serv1  
bucardo add table % db=serv2  
bucardo add all tables --herd=one_two db=serv1  
bucardo add all tables --herd=two_one db=serv2  
bucardo add sync sync_onetotwo relgroup=one_two db=serv1,serv2  
bucardo add sync sync_twotoone relgroup=two_one db=serv2,serv1  
bucardo list database  
bucardo list sync  
bucardo status  
mkdir /var/log/log.bucardo  
tail -f /var/log/bucardo/log.bucardo  
bucardo start  
bucardo show all  

### Testing  
Using DBeaver Enterprise to Generate Mock Data in tables, make 1500 row in DB Master  
![dbeaver-mock](\images\dbeaver-mock.JPG "dbeaver-mock")
![dbeaver-mock2](\images\dbeaver-mock2.JPG "dbeaver-mock2")  
<br/>
Check DB in Slave should be sama with data in DB Master  
![dbeaver-mock3](\images\dbeaver-mock3.JPG "dbeaver-mock3")  
<br/>
Check Bucardo status replication  
![dbeaver-mock4](\images\dbeaver-mock4.JPG "dbeaver-mock4")  
<br/>
Remove data from DB Slave in tab1, replace with 200 row data  
![dbeaver-mock5](\images\dbeaver-mock5.JPG "dbeaver-mock5")  
![dbeaver-mock6](\images\dbeaver-mock6.JPG "dbeaver-mock6") 
<br/> 
Check in DB Master, should be 200 row data, same with DB Slave  
![dbeaver-mock7](\images\dbeaver-mock7.JPG "dbeaver-mock7") 
<br/>

### Syntax for backup restore DB
su postgres
psql
pg_dump "db_name" > "place to save backup sql"
![backup-db1](\images\backup-db1.JPG "backup-db1")
<br/>
To restore, first create database to restore data
![backup-db2](\images\backup-db2.JPG "backup-db2")  
psql "db_name" < "backup sql"
<br/>
![backup-db3](\images\backup-db3.JPG "backup-db3")  
Check database in pgAdmin
<br/>
![backup-db4](\images\backup-db4.JPG "backup-db4")
![backup-db5](\images\backup-db5.JPG "backup-db5")