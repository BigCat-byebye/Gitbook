一，Redis主从配置
1.其master不需要做任何更改，只需要更改slave的配置文件即可，由于做实验的时候是在同一台，所以pid文件和db文件，port端口豪以及conf配置文件，都是要进行修改的
slave的配置文件修改如下
pidfile /var/run/redis_6380.pid
dbfilename dump_6380.rdb
dir ./data_6380/
port 6380
replicaof 127.0.0.1 6379
其中最重要的就是replicaof 后需要写master的ip和端口
2.启动即可
/usr/local/redis/src/redis_server ../redis.conf  (启动master）
/usr/local/redis/src/redis_server ../redis_slaver.conf (启动slave）
/usr/local/redis/src/redis_cli -p 6379 (连接master）
set ceshi 9
get ceshi
exit 
/usr/local/redis/src/redis_cli -p 6380 （连接slave）
get ceshi
如上文简单测试，主从复制即可确认成功

二，Redis集群配置
1.集群架构图，如下
![图片](https://images-cdn.shimo.im/avFOefILXe0opbgn/image.png!thumbnail)


在Redis的集群架构中，有几点是需要值得注意点的，首先，集群中的节点的不可用（fail）是需要靠该及集群所有master进行投票决定的，即ping-pong机制，超过半数master如法连接到该节点，则认为该节点不可用（fail）