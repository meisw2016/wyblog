<h1><center>使用docker搭建YAPI服务</center></h1>
# 集群说明

~~~
虚拟机 192.168.60.129	ranch2		ranch主机
docker 1.13.1升级最新版
yum -y remove docker*
安装国内阿里云镜像
yum install -y yum-utils device-mapper-persistent-data lvm2 sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
安装最新版本的docker
yum install docker-ce
安装ranch2.0
docker run -d --restart=unless-stopped -p 9080:80 -p 443:443 rancher/rancher

echo '{ "insecure-registries":["192.168.60.129:5000"] }' > /etc/docker/daemon.json
~~~
# 1.创建docker独立网络

~~~
docker network create --subnet=172.22.0.0/16 msw
~~~
# 2.启动MongoDB

~~~
docker run -d --name mongo-api mongo
~~~
# 3.获取yapi镜像

~~~
docker pull registry.cn-hangzhou.aliyuncs.com/anoy/yapi
~~~
# 4.初始化数据库索引及管理员账号

~~~
docker run -it --rm \
  --link mongo-yapi:mongo \
  --entrypoint npm \
  --workdir /api/vendors \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  run install-server
~~~
# 5.启动yapi服务

~~~
docker run -d \
  --name yapi \
  --link mongo-yapi:mongo \
  --workdir /api/vendors \
  -p 13000:3000 \
  registry.cn-hangzhou.aliyuncs.com/anoy/yapi \
  server/app.js
~~~
