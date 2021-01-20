# skywalking-es7-docker

```
docker-compose -f "docker-compose.yml" up -d --build
```


可能出现的问题

解决Docker容器 iptables问题

一、问题现象
最近在研究Docker容器日志管理时，启动容器出现iptables相关报错，具体问题如下

运行容器

[root@node-11 ~]# docker run -d -p 24224:24224 -p 24224:24224/udp -v /data:/fluentd/log fluent/fluentd
出现如下报错

docker: Error response from daemon: driver failed programming external connectivity on endpoint quizzical_thompson (c2b238f6b003b1f789c989db0d789b4bf3284ff61152ba40dacd0e01bd984653):  (iptables failed: iptables --wait -t filter -A DOCKER ! -i docker0 -o docker0 -p tcp -d 172.17.0.3 --dport 24224 -j ACCEPT: iptables: No chain/target/match by that name.
 (exit status 1)).
 
 
 二、解决办法
经过查阅资料得知是docker0网桥的原因，解决上面报错问题需要进行一下步骤
1.kill掉docker所有进程

[root@node-11 ~]# pkill docker 
2.清空nat表的所有链

[root@node-11 ~]# iptables -t nat -F
3.停止docker默认网桥docker0

[root@node-11 ~]# ifconfig docker0 down
4.删除docker0网桥

[root@node-11 ~]# brctl delbr docker0
5.重启docker服务

[root@node-11 ~]# systemctl restart docker
至此，成功运行docker容器

[root@node-11 ~]# docker run -d -p 24224:24224 -p 24224:24224/udp -v /data:/fluentd/log fluent/fluentd
644e43d03b9a2b30c062c8b5cde972b5514e6eef8a8ae95a6ab8c8004af6db5b

如何删除Docker网桥
虚拟网卡docker0其实是一个网桥,docker0这个网桥是在启动Docker Daemon时创建的，因此，这种删除方法并不能根本上删除docker0，下次daemon启动（假设没有指定-b参数）时，又会自动创建docker0网桥。
如果想删除它，只需要按照删除网桥的方法即可
# 停止docker service
systemctl stop docker
 
#关闭docker0网卡(如果没有ifconfig，则需要先安装(yum install net-tools))
ifconfig docker0 down
 
#删除docker0网卡(如果没有brctl，则需要先安装(yum install bridge-utils))
brctl delbr docker0