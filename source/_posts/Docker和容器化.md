---
title: Docker和容器化
tags: ["docker"]
overdue: false
categories: docker
date: 2022-01-29 15:58:48
updated: 2022-01-29 18:23:11
---

## 一、虚拟机和容器

### 1.1 虚拟化和虚拟机

虚拟化是一种通过软件层面创建虚拟资源（计算、存储、网络等）的技术。

虚拟机是通过软件创建出来一些虚拟的资源，从而在这些虚拟的资源上安装的操作系统。

VMware就是管理和创建虚拟机的软件，在VMware中，我们可以安装多种不同的操作系统，这些运行在VMware中的操作系统就称之为虚拟机。

虚拟化的好处：
1. 可以充分利用物理资源，在一台电脑上可以同时运行多个操作系统
2. 虚拟机之间彼此独立，互不干扰
3. 灵活管理资源，可以动态增加或减少资源
4. 便于迁移。比如hadoop_env这个镜像中包含了已经安装好hadoop等程序的centos7操作系统。

### 1.2 docker和容器

是一个开源项目 ，2013诞生，最初是 dotCloud 公司内部的一个业余项目。它基于 Google 公司推出的 Go 语言实现。 项目后来加入了 Linux 基金会，遵从了 Apache 2.0 协议，项目代码在 GitHub 上进行维护。 **Docker** 是一种基于容器的虚拟化技术，它通过轻量级的容器实现应用的打包、分发和运行。与传统的虚拟机相比，Docker 更加高效和灵活。Docker 自开源后受到广泛的关注和讨论，以至于 dotCloud 公司后来都改名为 Docker Inc。
利用虚拟化的技术，通过容器的方式，实现应用的打包、分发和运行。
这种容器包含了应用程序本身以及应用程序运行时所需要的一切环境

![image-20241128213506166](/image/image-20241128213506166.png)

容器和虚拟化的对比：

- 容器更加轻量化，Docker 容器共享宿主操作系统的内核，不需要为每个容器启动一个完整的操作系统实例，因此启动速度更快、资源占用更少。

- 一致性环境：Run anywhere，容器包含了应用程序本身和应用程序运行时所依赖的运行环境和配置信息等，因此不管容器在什么操作系统上执行，结果基本都是一致。（例外情况：容器依赖宿主机的网络、文件系统等）

- 便携性：容器可以打包成镜像，便于移植和部署。



### 1.3 docker的基本概念

- 容器（container）—— 类比面向对象中对象/实例
    - 容器是一个轻量级的、独立且可以执行的运行环境，用于运行应用程序。它包含代码、运行时、配置信息等，确保可以在不同环境中以相同的方式运行。
    - 容器化： 其实就是把程序放到容器中运行的一个过程。

- 镜像（image）  —— 类比面向对象中类
    - 镜像是一个只读模板，用于创建容器。它包含了应用程序的代码以及运行时所需要的所有依赖项。镜像可以理解为类似与虚拟机的快照，但是体积更小。

镜像和容器的关系，就像是面向对象编程中  类和对象 一样。镜像是静态的定义，容器是镜像在运行时的一个载体。容器可以被创建、启动、停止、删除。

镜像相当于类

容器相当于对象（实例）

- 容器引擎（docker Engine）
    - docker engine 是docker的一个核心组件，用于构建、运行和管理容器。包含两部分：docker daemon、docker cli（command line interface）

- dockerfile

    - 一组指令文件，用于定义如何构建docker的镜像。

    - ```bash
    1. 从 jdk 环境镜像开始
    2. 复制程序到某个工作目录下
    3. 执行启动命令
    ```

    - 有了dockerfile之后，可以通过docker build构建出来镜像文件

- repository

    - 类似于maven仓库，别人可以把自己构建好的镜像分享出来，那么其他人就可以获取这个镜像来执行程序。
    - 本地仓库：启动一个镜像时，优先从本地寻找，找不到的话再从远程仓库找。
    - 远程仓库：本地没有镜像时，可以通过docker pull命令拉取镜像。

### 1.4 docker的工作原理

- 内部原理
    - docker 通过linux的namespace和cgroups技术实现了容器的隔离和资源管理
        - namespace： 提供独立的运行环境（文件系统、进程树、网络通信等），类似于多用户系统
        - cgroups：限制容器的资源使用（比如 CPU、内存）

还是用了一些分层文件系统（overlayFS），是的多个容器可以高效共享镜像的只读部分，降低存储开销。



- 运行原理

![image-20241130170013637](/image/image-20241130170013637.png)

docker是一个 Client-Server架构，docker有一个服务端的守护进程（docker daemon）和客户端工具（cli），我们日常使用各种docker命令，其实就是cli发送给docker引擎进行交互的。



## 二、安装部署

1. 更新系统yum包

```bash
## 非必要，但是建议都更新一遍。
sudo yum update --skip-broken -y
```

2. 安装yum源和docker软件

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加yum软件源
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装docker
sudo yum install docker-ce

docker -v

# 启动 docker daemon
sudo systemctl start docker
## 设置开机自启动
sudo systemctl enable docker
```

![image-20241130095648634](/image/image-20241130095648634.png)

![image-20241130095720586](/image/image-20241130095720586.png)

3. 将hadoo用户添加到docker组

   ![image-20241130095901005](/image/image-20241130095901005.png)

   ```bash
   ## 默认情况下，只有root用户有权限访问docker，如果其他用户需要操作docker，可以将用户添加到docker组中。
   sudo usermod -aG docker hadoop
   
   exit
   # 重新登录
   ```

![image-20241130100138979](/image/image-20241130100138979.png)

## 三、基本操作

### 3.1 演示容器安装mysql

> 演示，安装一个基于docker的MySQL

1. 部署docker

   ```bash
   docker run -d \
     --name mysql \
     -p 3307:3306 \
     -e TZ=Asia/Shanghai \
     -e MYSQL_ROOT_PASSWORD=123 \
     mysql
   ```

   ![image-20241130100507767](/image/image-20241130100507767.png)

2. 遇到上面的报错，是因为当前在国内无法正常访问官方的docker仓库，需要配置国内的镜像代理

   > https://www.coderjia.cn/archives/dba3f94c-a021-468a-8ac6-e840f85867ea

   ```bash
   sudo vi /etc/docker/daemon.json
   {
       "registry-mirrors": [
       	"https://docker.unsee.tech",
           "https://dockerpull.org",
           "https://docker.1panel.live"
       ]
   }
   
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

3. 修改完仓库镜像的代理地址之后，我们重新执行安装MySQL的命令，发现正常下载包了。

4. MySQL安装完成后，可以通过任意的客户端工具连接到MySQL

   ```bash
   # 连接容器安装的mysql
   mysql -h hadoop -P 3307 -uroot -p
    # 密码是 123
    show databases;
    select version();
    
   # 连接虚拟机以前安装的mysql
   mysql -h hadoop  -uroot -p
    # 密码是 root123456
    show databases;
    select version();
   ```

   ![image-20241130102505770](/image/image-20241130102505770.png)

5. 大家可以发现，当我们执行了`docker run`命令之后，docker做的第一件事情，是去自动搜索本地仓库是否存在mysql:latest镜像，如果发现没有，则从远程仓库搜索，如果搜索到了，则自动下载并解压和安装mysql，这个过程是自动化的，完成不用人工干预。

    1. docker怎么知道下载什么镜像的？ —— 通过镜像名称[和tag]，tag可以理解为是版本号，默认是latest
    2. 我们怎么知道有什么镜像，镜像都有哪些tag？这些镜像启动的时候需要什么参数？（端口号、环境变量等）
    3. 其实这些镜像都是别人写好的，放到了镜像仓库中的，正常情况下，我们要安装一个基于docker的软件，都需要先在docker hub（或者其他代理地址）搜索并阅读相关的readme之后再安装，这样就知道有哪些tag、需要哪些启动参数。
    4. 另外，我们也可以自己构建自己的镜像，比如自己写的Java程序，也可以打包成镜像。

6. 总而言之，想要使用docker，需要先有镜像，想要得到镜像，需要自己制作或者从docker的镜像仓库下载。

7. 命令解读

   ```bash
   docker [container] run [OPTIONS] IMAGE [COMMAND] [ARG...]
   
   docker run -d \
     --name mysql \
     --restart always \
     -p 3307:3306 \
     -e TZ=Asia/Shanghai \
     -e MYSQL_ROOT_PASSWORD=123 \
     mysql
   ```

   > `docker run -d` ：创建并运行一个容器， `-d` 是让容器以后台的方式进行运行。（daemon）
   >
   > `--name mysql `：给容器起个名字，非必选参数，默认会生成奇怪的名字
   >
   > ` --restart always`： 重启策略，如果容器异常退出，则自动重启，或者重启宿主机之后，等docker daemon重启完，自动拉起该容器。
   >
   > ` -p 3307:3306`：设置端口映射，简单来说，就是将宿主机的某个端口转发到容器内的某个端口
   >
   > `-e TZ=Asia/Shanghai  -e MYSQL_ROOT_PASSWORD=123`：-e用来设置环境变量，每个容器的环境变量都不一样，可以自行查看镜像的说明。
   >
   > ​	格式是 key = value的形式。
   >
   > `mysql` ：镜像的名称，docker会根据这个名称搜索并下载镜像，优先从本地查找。
   >
   > ​	格式是  镜像名称:TAG，例如：mysql:oraclelinux9。默认情况下tag可以省略，省缺值是latest
>
>

### 3.2 基本命令

#### 3.2.1 镜像

> https://docs.docker.com/reference/cli/docker/image/

```bash
## 查看所有本地镜像
docker images
REPOSITORY：镜像名称
TAG：镜像标签（可以理解为就是版本号）
IMAGE ID：镜像ID
CREATED：镜像的创建日期（这个日期不是我们下载镜像的日期）
SIZE：镜像大小

## 搜索镜像
docker search 镜像名
NAME DESCRIPTION STARS OFFICIAL
nginx xxx         10000  true
nginx/nginx-ingress NGINX and NGINX Plus Ingress Controllers for Kubernetes         96  false

## 拉取镜像
docker pull 镜像名称[:TAG]
# 反向操作：push，前提是docker login 已经登录了。
docker push 镜像名称[:TAG]


## 删除本地镜像。  rmi = remove image
docker rmi nginx
# 只有镜像没有被任何的容器（包括正在运行的和已经stop的）使用的时候

# 有时候image太多，又想全部删除，怎么办呢？
docker rmi `docker images -q`
## docker images -q 是只返回所有镜像的id

```

#### 3.2.2 容器

> https://docs.docker.com/reference/cli/docker/container/

```bash
## 查看正在运行的容器
docker ps
CONTAINER ID   IMAGE     COMMAND（容器的启动命令）   CREATED   STATUS    PORTS(端口映射关系)     NAMES

## 查看所有的容器（包括已经停止的容器）
docker ps -a

## 查看停止的容器（不包括正在运行的）
docker ps -f status=exited

## 停止正在运行的容器
docker stop {容器名称|容器的id}

## 删除容器（类似于卸载软件）
docker rm {容器名称|容器的id}


[hadoop@hadoop docker]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
91a5a0c15755   mysql     "docker-entrypoint.s…"   3 minutes ago   Up 2 seconds   33060/tcp, 0.0.0.0:3308->3306/tcp, :::3308->3306/tcp   mysql2
[hadoop@hadoop docker]$ docker rm mysql2
Error response from daemon: cannot remove container "/mysql2": container is running: stop the container before removing or force remove

## 强制删除（不用停止容器，直接删除容器）
docker rm -f {容器名称|容器的id}

# 有时候container太多，又想全部删除，怎么办呢？
docker rm -f `docker ps -a -q`



######## 容器的启停 #########
## 创建并启动容器，启动容器所需的参数，需要自行查看镜像的介绍。
## 可选参数参考 https://docs.docker.com/reference/cli/docker/container/run/
docker run {镜像名称[:tag]}

## 停止正在运行的容器
docker stop {容器名称}


## 启动已经停止的容器
docker start {容器名称}


## 查看容器日志
## -n参数： 只显示最新的num行日期
## -f参数：Follow log output，联想 tail -f
docker logs [-n num] [-f] {容器名称}

# 查看容器详情
docker inspect {容器名称}
```

#### 3.2.3 容器的交互操作

```bash
docker run --name nginx -d -p 8080:80 nginx

## exec执行命令 ，bash是要执行的命令
## 合起来就是进入到nginx这个容器的命令行窗口
## 使用bash还是sh或者其他，取决于基础镜像是哪一个
docker exec -it nginx bash
docker exec -it nginx sh
docker exec -it nginx /bin/sh


docker exec -it nginx ls /usr/share/nginx/html/index.html

## 将nginx容器里面的/usr/share/nginx/html路径下的index.html 复制到 本地（宿主机）并重命名为index.html.bak
docker cp nginx:/usr/share/nginx/html/index.html index.html.bak
## 将本地（宿主机）的index.html文件复制到nginx容器里面的/usr/share/nginx/html路径下
docker cp index.html nginx:/usr/share/nginx/html
```

#### 3.2.4 容器和镜像的操作

```bash
## 我们修改了容器之后，希望保存这个修改，以后启动新的容器的时候就不用再次修改。
## 这个要求我们将容器保存为镜像。有点类似于拍快照
docker commit [-a 作者] {容器名称|容器id}  {镜像名称[:tag]} 

docker commit fe4815e1522b my_nginx 

## 通过新的镜像启动nginx
docker run --name nginx_1 -d -p 8081:80 my_nginx
```

![image-20241130120047743](/image/image-20241130120047743.png)

![image-20241130115943181](/image/image-20241130115943181.png)

## 四、进阶

### 4.1 数据卷和挂载绑定

> https://docs.docker.com/reference/cli/docker/volume/

容器就好像是一个简易版的操作系统，只不过系统中只安装了我们的程序运行所需要的环境，前边说到我们的容器是可以删除的，那如果删除了，容器中的程序产生的需要持久化的数据怎么办呢？容器运行的时候我们可以进容器去查看，容器一旦删除就什么都没有了。大家思考几个问题：

- 如果要升级 MySQL 版本，需要销毁旧容器，那么数据岂不是跟着被销毁了？
- MySQL、Nginx 容器运行后，如果我要修改其中的某些配置该怎么办？
- 我想要让 Nginx 代理我的静态资源怎么办？

因此，容器提供程序的运行环境，但是**程序运行产生的数据、程序运行依赖的配置都应该与容器解耦**

数据卷（volume）和挂载绑定就是来解决这个问题的，是用来将数据持久化到我们宿主机上，与容器间实现数据共享。

- 数据卷是 Docker 管理的持久化存储，它是一个特殊的目录，存在于 Docker 的管理之下，**是容器内目录和宿主机目录之间映射的桥梁**。数据卷可以通过 Docker CLI 创建，并且其生命周期独立于容器。

  ```bash
  ## 显示当前所有的数据卷
  docker volume ls
  
  ## 创建数据卷
  docker volume create my-vol
  
  ## 查看数据卷详情
  docker volume inspect my-vol
  
  
  ## 启动一个容器，并且挂载我们自己的数据卷
  docker run -d --name test_nginx --mount source=my-vol,target=/usr/share/nginx/html/ -p 8888:80 nginx
  
  ## 创建并启动容器后，可以在my-vol对应的宿主机的目录中看到nginx容器的/usr/share/nginx/html/中的内容。
  ## 修改my-vol对应的宿主机的目录中内容
  sudo vim /var/lib/docker/volumes/my-vol/_data/index.html 
  ## 随便修改点东西后保存，然后进行验证。
  
  ## 停止并删除 test_nginx 容器
  docker rm -f test_nginx
  ## 重新创建并行性一个nginx容器，使用同样的数据卷，观察修改的东西是否还在。
  docker run -d --name test_nginx --mount source=my-vol,target=/usr/share/nginx/html/ -p 8888:80 nginx
  
  
  docker volume rm my-vol
  
  
  
  #################### 第二种情况 ####################
  ## 不手动创建数据卷，让docker帮我们自动创建
  docker run -d --name test_nginx -v my-vol:/usr/share/nginx/html/ -p 8888:80 nginx
  ## 执行后发现，docker帮我们创建了一个名叫  my-vol的数据卷，并且挂载到了宿主机的/var/lib/docker/volumes/my-vol/_data/下
  
  
  ```

- 挂载绑定，和数据卷很像，但是不是以docker创建数据卷的方式呈现。

    - 可以发现，数据卷的目录结构比较深，不太好记忆 /var/lib/docker/volumes/my-vol/_data/
    - 很多情况下，我们会使用目录绑定代替数据卷，方便维护数据目录。

  ```bash
  docker run -d --name test_nginx -v ./html:/usr/share/nginx/html/ -p 8888:80 nginx
  
  -v my-vol:/usr/share/nginx/html/  # 会被识别为一个数据卷，叫做 my-vol，运行时如果没有这个卷，会自动创建，创建后会映射容器中的目录内容
  -v ./html:/usr/share/nginx/html/  # 会被识别为一个挂载绑定，绑定到当前的html目录，运行时如果没有这个目录，会自动创建，创建后容器中的内容会被覆盖
   # 挂载绑定不会出现在数据卷中，也就是 docker volume ls 看不到html这个数据卷
   docker volume ls
   
   
  ## 生产常用：比如把MySQL的配置文件用宿主机的配置文件进行替换，以后只需要维护好宿主机的配置文件即可。
  ## 挂载绑定可以是目录，也可以是文件，并且可以挂在多个
  docker run -d \
    --name mysql_bind \
    -v /etc/my.cnf:/etc/my.cnf \
   # -v /etc/hosts:/etc/hosts \
   # -v /hadoop/app/data:/data \
    -p 3308:3306 \
    -e TZ=Asia/Shanghai \
    -e MYSQL_ROOT_PASSWORD=123 \
    mysql
  
  ```

### 4.2 镜像结构

前面我们一直在使用别人准备好的镜像，那如果我要部署一个自己的项目，我该怎么把它打包成镜像呢？

镜像 之所以能够快速的 跨操作系统 部署应用，并且忽略宿主机的运行环境，是因为镜像中包含了程序运行所需要的 系统函数库、运行环境、配置信息、运行依赖等。因此，我们要构建一个镜像，实际上也是需要准备好这些东西。

举个例子，我们部署一个Java应用，大致流程如下：

- 准备一个操作系统
- 在这个操作系统上安装并配置好 jdk/jre
- 上传jar包（以及依赖）
- 运行jar包

那么，我们打包镜像也是这么几步：

- 准备一个操作系统（不是完整的操作系统，仅仅需要操作系统内核）（轻量化）
- 在这个操作系统上安装并配置好 jdk/jre
- 上传jar包（以及依赖）
- 运行jar包

如果每次打包，都需要从操作系统开始，一步一步往下配置，会比较麻烦。docker在构建的时候，实际上会把上述步骤简化，每隔一步骤其实就是在产生一些文件，可以简单理解为一个镜像就是 一堆特定文件 的集合。

在docker中，镜像文件不是随意对方的，他是会根据操作步骤分层叠放。每一层形成的文件会单独打包，并标记一个唯一id，每一层叫做一个  layer（层）。这样做的好处是什么呢？ —— 复用。

![image-20241130161313281](/image/image-20241130161313281.png)

从上图可以看到，新镜像是从base镜像一层一层叠加生成的，每次安装一个新的软件，就在原有的镜像基础上增加一层。

从这里也可以看出，如果很多镜像都用了相同的基础镜像，那么修改基础镜像就会影响到所有基于它构建的镜像。这样，当我们修改一个基础镜像时，可能会造成其他人的镜像无法正常运行，因此docker限制了不允许修改底层镜像，每次修改只能修改最上层。

如果多个容器共享一份基础镜像，当某个容器修改了基础镜像的内容，比如 /etc 下的文件，这时其他容器的 /etc 是不会被修改的，修改只会被限制在单个容器内。这就是容器 **Copy-on-Write** 特性。哪怕/etc下的文件来源于底层，他也会在上层保存修改的文件，当使用时，会从上往下逐层搜索文件，有文件就停止加载底层的同名文件。

最上层的层次叫做可写容器层。当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。

<img src="/image/image-20241130161938737.png" alt="image-20241130161938737" style="zoom:50%;" />

所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。只有**容器层是可写的，容器层下面的所有镜像层都是只读的**。镜像层数量可能会很多，所有镜像层会联合在一起组成一个统一的文件系统。如果不同层中有一个相同路径的文件，比如 /a，上层的 /a 会覆盖下层的 /a，也就是说用户只能访问到上层中的文件 /a。在容器层中，用户看到的是一个叠加之后的文件系统。

## 五、构建镜像

由于制作镜像的过程中，需要逐层处理和打包，比较复杂，所以 Docker 就提供了自动打包镜像的功能。我们只需要将打包的过程，每一层要做的事情用固定的语法写下来，交给 Docker build 去执行即可。

而这种记录镜像结构的文件就称为 **Dockerfile**，其对应的语法可以参考官方文档：

[https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/)



构建一个Java程序的镜像为例

- 创建一个maven项目，并配置打包插件，指定启动类。

  ```xml
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
  
      <groupId>org.example</groupId>
      <artifactId>JavaHttpServer</artifactId>
      <version>1.0-SNAPSHOT</version>
      <packaging>jar</packaging>
  
      <name>JavaHttpServer</name>
      <url>http://maven.apache.org</url>
  
      <properties>
          <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
      </properties>
  
      <dependencies>
      </dependencies>
  
      <build>
          <plugins>
              <!-- Maven Compiler Plugin -->
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-compiler-plugin</artifactId>
                  <version>3.8.1</version>
                  <configuration>
                      <source>8</source>
                      <target>8</target>
                  </configuration>
              </plugin>
  
              <!-- Maven Shade Plugin -->
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-shade-plugin</artifactId>
                  <version>3.2.4</version>
                  <executions>
                      <execution>
                          <phase>package</phase>
                          <goals>
                              <goal>shade</goal>
                          </goals>
                          <configuration>
                              <createDependencyReducedPom>false</createDependencyReducedPom>
                              <transformers>
                                  <transformer
                                          implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
  <!--                                    指定启动类-->
                                      <mainClass>org.example.App</mainClass>
                                  </transformer>
                              </transformers>
                          </configuration>
                      </execution>
                  </executions>
              </plugin>
          </plugins>
      </build>
  </project>
  
  ```

- 使用Java编写一个http服务，监听8080端口，有请求时返回 Hello, Docker World!

```java
package org.example;

import com.sun.net.httpserver.HttpServer;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.HttpExchange;

import java.io.IOException;
import java.io.OutputStream;
import java.net.InetSocketAddress;
/**
 * Hello world!
 */
public class App {
    public static void main(String[] args) throws IOException {
        // 创建 HTTP Server，监听 8080 端口
        HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);

        // 添加一个简单的上下文
        server.createContext("/", new HelloHandler());

        // 启动服务
        server.setExecutor(null); // 默认 executor
        System.out.println("Server started at http://localhost:8080/");
        server.start();
    }

    // 定义请求处理器
    static class HelloHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            String response = "Hello, Docker World!";
            exchange.sendResponseHeaders(200, response.getBytes().length);
            try (OutputStream os = exchange.getResponseBody()) {
                os.write(response.getBytes());
            }
        }
    }
}

```

- 编写Dockerfile，描述我们的程序是怎么一步一步构建出来并运行的。

  ```dockerfile
  # 使用 OpenJDK 基础镜像
  FROM openjdk:8-jre-alpine3.7
  
  # 创建工作目录
  WORKDIR /app
  
  # 复制生成的 JAR 文件到容器
  COPY target/JavaHttpServer-1.0-SNAPSHOT.jar app.jar
  
  # 设置容器启动时的默认命令
  CMD ["java", "-jar", "app.jar"]
  
  ```

- build镜像

  ```bash
  docker build -t {镜像名称} {dockerfile的路径}
  
  docker build -t java-http-server /home/hadoop/soft/docker_image/
  ```

  命令说明：

    - `docker build `： 固件一个docker镜像
    - `-t java-http-server`：-t参数指定镜像的名称，可以带上tag，不带的话默认是latest
    - `/home/hadoop/soft/docker_image/`：dockerfile文件所在的路径。dockerfile文件名字是固定的，不能写其他名字，也不能加其他后缀。

  ![image-20241130164018065](/image/image-20241130164018065.png)

- 启动我们打包好的镜像

  ```bash
  docker run -d --name my-java-http-server  -p 8080:8080 java-http-server
  ```

- dockerfile常用语法：

  | **指令**       | **说明**                                                     | **示例**                                                     |
    | -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | **MAINTAINER** | 镜像维护者的姓名和邮箱地址（非必须）                         | `MAINTAINER sufeng admin@xf233.io`                           |
  | **FROM**       | 指定基础镜像                                                 | `FROM centos:6`                                              |
  | **WORKDIR**    | 指定在创建容器后， 终端默认登录进来的工作目录                | `WORKDIR $CATALINA_HOME` `ENV CATALINA_HOME /usr/local/tomcat ` |
  | **ENV**        | 设置环境变量，可在后面指令使用                               | `ENV key value`                                              |
  | **COPY**       | 拷贝本地文件到镜像的指定目录                                 | `COPY ./xx.jar /tmp/app.jar`                                 |
  | **ADD**        | 将宿主机目录下（或远程文件）的文件拷贝进镜像，且会自动处理URL和解压tar压缩包。 |                                                              |
  | **RUN**        | 执行 Linux 的 shell 命令，一般是安装过程的命令               | `RUN yum install gcc`                                        |
  | **EXPOSE**     | 指定容器运行时监听的端口，是给镜像使用者看的                 | `EXPOSE 8080`                                                |
  | **USER**       | 指定该镜像以用户去执行，如果不指定，默认是`root`             | （一般不修改该配置）                                         |
  | **ENTRYPOINT** | 镜像中应用的启动命令，容器运行时调用                         | `ENTRYPOINT java -jar xx.ja`                                 |
  | **CMD**        | 指定容器启动后要干的事情                                     | `CMD echo "hello world"`                                     |



- 共享镜像给别人

    - 常用于离线环境的镜像分享

  ```bash
  ## 当我们需要把镜像共享给别人使用，又无法上传到仓库时，或者需要不同的环境间传输镜像，可以使用导入导出的方式
  ## 导出镜像：
  docker save -o {保存的文件名} {要导出的镜像[:tag]}
  # 例如：
  docker save -o java-http-server.tar java-http-server:latest
  ## 得到的java-http-server.tar 就可以共享了，别人使用时只需要通过docker load导入即可。
  docker load -i java-http-server.tar 
  
  ```









一键安装脚本 https://get.docker.com

docker镜像仓库 https://docker.unsee.tech