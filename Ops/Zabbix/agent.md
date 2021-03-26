# 监控客户端操作

## 操作系统

### 下载客户端

Zabbix客户端官方下载地址 https://www.zabbix.com/cn/download_agents

Red Hat/CentOS 也可以从阿里镜像下载 https://mirrors.aliyun.com/zabbix/zabbix/4.0/rhel/7/x86_64/

### 开机启动

Windows使用msi安装完毕后，可以在《服务》找到zabbix agent，默认是开机启动的

Red Hat/CentOS使用rpm包安装后，可配置开机启动

### 配置文件

Windows安装目录的zabbix_agentd.conf

Red Hat/CentOS在/etc/zabbix/zabbix_agentd.conf

Server=zabbix服务ip地址

ServerActive=zabbix服务ip地址

Hostname=XXXX

配置文件修改后需要重启服务

### 开启防火墙端口

Zabbix服务端通过访问客户端的10050端口，所以需要在防火墙上配置开放对应的端口。

CentOS`firewall-cmd --permanent --zone=public --add-service=zabbix-agent`
或者`firewall-cmd --permanent --zone=public --add-port=10050/tcp`来开启。
执行`firewall-cmd --reload`生效

## 数据库

Zabbix服务端通过ODBC来实时监控数据库。

数据库需要开通监控的用户并授予对应的权限。

- MySQL
    需要授权用户权限如下
    `GRANT USAGE,REPLICATION CLIENT,PROCESS,SHOW DATABASES,SHOW VIEW ON *.* TO '<username>'@'%';`

- Oracle
    需要授权用户权限如下
    ```
    CREATE USER zabbix_mon IDENTIFIED BY <PASSWORD>;
    – Grant access to the zabbix_mon user.
    GRANT CONNECT, CREATE SESSION TO zabbix_mon;
    GRANT SELECT ON V_$instance TO zabbix_mon;
    GRANT SELECT ON V_$database TO zabbix_mon;
    GRANT SELECT ON v_$sysmetric TO zabbix_mon;
    GRANT SELECT ON v$recovery_file_dest TO zabbix_mon;
    GRANT SELECT ON v$active_session_history TO zabbix_mon;
    GRANT SELECT ON v$osstat TO zabbix_mon;
    GRANT SELECT ON v$restore_point TO zabbix_mon;
    GRANT SELECT ON v$process TO zabbix_mon;
    GRANT SELECT ON v$datafile TO zabbix_mon;
    GRANT SELECT ON v$pgastat TO zabbix_mon;
    GRANT SELECT ON v$sgastat TO zabbix_mon;
    GRANT SELECT ON v$log TO zabbix_mon;
    GRANT SELECT ON v$archive_dest TO zabbix_mon;
    GRANT SELECT ON v$asm_diskgroup TO zabbix_mon;
    GRANT SELECT ON sys.dba_data_files TO zabbix_mon;
    GRANT SELECT ON DBA_TABLESPACES TO zabbix_mon;
    GRANT SELECT ON DBA_TABLESPACE_USAGE_METRICS TO zabbix_mon;
    GRANT SELECT ON DBA_USERS TO zabbix_mon;
    ```

## 中间件

Zabbix服务端通过JMX来实时监控中间件。

中间件需要开启JMX并远程提交到Zabbix服务端。

- [Tomcat](jmx.md)

- [WebSphere](jmx.md)

## 网络设备

Zabbix服务端通过SNMP协议来实时监控网络设备。

网络设备需要提供IP、端口、团体名(COMMUNITY)
