# 防火墙

## 服务配置地址

- 内置服务 /usr/lib/firewalld/services/
- 自定义服务 /etc/firewalld/services/

自定义服务参考
```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Redis</short>
  <description>Redis Server</description>
  <port protocol="tcp" port="6379"/>
</service>
```

## firewall命令

### 添加、删除服务

```
# --add-service|--remove-service
firewall-cmd --permanent --zone=public --add-service=http
```

### 添加、删除端口
```
# --add-port|--remove-port
firewall-cmd --permanent --zone=public --add-port=8080/tcp
```

### 其他

```
firewall-cmd --reload
firewall-cmd --list-ports
firewall-cmd --list-services
```

