# MySQL

## 安装mariadb

1. 使用命令`yum install mariadb-server`安装数据库

## ODBC配置和使用

yum install unixODBC unixODBC-devel mysql-connector-odbc

## 配置mariadb
```
mysql -u root
grant all privileges on *.* to 'root'@'localhost' identified by 'root';
flush privileges;
revoke all privileges on *.* from 'root'@'%';
show grants;
```
   
## 初始化数据库DRP
```
create database drp default charset utf8 collate utf8_general_ci;
grant all privileges on drp.* to 'drp'@'%' identified by 'drp';
flush privileges;
```

## 导出数据库
```
mysqldump -uroot -p123 test > test.dump
# 导出存储过程
-R (--routines:导出存储过程以及自定义函数)
```


## 导入数据库
```
mysql -uroot -p

use drp

source /root/html/drp.sql
```


## 重置root密码
```
# 安全模式进行MariaDB
systemctl stop mariadb
systemctl set-environment MYSQLD_OPTS="--skip-grant-tables"
systemctl start mariadb
systemctl status mariadb

# 进入bash,连接密码库
mysql -uroot
use mysql;
update user set Password=PASSWORD("admin@1234") where User='root';
flush privileges; 

systemctl stop mariadb
systemctl unset-environment MYSQLD_OPTS
systemctl start mariadb
systemctl status mariadb
```
