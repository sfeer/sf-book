# Docker

## todo

docker、docker-compose、docker swarm和k8s的区别

https://www.jianshu.com/p/2a9ae69c337d

## 介绍

### Docker
Docker 这个东西所扮演的角色，容易理解，它是一个容器引擎，也就是说实际上我们的容器最终是由Docker创建，运行在Docker中，其他相关的容器技术都是以Docker为基础，它是我们使用其他容器技术的核心。

### Docker-Compose
Docker-Compose 是用来管理你的容器的，有点像一个容器的管家，想象一下当你的Docker中有成百上千的容器需要启动，如果一个一个的启动那得多费时间。有了Docker-Compose你只需要编写一个文件，在这个文件里面声明好要启动的容器，配置一些参数，执行一下这个文件，Docker就会按照你声明的配置去把所有的容器启动起来，只需docker-compose up即可启动所有的容器，但是Docker-Compose只能管理当前主机上的Docker，也就是说不能去启动其他主机上的Docker容器

### Docker-Swarm
Docker-Swarm 是一款用来管理多主机上的Docker容器的工具，可以负责帮你启动容器，监控容器状态，如果容器的状态不正常它会帮你重新帮你启动一个新的容器，来提供服务，同时也提供服务之间的负载均衡，而这些东西Docker-Compose 是做不到的

### Kubernetes
Kubernetes它本身的角色定位是和Docker Swarm 是一样的，也就是说他们负责的工作在容器领域来说是相同的部分，都是一个跨主机的容器管理平台，当然也有自己一些不一样的特点，k8s是谷歌公司根据自身的多年的运维经验研发的一款容器管理平台。而Docker Swarm则是由Docker 公司研发的。

既然这两个东西是一样的，那就面临选择的问题，应该学习哪一个技术呢?实际上这两年Kubernetes已经成为了很多大公司的默认使用的容器管理技术，而Docker Swarm已经在这场与Kubernetes竞争中已经逐渐失势，如今容器管理领域已经开始已经逐渐被Kubernetes一统天下了。所以建议大家学习的时候，应该多考虑一下这门技术在行业里面是不是有很多人在使用。

需要注意的是，虽然Docker Swarm在与Kubernetes的竞争中败下阵来，但是这个跟Docker这个容器引擎没有太大关系，它还是整个容器领域技术的基石，Kubernetes离开他什么也不是。

### Docker-Hub
Docker镜像仓库

## 安装
较旧的 Docker 版本称为 docker 或 docker-engine 。如果已安装这些程序，请卸载它们以及相关的依赖项。
```
sudo yum remove docker \
	  docker-client \
	  docker-client-latest \
	  docker-common \
	  docker-latest \
	  docker-latest-logrotate \
	  docker-logrotate \
	  docker-engine
```

Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩写为 EE）。
企业版包含了一些收费服务，个人开发者一般用不到。下面的介绍都针对社区版。

1. 安装依赖包
```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

2. 安装稳定版Yum源
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

3. 安装Docker社区版
```
yum install docker-ce docker-ce-cli containerd.io
```

4. 开启Docker
```
systemctl start docker
systemctl enable docker
```

5. 安装完成后验证
```
docker version
```

## Docker配置镜像加速器

针对Docker客户端版本大于 1.10.0 的用户
您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://apkqhacm.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Docker配置远程访问

docker.service 中 dockerd 的 -H 参数不能与 daemon.json 中的 hosts 键值对冲突。
```
# 配置hosts  "hosts": ["tcp://0.0.0.0:2375", "fd://"]
vi /etc/docker/daemon.json
# 删除 -H fg://参数
vi /lib/systemd/system/docker.service
```

## Docker常用命令

```
docker search xxxx

# 列出本机的所有 image 文件。
docker image ls

# 删除 image 文件
docker image rm [imageName]

# 列出本机正在运行的容器
docker container ls

# 列出本机所有容器，包括终止运行的容器
docker container ls --all

# 删除容器（慎重）
docker container rm [containerID]

# 终止容器
docker container kill [containerID]

# 列出所有网络
docker network ls

# 执行命令
docker exec -it [containerName] /bin/sh
docker exec -it [containerName] pwd
```

## 更新容器

docker pull xxxx:latest

docker stop xxxx && docker rm xxxx

docker run xxxx... 重新安装

## 网络相关

容器间网络是由docker分配的, docker本身ip 172.17.0.1

方式一：内部网络
查看容器ip `docker inspect 容器ID/容器名称`
```
docker exec -it [containerName] ifconfig
```

方式二：使用link
在docker run启动时添加link

方式三：自定义bridge网络

## CI/CD

.gitlab-ci.yml示例
```yaml
before_script:
  - npm -v
  - docker -v

build:
  stage: build
  script:
    - npm install --registry=https://registry.npm.taobao.org
    - npm run build
    - docker build -f docker/gzzy/Dockerfile --force-rm -t pj-gzzy-web .
    - docker ps -a|grep pj-gzzy-web &> /dev/null && docker rm -f pj-gzzy-web
    - "docker run -d -p 8081:80 \
      --network aiop_default \
      --link pj-gzzy \
      --restart always \
      --name pj-gzzy-web \
      pj-gzzy-web:latest"
    - npm run build-sh-h5
    - docker build -f docker/shrz/Dockerfile --force-rm -t pj-shrz-web .
    - docker ps -a|grep pj-shrz-web &> /dev/null && docker rm -f pj-shrz-web
    - "docker run -d -p 8082:80 \
      --network aiop_default \
      --link pj-shrz \
      --restart always \
      --name pj-shrz-web \
      pj-shrz-web:latest"
  tags:
    - aiop
```

## 安装docker-compose

```
curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

idea配置docker-compose需要下载exe
[官方GitHub下载地址](https://github.com/docker/compose/releases)


## 示例

```
variables:
  DOCKER_HOST: tcp://192.168.0.215:2375/ # 设置docker地址，可以在gitlab ci上统一配置

before_script:
  - npm -v # 检查npm
  - docker -v # 检查docker

build:
  stage: build
  script:
    - npm install --registry=https://registry.npm.taobao.org
    - npm run build
    # -f 指定要使用的Dockerfile路径
    # --tag, -t 镜像的名字及标签
    - docker build -f docker/Dockerfile -t aiop-web .
    # -f 通过 SIGKILL 信号强制删除一个运行中的容器
    - docker rm -f aiop-web
    # -d 后台运行容器，并返回容器ID
    # -p 指定端口映射，格式为：主机(宿主)端口:容器端口
    # --network 加入docker-compose默认网络
    # --link 添加链接到另一个容器
    # --restart 自动重启
    # --name 名称
    - docker run -d -p 8080:80 --network aiop_default --link aiop-admin --restart always --name aiop-web aiop-web:latest
  tags:
    # gitlab ci runner会执行制定tag的任务
    - aiop
```

## TODO

结合开发工程整理
