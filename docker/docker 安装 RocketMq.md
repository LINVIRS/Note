# docker 安装 RocketMq

## 1.0 查找我们需要的镜像 

```
//命令
docker search rocketmq

docker没有官方镜像  我们选一个较好的 
```

![image-20201007115924400](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20201007115924400.png)

## 2.0查看镜像所有版本以及内容

```java
//如果查看其他的版本 将{}内容替换即可 找到版本 
curl https://registry.hub.docker.com/v1/repositories/{foxiswho/rocketmq}/tags\
| tr -d '[\[\]" ]' | tr '}' '\n'\
| awk -F: -v image='{foxiswho/rocketmq}' '{if(NR!=NF && $3 != ""){printf("%s:%s\n",image,$3)}}'
```

![image-20201007120135487](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20201007120135487.png)

## 3.0 启动 NameServer

```
//选择一个版本号  这里最新 4.7.0 
docker run -d -p 9876:9876 --name rmqserver  foxiswho/rocketmq:server-4.7.0 
```

### 3.1docker ps 看到如下图 

![image-20201007120848339](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20201007120848339.png)

## 4.0启动broker

### 4.1首先我们要在Ecs目录下创建 一个 broker.conf 文件 （先创建 不然 broker 无法启动）

![image-20201007120527862](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20201007120527862.png)

### 4.2broker.conf 内容如下

```java
brokerClusterName = DefaultCluster
brokerName = broker-a
brokerId = 0
deleteWhen = 04
fileReservedTime = 48
brokerRole = ASYNC_MASTER
flushDiskType = ASYNC_FLUSH
brokerIP1 = {自己的公网ip}
```

### 4.3 运行命令

```java
docker run -d -p 10911:10911 -p 10909:10909\
 --name rmqbroker --link rmqserver:namesrv\
 -e "NAMESRV_ADDR=namesrv:9876" -e "JAVA_OPTS=-Duser.home=/opt"\
 -e "JAVA_OPT_EXT=-server -Xms128m -Xmx128m"\
 -v /conf/broker.conf:/etc/rocketmq/broker.conf \
 foxiswho/rocketmq:broker-4.5.1
```

### 4.4 docker ps 运行 看到 下图

![image-20201007120802049](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20201007120802049.png)

## 5.0 启动 可视化界面

### 5.1运行命令

```
docker run -d --name rmqconsole -p 8180:8080 --link rmqserver:namesrv\
 -e "JAVA_OPTS=-Drocketmq.namesrv.addr=namesrv:9876\
 -Dcom.rocketmq.sendMessageWithVIPChannel=false"\
 -t styletang/rocketmq-console-ng
```

### 5.2docker ps 出现 下图 即为成功

![image-20201007121031990](C:\Users\wanglin\AppData\Roaming\Typora\typora-user-images\image-20201007121031990.png)



## 6.0总结



```
参考：https://blog.csdn.net/fenglibing/article/details/92378090
```

