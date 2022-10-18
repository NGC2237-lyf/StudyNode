# Docker概述

docker思想来源于**集装箱**

正常：

java -- apk -- 发布（应用商店） -- 第三方使用apk -- 安装即可用

java -- jar（环境） -- 打包带上环境（**镜像**） -- （Docker 仓库：商店） -- 下载发布的镜像 -- 直接运行即可

------

**Docker 核心思想**：**隔离**（可以将服务器用到极值）

Docker 是一种虚拟化技术，小巧（几个M，秒级启动）

------

# Docker优点

容器**没有自己的内核**，没有虚拟化我们的硬件，所以轻便

容器之间**相互隔离**，每个容器都有自己的文件系统，互不影响

**应用更快速的交付和部署**

- 传统：一堆帮助文档，安装程序
- Docker：打包镜像发布测试，一键运行

**更便捷的升级和扩缩容**

**更简单的系统运维**（在容器化之后，测试环境都是高度一致的）
**更高效的计算资源利用**

------

# 理解相关概念

## 镜像（image）

docker镜像就相当于一个模板，可用通过模板来创建容器服务，tomcat镜像==》 run ==》tomcat01容器（提供服务器）

## 容器（container）

Docker 利用容器技术，独立运行一个或者一组应用，通过镜像来创建的。

## 仓库（repository）

仓库就是存放镜像的地方

仓库分为**公有仓库**和**私有仓库**

------

# 安装Docker

`查看系统版本`

```BASH
[root@lvyanfang ~]# cat /etc/os-release
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

```BASH
# 卸载旧版本
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# 安装工具包
yum install -y yum-utils
# 安装镜像
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo  # 默认是国外的
    
yum-config-manager \
    --add-repo \
	http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 阿里云
# 更新 yum 软件包索引	
yum makecache fast
# 安装 Docker （引擎）  docker-ce：社区版   ee：企业版
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
# 启动 Docker
systemctl start docker
# 查看是否启动成功
docker version
# 启动 hello-world
docker run hello-world
# 查看镜像
docker images
```

```BASH
# 卸载依赖
yum remove docker-ce docker-ce-cli containerd.io docker-compose-plugin
# 删除资源
rm -rf /var/lib/docker
# /var/lib/docker  默认的工作路径
```

```BASH
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
	"registry-mirrors":["https://qiyb9988.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload
sudo systemctl restart docker
```

![image-20221010213705264](D:\笔记\照片\image-20221010213705264.png)

# 底层原理

- Docker 是如何工作的？

Docker 是一个 **Client-Server 结构**的系统，Docker 的守护进程运行在主机上。通过 Socket 从客户端访问！

DockerServer 接收到 Docker-Client 的指令，就会执行

![image-20221010214330746](D:\笔记\照片\image-20221010214330746.png)

- Docker 为什么比 VM 快？

新建容器（重新开一个服务）的时候，docker不需要像虚拟机一样重新加载一个操作系统内核（Docker有着比虚拟机更少的抽象层）

docker利用的是宿主机的操作系统

# Docker常用命令

## 帮助命令

```BASH
docker version
docker info
docker 命令 --help
```

帮助文档地址：https://docs.docker.com/engine/reference/run/

------

## 镜像命令

docker images：查看所有本地上的镜像

- 语法：docker images [OPTIONS] [REPOSITORY[:TAG]]

```BASH
[root@lvyanfang ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   12 months ago   13.3kB

REPOSITORY ：镜像的名称
TAG：        镜像的标签
IMAGE ID：   镜像的 id
CREATED：    镜像的创建时间
SIZE：       镜像的大小

可选项:
  -a, --all             显示所有信息
  -q, --quiet           只显示 id

```



------

## 容器命令

