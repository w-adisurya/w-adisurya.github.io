---
layout: post
title: "High Availability PHP and Apache using heartbeat"
date: 2019-10-16 14:10:00 +0700
categories: [IT]
---

**Requirement:**

|          	| User Requirement 	| Installed                         	| How to check        	|
|----------	|------------------	|-----------------------------------	|---------------------	|
| OS       	| centos6.x        	| CentOS release 6.9 (Final)        	| /etc/centos-release 	|
| Apache   	| bebas            	| httpd-2.2.15-69.el6.centos.x86_64 	| rpm -q httpd        	|
| PHP      	| 7                	| 7.0.33                            	| php -v              	|
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

### Both Server config
	yum install epel-release
	yum install httpd -y
	yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-6.noarch.rpm
	yum install http://rpms.remirepo.net/enterprise/remi-release-6.rpm
	yum install yum-utils
	yum-config-manager --enable remi-php70
	yum install php
	yum install heartbeat -y
	service httpd start
	chkconfig httpd

### Master Server Config
echo "node5 server1 hidup" > /var/www/html/index.html  
cd /etc/ha.d/  
touch authkeys ha.cf haresources  
	
vim /etc/ha.d/authkeys

	auth3
	3 md5 test

vim /etc/ha.d/ha.cf

	bcast eth0
	keepalive 2
	warntime 5
	deadtime 10
	initdead 20
	autojoin none
	node node5.test
	node node6.test
	auto_failback on
	debugfile /var/log/ha-debug
	logfile /var/log/ha-log
	logfacility local0
	
vim /etc/ha.d/haresources

	node5.test IPaddr::192.168.10.10/24/eth0 httpd

scp /etc/ha.d/ha.cf 192.168.10.11:/etc/ha.d/  
scp /etc/ha.d/haresources 192.168.10.11:/etc/ha.d/  
scp /etc/ha.d/authkeys 192.168.10.11:/etc/ha.d/  

### Slave Server Config
	echo "node5 server1 sedang down, saat ini anda ada di node6 server2" > /var/www/html/index.html

### Check HA
	service heartbeat start

Open browser, cek 192.168.10.10, test failover  
![apache](\images\apache-heartbeat.jpg "Apache HA test")

## Configure Web Directory Replication
### Master Server (Client Machine) Config
Setup SSH Keys for password less logon :

	ssh-keygen
	cat ~/.ssh/id_rsa.pub | ssh root@192.168.10.11 "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && 
	chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
	ssh root@192.168.10.11
	
![passwordless-logon](\images\passwordless-logon.jpg "passwordless-logon")  
<br/>				<!--- Memberikan blank line --->
Setup Crontab and rsync :

	crontab -e
	* * * * * /usr/bin/rsync -rtvuc --delete-before /var/www/html/ root@192.168.10.11:/var/www/html/

Setting diatas akan menjalankan sinkronisasi folder /var/www/html dari Master Server ke Slave Server setiap 1 menit, perubahan yang disinkronisasi hanya perubahan di Master Server.

![crontab](\images\crontab.JPG "crontab")

