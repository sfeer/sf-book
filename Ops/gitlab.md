# GitLab入门及常见问题


## 关于GitLab角色

[官方权限说明](https://docs.gitlab.com/ce/user/permissions.html)

主要是对保护的版本无法提交push和合并merge
在项目的setting > Repository Settings > Protected Branches
下面对master版本允许Developers角色操作

## 关于CI/CD

### 安装GitLab Runner

RHEL/CentOS/Fedora
```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | sudo bash
sudo yum install gitlab-runner
```

### 注册GitLab Runner

此处是将你的GitLab Runner注册到GitLab page上，让GitLab page可以和你的Runner通信。
GitLab的项目/setting/ci_cd, 展开Runners可查看token
*你可以通过`GitLab page -> Settings -> CI/CD -> Runners`来获得URL和TOKEN*

1. 运行注册命令
```
gitlab-runner register
```

2. 输入GitLab URL
```
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
```

3. 输入GitLab TOKEN
```
Please enter the gitlab-ci token for this runner
```

4. 输入对这个Runner的表述
```
Please enter the gitlab-ci description for this runner
```

5. 输入Runner的tag（逗号分割，和.gitlab-ci.yml的tags配置有关）
```
Please enter the gitlab-ci tags for this runner (comma separated):
```

6. 输入Runner的executor（常用执行器的ssh，shell，docker）
```
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
```

*注册完成后你可以在/etc/gitlab-runner里发现 config.toml文件，该文件是Runner的配置文件*

### 常见命令

```
systemctl restart gitlab-runner
systemctl status gitlab-runner

gitlab-runner list
cat /etc/gitlab-runner/config.toml
```

### 环境变量

`GitLab page -> Settings -> CI/CD -> Variables`
可设置`JAVA_HOME`及密码、密钥

### 发布SpringBoot应用

#### 安装mvn

[官方文档](http://maven.apache.org/install.html)

#### 安装docker-compose

[详情](docker.md#安装docker-compose)


## 待整理

github action 发布vue 到github pages
