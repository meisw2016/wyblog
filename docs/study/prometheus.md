<h1><center>docker搭建prometheus</center></h1>
# 镜像下载

~~~json
docker pull prom/node-exporter
docker pull prom/prometheus
docker pull grafana/grafana
~~~
# docker创建网络

~~~json
创建docker网络
docker network create -d overlay net2

创建docker网络并指定网段
docker network create --subnet=172.19.0.0/16 net2

链接两个docker网络之间的通信
docker network connect ov_net1 bbox3

~~~
# docker网络相关

~~~json
docker network create docker-network 	//docker-network是局域网的名字，自定义
docker network ls  //查看已有的network
docker network connect	将容器连接到网络。
docker network create	创建新的 Docker 网络。默认情况下，在 Windows 上会采用 NAT 驱动，在 Linux 上会采用
Bridge 驱动。可以使用 -d 参数指定驱动（网络类型）。
docker network connect     将容器连接到网络
docker network disconnect	断开容器的网络。
docker network inspect	提供 Docker 网络的详细配置信息。
docker network ls	用于列出运行在本地 Docker 主机上的全部网络。
docker network prune	删除 Docker 主机上全部未使用的网络。
docker network rm	删除 Docker 主机上指定网络。

docker绑定网卡启动镜像
docker run -itd --name bbox1 --network ov_net1 busybox
bbox1的自定义镜像名称
ov_net1 为自定义网卡名称
busybox为原始镜像
~~~

# 启动node-exporter

~~~json
docker run -d -p 20101:9100 \
  --net prometheus21 --ip 172.21.0.101 \
  --name exporter \
  -v "/proc:/host/proc:ro" \
  -v "/sys:/host/sys:ro" \
  -v "/:/rootfs:ro" \
  prom/node-exporter
~~~