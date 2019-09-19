# 端口转发
## 基于软件`rinetd`
1. 安装
``` shell
wget https://boutell.com/rinetd/http/rinetd.tar.gz && tar xzvf rinetd.tar.gz -C /usr/local/ && cd /usr/local/rinetd/ && sed -i 's/65536/65535/g rinetd.c' && make && make install 
```
2. 配置
``` shell
[root@zhibo rinetd]# cat /etc/rinetd.conf
0.0.0.0 19903 34.92.76.173 19903

# 格式如上：source_ip source_port remote_ip remote_port
```