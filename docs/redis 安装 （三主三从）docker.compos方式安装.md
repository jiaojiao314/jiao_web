

# redis 安装 （三主三从）docker.compos方式安装

参考链接：[https://www.jianshu.com/p/b7dea62bcd8b](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/b7dea62bcd8b)



参考镜像：[https://github.com/publicisworldwide/docker-stacks/tree/master/oracle-linux/environments/storage/redis-cluster](https://link.zhihu.com/?target=https%3A//github.com/publicisworldwide/docker-stacks/tree/master/oracle-linux/environments/storage/redis-cluster)



## 1.创建数据目录



```
[root@localhost redis]# pwd
/data/redis
[root@localhost redis]# mkdir 700{1..6}/data  ##这个目录和下边yml文件的volumes对应
[root@localhost redis]# ls 
7001  7002  7003  7004  7005  7006  docker-compose.yml
```



### 1.1 导入镜像

[📎redis-cluster.tar](https://www.yuque.com/attachments/yuque/0/2020/tar/671316/1591261142509-2cbff7a8-bf0e-4c79-bb08-647ad1e6252e.tar)

docker load < redis-cluster.tar

## 2.创建一个docker-compose.yml文件，内容如下：



```
version: '3'

services:
 redis1:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/7001/data:/data
  environment:
   - REDIS_PORT=7001

 redis2:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/7002/data:/data
  environment:
   - REDIS_PORT=7002

 redis3:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/7003/data:/data
  environment:
   - REDIS_PORT=7003

 redis4:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/7004/data:/data
  environment:
   - REDIS_PORT=7004

 redis5:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/7005/data:/data
  environment:
   - REDIS_PORT=7005

 redis6:
  image: publicisworldwide/redis-cluster
  network_mode: host
  restart: always
  volumes:
   - /data/redis/7006/data:/data
  environment:
   - REDIS_PORT=7006
```



这里用的别人一个现成的镜像 publicisworldwide/redis-cluster ，自己制作也可以，后续会发布制作过程，不过不建议自己做，有什么需要更改的可以在现有的镜像基础上进行更改之后commit。镜像可以自己下载，也可以等启动服务的时候他自动下载。



这里用的是host网络模式，redis挂载到之前创建的目录下。



如果不想使用主机模式可以文章开头的链接，具体我这里就不写了，自己看。至于docker-compose的安装网上一大堆文章，这里不赘述。



## 3.创建文件之后，启动服务



```
[root@k8s-m2 redis]# docker-compose up -d 
Creating redis_redis1_1 ... done
Creating redis_redis5_1 ... done
Creating redis_redis4_1 ... done
Creating redis_redis3_1 ... done
Creating redis_redis6_1 ... done
Creating redis_redis2_1 ... done 
[root@k8s-m2 redis]# docker-compose ps 
     Name                   Command               State   Ports
---------------------------------------------------------------
redis_redis1_1   /usr/local/bin/entrypoint. ...   Up           
redis_redis2_1   /usr/local/bin/entrypoint. ...   Up           
redis_redis3_1   /usr/local/bin/entrypoint. ...   Up           
redis_redis4_1   /usr/local/bin/entrypoint. ...   Up           
redis_redis5_1   /usr/local/bin/entrypoint. ...   Up           
redis_redis6_1   /usr/local/bin/entrypoint. ...   Up
```



状态都为Up，说明服务均正常启动，如果有状态不正常的容器，看看自己服务器的端口是不是被占用了。



## 4.redis-cluster集群配置



上述只是启动了6个redis容器，并没有设置集群，随便进入一个redis容器，执行下面的命令，IP替换成自己宿主机的IP



```
[root@k8s-m2 redis]# docker exec -ti d3a904cc0d5a bash 
root@k8s-m2:/data# redis-cli --cluster create 192.168.1.100:7001 192.168.1.100:7002 192.168.1.100:7003 192.168.1.100:7004 192.168.1.100:7005 192.168.1.100:7006 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.1.100:7004 to 192.168.1.100:7001
Adding replica 192.168.1.100:7005 to 192.168.1.100:7002
Adding replica 192.168.1.100:7006 to 192.168.1.100:7003
>>> Trying to optimize slaves allocation for anti-affinity
[WARNING] Some slaves are in the same host as their master
M: 7237efa105ff4246c02da08c0e65c18246755f2f 192.168.1.100:7001
   slots:[0-5460] (5461 slots) master
M: 241d6eb2abcd9a6d41db5231fcd80fac1d452bee 192.168.1.100:7002
   slots:[5461-10922] (5462 slots) master
M: 493083f2733bc181295f5b9a9fa9bc6fdf821958 192.168.1.100:7003
   slots:[10923-16383] (5461 slots) master
S: b60ef6cbea0ae06566d17db4a1b4452e56543419 192.168.1.100:7004
   replicates 241d6eb2abcd9a6d41db5231fcd80fac1d452bee
S: b48e546e963f694eac91d49b285040bb2bec5513 192.168.1.100:7005
   replicates 493083f2733bc181295f5b9a9fa9bc6fdf821958
S: 259de2d6be917e64cdc01604521da45cc1f8d842 192.168.1.100:7006
   replicates 7237efa105ff4246c02da08c0e65c18246755f2f
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
....
>>> Performing Cluster Check (using node 192.168.1.100:7001)
M: 7237efa105ff4246c02da08c0e65c18246755f2f 192.168.1.100:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 241d6eb2abcd9a6d41db5231fcd80fac1d452bee 192.168.1.100:7002
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: 493083f2733bc181295f5b9a9fa9bc6fdf821958 192.168.1.100:7003
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: b60ef6cbea0ae06566d17db4a1b4452e56543419 192.168.1.100:7004
   slots: (0 slots) slave
   replicates 241d6eb2abcd9a6d41db5231fcd80fac1d452bee
S: 259de2d6be917e64cdc01604521da45cc1f8d842 192.168.1.100:7006
   slots: (0 slots) slave
   replicates 7237efa105ff4246c02da08c0e65c18246755f2f
S: b48e546e963f694eac91d49b285040bb2bec5513 192.168.1.100:7005
   slots: (0 slots) slave
   replicates 493083f2733bc181295f5b9a9fa9bc6fdf821958
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

****

至此，基于docker部署的redis-cluster集群就已经完成