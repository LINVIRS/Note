

# docker 安装 以及简单项目部署

## 1.0 准备阶段

### 1.0.1 购买服务器

首先我们确保我们的运行是在Liunx环境运行 ,所以我们需要一台liunx服务器，没有小伙伴可以购买阿里云学生机 。当然我们也为了没钱的小伙伴准备了人性化的活动，阿里云高校计划。（当然规定是全日制在校学生才可以）

```xml
阿里云高校计划传送门：https://developer.aliyun.com/adc/student/ 
阿里云学生机购买传送门:https://promotion.aliyun.com/ntms/act/campus2018.html
```

### 1.0.2 准备下载连接工具

当买好阿里云服务器后 我们需要去xftp官网申请一下xftp 和xshell  （学生免费） 

这款工具简而言之就是方便我们连接远程我们的服务器。

冲!我们的准备工作就完成的。

## 2.0 docker 安装

### 2.1 docker官网 

```
地址：https://docs.docker.com/engine/install/centos/  
```

####  2.1.1 运行命令

```java
//清除服务器上docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
-----------------------------------------
//安装工具
sudo yum install -y yum-utils
-----------------------------------------
//配置镜像（官网镜像太慢了）所以我们找到一个阿里的镜像地址 听懂了没有？
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
-----------------------------------------
sudo yum install docker-ce docker-ce-cli containerd.io
------------------------------------------
 //启动docker   
sudo systemctl start docker
------------------------------------------
 //输入命令  查看当前容器
 docker ps    
  //出现CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES 
    代表安装docker成功！ 下班 byebye。
  
```

2.1.2 配置阿里镜像加速（这一步可以不用配 配了就会更快更高效 便捷）

```java
//阿里镜像加速教程
https://developer.aliyun.com/article/692983
```

2.1.3

```
我们需要去阿里云去开放我们的安全组规则 
例如你需要访问你服务器8089端口 这时候你就需要配置服务器安全组规则其中一个8089
当然一个个端口开放比较麻烦 偷懒的小技巧是 开放所有端口（可能有安全问题）

```

![image-20200903132119745](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20200903132119745.png)

-----------------------------------------------------------------------------------------------------------------------------

![image-20200903131947153](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20200903131947153.png)

docker 安装完毕！

----------------------------------------------------------------------------------------------------------------------------

## 3.0 简单项目部署

###  3.0.1 准备工作

​     

```java
1.准备对应版本JDK liunx 压缩包 自己去官网就可以下载：https://www.oracle.com/cn/java/technologies/javase-downloads.html
2.编写一个dockerfile
// dockerfile内容如下
---------------------------------------------------------------------------------
# Docker image for springboot file run
# VERSION 0.0.1
# Author: wl

# 建立一个新的镜像文件，配置模板：新建立的镜像是以centos为基础模板
# 因为jdk必须运行在操作系统之上
FROM centos:7

# 作者名  作者邮箱
MAINTAINER wl<1987153442@qq.com>

#设置系统编码
RUN yum install kde-l10n-Chinese -y
RUN yum install glibc-common -y
RUN localedef -c -f UTF-8 -i zh_CN zh_CN.utf8
#RUN export LANG=zh_CN.UTF-8
#RUN echo "export LANG=zh_CN.UTF-8" >> /etc/locale.conf
#ENV LANG zh_CN.UTF-8
ENV LC_ALL zh_CN.UTF-8


# 创建一个新目录来存储jdk文件
RUN mkdir /usr/local/java

#将jdk压缩文件复制到镜像中，它将自动解压缩tar文件（版本不一致自己调换版本）
ADD jdk-11.0.6_linux-x64_bin.tar.gz /usr/local/java/

# 创建软连接 ln -s 源文件 目标文件（主要自己上传jdk版本名字）
# RUN ln -s /usr/local/java/jdk-11.0.6 /usr/local/java/jdk

# 设置环境变量
ENV JAVA_HOME /usr/local/java/jdk-11.0.6
ENV PATH $JAVA_HOME/bin:$PATH

# VOLUME 指定了临时文件目录为/tmp
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp

# 运行jar包
# 运行.sh文件 这里注意 .sh 文件放置于/usr/local/idea 位置以及文件名字 可以自定义
CMD ["sh","-c","/usr/local/idea/wljar.sh"]
--------------------------------------------------------------------
 3   //编写xxx.sh 文件
    //1.jar 是jar包名字 前面是jar包存在路径
    //.sh文件内容如下
--------------------------------------------------------------------
    java -jar /usr/local/idea/1.jar
----------------------------------------------------------------------
    4准备一个jar包
-----------------------------------------------------------------
    5将jar包 jdk 以及 .sh 文件 以及dockerfile 丢到服务器中（依靠xftp）
```

### 3.0.2 docker 运行项目

```java
//在dockerfile 所在路径下运行命令：wldemojar 是指代生成镜像的名字 此处可以替换
docker build -t wldemojar . 
------------------------------------------------------------------------------------
//查看刚才创建镜像    
docker images 
//看见以下信息说明创建成功    
//wldemojar                                                     latest              279c07755469        4 seconds ago       695MB    
-------------------------------------------------------------------------------------
 // 上一步执行中可能出现 .sh permmison defined 权限不足问题  
 // 解决办法：进入.sh所在目录执行 下面命令 
  chmod u+x *.sh
-------------------------------------------------------------------------------------
 //运行docker run 命令 使容器生效 至此简单部署全部完成   -v是容器挂载  --name后参数是镜像名字
docker run --name thp -it -v /usr/local/idea:/usr/local/idea --privileged=true -d -p 8081:8081 jar    
-----------------------------------------------------------------------------------
//容器id 
 d69828275dfbc53ab6724d746e5fefaf017f125af79e298d1748eb8d7e34d921
//部署成功 可以访问接口
```

### 3.0.3 一些命令解释

![image-20200903131208397](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20200903131208397.png)



```
上述命令用到了容器挂载：容器挂载指的就是 容器内部数据 挂载在liunx服务器外部 这种挂载好处就是我们可以在容器外部去操作容器内部数据。
// -p 命令 -p 8080:8080 第一个8080指代jar包中运行端口号 后一个8080 指我们映射出来的端口号
```

```
//docker ps -aq 查看所有容器 
//docker ps 查看当前运行容器
//docker stop 容器id  停止指定id容器（批量停止所有容器）
docker stop $(docker ps -aq)
//docker rm 容器id 删除一个容器 （这儿是批量删除所有容器）
docker rm $(docker ps -aq)
//docker images 查看当前docker存在镜像
dokcer rmi 镜像名字 删除镜像
docker rmi 镜像id 

```



## 5.0 docker安装GitLab：

```java
----------------------------------------------------------------------------------
	//下拉最新gitLab
	docker pull gitlab/gitlab-ce:latest
----------------------------------------------------------------------------------
    # 启动
    docker run --detach \
    --publish 443:443 --publish 80:80  --publish 222:22 \
    --name gitlab \
    --memory 4g \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest

----------------------------------------------------------------------------------------------
修改ssh端口 （至关重要）
vi /etc/ssh/sshd_config
找到  #port   将其改为 port 任意端口 注意不要和已知端口冲突
# 2.重启sshd服务       
systemctl restart sshd       
#3.重启container
#4.0查看当前运行容器
docker ps 
#5.0找到containerId 
docker restart containerId
重启gitlab至此gitlab搭建完成
这时候可以等待访问你的gitLab   地址为服务器ip地址        
-------------------------------------------------------------------------------------------
--hostname gitlab.example.com: 设置主机名或域名
--detach 如果在docker run后面追加-d=true或者-d，那么容器将会运行在后台模式。此时所有I/O数据只能通过网络资源或者共享卷组来进行交互。
--publish 443:443：将http：443映射到外部端口443   https 访问端口
--publish 80:80：将web：80映射到外部端口80 web访问端口
--publish 222:22：将ssh：22映射到外部端口222  ssh 访问端口
--name gitlab: 运行容器名
--restart always: 自动重启
--volume /srv/gitlab/config:/etc/gitlab: 挂载目录
--volume /srv/gitlab/logs:/var/log/gitlab: 挂载目录
--volume /srv/gitlab/data:/var/opt/gitlab: 挂载目录
--privileged=true 使得容器内的root拥有真正的root权限。否则，container内的root只是外部的一个普通用户权限
---------------------------------------------------------------------------------------------

# 参考
https://segmentfault.com/a/1190000019772866?utm_source=sf-related
```



## 4.0结语

这是docker容器技术初探。简简单单容器部署jar包。

不足之处在于 每次部署需要终止容器 删除镜像 替换jar包 或者.sh文件有问题就需要吧镜像容器全部删除

重新运行docker build 以及docker run  命令 比较麻烦 很讨厌 下次分享更简单部署	