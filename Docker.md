# Docker

## 安装

### yum安装

参考链接：

https://www.runoob.com/docker/centos-docker-install.html

http://t.csdn.cn/bOzWw

注意：新系统在执行yum操作之前先确认服务器环境，若不具备外网环境或当前系统版本已不受支持，请先更换yum源以及epel源。

1. 卸载原有Docker...

   - ```
     yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
     ```

2. 安装所需的软件包 yum-utils|device-mapper-persistent-data|lvm2

   - ```
     yum install -y yum-utils device-mapper-persistent-data lvm2
     ```

3. 设置阿里云仓库（国内仓库稳定）

   - ```
     yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
     ```

4. 安装docker-ce（社区版）

   - ```
     yum install docker-ce
     ```

5. 测试docker是否安装成功

   - ```
     docker -v
     ```

6. 启动docker

   - ```
     systemctl start docker
     ```

7. 启动|状态|停止 docker 指令

   - ```
     systemctl status|start|stop docker
     ```

8. 配置docker阿里云镜像

   - https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
   - 根据网址操作文档操作

9. 测试

   - download nginx latest

     - ```
       docker pull nginx:latest
       ```

   - check images

     - ```
       docker images
       ```

   - start container

     - ```
       docker run --name nginx-test -p 8080:80 -d nginx
       ```

   - 访问nginx ：ip:8080 (要开放端口号，或者关闭防火墙)

     - httpdeathsky.top:8080

## docker命令

### docker镜像操作

```shell
#查看本地所有镜像
docker images
#查看镜像实例运行状态
docker ps -a
#查看docker版本
docker -v
#查看docker信息
docker info
#查看公共镜像仓库
docker search `xxx`
#下载镜像
docker pull
#获取镜像信息
docker inspect `镜像ID`
#添加镜像标签
docker tag `名称:[旧标签]` `新名称:[新标签]`
#删除镜像
docker rmi `镜像ID号` || `仓库名称:标签`
rmi是rm images的缩写
#加载所有镜像ID
docker images -q
#批量删除所有镜像
docker rmi `docker images -q`
#批量删除nginx镜像
docker rmi `docker images|grep "nginx"`
#注意：如果该镜像已经被容器使用，正确的做法是先删除依赖该镜像的所有容器，再去删除镜像
#先通过docker-compose down 停止服务 后再次执行删除

#导出/导入镜像

#导出镜像
#格式：docker save -o 存储文件名 存储的镜像
#存出镜像命名为nginx存在当前目录下
docker save -o nginx_v1 nginx:latest
#将导出的镜像以scp方式导到别的服务器上
scp nginx_v1 @root:192.168.59.111:/opt

#导入镜像，可以异地导入，但是必须要有docker引擎，并且版本不可以差太多
#格式：docker load < 存出的文件
docker load < nginx_v1
```

### docker容器操作

```shell
#容器创建
docker create
新创建的容器默认处于停止状态，不运行任何程序，需要在其中发起一个进程来启动容器
格式：docker create [选项] 镜像
常用选项：
-i：让容器的输入保持打开
-t：让 Docker 分配一个伪终端
 
docker create -it nginx:latest /bin/bash

#启动容器
#格式：docker start 容器的ID/名称
docker start b2a57b3ea48a

#查看运行中的容器
docker ps -a
docker container ls -a

docker ps
docker container ls

#创建并启动容器
#docker run = docker create && start
#容器是一个与其中运行的 shell 命令共存亡的终端，命令运行容器运行， 命令结束容器退出

#注：docker容器默认会把容器内部第一个进程，也就是pid=1的程序作为docker容器是否正在运行的依据，如果docker容器中pid=1的进程挂了，那么docker容器便会直接退出，也就是说Docker容器中必须有一个前台进程，否则认为容器已经挂掉。

#当利用 docker run 来创建容器时，Docker在后台的标准运行过程是
#检查本地是否存在指定的镜像。当镜像不存在时，会从公有仓库下载；
#利用镜像创建并启动一个容器；
#分配一个文件系统给容器，在只读的镜像层外面挂载一层可读写层；
#从宿主主机配置的网桥接口中桥接一个虚拟机接口到容器中；
#分配一个地址池中的 IP 地址给容器；
#执行用户指定的应用程序，执行完毕后容器被终止运行。

#加 -d 选项让 Docker 容器以守护形式在后台运行。并且容器所运行的程序不能结束。

#示例1：
docker run -itd nginx:latest /bin/bash
 
#示例2：执行后退出
docker run centos:7 /usr/local/bash -c ls /   
 
#示例3：执行后不退出，以守护进程方式执行持续性任务
docker run -d centos:7 /usr/local/bash -c "while true;do echo hello;done"

#容器删除
docker rm 'contain-name'

#容器退出
exit
#退出但不关闭容器
ctrl+q+p

#查看所有容器的ID
docker ps -aq

#查看容器详细信息
docker container inspect 容器名或者运行号码

#可查看容器内进程信息
docker container top 容器的ID

#查看容器的日志信息
docker container logs [-ft] 容器的ID
-f 为持续监控，-t 为更加详细显示

#进入容器——docker exec
#会创建前台进程，但是会在输入exit后终止进程
docker run -it
#会通过连接stdin，连接到容器内输入输出流，会在输入exit后终止容器进程
docker attach
#会连接到容器，可以像sSH一样进入容器内部，进行操作，可以通过exit退出容器，不影响容器运行
docker exec -it

#需要进入容器进行命令操作时，可以使用 docker exec 命令进入运行着的容器。
#格式：docker exec -it 容器ID/名称 /bin/bash
-i 选项表示让容器的输入保持打开；
-t 选项表示让 Docker 分配一个伪终端。

#示例：进入（三种方式）
docker run -itd centos:7 /bin/bash  #先运行容器
docker ps -a 
#1.使用run进入，可以使用ctrl+d退出，直接退出终端
docker run -it centos:7 /bin/bash 
 
#2.想永久性进入，退出后还是运行状态，用docker exec
docker ps -a 
docker exec -it b99e0771c4e1  /bin/bash
 
#3.会通过连接stdin，连接到容器内输入输出流，公在输入exit后终止容器进程（临时性的，不推荐）
docker attach

#容器导出/导入
docker export

#用户可以将任何一个 Docker 容器从一台机器迁移到另一台机器。在迁移过程中，可以使用docker export 命令将已经创建好的容器导出为文件，无论这个容器是处于运行状态还是停止状态均可导出。可将导出文件传输到其他机器，通过相应的导入命令实现容器的迁移

#导出格式：docker export 容器ID/名称 > 文件名
docker export b99e0771c4e1 > centos_7
 
#导入格式：cat 文件名 | docker import – 镜像名称:标签
docker import centos_7  centos：v1	#导入后会生成镜像，但不会创建容器
cat centos_7 |docker import - centos:v2

#删除容器
#格式：docker rm [-f] 容器ID/名称
1.#不能删除运行状态的容器，只能-f强制删除，或者先停止再删除
docker rm 3224eb044879

2.#已经退出的容器，可以直接删除
docker rm 1270a6791069 

3.#基于名称匹配的方式删除
docker rm -f distracted_panini

4.#删除所有运行状态的容器
docker rm -f `docker ps -q`

5.#删除所有容器
docker rm -f `docker ps -aq`

6.#有选择性的批量删除 （正则匹配）
docker ps -a l awk ' {print "docker rm "$1}'l bash

7.#删除退出状态的容器
for i in `dockef ps -a l grep -i exit / awk '{print $1}' '; do docker rm -f $i;done

#查看docker资源
docker stats
```

### docker仓库操作

```shell
#首先下载registry 镜像
docker pull registry
 
#在daemon.json文件中添加私有镜像仓库地址
vim /etc/docker/daemon.json
{
"insecure-registries": ["192.168.80.10:5000"],   #添加，注意用逗号结尾
"registry-mirrors": ["https://6ijb8ube.mirror.aliyuncs.com"]
}
 
systemctl restart docker.service
 
#运行registry 容器
docker run -itd -v /data/registry:/var/lib/registry -p 5000: 5000 --restart=always --name registry registry:latest
-------------------------------------------------
-itd: 在容器中打开一个伪终端进行交互操作，并在后台运行
-v: 把宿主机的/data/registry目录绑定到容器/var/lib/registry目录(这个目录是registry容器中存放镜像文件的目录)，来实现数据的持久化;
-p:映射端口;访问宿主机的5000端口就访问到registry容器的服务了
--restart=always: 这是重启的策略，在容器退出时总是重启容器
--name registry: 创建容器命名为registry
registry:latest:这个是刚才pull下来的镜像
---------------------------------------------------
Docker容器的重启策略如下:
no:默认策略，在容器退出时不重启容器
on-failure:在容器非正常退出时(退出状态非0)，才会重启容器
on-failure:3 :在容器非正常退出时重启容器，最多重启3次
always:在容器退出时总是重启容器
unless-stopped:在容器退出时总是重启容器，但是不考虑在Docker守护进程启动时就已经停止了的容器
 
#为镜像打标签
docker tag centos:7 192.168.80.10:5000/centos:v1
 
#上传到私有仓库
docker push 192.168.80.10:5000/centos:v1
 
#列出私有仓库的所有镜像
curl http://192.168.80.10:5000/v2/_catalog
 
#出私有仓库的centos 镜像有哪些tag
curl http://192.168.80.10:5000/v2/centos/tags/list
 
#先删除原有的 centos 的镜像，再测试私有仓库下载
docker rmi -f 8652b9f0cb4c
docker pull 192.168.80.10:5000/centos:v1 
```
