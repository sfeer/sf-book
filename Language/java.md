# Java

## JDBC

Java Database Connectivity，简称JDBC

常用数据库JDBC

| 数据库 | 驱动 | 说明
| ---- | ---- | ----
| MySQL | com.mysql.cj.jdbc.Driver |
| MySQL for 5.1 | com.mysql.jdbc.Driver |
| MariaDB | org.mariadb.jdbc.Driver | 参数同MySQL

[IDEA指定jdk连接](IDE/README.md#数据库连接)
    

### 常见问题

- jdk8连接sqlserver报错
    * 错误描述
        ```
        Caused by: com.microsoft.sqlserver.jdbc.SQLServerException: 驱动程序无法通过使用安全套接字层(SSL)加密与 SQL Server 建立安全连接。错误:“SQL Server 未返回响应。连接已关闭。
        ```
    * 错误原因
    
        在jdk1.8.0_161版本之后会出现问题
    
        在171的Release Notes里有一条关于ssl的安全性修复：3DES算法将不被ssl使用
    
    * 错误处理
    
        修改`jre\lib\security\java.security`，删除jdk.tls.disabledAlgorithms中的3DES_EDE_CBC

