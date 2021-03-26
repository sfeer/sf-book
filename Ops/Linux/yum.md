# YUM

## 基本信息

更新`yum update`

清除`yum clean all`

列出`yum list installed` `yum list|grep zabbix`

生成缓存`yum makecache fast`

repo配置地址`/etc/yum.repos.d`


## 阿里镜像源

[CentOS 镜像](https://developer.aliyun.com/mirror/centos)

CentOS，是基于 Red Hat Linux 提供的可自由使用源代码的企业级 Linux 发行版本；是一个稳定，可预测，可管理和可复制的免费企业级计算平台。

[Epel 镜像](https://developer.aliyun.com/mirror/centos)

EPEL (Extra Packages for Enterprise Linux), 是由 Fedora Special Interest Group 维护的 Enterprise Linux（RHEL、CentOS）中经常用到的包。

[Docker CE 镜像](https://developer.aliyun.com/mirror/docker-ce)

Docker CE 是免费的 Docker 产品的新名称，Docker CE 包含了完整的 Docker 平台，非常适合开发人员和运维团队构建容器 APP。

## 本地源

### 从阿里云镜像站点下载iso镜像
[CentOS 镜像](http://mirrors.aliyun.com/centos/)

1. 下载CentOS的iso

http://mirrors.aliyun.com/centos/7.8.2003/isos/x86_64/CentOS-7-x86_64-DVD-2003.iso

2. 挂载iso

上传iso文件到/tmp目录
`mount -o loop -t iso9660 /tmp/CentOS-7-x86_64-DVD-2003.iso /mnt/vcdrom`

3. 建立本地源

配置local.repo
```
[local]
name=local
baseurl=file:///mnt/vcdrom
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### 创建自定义repo

1. 获取rpm文件
    - 方式一：
        从aliyun的镜像站找到链接下载并上传到服务器
    - 方式二：
        在能联网的centos上使用yum命令`yum install --downloadonly --downloaddir=/tmp/zabbix zabbix-get zabbix-java-gateway`下载

2. 安装createrepo`yum install createrepo`

3. 构建repo`createrepo /usr/myrepo`

4. 在`/etc/yum.repos.d`目录下新建repo文件

`vim /etc/yum.repos.d/myrepo.repo`
```
[myrepo]
name=myrepo
baseurl=file:///usr/myrepo
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```




