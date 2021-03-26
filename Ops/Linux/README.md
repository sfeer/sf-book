# Linux

## 基础知识

### 查看内核版本 

```
# 查看内核相关信息
cat /proc/version
# 查看内核名称，即操作系统名称
uname -s
Linux
# 查看机器硬件名称，即CPU类型
uname -m
x86_64
# 查看 CentOS 版本
cat /etc/redhat-release
CentOS Linux release 7.7.1908 (Core)
```

### 系统信息
```
# 查看内核相关信息
cat /proc/version

# 查看系统中断信息（通过eth0绑定的中断记录查看网卡的中断是否均匀分配到每个CPU）
cat /proc/interrupts

# CPU信息
cat /proc/cpuinfo
# cpu cores 核数
# processor 逻辑CPU个数
# physical id 物理CPU个数
dmidecode -t processor

# 内存信息
cat /proc/meminfo
dmidecode -t memory

# hosts文件
cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# DNS配置
cat /etc/resolv.conf
DNS1=8.8.8.8
DNS2=114.114.114.114
nameserver 114.114.114.114
nameserver 8.8.8.8
search localdomain

# 映射端口文件
cat /etc/services|grep mysql
mysql           3306/tcp                        # MySQL
mysql           3306/udp                        # MySQL
mysql-cluster   1186/tcp                # MySQL Cluster Manager
mysql-cluster   1186/udp                # MySQL Cluster Manager
mysql-cm-agent  1862/tcp                # MySQL Cluster Manager Agent
mysql-cm-agent  1862/udp                # MySQL Cluster Manager Agent
mysql-im        2273/tcp                # MySQL Instance Manager
mysql-im        2273/udp                # MySQL Instance Manager
mysql-proxy     6446/tcp                # MySQL Proxy
mysql-proxy     6446/udp                # MySQL Proxy

# 账户密码
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
# 用户名:密码:UID:GID:描述:主目录:登录后执行的命令（nologin或者创建环境变量）
```

### 文件权限

```
ll /usr/bin
lrwxrwxrwx    1 root root          13 Sep 24  2019 pango-querymodules-64 -> /usr/bin/true
-rwxr-xr-x    1 root root       50656 Sep 14  2019 pango-view
-rwxr-xr-x    1 root root        4737 Jun 24  2015 passmass
-rwsr-xr-x    1 root root       27856 Aug  9  2019 passwd

# 第一位显示 l链接 -文件 d文件夹
# 权限分三组：所有者|所属组|其他人
# r读 w写 x执行 s特殊

# chmod u+x xxx.sh
# u:所有者, g:所属组,o:其他人,a:所有

# setuid特殊权限
# Linux系统中某个可执行文件属于root并且有setuid
# 当一个普通用户mike运行这个程序时
# 产生的进程的有效用户和实际用户分别是root mike
chmod u+s /usr/bin/passwd
```

### 性能分析

```
# CPU
top
# 内存
free
# 磁盘I/O
iostat -x 10
```

### 磁盘空间

```
df -h
du -sh *
```

### 同步时间

后台服务对时间比较敏感，需保证集群内所有服务器的本地时间一致，默认安装后检查crontab -l 显示有如下条目：
```
#SyncTime
*/23 * * * * /etc/cron.hourly/SyncTime.sh > /dev/null 2>&1
```
修改 /etc/cron.hourly/SyncTime.sh
```
#将 cn.ntp.org.cn 修改为集群内机器能访问的 NTP 服务器 IP
/usr/sbin/ntpdate  cn.ntp.org.cn >/dev/null 2>&1
```

### 设置ip

`vi /etc/sysconfig/network-scripts/ifcfg-ens33`

### 查找服务目录

`ps -ef | grep tomcat` 得到进程号
`ls -l /proc/xxxx/cwd`

### 关闭SELinux

1. 临时关闭
setenforce 0 ：用于关闭selinux防火墙，但重启后失效。

[root@localhost ~]# setenforce 0
[root@localhost ~]# /usr/sbin/sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   permissive
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      28

2. 永久关闭
修改selinux的配置文件，重启后生效。

打开 selinux 配置文件
[root@localhost ~]# vim /etc/selinux/config
修改 selinux 配置文件
将SELINUX=enforcing改为SELINUX=disabled，保存后退出

### 系统服务

[systemd](https://acherstyx.github.io/2020/07/07/Systemd%E6%9C%8D%E5%8A%A1%E8%84%9A%E6%9C%AC/)
`/usr/lib/systemd/system`： 系统服务，运行具有管理员权限。
`/usr/lib/systemd/user`：用户服务，只有在用户登陆之后才开始运行。

## 常用命令

### yum

```
# 更新
yum update
# 安装
yum install xxx
# 查询
yum search xxx
# 列出已装
yum list installed
# 清理
yum clean all
```

配置目录`/etc/xxx/...`
主体目录`/usr/lib/xxx/...`
文档目录`/usr/share/doc/xxx/...`

### rpm

```
# 安装
rpm -ivh xxx.rpm
# 卸载
rpm -e 包名
# 查询
rpm -qa | grep mysql
# 列出安装目录
rpm -ql php-mysql-5.4.16-46.1.el7_7.x86_64
/etc/php.d/mysql.ini
/etc/php.d/mysqli.ini
/etc/php.d/pdo_mysql.ini
/usr/lib64/php/modules/mysql.so
/usr/lib64/php/modules/mysqli.so
/usr/lib64/php/modules/pdo_mysql.so
```

### ifconfig

```
ifconfig eth0 up
ifconfig eth0 down
```

### tar

解压/打包命令

-cvf 打包
-xvf 解压
-z   gzip
```
# 压缩所有log文件
tar -cvf log.tar *.log
# 解压log.tar
tar -xvf log.tar
```

### find

```
# 列出目录下java文件
find . -name "*.java"
# 列出目录下一般文件
find . -type f
```

### df

查看磁盘空间使用情况
-h 易读的格式显示空间
```
df -h
```

### sed

流编辑器（常用语文本替换）

### awk

文本分析工具
```
cat /etc/passwd |awk  -F ':'  '{print $1}'  
root
daemon
bin
sys
```

### cut

剪切文件内容

### wc

统计文本行数、字数等
```
cat /etc/passwd | wc -l
```

### crontab

调度管理命令

调度文件格式说明

|含义  |范围
|---- |----
|分钟  |0-59
|小时  |0-23
|日期  |1-31
|月份  |1-12
|星期几 |0-7（0和7都代表星期日）
|执行命令|xxx.sh

- 星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
- 逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
- 中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
- 正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次。

```
# crond服务（crontab基于crond服务）
systemctl status crond
# 查看调度
crontab -l
# 删除调度
crontab -r
# 编辑调度（操作同vi）
crontab -e
```

### mount

将设备挂载到文件目录

-t 指定文件类型（cat /etc/filesystems 查看文件类型）
-o 选项

```
# 查看分区情况
fdisk -l
# 挂载设备
mount /dev/sdb1 /mnt
```

### systemctl

CentOS7开始使用systemctl命令
```
# 查看日志
journalctl -xe
journalctl -f -u tomcat
```

### firewall

服务配置地址：
内置服务 `/usr/lib/firewalld/services/`
自定义服务 `/etc/firewalld/services/`

添加、删除服务（--add-service|--remove-service）
`firewall-cmd --permanent --zone=public --add-service=http`

添加、删除端口（--add-port|--remove-port）
`firewall-cmd --permanent --zone=public --add-port=8080/tcp`

重新加载
```
firewall-cmd --reload
firewall-cmd --list-ports
firewall-cmd --list-services
```

### nohup

后台执行命令
```
# 启动微服务
nohup java -jar xxx.jar >/dev/null 2>&1 &
# 导入大量数据
nohup mysql -udrp -pr00t@126.com drp<drp.sql >/dev/null 2>&1 &
```

### scp

scp 是 secure copy 的缩写, scp 是 linux 系统下基于 ssh 登陆进行安全的远程文件拷贝命令。
```
usage: scp [-12346BCpqrv] [-c cipher] [-F ssh_config] [-i identity_file]
           [-l limit] [-o ssh_option] [-P port] [-S program]
           [[user@]host1:]file1 ... [[user@]host2:]file2
-P ssh端口
-r 复制目录
```

### rdtscp指令

查看cpu是否支持 **rdtscp** `grep rdtscp /proc/cpuinfo`

### 查看网卡中断

查看系统中断信息 `cat /proc/interrupts`

```
TODO
印象笔记官网上有
setuid
Linux系统中某个可执行文件属于root并且有setuid，当一个普通用户mike运行这个程序时，产生的进程的有效用户和实际用户分别是root mike

ifconfig参数
find参数
mount/umount参数
/etc/fstab

查看磁盘空间
用法：df [选项]... [文件]...
Show information about the file system on which each FILE resides,
or all file systems by default.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all             include pseudo, duplicate, inaccessible file systems
  -B, --block-size=SIZE  scale sizes by SIZE before printing them; e.g.,
                           '-BM' prints sizes in units of 1,048,576 bytes;
                           see SIZE format below
      --direct          show statistics for a file instead of mount point
      --total           produce a grand total
  -h, --human-readable  print sizes in human readable format (e.g., 1K 234M 2G)
  -H, --si              likewise, but use powers of 1000 not 1024
  -i, --inodes		显示inode 信息而非块使用量
  -k			即--block-size=1K
  -l, --local		只显示本机的文件系统
      --no-sync		取得使用量数据前不进行同步动作(默认)
      --output[=FIELD_LIST]  use the output format defined by FIELD_LIST,
                               or print all fields if FIELD_LIST is omitted.
  -P, --portability     use the POSIX output format
      --sync            invoke sync before getting usage info
  -t, --type=TYPE       limit listing to file systems of type TYPE
  -T, --print-type      print file system type
  -x, --exclude-type=TYPE   limit listing to file systems not of type TYPE
  -v                    (ignored)
      --help		显示此帮助信息并退出
      --version		显示版本信息并退出

所显示的数值是来自 --block-size、DF_BLOCK_SIZE、BLOCK_SIZE 
及 BLOCKSIZE 环境变量中第一个可用的 SIZE 单位。
否则，默认单位是 1024 字节(或是 512，若设定 POSIXLY_CORRECT 的话)。

SIZE is an integer and optional unit (example: 10M is 10*1024*1024).  Units
are K, M, G, T, P, E, Z, Y (powers of 1024) or KB, MB, ... (powers of 1000).

FIELD_LIST is a comma-separated list of columns to be included.  Valid
field names are: 'source', 'fstype', 'itotal', 'iused', 'iavail', 'ipcent',
'size', 'used', 'avail', 'pcent', 'file' and 'target' (see info page).

GNU coreutils online help: <http://www.gnu.org/software/coreutils/>
请向<http://translationproject.org/team/zh_CN.html> 报告df 的翻译错误
要获取完整文档，请运行：info coreutils 'df invocation'
df -BG 显示磁盘GB的空间信息

crontab参数和6个域的意义
/etc/hosts /etc/resolv.conf /etc/networks /etc/services /etc/passwd详解
sed awk cut wc了解
A B C D类IP地址
/proc/interrupts CPU 内存等查看
DHCP SNMP了解
http https mysql nginx ssh ftp等常用软件默认端口
RPM参数
TCP UCP了解
VLAN了解
tar zip压缩/解压命令了解
```

## 相关知识

### VLAN

虚拟局域网，VLAN之间通讯隔离

### TCP/UDP

TCP：Transmission Control Protocol 传输控制协议 面向连接 可靠 少量数据
UDP：User Datagram Protocol 用户数据报协议 无连接 不可靠 大量数据
同属传输层协议

### DHCP/SNMP

DHCP：Dynamic Host Configuration Protocol 动态主机配置协议 分配IP地址
SNMP：Simple Network Management Protocol 简单网络管理协议 MIB库 管理网络节点
同属应用层协议
