Docker

[TOC]



**虚拟机**

待环境安装的一种解决方案。它可以在一种操作系统里面运行另外一种操作系统，比如在Windows系统里面运行Linux系统。应用程序对此毫无感知，因此虚拟机看上去就和真实系统一模一样，而对于底层系统来说，虚拟机就是一个普通的文件，不需要就删除，对于其他的部分毫无影响。这类虚拟机完美的运行了另外的一套系统，能够使应用程序，操作系统，和硬件三者之间的逻辑不变。

**缺点**：启动慢，冗余步骤多，资源占用多

**Linux容器**

Linux容器不是模拟一个完整的操作系统，而是对进程进行隔离。有了容器，就可以将软件运行所需要的所有资源打包到一个隔离的环境中。容器与虚拟机不同，不需要捆绑一整套的操作系统，只是需要软件工作所需要的库资源和设置。

**Docker**是一个开源的应用容器引擎；是一个轻量级容器技术；

Docker支持将软件编译成一个镜像；然后在镜像中各种软件做好配置，将镜像发布出去，其他使用者可以直接使用这个镜像；

![mark](http://www.hhaxmm.cn/blog/20190119/5BDjAHPlVk38.png)



<font color='red'>运行中的这个镜像称为容器</font>， 容器启动是非常快速的。

- 传统的虚拟机技术是虚拟出一套硬件之后，在其上运行一个完整的操作系统，在该系统上运行所需要的进程
- 而容器内的应用进程是直接运行与宿主的内核，容器自己没有内核，而且也没有进行硬件虚拟，因此容器跟加轻便
- 每个容器相互隔离

**DevOps:自开发运维**：今天的优势会被明天的趋势所替代

## 2、核心概念

<font color='red'>docker主机(Host)：安装了Docker程序的机器（Docker直接安装在操作系统之上）</font>；

docker客户端(Client)：连接docker主机进行操作；

docker仓库(Registry)：用来保存各种打包好的软件镜像；Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。    

docker镜像(Images)：软件打包好的镜像；放在docker仓库中；

docker容器(Container)：镜像启动后的实例称为一个容器；容器是独立运行的一个或一组应用(运行5次tomcat就有5个tomcat实例)

![mark](http://www.hhaxmm.cn/blog/20190119/o3c39DTHHX0o.png)

使用Docker的步骤：

1）、安装Docker

2）、去Docker仓库找到这个软件对应的镜像；

3）、使用Docker运行这个镜像，这个镜像就会生成一个Docker容器；

4）、对容器的启动停止就是对软件的启动停止；



## 3、安装Docker

#### 1）、安装linux虚拟机

2）、在linux虚拟机上安装docker

步骤：

```shell
1、检查内核版本，必须是3.10及以上 yum update 升级内核版本
uname -r
2、安装docker
yum install docker
3、输入y确认安装
4、启动docker
[root@localhost ~]# systemctl start docker
[root@localhost ~]# docker -v
Docker version 1.12.6, build 3e8e77d/1.12.6
5、开机启动docker
[root@localhost ~]# systemctl enable docker
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
6、停止docker
systemctl stop docker
```

![1547874660048](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1547874660048.png)



## 4、Docker常用命令&操作

| 操作 | 命令                                            | 说明                                                     |
| ---- | ----------------------------------------------- | -------------------------------------------------------- |
| 检索 | docker  search 关键字  eg：docker  search redis | 我们经常去docker  hub上检索镜像的详细信息，如镜像的TAG。 |
| 拉取 | docker pull 镜像名:tag                          | :tag是可选的，tag表示标签，多为软件的版本，默认是latest  |
| 列表 | docker images                                   | 查看所有本地镜像                                         |
| 删除 | docker rmi image-id                             | 删除指定的本地镜像                                       |

https://hub.docker.com/

### 1)、阿里云镜像加速

软件镜像（QQ安装程序）----运行镜像(镜像是分层的)----产生一个容器（正在运行的软件，运行的QQ）；

阿里云镜像加速服务

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

```json
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://sz8ysd54.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 2）、容器操作

步骤：

```shell
1、搜索镜像
[root@localhost ~]# docker search tomcat
2、拉取镜像
[root@localhost ~]# docker pull tomcat
3、根据镜像启动容器
docker run --name mytomcat -d tomcat:latest
4、docker ps  
查看运行中的容器
5、 停止运行中的容器
docker stop  容器的id
6、查看所有的容器
docker ps -a   （ps命令是查看正在运行中的）
7、启动容器
docker start 容器id
8、删除一个容器
 docker rm 容器id
9、启动一个做了端口映射的tomcat
[root@localhost ~]# docker run -d -p 8888:8080 tomcat
-d：后台运行
-p: 将主机的端口映射到容器的一个端口    主机端口:容器内部的端口

10、为了演示简单关闭了linux的防火墙
service firewalld status ；查看防火墙状态
service firewalld stop：关闭防火墙
11、查看容器的日志
docker logs container-name/container-id

更多命令参看
https://docs.docker.com/engine/reference/commandline/docker/
可以参考每一个镜像的文档

```

| 操作               | 命令                                                         | 说明                                                      |
| ------------------ | ------------------------------------------------------------ | --------------------------------------------------------- |
| 运行               | docker run --name container-name -d image-name eg:docker run –name myredis –d redis | --name：自定义容器名 -d：后台运行 image-name:指定镜像模板 |
| 列表               | docker ps（查看运行中的容器）；                              | 加上-a；可以查看所有容器                                  |
| 停止               | docker stop container-name/container-id                      | 停止当前你运行的容器                                      |
| 启动               | docker start container-name/cont ainer-id                    | 启动容器                                                  |
| 删除               | docker rm **container-id**                                   | 删除指定容器                                              |
| 端口映射           | -p 6379:6379 eg:docker run -d -p 6379:6379 --name myredis docker.io/redis | -p: 主机端口(映射到)容器内部的端口                        |
| 容器日志           | docker logs container-name/container-id                      |                                                           |
| 更多命令           | https://docs.docker.com/engine/reference/commandline/docker/ |                                                           |
| 运行并且伪终端     | docker run -it image-id，这种方式是进入到容器的实例中进行交互，退出这种交互有两种方式，一种是使用exit(容器停止并且退出)，另外的一种是使用Ctrl+P+Q 容器不停止退出; | 运行并且以伪终端的方式进行运行。                          |
| 查运行进程         | docker  top container-id                                     |                                                           |
| 查容器细节         | docker inspect container-id                                  |                                                           |
| 进入正在运行的容器 | docker  exec  -t  container-id  /bin/bash重新与程序进行交互  | 直接的得到结果                                            |
|                    |                                                              |                                                           |
|                    |                                                              |                                                           |

### 3）、安装MySQL示例

1、安装mysql 2、安装redis 3、 安装rabbitmq 4、 安装elasticsearch    

```shell
docker pull mysql
```

错误的启动

```shell
[root@localhost ~]# docker run --name mysql01 -d mysql
42f09819908bb72dd99ae19e792e0a5d03c48638421fa64cce5f8ba0f40f5846

mysql退出了
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
42f09819908b        mysql               "docker-entrypoint.sh"   34 seconds ago      Exited (1) 33 seconds ago                            mysql01
538bde63e500        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       compassionate_
goldstine
c4f1ac60b3fc        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       lonely_fermi
81ec743a5271        tomcat              "catalina.sh run"        About an hour ago   Exited (143) About an hour ago                       sick_ramanujan


//错误日志
[root@localhost ~]# docker logs 42f09819908b
error: database is uninitialized and password option is not specified 
  You need to specify one of MYSQL_ROOT_PASSWORD, MYSQL_ALLOW_EMPTY_PASSWORD and MYSQL_RANDOM_ROOT_PASSWORD；这个三个参数必须指定一个
```

正确的启动

```shell
[root@localhost ~]# docker run --name mysql01 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
b874c56bec49fb43024b3805ab51e9097da779f2f572c22c695305dedd684c5f
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
b874c56bec49        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 3 seconds        3306/tcp            mysql01
```

做了端口映射

```shell
[root@localhost ~]# docker run -p 3306:3306 --name mysql02 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
ad10e4bc5c6a0f61cbad43898de71d366117d120e39db651844c0e73863b9434
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ad10e4bc5c6a        mysql               "docker-entrypoint.sh"   4 seconds ago       Up 2 seconds        0.0.0.0:3306->3306/tcp   mysql02
```



几个其他的高级操作

```
docker run --name mysql03 -v /conf/mysql:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
把主机的/conf/mysql文件夹挂载到 mysqldocker容器的/etc/mysql/conf.d文件夹里面
改mysql的配置文件就只需要把mysql配置文件放在自定义的文件夹下（/conf/mysql）

docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
指定mysql的一些配置参数

```

redis运行的时候的命令

```

```



安装zookeeper

## docker镜像原理

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种分层、轻量级并且高性能的文件系统，它支持<font color='red'>对文件系统的修改作为一次提交来一层层的叠加</font>，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

**Docker镜像加载原理**：
   docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。
bootfs(boot file system)主要包含bootloader和kernel, bootloader主要是引导加载kernel, Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权已由bootfs转交给内核，此时系统也会卸载bootfs。

rootfs (root file system) ，在bootfs之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs就是各种不同的操作系统发行版，比如Ubuntu，Centos等等。 

**最大的一个好处就是** - 共享资源

比如：有多个镜像都从相同的 base 镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，
同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。 ￼

### docker commit

```
docker commit : 提交容器副本使之成为一个新的镜像

docker commit -m=“提交的描述信息” -a=“作者” 容器ID 要创建的目标镜像名:[标签名]
```



## Docker 容器数据卷

一句话：有点类似我们Redis里面的rdb和aof文件

先来看看Docker的理念：
*  将运用与运行的环境打包形成容器运行 ，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
*  容器之间希望有可能共享数据

Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一部分保存下来，
那么当容器删除后，数据自然也就没有了。

为了能保存数据在docker中我们使用卷。 卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷

特点：
1：数据卷可在容器之间共享或重用数据
2：卷中的更改可以直接生效
3：数据卷中的更改不会包含在镜像的更新中
4：数据卷的生命周期一直持续到没有容器使用它为止

 ### 具体的操作

 **直接命令添加**：

```
docker run -it -v /宿主机绝对路径目录:/容器内目录      镜像名 
```

```txt
-it：这是两个参数，
-i：交互式操作
-t 终端。我们这里打算进入 bash 执行一些命令并查看返回结果，因此我们需要交互式终端。
```

举个例子

```\
docker run -it -v /myDateVolume:/dataVolumeContainer 镜像名
```

加权限（值允许主机进行单方面的操作）

```
docker run -it -v /宿主机绝对路径目录:/容器内目录:ro 镜像名 
```

**dockerfile添加**：

-  主机根目录下新建mydocker文件夹并进入
- 可在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷

```java
VOLUME["/dataVolumeContainer","/dataVolumeContainer2","/dataVolumeContainer3"]
 
说明：
 
出于可移植和分享的考虑，用-v 主机目录:容器目录这种方法不能够直接在Dockerfile中实现。
由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。
目录是容器的目录
```
- File构建

```dockerfile
FROM centos
VOLUME["/dataVolumeContainer","/dataVolumeContainer2"]
CMD echo "finished,.....success1"
CMD /bin/bash
```

- build后生成镜像

```:new_zealand:
docker bulid -f dockfile位置 -t 镜像名（自己起的名字哦）
```

- 通过上述步骤，容器内的卷目录地址已经知道
  对应的主机目录地址哪？？

```
默认的位置：
使用docker inspect查看
```

![mark](http://www.hhaxmm.cn/blog/20190119/b26pt3uDMmuc.png)



## Dockerfile

Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本。

编写Dockerfile文件

1. docker build
2. docker run

### Dockerfile内容基础知识

1：每条保留字指令都必须为大写字母且后面要跟随至少一个参数

2：指令按照从上到下，顺序执行

3：#表示注释

4：每条指令都会创建一个新的镜像层，并对镜像进行提交

### Dockerfile大致的执行流程

（1）docker从基础镜像运行一个容器

（2）执行一条指令并对容器作出修改

（3）执行类似docker commit的操作提交一个新的镜像层

（4）docker再基于刚提交的镜像运行一个新容器

（5）执行dockerfile中的下一条指令直到所有指令都执行完成

### DockerFile体系结构(保留字指令)

| FROM       | 基础镜像，当前新镜像是基于哪个镜像的                         |
| ---------- | ------------------------------------------------------------ |
| MAINTAINER | 镜像维护者的姓名和邮箱地址                                   |
| RUN        | 容器构建时需要运行的命令                                     |
| EXPOSE     | 当前容器对外暴露出的端口                                     |
| WORKDIR    | 指定在创建容器后，终端默认登陆的进来工作目录，一个落脚点     |
| ENV        | 用来在构建镜像过程中设置环境变量                             |
| ADD        | 将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包|
| COPY       | 类似ADD，拷贝文件和目录到镜像中。
将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置 |
| VOLUME     | 容器数据卷，用于数据保存和持久化工作                         |
| CMD        | 指定一个容器启动时要运行的命令,Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换 |
| ENTRYPOINT | ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数------(追加) |
| ONBUILD    | 当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发 |

### Dockerfile案例

<font color='red'>案例1：自定义mycentos目的使我们自己的镜像具备如下:</font>

-  登陆后的默认路径
- vim编辑器
- 查看网络配置ifconfig支持

默认的从官方下载的centos是不支持的

1. 编写dockerfile文件

```dockerfile
FROM centos
MAINTAINER river<hhriver0601@163.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH
CMD echo "finished,.....ok"
CMD /bin/bash
```

2. docker build 构建

```powershell
docker build -f dockerfile路径 -t 新镜像名字:TAG .
# 举例子如下所示
docker build -f /root/mydocker/Dockerfile1 -t mycentos:1.3 .
```

3. 运行

```
docker run -it 新镜像名字:TAG 
```



<font color='red'>案例二：CMD/ENTRYPOINT 镜像案例</font>

都是指定一个容器启动时要运行的命令，Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效，CMD 会被 docker run 之后的参数替换

tomcat的讲解演示 

```powershell
docker run -it -p 8888:8080 tomcat ls -l
```

docker run 之后的参数会被当做参数传递给 ENTRYPOINT，之后形成新的命令组合

制作CMD版可以查询IP信息的容器

```powershell
FROM centos
RUN yum install -y curl
CMD [ "curl", "-s", "http://ip.cn" ]
```

问题：如果我们希望显示HTTP头信息，就需要加上-i 参数

我们可以看到可执行文件找不到的报错，executable file not found。之前我们说过，跟在镜像名后面的是 command，运行时会替换 CMD 的默认值。因此这里的 -i 替换了原来的 CMD，而不是添加在原来的 curl -s http://ip.cn 后面。而 -i 根本不是命令，所以自然找不到。那么如果我们希望加入 -i 这参数，我们就必须重新完整的输入这个命令：$ docker run myip curl -s http://ip.cn -i

制作ENTRYPOINT 版可以查询IP信息的容器

```dockerfile
FROM centos
RUN yum install -y curl
ENTRYPOINT [ "curl", "-s", "http://ip.cn" ]
```

<font color='red'>案例三：自定义tomcat</font>

自定义镜像Tomcat9

```dockerfile
FROM         centos
MAINTAINER    zzyy<zzyybs@126.com>
#把宿主机当前上下文的c.txt拷贝到容器/usr/local/路径下
COPY c.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk-8u171-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.8.tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_171
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.8
ENV CATALINA_BASE /usr/local/apache-tomcat-9.0.8
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
#容器运行时监听的端口
EXPOSE  8080
#启动时运行tomcat
# ENTRYPOINT ["/usr/local/apache-tomcat-9.0.8/bin/startup.sh" ]
# CMD ["/usr/local/apache-tomcat-9.0.8/bin/catalina.sh","run"]
CMD /usr/local/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/logs/catalina.out
 
```

run执行:(-v是容器数据卷的指令)

```
 docker run -d -p 9080:8080 --name myt9
 # 这个命令加入在主机的test文件夹下发布文件，就会和docker镜像里面的文件共享
 -v /zzyyuse/mydockerfile/tomcat9/test:/usr/local/apache-tomcat-9.0.8/webapps/test
 -v /zzyyuse/mydockerfile/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.8/logs --privileged=true zzyytomcat9
```

![1547891418997](C:\Users\huanghe\AppData\Roaming\Typora\typora-user-images\1547891418997.png)

## 本地镜像推送到阿里云

阿里云开发者平台：https://dev.aliyun.com/search.html

创建仓库镜像

![mark](http://www.hhaxmm.cn/blog/20190201/QGoBDu1FXu8D.png)

将镜像推送到registry

```dockerfile
$ sudo docker login --username=735597346@qq.com registry.cn-hangzhou.aliyuncs.com
$ sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/riverzmm/myapp:[镜像版本号]
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/riverzmm/myapp:[镜像版本号]
```



