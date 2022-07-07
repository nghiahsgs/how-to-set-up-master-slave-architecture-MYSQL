# how-to-set-up-master-slave-architecture-MYSQL
how to set up master slave architecture MYSQL


# STEP A: install mysql for 3 vps (1 master and 2 slave)

1/ install mysql
```
sudo apt-get update
sudo apt-get install mysql-server
```

2/ change password for root
```
sudo mysql
```
```sql
SELECT user,authentication_string,plugin,host FROM mysql.user;

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';

FLUSH PRIVILEGES;

SELECT user,authentication_string,plugin,host FROM mysql.user;

```

check status mysql
```
sudo /etc/init.d/mysql status
```

3/ install php myadmin
```
apt update && upgrade

sudo add-apt-repository universe

apt install phpmyadmin
```

4/ check phpmyadmin install successful or not
```
ip_vps/phpmyadmin/

```
5/ Open firewall
```
sudo apt install ufw
```

```
ufw enable
ufw status
ufw allow 80
ufw allow 443
ufw allow 3306
```

6/ Fix 404 eror for php myadmin
```
sudo ln -s /etc/phpmyadmin/apache.conf /etc/apache2/conf-available/phpmyadmin.conf
sudo a2enconf phpmyadmin.conf
sudo systemctl restart apache2
```

```
sudo apt-get install libapache2-mod-php
```

# STEP B: Config for master vps
1/ open file config mysql to edit
```
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

comment the bind-address
```
#bind-address =127.0.0.1
```

change the following fields
```
server-id – Unique ID of the MySQL server. This ID can not be re-used in any nodes in the cluster. (for ex: set server-id=1)

log-bin – This is the file in which all the replication information is stored.

max_binlog_size – Size of the binlog file.
```

reset mysql
```
systemctl restart mysql
```

2/ Create a new user for the Replication service

```
mysql -u root -p
```

```sql
CREATE USER replication_user@192.168.178.137 IDENTIFIED WITH mysql_native_password BY 'StrongP@ssw0rd';
```



where 192.168.178.137 is ip of slave1


```sql
GRANT REPLICATION SLAVE on *.* to replication_user@192.168.178.137;
```

```sql
SHOW GRANTS FOR replication_user@192.168.178.137;
```


# STEP C: Config for slave vps
1/ open file config mysql to edit
```
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

comment the bind-address
```
#bind-address =127.0.0.1
```

change the following fields
```
server-id – Unique MySQL server-id. (for ex: set server-id = 2)

log_bin – Enables binary logging in slave node

max_binlog_size – Size of the binlog file.

slow_query_log – Enables slow query log
```

reset mysql
```
systemctl restart mysql
```

2/ Connect slave server to master server
```
go to master server
mysql -u root -p
```

``` sql
SHOW MASTER STATUS\G

Take a note current file log and position
```

```
go to slave server
mysql -u root -p
```

```sql
STOP SLAVE;
```

```sql
CHANGE MASTER TO MASTER_HOST='192.168.178.136', MASTER_USER='replication_user', MASTER_PASSWORD='StrongP@ssw0rd', MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=1050;
```

where 192.168.178.136 is ip of master server; mysql-bin.000003 and 1050 are file name and position binary file of mysql master, 

```sql
START SLAVE;
```

```sql
SHOW SLAVE STATUS\G
```

```
check 2 important fields
Slave_IO_Running: yes
Slave_IO_Running: yes
```


# refer article
```
https://vitux.com/mysql-master-slave-replication-on-ubuntu/
```