# 企业微信私有化

## T201企业微信私有化运维-部署操作

### 一、企业微信私有化介绍

#### 企业微信SaaS和私有化部署区别    

相同的产品平台，不同的交付模式
* 企业微信SaaS：面向所有中小企业 | 免费注册使用 | 快速功能迭代 | 开放的第三方应用市场
* 私有化部署：面向政府、金融、大型央企 | 许可证、有偿服务 | 需要定制化应用开发

#### 企业微信本地版和政务微信区别

同为私有化部署，后端部署方式一致
* 企业微信本地版：面向金融、央企 | 没有统一包 | 定制企业logo | 企业自主证书、自助上架 | 不支持微信红包
* 政务微信：面向政务公检法 | 政务微信统一发包 | 统一政务微信logo | 腾讯统一证书、统一上架 | 支持微信红包

### 二、企业微信私有化架构

#### 企业微信后端主要软件模块

* 接入层：反向代理-wwlnginx | 正向代理-socks5
* 连接层：短连接-wwlproxy | 长连接-wwlconn | 客户端CGI-wwllogic | 
文件CGI-wwlhttpsvr | 开放平台CGI-wwlopenlogic | 管理端CGI-wwlmngnjlogic
* 逻辑层：苹果/安卓推送-wwlapns | 登录后台-wwlauthsvr | 账号逻辑-wwlaccountsvr | 
消息推送中心-wwlpush | 聊天后台-wwlchatlogicsvr | 外部联系人-wwlcontactmq | 
云端互联-wwlreportsvr | 应用后台-wwlappsvr | 企业搜索-wwlcorpseek | 
自助查询-wwlselfhelpsvr | 管理后台-wwlmanage | 组织架构-wwlcorpsvr | 
统计监控-wwlossagent | Session后台-wwlsessionsvr | 通用MQ-wwlmqsvr | 
跨集群同步队列-wwlidmqsvr | 开放平台后台-wwlopenlogic | 序列产生器-wwlidallocsvr | 
跨集群同步服务-wwlidcsyncsvr | 公安互联模块-wwlgalogic | 公安互联回调-wwlgapcallback | 
解析音视频-wwlaudiosvr
* 存储层：结构数据存储-wwlpkv | 消息存储-wwlxkv | 临时存储-wwlredis | 
通用mysql存储-wwlmysql | 临时文件存储-wwlfile | 永久文件存储-wwlopenmedia

#### 简易网络拓扑

客户端》接入区》逻辑服务集群》存储服务集群

1. 接入服务器：导入访问流量 | 负载均衡 | 对外做正向代理
2. 逻辑服务器：逻辑运算类服务 | 临时文件存储 | 收藏文件存储
包含长短连接和CGI及微服务（即上述连接层和逻辑层）
3. 存储服务器：wwlxkv-即时通信消息存储 | wwlpkv-组织架构结构化数据存储 | wwlmysql-运维统计、日志数据 | 
XKV存储服务：基于三机协商 | 二副本一校验
PaxosKV存储服务：基于Paxos协议 | 三副本

**注：wwlpkv wwlxkv 1.5以前只有xkv，二者都是腾讯自研。**
**注：存储服务器三的倍数，同组两台坏掉会导致存储失败。**
**注：腾讯官方对外不承诺支持横向扩容。**
**注：逻辑和存储机可以混合部署在同一台机器上**

### 三、部署方案设计

#### 评估

人数估计
活跃用户大概占30%
2万 -> 6000人

每2秒1条消息，平均消息大小1KB
1KB * 6000 * 0.5 = 24Mbps

10%文件消息，平均下载带宽50KBps每用户
6000 * 0.5 * 10% * 50KB * 8 = 120Mbps

每台接入机最多支持28232个客户端连接（短连接、长连接数总和），扣除预留的大概一台接入机2.7w的客户端
```
cat /etc/sysctl.conf
net.ipv4.ip_local_port_range=32768 61000   <-- 28232个临时端口
```

| 部署要求 | 需要机器台数  | 备注
| --- | --- | ---
| 200人以下 | 1接入+1测试 | 如果存储机可以连接互联网，不需要接入机
| 2万人以下 | 2接入+3逻辑存储混合部署 | 
| 5万人以下 | 2接入+3逻辑+3存储 |
| 10万人以下 | 4接入+6逻辑+6存储 |

#### 部署方案

1. 单服务器部署（POC）
2. 独立接入机（2台高可用），逻辑存储混合（3倍）
3. 独立接入机（2条高可用），逻辑存储独立（3倍）
4. 内外网隔离接入机
    逻辑节点代理指向外网的接入机
5. 多类网络部署架构
    TCPBRIDGE桥接网闸设备
    跨网数据脱敏

### 四、部署准备和部署过程

#### 资源要求

硬件环境

- 强烈建议物理机
- CPU >= 24核，支持 rdtscp 指令 `grep rdtscp /proc/cpuinfo`
- 内存 >= 64G
- 磁盘本地磁盘： wwlpkv wwlxkv 对本地磁盘做了优化
- 存储节点做SSD **RAID10** 存储空间 >1T 每台
- 逻辑节点和存储节点必须为3的倍数
- 网卡带宽 > 1000Mbps，双网卡bounding

操作系统环境

- Centos6.5、RedHat6.5 Basic Server

网络策略

- 端口进：80/433，8080
- 端口出：80/433，8080，2195（苹果推送），2196（苹果推送）

#### 磁盘情况

- 安装目录：/home/wwlocal
- 存储节点关注：/home/wwlocal/wwlxkv /home/wwlocal/wwlpkv
- 逻辑节点关注：/home/wwlocal/wwlfilesvr /home/wwlocal/wwlopenmediasvr
- 关注磁盘IO速度
    顺序写 100MB/s 1万人以上 500MB/s
    随机读 100MB/s 1万人以上 500MB/s

#### 网络连通性

接入机需要关闭tcp_tw_recycle，关闭防止服务器快速回收socket超时的连接
```
vim /etc/sysctl.conf
net.ipv4.tcp_tw_recycle=0 <- 设置成0
sysctl -p
```
各个服务器时间同步
```
/usr/sbin/ntpdate cn.ntp.org.cn
```

测试接入机的端口联通性
进口：python -m SimpleHTTPServer 80/8080，然后外网访问
出口：外网机器wget或者curl命令来下载文件，测试一批腾讯的服务域名

#### 安装包

package安装包 patch补丁

#### 环境情况记录

《部署前环境检查表》、《部署验收报告》

#### 部署过程

- 解压安装包
```
tar -zxf wwlocal-1.5.0.tar.gz -C /home/
cd /home/wwlocal
ls
```

- 部署接入机

修改nginx配置（wwlnginx目录下）

`vi /home/wwlocal/wwlnginx/conf/nginx.conf`

执行SETUPPROXY.sh

`sh /home/wwlocal/wwlops/SETUPPROXY.sh`

检查 socks5proxy ss5代理
`curl -Iv --socks5 代理IP:8081 -U`

- 逻辑机、存储机

修改IP列表文件 /home/wwlocal/conf/gloal/ip.lst
```
LIP1=XX.XX.XX.XX
LIP2=XX.XX.XX.XX
LIP3=XX.XX.XX.XX

SIP1=XX.XX.XX.XX
SIP2=XX.XX.XX.XX
SIP3=XX.XX.XX.XX
```
修改全局配置 /home/wwlocal/conf/global/localglobal.conf，修改外网ip或域名

修改代理配置 /home/wwlocal/conf/global/wwlsocks5proxy_cli.conf，接入机IP0，IP1

执行安装脚本 sh /home/wwlocal/wwlops/SETUP.sh

检查部署情况 sh /home/wwlocal/wwlops/check.sh，对失败的服务单独重启和查看启动日志定位错误原因

检查服务是否频繁重启 grep restart /home/wwlocal/log/service/date+%Y%m%d.log

初始化企业数据（仅需在一台机器上执行一次）sh /home/wwlocal/wwlops/INIT.sh

#### License验证

- 浏览器打开http://接入机地址
- 在管理端导入License文件
- 纯内网验证，还需要额外的Token
    `/home/wwlocal/wwlops/tools/ToolsAuthLicense "license内容" "token内容"`

#### 部署后功能验证

- 确认部署网络拓扑
- 各端的功能可用性
- 找客户接口人确认工作（签字）

## T202企业微信私有化运维-运维支持

### 架构简介

#### 核心功能调用链

客户端登录

nginx > wwlproxy-短连接 > wwllogic-客户端CGI > wwlauthsvr-登录后台 > wwlxkv-消息存储

客户端发消息

nginx > wwlproxy-短连接 > wwlogic-客户端CGI >wwlchatlogicsvr-聊天后台 > wwlxkv-消息存储

客户端推送消息

上面的wwlchatlogicsvr-聊天后台 > wwlmqasyncsvr > wwlpush > wwlconn或ss5

1.6版本开始 核心存储逐渐xkv替换为pkv

2.0版本开始 引入了独立文件存储wwldfs，逐步代替wwlfilesvr

和客户端有长连接时消息通过长连接推送，苹果/小米/华为推送使用ss5

### 运维规范

#### 目录结构

产品运行目录 /home/wwlocal

服务的目录结构<br>
/home/wwlocal/wwl* 逻辑层模块<br>
/home/wwlocal/bin 启停、监控脚本<br>
/home/wwlocal/sbin 服务的可执行二进制<br>
/home/wwlocal/conf 配置文件<br>
/home/wwlocal/data 数据目录<br>
/home/wwlocal/log 预留未用

#### 配置管理

- Server配置（当前模块的配置信息）<br>
    /home/wwlocal/wwllogic/conf/wwllogic.conf
- Client配置（提供其他模块调用的配置信息）<br>
    /home/wwlocal/conf/global/wwllogic_cli.conf
- 全局配置（当前客户端的）可以配置是否开启红包等功能<br>
    /home/wwlocal/conf/global/localglobal.conf

#### 路由访问

- 路由配置自动化<br>
    ip.lst配置模版自动生成
- 基于本地cli文件，无中心调度<br>
    各自读取 /home/wwlocal/conf/global/xx_cli.conf
- 接入层权重轮询<br>
    Nginx配置
- 逻辑层多机一致性哈希容灾<br>
    核心存储层分组三机容灾

#### 定时任务

- 接入机<br>
    crontab -l<br>
    同步时间<br>
    清理日志<br>
    监控ss5<br>
- 逻辑/存储机
    crontab -l<br>
    同步时间<br>
    本地监控分区表<br>
    红包DB按年建表<br>
    监控服务 Mon_all 监控所有服务Mon_开头，监控目的防止服务失败，记录服务运行情况（日志里的restart）<br>

#### 后台日志

- 接入机<br>
    /home/wwlocal/bin/clean_connlog.sh 每天3点切割nginx和ss5日志并压缩、保留七天

- 逻辑/存储机<br>
    /home/wwlocal/bin/clear_disk.sh<br>
    每15分钟清理压缩指定目录文件，日志保留14天

    /home/wwlocal/bin/clean_connlog.sh<br>
    每天3点切割nginx和ss5日志并压缩、保留七天

    消息存储管理端配置清理规则

    临时文件存储自动清理<br>
    默认单击保留200G或者365天

#### 操作规范

- 单个服务<br>
    启停：/home/wwlocal/服务名/bin/服务名Tool start/stop/restart<br>
    监控：/home/wwlocal/服务名/bin/Mon_服务名(定时任务自动执行该脚本)

- 本机上的所有服务<br>
    启动：/home/wwlocal/wwlops/START.sh<br>
    停止：/home/wwlocal/wwlops/STOP.sh<br>
    检查：/home/wwlocal/wwlops/CHECK.sh (-h 查看参数)

- 运维变更

### 故障处理

#### 日志分析

- 接入机日志
    - nginx日志：/home/wwlocal/wwlnginx/目录下
    - ss5日志：/var/log/ss5/目录下
- 逻辑/存储机日志信息
    - 监控日志 /home/wwlocal/log/service/[date+%Y%m%d].log grep restart
    - 程序日志 /home/wwlocal/log/error/[date+%Y%m%d].log grep error
    - 常用命令（alice别名） teer=tail+error,geer=grep+error,tsvc=tail+svc
    - 常用工具 CHECK.sh wwlops_tools
- 企业帐号信息
    - 获取corpid<br>
    /home/wwlocal/wwlops/wwlops_tools get_corpid<br>
    cat /home/wwlocal/conf/global/localglobal.conf|grep '^corpid'|awk -F = '{print$2} '
    - 获取vid（用户id）<br>
    /home/.wwlocal/wwlops/wwlops_tools get_vid<br>
    /home/wwlocal/wwlops/tools/wwlocal_tools CropAcctid2Vid $CORPID $USERID
    
1. 获取失败用户的vid：<br>
    wwlops_tools getvid 帐号
2. 收集关键日志：<br>
    在所有逻辑存储服务器上执行 terr|grep 失败用户的vid<br>
    只看错误日志 gerr
3. 触发复现失败：<br>
    收集屏幕打印日志，进行分析或者交给企业微信团队协助分析

windows企业微信客户端 ctrl+shift+alt+d 开启debug模式<br>
会话右键roomid=群的id 用户会话的消息右键获取vid=消息发送者、消息ID

#### 队列日志

处理耗时
```
cd /home/wwlocal/log/error
tailf 2019041721.log | grep "wwlchatlogicsvr" | grep "RUN "
RUN 0 34 xxxxxxxxxx CMD 41 ip xx.xx.xx.xx io 18 67999
# 其中0是队列等待耗时ms
# 其中34是WORKER处理耗时ms
# 其中41是请求命令字 command-id
# 其中ip后面是请求上一层来源IP
# 其中18是请求包大小
# 其中67999是请求返回包大小
```

处理队列<br>
CHECK queue<br>
队列名<br>
delay：延时任务数<br>
ready：就绪任务数<br>
reserve：正在处理的任务worker数，小于最大并发数<br>
wait：等待的任务数，因UIN串行化而等待的任务<br>
qsize：队列的堆积数（常见判断客户端延时）<br>

#### 错误码说明

常见错误码
- 201/-202： 端口连接不上或者client读写超时
- 203：ss5代理超时，可能外网网络问题
- 1233/-1234/-1235：服务过载相关
- 2000/-2001：wwlxkv版本相关

#### 客户端收集日志

Android
1. 正常登录获取日志<br>
    我》设置》关于政务微信》发送日志》选择发送方式
2. 不能登录获取日志<br>
    sdcard里取日志<br>
    长按首页政务微信logo可以发日志
3. 客户端Carsh闪退<br>
    sdcard里取日志
4. 开启调试模式<br>
    我》设置》关于政务微信》长按政务微信的图标，开启调试开关<br>
    常用的有【调试信息】【检查网络连通性】等功能工具
    
iOS
1. 正常登录获取日志<br>
    我》设置》关于政务微信》发送日志》选择发送方式
2. 客户端Carsh闪退<br>
    设置》隐私》分析》分析数据 里找到当天的Crash log
3. 开启调试模式<br>
    去到我的页签上面标题“我”狂点五下。

Windows
1. pc端日志获取路径<br>
    %appdata%\Tencent\WXWorkLocal
2. pc端获取日志快捷键<br>
    ctrl+shift+alt+f
3. 正常登录获取日志<br>
    我》设置》关于政务微信》发送日志》选择发送方式
4. 闪退，取dump文件<br>
    任务管理器找到WXWorkLocal.*32<br>
    右键选择“创建转储文件”
5. 开启调试模式<br>
    ctrl+shift+alt+d

#### 常见问题

- 忘记管理员密码<br>
    初始密码在localglobal.conf文件中的admin password，自动生成<br>
    重置admin密码<br>
    wwlops_tools update_adminpwd<br>
    或者 /home/wwlocal/wwlops/tools/ToolsUpdateAdminPwd $CORPID

- Lincense验证报错
    - 错误码-1/-202 网络问题，内网的使用内网验证方式
    - 错误码500 license错误

- 消息接收有明显延迟
    - 消息半分钟一批一批到达客户端（包括客户端无法看到“对方正在输入”），长连接断开，检查8080端口
    - 排查技巧：<br>
        telnet服务器的8080端口的连通性
        客户是否有负载均衡F5的配置，代理模式是不是4层代理
    
- 没有收到消息推送
    - 检查ss5代理到手机厂商的推送服务器是否通
    - CHECK.sh -e 排查网络连通性
    - 是否证书有问题（上线环境）

### 版本升级

#### 方案准备

准备方案，详细到命令级别

#### 环境巡查

- root权限申请
- 磁盘容量检查
- 服务检查
- CPU负载正常（80%以下）
- 版本号检查（每个逻辑服务器服务节点）<br>
    `grep version /home/wwlocal/conf/global/localglobal.conf`

#### 预约时间

- 协商时间
- 业务量较低的时候进行操作
- 协助客户准备安民通告

#### 实施升级

- 准备好升级包 校验md5sum 解压/home/wwpatch
- 备份关键配置文件
    /home/wwlocal/conf/global/ip.lst、localglobal.conf、wwlsocks5proxy_cli.conf
- 每台服务器节点依次执行 install.sh（升级后需要重启服务，命令执行相隔1-2分钟）
- 检查服务<br>
    CHECK.sh（所有服务节点升级后执行）<br>
    grep restart /home/wwlocal/log/service/[date+%Y%m%d].log
    

### 功能检查

- 检查基础功能
- 检查应用功能
- 检查新feature及开关
- 完善功能验证文档（签字）

### 迁移扩容

场景：

- 服务器纵向升级（硬件加配）
    - 判断机器角色
    - 停机器服务，实施硬件变更
    - 恢复服务
    - 检查服务、测试功能
- 服务器更换、机房搬迁
    - 判断机器角色
    - 部署新机
    - rsync原机器数据到新机
    - 停原机，再次rsync
    - 修改IP.lst， CHANGEIP.sh
    - 启动新机的服务START.sh
    - 检查服务、测试功能
- 横向扩容（增加服务器）
    - 增加接入机
        - 配置网络策略
        - 流量切换
        - 如果接入机作为ss5代理，修改逻辑器的client配置上添加ip
    - 增加逻辑机（3倍）
        - 部署新机
        - 调整临时/收藏文件存储权重
        - 修改IP.lst， CHANGEIP.sh
        - 启动新机的服务START.sh
        - 修改接入机配置（增加流量导入的IP）
        - 检查服务、测试功能
    - 增加存储机（3倍）
        - 2.0后才支持存储机横向扩容
        - 基于PKV
        - 部署新机
        - 运行迁移工具、短暂停服再同步
        - 检查数据
        - 启动服务、检查服务、测试功能
        
优先考虑停服操作、或低峰期在线操作

和升级一样5个步骤

方案准备》环境巡查》预约时间》实施迁移》功能检查
