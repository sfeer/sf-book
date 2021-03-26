# Zabbix

## 介绍

Zabbix 是一个基于 WEB 界面的提供分布式系统监视以及网络监视功能的企业级的开源解决方案。

它能监视各种网络参数，保证服务器系统的安全运营

提供灵活的通知机制以让系统管理员快速定位，解决存在的各种问题

- 阿里云镜像地址：https://mirrors.aliyun.com/zabbix
- 官方主页：https://www.zabbix.com
- Github：https://github.com/zabbix

## 安装

参加[官方教程](https://www.zabbix.com/cn/download)

CentOS修改官方镜像为阿里云的镜像： 

`sed -i 's/repo.zabbix.com/mirrors.aliyun.com\/zabbix/g' /etc/yum.repos.d/zabbix.repo`

默认安装会开启`odbc`所以安装的依赖里会有`unixODBC`

### 环境变量设置

systemd启动的服务不会读取/etc/profile
导致动态链接库的地址不能正常加载

```
cat /usr/lib/systemd/system/zabbix-server.service 
[Unit]
Description=Zabbix Server
After=syslog.target
After=network.target
After=mysql.service
After=mysqld.service
After=mariadb.service
After=postgresql.service
After=pgbouncer.service
After=postgresql-9.4.service
After=postgresql-9.5.service
After=postgresql-9.6.service
After=postgresql-10.service
After=postgresql-11.service
After=postgresql-12.service

[Service]
Environment="CONFFILE=/etc/zabbix/zabbix_server.conf"
EnvironmentFile=-/etc/sysconfig/zabbix-server
Type=forking
Restart=on-failure
PIDFile=/run/zabbix/zabbix_server.pid
KillMode=control-group
ExecStart=/usr/sbin/zabbix_server -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s
TimeoutSec=0

[Install]
WantedBy=multi-user.target
```

添加`Environment="LD_LIBRARY_PATH=/opt/dmdbms/bin:/opt/ShenTong/odbc/lib"`这行地址解决了这个问题

完成后需要执行
```
systemctl daemon-reload
systemctl restart 
```

## 配置

### 模版

https://share.zabbix.com 上面有官方分享的模版

### [告警](alert.md)

### ODBC

### [JMX](jmx.md)

### SNMP

网络设备主要基于snmpv2的版本，测试snmp的工具

内置思科和华为的snmpv2的监控模版

也有基于snmpv2的通用模版

## 待整理

### Zabbix 数据库
官方文档
cat /usr/share/doc/zabbix-agent-5.0.3/userparameter_mysql.conf

### Zabbix 中间件

JavaGateway=127.0.0.1  #修改为zabbix-java-gateway所在主机的ip地址，这里是和zabbix-server安装在同一台主机所以为127.0.0.1
JavaGatewayPort=10052  #因为zabbix-java-gateway  默认监控端口为10052
StartJavaPollers=5     #zabbix-java-gateway 默认启动工作线程数量为5

### 帮助文档
 
https://www.zabbix.com/documentation/current/manual/config/templates_out_of_the_box/odbc_checks
https://git.zabbix.com/projects/ZBX/repos/zabbix/browse/templates/db/mysql_odbc/README.md
https://www.zabbix.com/documentation/current/manual/config/items/itemtypes/odbc_checks/unixodbc_mysql

### 常用命令

zabbix_get -s xxx.xxx.xxx.xxx -p 10050 -k "system.uname"
zabbix_sender -s "aiop-db" -z 127.0.0.1 -p 10051 -k "mysql.version" -o "hehe" -vv
zabbix前端监控项可以测试

### Docker

#### Zabbix数据库 MySQL
```
docker pull mysql:5.7

docker run --name mysql-server -t \
-e MYSQL_DATABASE="zabbix" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-e MYSQL_ROOT_PASSWORD="zabbix" \
-p 127.0.0.1:3306:3306 \
-d mysql:5.7 \
--character-set-server=utf8 --collation-server=utf8_bin
```

#### Zabbix服务器
```
docker pull zabbix/zabbix-server-mysql:centos-4.0-latest

docker run --name zabbix-server-mysql -t \
--link mysql-server:mysql \
-e DB_SERVER_HOST="mysql-server" \
-e MYSQL_DATABASE="zabbix" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-e MYSQL_ROOT_PASSWORD="zabbix" \
-p 10051:10051 \
-d zabbix/zabbix-server-mysql:centos-4.0-latest
```

#### Zabbix管理后台
```
docker pull zabbix/zabbix-web-nginx-mysql:centos-4.0-latest

docker run --name zabbix-web-nginx-mysql -t \
--link mysql-server:mysql \
--link zabbix-server-mysql:zabbix-server \
-e DB_SERVER_HOST="mysql-server" \
-e MYSQL_DATABASE="zabbix" \
-e MYSQL_USER="zabbix" \
-e MYSQL_PASSWORD="zabbix" \
-e MYSQL_ROOT_PASSWORD="zabbix" \
-e PHP_TZ="Asia/Shanghai" \
-p 80:80 \
-d zabbix/zabbix-web-nginx-mysql:centos-4.0-latest
```

#### docker-compose配置
```
version: '3'
services:
  aiop-admin:
    build:
      context: ./
      dockerfile: ./aiop-admin/Dockerfile
    restart: always
    container_name: aiop-admin
    image: aiop-admin
    ports:
      - 8081:8081

  aiop-wx:
    build:
      context: ./
      dockerfile: ./aiop-wx/Dockerfile
    restart: always
    container_name: aiop-wx
    image: aiop-wx
    ports:
      - 8082:8082

  zabbix-mysql:
    container_name: zabbix-mysql
    restart: always
    image: mysql:5.7
    ports:
      - 127.0.0.1:3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix
      MYSQL_DATABASE: zabbix

  zabbix-server-mysql:
    container_name: zabbix-server-mysql
    restart: always
    image: zabbix/zabbix-server-mysql:alpine-4.0-latest
    ports:
      - 10051:10051
    depends_on:
      - zabbix-mysql
    links:
      - zabbix-mysql:mysql
    environment:
      DB_SERVER_HOST: zabbix-mysql
      MYSQL_USER: zabbix
      MYSQL_DATABASE: zabbix
      MYSQL_PASSWORD: zabbix

  zabbix-web-nginx-mysql:
    container_name: zabbix-web-nginx-mysql
    restart: always
    image: zabbix/zabbix-web-nginx-mysql:alpine-4.0-latest
    ports:
      - 8082:8080
    depends_on:
      - zabbix-mysql
      - zabbix-server-mysql
    links:
      - zabbix-mysql:mysql
      - zabbix-server-mysql:zabbix-server
    environment:
      DB_SERVER_HOST: zabbix-mysql
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix
      MYSQL_DATABASE: zabbix
      ZBX_SERVER_HOST: zabbix-server-mysql
      PHP_TZ: Asia/Shanghai
```
