<h1><center>监控系统搭建介绍</center></h1>
* 主要功能点介绍

~~~json
1. 权限系统
2. 微服务架构模型
	consul,zuul,springboot
3. 日志监控elk
	elasticsearch,logstash,kibana,filebeat,metricbeat,hartbeat等
4. docker容器搭建oracle数据库,nginx集群,redis集群
~~~

* docker搭建oracle数据库
** docker安装

~~~json
1. 下载docker-ce的repo
curl https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
2. 安装依赖（这是相比centos7的关键步骤）
yum install https://download.docker.com/linux/fedora/30/x86_64/stable/Packages/containerd.io-1.2.6-3.3.fc30.x86_64.rpm
3. 安装docker-ce
yum install docker-ce
4. 启动docker
systemctl start docker
~~~