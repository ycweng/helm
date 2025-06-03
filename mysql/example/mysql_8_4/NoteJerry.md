# note from jerry

    1. 安裝MySQL
```shell
dnf localinstall -y https://dev.mysql.com/get/mysql84-community-release-el8-1.noarch.rpm
dnf module disable mysql

dnf install mysql-community-server mysql-shell
dnf install mysql-router

systemctl start mysqld.service
systemctl enable mysqld.service
systemctl status mysqld
grep 'temporary password' /var/log/mysqld.log

mysql_secure_installation
alter user 'root'@'localhost' identified by 'e_kR40s*';
flush privileges;

create user 'root'@'%' identified  by 'e_kR40s*';
grant all privileges on *.* to  'root'@'%';
flush privileges;
```
    2. MySQL Innodb cluster
        a. vi /etc/hosts
```shell
10.100.54.31 mongodb1 innodb-node1 node1
10.100.54.32 mongodb2 innodb-node2 node2
10.100.54.33 mongodb3 innodb-node3 node3
```
b. all node
```shell
vi /etc/my.cnf

server-id=32
log-bin=/mysql/binlog/mybin
relay-log=/mysql/relaylog/relay-log

plugin-dir= /usr/lib64/mysql/plugin
enforce_gtid_consistency = ON
gtid-mode=ON
log-slave-updates = ON
binlog-checksum=NONE
slave-parallel-workers=8
slave_preserve_commit_order=ON

disabled_storage_engines="MyISAM,BLACKHOLE,FEDERATED,ARCHIVE,MEMORY"


plugin_load_add='group_replication.so'
group_replication_start_on_boot = OFF
group_replication_bootstrap_group = OFF
group_replication_ip_allowlist = '127.0.0.1/8,10.100.54.0/24'
loose-group_replication_single_primary_mode = ON
group_replication_ssl_mode         = REQUIRED
group_replication_recovery_use_ssl = ON

init_connect='SET NAMES utf8mb4'
binlog_expire_logs_seconds=604800
slow_query_log            = on
slow_query_log_file       = mysql-slow.log
max_connections           = 2000
max_connect_errors        = 100
innodb_file_per_table     = 1
validate_password.policy=LOW
validate_password.length  = 6
default-time-zone = '+08:00'
interactive_timeout=25920000
wait_timeout=25920000
transaction_isolation='READ-COMMITTED'

systemctl restart mysqld
```
        c. all node
```shell
SET SQL_LOG_BIN=0;
CREATE USER icadmin@'10.100.54.%' IDENTIFIED BY 'Sq2%OIcI3#P';
GRANT ALL ON *.* TO icadmin@'10.100.54.%' WITH GRANT OPTION;
SET SQL_LOG_BIN=1;
```
        d. all node
```shell
mysqlsh
\c icadmin@10.100.54.31:3306
\c icadmin@10.100.54.32:3306
\c icadmin@10.100.54.33:3306
```
        e.  MySQL  10.100.54.31:3306 ssl  SQL > \js
```shell
dba.checkInstanceConfiguration('icadmin@10.100.54.31:3306');
dba.checkInstanceConfiguration('icadmin@10.100.54.32:3306');
dba.checkInstanceConfiguration('icadmin@10.100.54.33:3306');
```
        f. 
```shell
dba.configureInstance('icadmin@10.100.54.31:3306', {clusterAdmin: "'icadmin'@'10.100.54.%'"});
dba.configureInstance('icadmin@10.100.54.32:3306', {clusterAdmin: "'icadmin'@'10.100.54.%'"});
dba.configureInstance('icadmin@10.100.54.33:3306', {clusterAdmin: "'icadmin'@'10.100.54.%'"});
```
        g. 
```shell
var cluster=dba.getCluster()
cluster.addInstance('icadmin@10.100.54.32:3306')
cluster.status()
cluster.addInstance('icadmin@10.100.54.33:3306')
cluster.status()
```



    3. MySQL Router
        a. mysqlrouter –version
```shell
mysqlrouter --bootstrap icadmin@10.100.54.31:3306 --user=mysqlrouter
systemctl start mysqlrouter.service
systemctl enable mysqlrouter.service
cat /etc/mysqlrouter/mysqlrouter.conf
```
b.

    4. Note
        a. primary db read write
other read only
b. Need PK
c. ERROR 3100 (HY000): Error on observer while running replication hook 'before_commit'

Looking into MySQL error shows:
Error on session 157083. Transaction of size 261566573 exceeds specified limit 150000000. To increase the limit please adjust group_replication_transaction_size_limit option.
From MySQL 8.0, the default setting for this system variable is 150000000 bytes (approximately 143 MB).
d. 