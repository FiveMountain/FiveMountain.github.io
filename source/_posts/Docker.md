---
title: Docker
date: 2021-09-10 13:36:49
categories:
- Learning Notes
---

> Docker文档：[https://docs.docker.com](https://docs.docker.com)

# 安装&卸载

## 环境

```shell
# 内核版本
$ uname -r
3.10.0-957.21.3.el7.x86_64

# 系统信息
$ cat /etc/os-release 
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

## 安装

```shell
# 卸载旧版本
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 需要的安装包
$ sudo yum install -y yum-utils

# 使用阿里云镜像
$ sudo yum-config-manager \
    --add-repo \
	http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新yum软件包索引
$ yum makecache fast

# 安装Docker Engine（最新版本）
$ sudo yum install docker-ce docker-ce-cli containerd.io

# 安装Docker Engine（指定版本）
$ yum list docker-ce --showduplicates | sort -r

$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io

# 启动Docker服务
$ systemctl start docker

# 设置自启动
$ systemctl enable docker

# 查看版本
$ docker version

# hello-world测试
$ docker run hello-world
```

> 防火墙相关

```shell
# 开启
systemctl start firewalld
# 重启
systemctl restart firewalld
# 关闭
systemctl stop firewalld
# 查看防火墙规则
firewall-cmd --list-all
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放端口
firewall-cmd --zone=public --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --zone=public --permanent --remove-port=80/tcp
# 防火墙配置重新加载(修改配置后需执行)
firewall-cmd --reload
```

## 卸载

```shell
# 卸载
$ sudo yum remove docker-ce docker-ce-cli containerd.io

# 删除images、containers、volumes
$ sudo rm -rf /var/lib/docker
$ sudo rm -rf /var/lib/containerd
```

# 常用命令

> [https://docs.docker.com/reference/](https://docs.docker.com/reference/)

![DockerCommand](DockerCommand.png)

> 帮助命令

| Command |             Information             |              Annotation              |
| :-----: | :---------------------------------: | :----------------------------------: |
|  info   |   Display system-wide information   | docker系统信息，包括镜像和容器的数量 |
| version | Show the Docker version information |            docker版本信息            |

> 镜像命令

| Command |                  Information                  |    Annotation    |
| :-----: | :-------------------------------------------: | :--------------: |
| images  |                  List images                  | 查看本地所有镜像 |
|  pull   | Pull an image or a repository from a registry |     下载镜像     |
|   rmi   |           Remove one or more images           |     移除镜像     |
| search  |       Search the Docker Hub for images        |     搜索镜像     |

> 容器命令

| Command |             Information              |      Annotation      |
| :-----: | :----------------------------------: | :------------------: |
|  kill   | Kill one or more running containers  |     强制停止容器     |
|   ps    |           List containers            |   列出运行中的容器   |
| restart |    Restart one or more containers    |       重启容器       |
|   rm    |    Remove one or more containers     |       移除容器       |
|   run   |   Run a command in a new container   |    新建容器并启动    |
|  start  | Start one or more stopped containers |   启动已停止的容器   |
|  stop   | Stop one or more running containers  | 停止当前正在运行容器 |

> 其它命令

| Command |                         Information                          |            Annotation            |
| :-----: | :----------------------------------------------------------: | :------------------------------: |
| attach  | Attach local standard input, output,<br/>and error streams to a running container |      进入容器正在执行的终端      |
|  build  |               Build an image from a Dockerfile               |      由Dockerfile创建image       |
| commit  |        Create a new image from a container's changes         | 提交镜像，保存容器当前状态，快照 |
|   cp    |                      Copy files/folders                      |       容器与主机间文件拷贝       |
|  exec   |             Run a command in a running container             |    进入容器，打开一个新的终端    |
| inspect |        Return low-level information on Docker objects        |      查看容器/镜像的元数据       |
|  logs   |                Fetch the logs of a container                 |           查看容器日志           |
| network |                       Manage networks                        |             网络相关             |
|   top   |         Display the running processes of a container         |       查询容器内部进程信息       |
| volume  |                        Manage volumes                        |            数据卷操作            |

> Dockerfile

|  Command   |                            Format                            |                          Annotaion                           |
| :--------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|    FROM    |                 FROM image / FROM image:tag                  |                 基础镜像，一切从这里开始构建                 |
| MAINTAINER |               MAINTAINER user_name user_email                |                        指定维护者信息                        |
|    RUN     |    RUN command<br/>RUN ["EXECUTABLE", "param1", "param2"]    |                     构建时需要运行的命令                     |
|   EXPOSE   |                EXPOSE port [port2,port3,...]                 |                         指定对外端口                         |
|    ADD     |                         ADD src dest                         |                      复制，tar自动解压                       |
|    COPY    |                        COPY src dest                         |                  复制，dest不存在时自动创建                  |
|    ENV     |                        EVN key value                         |                      构建时设置环境变量                      |
|   VOLUME   |                       VOLUME ["/data"]                       |                     设置卷，挂在主机目录                     |
|    USER    |                        USER username                         |   指定容器运行时的用户名或UID，后续的RUN也会使用指定的用户   |
|  WORKDIR   |                        WORKDIR /path                         |                         设置工作目录                         |
|    CMD     | CMD ["executable","param1","param2"]<br/>CMD command param1 param2<br/>CMD ["param1","param2"] | 每个Dockerfile只能有一个CMD命令，多个CMD命令只执行最后一个。<br/>若容器启动时指定了运行的命令，则会覆盖掉CMD中指定的命令。 |
| ENTRYPOINT | ENTRYPOINT ["executable","param1","param2"]<br/>ENTRYPOINT command param1,param2 | 每个Dockerfile中只能有一个ENTRYPOINT，当有多个时最后一个生效。<br/>这些命令不能被docker run提供的参数覆盖。 |
|  ONBUILD   |                    ONBUILD [INSTRUCTION]                     |  指定当所创建的镜像作为其他新建镜像的基础镜像时所执行的指令  |

