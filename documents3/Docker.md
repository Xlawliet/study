# Docker

网址：https://www.runoob.com/docker/centos-docker-install.html

## 安装

安装命令：

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror aliyun
```

国内daocloud一件安装：

```shell
curl -sSL https://get.daocloud.io/docker | sh
```



## 手动安装

较旧的Docker版本称为docker或docker-engine，安装新版本时需要先卸载相关的依赖项。

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```



### 设置docker仓库

安装engine-community之前需要设置docker仓库，方便从仓库对docker进行安装和更新。

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

比较稳定的仓库：

官方源：

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

阿里云：

```shell
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

清华大学源：

```shell
sudo yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```



### 安装Docker Engine-Community

```shell
sudo yum install docker-ce docker-ce-cli containerd.io
```

如果提示您接受 GPG 密钥，请选是。

若要安装特定版本，从存储库中列出可用版本，然后选择安装：

```shell
$ yum list docker-ce --showduplicates | sort -r
```

通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1。

```shell
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```





### 启动docker

```shell
######启动命令
sudo systemctl start docker

######通过运行hello-world验证是否正确安装
sudo docker run hello-world

```



### 卸载docker

```shell
#####删除安装包
yum remove docker-ce

######删除镜像、配置文件等内容
rm -rf /var/lib/docker
```





## 使用

### Docker客户端

通过docker命令来查看Docker客户端的所有命令选项。

`docker`

可以通过`docker command --help`了解使用方法

例如查看`docker stats --help`的使用方法。

### 

### 容器使用

#### 获取镜像

例如如果本地没有镜像，可以通过`docker pull`来载入镜像：

```shell
docker pull ubuntu
```

#### 启动容器

```
docker run -iu ubuntu /bin/bash
```

参数说明：

- **-i**: 交互式操作。
- **-t**: 终端。
- **ubuntu**: ubuntu 镜像。
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

要推出终端，直接输入`exit`



#### 启动已停止运行的容器

```shell
docker ps -a
```

通过上述命令获取容器id，用下面的命令来启动停止的容器

```
docker start -id
```



#### 后台运行

```shell
docker run -itd --name ubuntu-test ubuntu /bin/bash
```

通过参数-d指定容器的运行模式，该参数使得容器才后台运行，不会进入。

通过ubuntu镜像创建一个名为ubuntu-test的容器。



#### 停止一个容器

```shell
docker stop <容器id>
```

#### 重启一个容器

```shell
docker restart <容器id>
```



#### 进入容器

在使用 `-d` 参数时，容器启动后会进入后台。此时要进入容器可以通过以下命令：

- docker attach
- docker exec （推荐，因为推出容器终端不会使得容器停止）



### 导出和导入容器

#### 导出容器

```shell
docker export <容器id> ubuntu.tar
```

导出容器快照到本地文件ubuntu.tar



#### 导入容器快照

```
cat docker/ubuntu.tar |docker import - test/ubuntu:v1
```

将快照文件ubuntu.tar 导入到镜像test/ubuntu:v1

也可以通过指定URL的活着某个目录来导入：

```shell
docker import http://example.com/exampleimage.tgz example/imagerepo
```



## 删除容器

```shell
docker rm -r <容器id>
```





## 运行

运行一个web应用

```shell
docker pull training/webapp #载入镜像
docker run -d -P training/webapp python app.py 
```

- -d：容器在后台运行
- -P：将容器内部使用的网络端口随机映射到主机上



通过-p参数来设置不一样的端口：

```shell
docker run -d -p 5000:5000 training/webapp python app.py
```

使得app.py的运行端口通过主机5000映射到docker的5000上。



#### 网络端口的快捷方式

`docker port <容器id或容器name>`

#### 查看web应用程序的程序日志

`docker logs -f <容器id/容器name>`

#### 查看web应用程序容器内的进程

`docker top <容器id/容器name>`

#### 检查web应用程序

`docker inspect <容器id/容器name>`

#### 移除web应用容器

前提是容器必须是停止状态。

`docker rm <容器id/容器name>`



## 镜像

#### 列出镜像列表

`docker images`



## 容器连接

#### 新建docker网络

`docker network create -d bridge test-net`

#### 查看docker网络

`docker network ls`

#### 连接容器

```shell
#运行一个容器并连接到新建的网络
docker run -itd --name test1 --network test-net ubuntu /bin/bash
#运行另一个容器并加入这个网络
docker run -itd --name test2 --network test-net ubuntu /bin/bash
```



# 问题

### Unable to find image 'hello-world:latest' locally

本地没有找到hello-world镜像，也没有从仓库中拉取到镜像

#### 原因

docker服务器在国外，导致由于墙的问题无法正常拉取镜像。

#### 解决方案

创建文件daemon.json

```shell
touch /etc/docker/.daemon.json
```

配置阿里云镜像

```shell
{ 
"registry-mirrors": ["https://alzgoonw.mirror.aliyuncs.com"] 
}
```

重启服务

```shell
systemctl restart docker
sudo systemctl status docker
```



