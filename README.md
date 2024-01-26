- [1. 安装 Docker](#1-安装-docker)
  - [1.1. ubuntu 安装 docker](#11-ubuntu-安装-docker)
- [2. 镜像](#2-镜像)
  - [2.1. 拉取镜像](#21-拉取镜像)
  - [2.2. 创建自己的镜像](#22-创建自己的镜像)
    - [2.2.1. 用容器创建镜像](#221-用容器创建镜像)
    - [2.2.2. 利用 Dockerfile 来创建镜像](#222-利用-dockerfile-来创建镜像)
  - [2.3. 存出和载入镜像](#23-存出和载入镜像)
    - [2.3.1. 存出镜像](#231-存出镜像)
    - [2.3.2. 载入镜像](#232-载入镜像)
    - [2.3.3. 移除镜像](#233-移除镜像)
- [3. 容器](#3-容器)
  - [3.1. 启动容器](#31-启动容器)
    - [3.1.1. 新建并启用容器](#311-新建并启用容器)
    - [3.1.2. 启用已经终止的容器](#312-启用已经终止的容器)
    - [3.1.3. 后台运行的方式启用容器](#313-后台运行的方式启用容器)
  - [3.2. 进入容器](#32-进入容器)
    - [3.2.1. attach 命令](#321-attach-命令)
    - [3.2.2. nsenter 命令](#322-nsenter-命令)
  - [3.3. 导入和导出容器](#33-导入和导出容器)
    - [3.3.1. 导出容器](#331-导出容器)
    - [3.3.2. 导入容器](#332-导入容器)
  - [3.4. 终止容器](#34-终止容器)
  - [3.5. 删除容器](#35-删除容器)
- [4. 仓库](#4-仓库)
- [5. 数据管理](#5-数据管理)
  - [5.1. 数据卷](#51-数据卷)
  - [5.2. 创建数据卷](#52-创建数据卷)
- [6. 使用网络](#6-使用网络)
- [7. 項目打包爲docker](#7-項目打包爲docker)
  - [7.1. Dockerfile生成鏡像](#71-dockerfile生成鏡像)
  - [7.2. 鏡像轉存爲文件](#72-鏡像轉存爲文件)
  - [7.3. 通過文件載入鏡像](#73-通過文件載入鏡像)
  - [7.4. 單獨運行docker](#74-單獨運行docker)
  - [7.5. docker-compose.yml](#75-docker-composeyml)



## 1. 安装 Docker

### 1.1. ubuntu 安装 docker

```shell
sudo apt-get update
sudo apt-get install -y docker.io
```

## 2. 镜像

### 2.1. 拉取镜像

```shell
#拉取镜像
sudo docker pull ubuntu:20.04
##用镜像创建容器
sudo docker run -t -i ubuntu:20.04 /bin/bash
#显示本地已有的镜像
sudo docker images
```

### 2.2. 创建自己的镜像

#### 2.2.1. 用容器创建镜像

```shell
sudo docker run -i -t ubuntu:20.04 /bin/bash
#改变内容
touch test.txt
#退出容器
exit
#修改镜像内容
sudo docker commit -m "Added json gem" -a "Docker Newbee" 8750c18cb080 ubuntu:v2
#查看创建的镜像
sudo docker images
```

其中， -m 来指定提交的说明信息，跟我们使用的版本控制工具一样； -a 可以指定更新的用户信息；之后是用来创建镜像的容器的 ID；最后指定目标镜像的仓库名和 tag 信息。创建成功后会返回这个镜像的 ID 信息。

#### 2.2.2. 利用 Dockerfile 来创建镜像

Dockerfile 基本的语法是使用 # 来注释,FROM 指令告诉 Docker 使用哪个镜像作为基础,接着是维护者的信息,RUN 开头的指令会在创建中运行，比如安装一个软件包，在这里使用 apt-get 来安装了一些软件

```
# This is a comment
FROM ubuntu:14.04
MAINTAINER Docker Newbee <newbee@docker.com>
RUN touch /zjtext.txt

```

编写完成 Dockerfile 后可以使用 docker build 来生成镜像。

```
sudo docker build -t="ubuntu:v3" .
```

其中 -t 标记来添加 tag，指定新的镜像的用户信息。 “.” 是 Dockerfile 所在的路径（当前目录），也可以替
换为一个具体的 Dockerfile 的路径。

查看创建的镜像

```
sudo docker images
```

### 2.3. 存出和载入镜像

#### 2.3.1. 存出镜像

如果要导出镜像到本地文件，可以使用 docker save 命令。

```
sudo docker save -o ubuntu_14.04.tar ubuntu:14.04
```

#### 2.3.2. 载入镜像

可以使用 docker load 从导出的本地文件中再导入到本地镜像库，例如

```shell
sudo docker load --input ubuntu_14.04.tar
#或者
sudo docker load < ubuntu_14.04.tar
```

#### 2.3.3. 移除镜像

如果要移除本地的镜像，可以使用 docker rmi 命令。注意 docker rm 命令是移除容器。

```
sudo docker rmi ubuntu:v3
```

## 3. 容器

简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。
本章将具体介绍如何来管理一个容器，包括创建、启动和停止等。

### 3.1. 启动容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（stopped）的容器重新启动。

#### 3.1.1. 新建并启用容器

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：
• 检查本地是否存在指定的镜像，不存在就从公有仓库下载
• 利用镜像创建并启动一个容器
• 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
• 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
• 从地址池配置一个 ip 地址给容器
• 执行用户指定的应用程序
• 执行完毕后容器被终止

```shell
sudo docker run -t -i ubuntu:v2 /bin/bash
```

其中， -t 选项让 Docker 分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输
入保持打开。

#### 3.1.2. 启用已经终止的容器

可以利用 docker start 命令，直接将一个已经终止的容器启动运行。

```
sudo docker start 9e65ca71da2a
```

#### 3.1.3. 后台运行的方式启用容器

还可以在启用容器的时候加-d 让其后台运行。

```
sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

查看后台运行的容器

```
sudo docker ps
```

要获取容器的输出信息，可以通过 docker logs 命令。

```
sudo docker logs insane_babbage
```

### 3.2. 进入容器

某些时候需要进入容器进行操作，有很多种方法，包括使用 docker attach 命令或 nsenter 工具等。

#### 3.2.1. attach 命令

```
sudo docker attach agitated_jackson
```

最后面是容器名，当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。

#### 3.2.2. nsenter 命令

pass

### 3.3. 导入和导出容器

#### 3.3.1. 导出容器

如果要导出本地某个容器，可以使用 docker export 命令。

```
sudo docker export 7691a814370e > ubuntu.tar
```

#### 3.3.2. 导入容器

可以使用 docker import 从容器快照文件中再导入为镜像，例如

```
cat ubuntu.tar | sudo docker import - test/ubuntu:v1.0
```

### 3.4. 终止容器

可以使用 docker stop 来终止一个运行中的容器。此外，当 Docker 容器中指定的应用终结时，容器也自动终止。

```
sudo docker stop 9e65ca71da2a
```

### 3.5. 删除容器

可以使用 docker rm 来删除一个处于终止状态的容器。

```
sudo docker rm trusting_newton
```

如果要删除一个运行中的容器，可以添加 -f 参数。Docker 会发送 SIGKILL 信号给容器。

## 4. 仓库

pass

## 5. 数据管理

### 5.1. 数据卷

数据卷是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：
• 数据卷可以在容器之间共享和重用
• 对数据卷的修改会立马生效
• 对数据卷的更新，不会影响镜像
• 卷会一直存在，直到没有容器使用数据卷，类似于 Linux 下对目录或文件进行 mount。

### 5.2. 创建数据卷

在用 docker run 命令的时候，使用 -v 标记来创建一个数据卷并挂载到容器里。在一次 run 中多次使用可以挂载多个数据卷。

指定挂载一个本地主机的目录到容器

```
sudo docker run -it -P -v /src/webapp:/opt/webapp ubuntu:v2
```

## 6. 使用网络

容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 -P 或 -p 参数来指定端口映射。

当使用 -P 标记时，Docker 会随机映射一个 49000~49900 的端口到内部容器开放的网络端口。

```
sudo docker run -d -p 5000:5000 -p 3000:80 training/webapp
```

## 7. 項目打包爲docker
### 7.1. Dockerfile生成鏡像
构建镜像：在包含 Dockerfile 的目录中运行以下命令来构建镜像。yourapp 是你给镜像起的名字，. 指的是当前目录（包含 Dockerfile）。
```
docker build -t yourapp .
docker images
```
### 7.2. 鏡像轉存爲文件
```
sudo docker export 7691a814370e > app.tar
```
### 7.3. 通過文件載入鏡像
```
sudo docker load --input ubuntu_14.04.tar
#或者
sudo docker load < ubuntu_14.04.tar
```
### 7.4. 單獨運行docker
```
sudo docker run -t -i ubuntu:v2 /bin/bash
```
### 7.5. docker-compose.yml
通過docker-compose.yml
```
version: "3"
services:
  nginx:
    image: dji/nginx:xinjiang-yining-kuang-001
    restart: "always"
    ports:
      - "3:8080"
    volumes:
      - /etc/localtime:/etc/localtime
  cloud_api_sample:
    image: dji/cloud_api_sample:xinjiang-yining-kuang-001
    ports:
      - "2:6789"
    volumes:
      - /etc/localtime:/etc/localtime
    hostname: cloud_api_sample
    restart: "always"
```
要使用这个 docker-compose.yml 文件，你需要安装Docker Compose工具，然后在包含这个文件的目录中运行以下命令：
```
docker-compose up
```
这个命令会启动定义在 docker-compose.yml 文件中的所有服务。如果你想在后台运行这些服务，可以添加 -d 标志：
```
docker-compose up -d
```
要停止并移除由 docker-compose.yml 文件定义的所有容器，可以使用：
```
docker-compose down
```