docker安装脚本

```sh
# 卸载旧版本
yum remove docker \
             docker-client \
             docker-client-latest \
             docker-common \
             docker-latest \
             docker-latest-logrotate \
             docker-logrotate \
             docker-engine

# yum-util提供yum-config-manager功能
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

# 设置yum源
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# 安装最新版docker
yum install -y docker-ce
```

