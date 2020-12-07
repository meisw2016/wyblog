# docker安装consul集群

~~~
docker run -d --name consul01 --network consul  -p 21001:8500 consul:release agent --server=true --bootstrap-expect=3 --client=0.0.0.0 --datacenter=bj -ui
docker run -d --name=consul02 --network consul consul:release agent --server=true --client=0.0.0.0 --datacenter=bj --join 172.19.0.2 
docker run -d --name=consul03 --network consul consul:release agent --server=true --client=0.0.0.0 -datacenter=bj --join 172.19.0.2 
docker run -d --name=consul04 --network consul consul:release agent --server=false --client=0.0.0.0 -datacenter=bj --join 172.19.0.2
~~~
# 查看防火墙

~~~
systemctl status firewalld查看firewalld状态
systemctl start firewalld开启防火墙
systemctl stop firewalld

firewall-cmd --permanent --zone=public --add-port=21001/tcp
~~~

~~~
docker run -d --name consul01 --network consul  -p 8500:8500 consul:release agent --server=true --bootstrap-expect=3 --client=0.0.0.0 --datacenter=bj -ui
docker run -it  --name registry -p 5000:5000 -v /opt/registry:/var/lib/registry -v /var/run/docker.sock:/var/run/docker.sock -d registry:release
~~~
# linux安装oracle数据库
~~~
oracle/appmon123
创建用户组
groupadd oinstall
groupadd dba

创建oracle用户和密码
useradd -g oinstall -g dba -m oracle
passwd oracle

yum install -y automake autotools-dev binutils bzip2 elfutils expat \
gawk gcc gcc-multilib g++-multilib lib32ncurses5 lib32z1 \
ksh less lib32z1 libaio1 libaio-dev libc6-dev libc6-dev-i386 \
libc6-i386 libelf-dev libltdl-dev libodbcinstq4-1 libodbcinstq4-1:i386 \
libpth-dev libpthread-stubs0-dev libstdc++5 make openssh-server rlwrap \
rpm sysstat unixodbc unixodbc-dev unzip x11-utils zlibc unzip cifs-utils \
libXext.x86_64  glibc.i686


groupadd -g 502 oinstall
groupadd -g 503 dba
groupadd -g 504 oper
groupadd -g 505 asmadmin
useradd -u 502 -g oinstall -G oinstall,dba,asmadmin,oper -s /bin/bash -m oracle
passwd oracle


5.修改操作系统配置
操作用户：root
操作文件：/etc/security/limits.conf

vim /etc/security/limits.conf
在文件的末尾添加如下配置项。

oracle          soft      nproc   2047
oracle          hard      nproc   16384
oracle          soft      nofile  1024
oracle          hard      nofile  65536
oracle          soft      stack   10240
~~~

# --创建oracle网络

~~~
docker network create --subnet=172.20.0.0/16 oracle

docker run --privileged --name oracle11g -p 1521:1521 -v /data/oracle:/install jaspeen/oracle-11g
docker run --privileged --name oracle --network oracle -p 21003:1521 -v /data/oracle:/install jaspeen/oracle-11g
docker run --privileged --name oracle --network oracle -p 21003:1521 -v /dev/shm:/install jaspeen/oracle-11g
docker run --privileged --name oracle --network oracle -p 21003:1521 -v /home:/install jaspeen/oracle-11g
~~~

# 配置oracle

~~~
docker exec -it oracle11g /bin/bash
su - oracle
sqlplus / as sysdba
SQL> alter user scott account unlock;
SQL> commit;
SQL> conn scott/tiger;

alter user appmon default tablespace IOMP_BPM
~~~
# 创建表空间

~~~
create tablespace IOMP
logging
datafile '/opt/oracle/app/oradata/orcl/IOMP.dbf'
size 32m
autoextend on
next 32m maxsize 2048m
extent management local;

SELECT a.tablespace_name "表空间名",
total/1024/1024  "表空间大小单位M",
free/1024/1024 "表空间剩余大小单位M",
(total - free)/1024/1024 "表空间使用大小单位M",
 Round((total - free) / total, 4) * 100 "使用率   [[%]]"FROM 
(SELECT tablespace_name,Sum(bytes) free FROM DBA_FREE_SPACE GROUP BY tablespace_name) a,
(SELECT tablespace_name,
 Sum(bytes) total FROM DBA_DATA_FILES GROUP BY tablespace_name) b WHERE a.tablespace_name = b.tablespace_name;
 ---
 select a.tablespace_name, total, free, total-free as used, substr(free/total * 100, 1, 5) as "FREE%", substr((total - free)/total * 100, 1, 5) as "USED%" from 
(select tablespace_name, sum(bytes)/1024/1024 as total from dba_data_files group by tablespace_name) a, 
(select tablespace_name, sum(bytes)/1024/1024 as free from dba_free_space group by tablespace_name) b
where a.tablespace_name = b.tablespace_name
order by a.tablespace_name;

 --查看当前连接数
  select count(*) from v$session;

 --查询所有表空间
 select tablespace_name, sum(bytes)/1024/1024 from dba_data_files group by tablespace_name;
 
 select * from dba_tablespaces;
 
 select * from v$tablespace;
 --查询用户下的表空间
 select * from dba_users where username='CLOUDMON01';
 select * from dba_users where username = 'APPMON';
 --查看表空间是否自动扩容
 select tablespace_name,file_name,autoextensible from dba_data_files;
 
 --查询表空间下的用户
  select distinct s.owner from dba_segments s where s.tablespace_name ='IOMP';  
 
 --表空间扩容
 alter tablespace IOMP_BPM add datafile '/opt/oracle/app/oradata/orcl/IOMP_BPM.dbf' size 2048M AUTOEXTEND on next 2016m
 
 ALTER DATABASE DATAFILE '/opt/oracle/app/oradata/orcl/IOMP_BPM.dbf' AUTOEXTEND ON NEXT 2016M;
 --删除表空间
 drop tablespace IOMP_BPM;
 --删除有数据的表空间
 drop tablespace IOMP_BPM including contents and datafiles;
--创建表空间
create tablespace IOMP_BPM
logging
datafile 'E:\install\oracle\product\10.2.0\oradata\orcl\IOMP_BPM.dbf'
size 2016m
autoextend on
next 2016m maxsize 2048m
extent management local;

--导入dump文件
impdp cloudmon01/cloudmon01@ORCL full=Y directory=aa dumpfile=cloudmon01.dbbak.0624.dmp transform=segment_attributes:n;

创建用户并指定表空间
create user cloudmon01 identified by "cloudmon01" default tablespace iomp;
给用户授权
grant connect,resource,dba to cloudmon01;
grant connect to cloudmon01 with admin option; 
grant javasyspriv to cloudmon01; 
grant java_admin to cloudmon01; 
grant resource to cloudmon01 with admin option; 
grant administer database trigger to cloudmon01 with admin option; 
grant alter any index to cloudmon01; 
grant alter any procedure to cloudmon01; 
grant alter any table to cloudmon01; 
grant alter session to cloudmon01; 
grant alter tablespace to cloudmon01; 
grant analyze any to cloudmon01; 
grant audit any to cloudmon01; 
grant comment any table to cloudmon01; 
grant create any index to cloudmon01; 
grant create any materialized view to cloudmon01 with admin option; 
grant create any procedure to cloudmon01; 
grant create any sequence to cloudmon01; 
grant create any synonym to cloudmon01 with admin option; 
grant create any table to cloudmon01 with admin option; 
grant create any trigger to cloudmon01; 
grant create any view to cloudmon01 with admin option; 
grant create cluster to cloudmon01; 
grant create job to cloudmon01; 
grant create library to cloudmon01; 
grant create materialized view to cloudmon01;

grant create procedure to cloudmon01; 
grant create profile to cloudmon01; 
grant create sequence to cloudmon01; 
grant create session to cloudmon01; 
grant create synonym to cloudmon01; 
grant create table to cloudmon01; 
grant create trigger to cloudmon01; 
grant create type to cloudmon01; 
grant create view to cloudmon01; 
grant debug any procedure to cloudmon01; 
grant debug connect session to cloudmon01; 
grant delete any table to cloudmon01 with admin option; 
grant drop any index to cloudmon01; 
grant drop any materialized view to cloudmon01; 
grant drop any synonym to cloudmon01 with admin option; 
grant drop any table to cloudmon01; 
grant drop any view to cloudmon01; 
grant insert any table to cloudmon01 with admin option; 
grant on commit refresh to cloudmon01; 
grant select any dictionary to cloudmon01; 
grant select any table to cloudmon01 with admin option; 
grant unlimited tablespace to cloudmon01 with admin option; 
grant update any table to cloudmon01 with admin option;
~~~

# docker搭建redis cluster集群

~~~
docker network create --subnet=172.21.0.0/16 redis


for port in $(seq 21010 21015); \
do \
  mkdir -p ./${port}/conf  \
  && PORT=${port} envsubst < ./redis-cluster.tmpl > ./${port}/conf/redis.conf \
  && mkdir -p ./${port}/data; \
done


for port in $(seq 21010 21015); \
do \
   docker run -it -d -p ${port}:${port} -p 1${port}:1${port} \
  --privileged=true -v /home/soft/redis/${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  --privileged=true -v /home/soft/redis/${port}/data:/data \
  --restart always --name redis-${port} --net redis \
  --sysctl net.core.somaxconn=1024 redis redis-server /usr/local/etc/redis/redis.conf; \
done


上面安装redis失败，试试以下操作
docker run -it -d --name r1 -p 5001:6379 --net=redis --ip 172.21.0.12 redis bash 
docker  run -it -d --privileged=true --name r1 -v /home/redis/redis.conf:/usr/redis/redis.conf -p 5001:6379 --net=redis --ip 172.21.0.12 redis:meisw bash
docker  run -it -d --name r2 -v /home/redis/r2/redis.conf:/usr/redis/redis.conf -p 5002:6379 --net=redis --ip 172.21.0.13 redis:meisw bash
docker  run -it -d --name r3 -v /home/redis/r3/redis.conf:/usr/redis/redis.conf -p 5003:6379 --net=redis --ip 172.21.0.14 redis:meisw bash
docker  run -it -d --name r4 -v /home/redis/r4/redis.conf:/usr/redis/redis.conf -p 5004:6379 --net=redis --ip 172.21.0.15 redis:meisw bash
docker  run -it -d --name r5 -v /home/redis/r5/redis.conf:/usr/redis/redis.conf -p 5005:6379 --net=redis --ip 172.21.0.16 redis:meisw bash
docker  run -it -d --name r6 -v /home/redis/r6/redis.conf:/usr/redis/redis.conf -p 5006:6379 --net=redis --ip 172.21.0.17 redis:meisw bash

./redis-trib.rb create --replicas 1 172.21.0.12:6379 172.21.0.13:6379 172.21.0.14:6379 172.21.0.15:6379 172.21.0.16:6379 172.21.0.17:6379

进入容器
docker exec -it r1 /bin/bash
编辑配置文件
vim /usr/redis/redis.conf
apt-get update
apt-get install vim 

cd /usr/redis/src
./redis-server ../redis.conf & 以后台方式启动
配置redis节点
daemonize yes #以后台进程运行
cluster-enabled yes #开启集群
cluster-config-file nodes.conf #集群配置文件
cluster-node-timeout 15000 #超时时间
appendonly yes #开启AOF

daemonize yes
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
appendonly yes
bind 172.21.0.14

安装redis-trib.rb
docker exec -it r1 bash
cd /usr/redis/
mkdir cluster
cp /usr/redis/src/redis-trib.rb /usr/redis/cluster/
cd /usr/redis/cluster
apt-get install ruby
apt-get install rubygems
gem install redis

启动redis
cd /usr/redis/src
./redis-server ../redis-config

连接到redis服务器
./redis-cli -h  127.0.0.1 -p 6379
设置密码
config get requirepass 123456
验证密码
auth 123456
~~~

# docker安装nginx

~~~
docker run -d  --privileged=true -p 21080:80 --net=47467db801c8 --ip 172.22.0.3 --name nginx \
-v /data/nginx/www:/usr/share/nginx/html \
-v /data/nginx/conf:/etc/nginx/nginx.conf \
-v /data/nginx/logs:/var/log/nginx nginx:release

docker run -d  --privileged=true -p 24380:80 --net=47467db801c8 --ip 172.22.0.2 --name nginx \
-v /data/nginx/www:/usr/share/nginx/html \
-v /data/nginx/conf:/etc/nginx/nginx.conf \
-v /data/nginx/logs:/var/log/nginx nginx:release

docker run -d  --privileged=true -p 24381:80 --net=47467db801c8 --ip 172.22.0.3 --name nginx2 \
-v /data/nginx2/www:/usr/share/nginx/html \
-v /data/nginx2/conf:/etc/nginx/nginx.conf \
-v /data/nginx2/logs:/var/log/nginx nginx:release

创建Redis集群
./redis-trib.rb create --replicas 1 172.21.0.12:6379 172.21.0.13:6379 172.21.0.14:6379 172.21.0.5:6379 172.21.0.16:6379 172.21.0.17
~~~
# 查看docker容器ip的神器

~~~
curl -L https://github.com/hlwojiv/tools/releases/download/1.0/docker-allip -o /usr/local/bin/docker-allip && chmod +x /usr/local/bin/docker-allip
查看docker容器所有IP命令
docker-allip
~~~

# docker部署elk

~~~
docker network create --subnet=172.23.0.0/16 elk

docker run -d --name=elasticsearch --net elk -p 21002:9200 --ip 172.23.0.2 -p 9300:9300 \
-v /data/elasticsearch.yml:/usr/share/elasticsearch/elasticsearch.yml elasticsearch:6.6.2

docker run -d --name=es-header --net elk -p 24101:9100 --ip 172.23.0.4 docker.io/mobz/elasticsearch-head:5

docker run --name kibana --privileged=true --net=elk \
--link elasticsearch:elasticsearch \
-p 21004:5601 \
-d kibana:release

下载heartbeat
curl -L -O https://artifacts.elastic.co/downloads/beats/heartbeat/heartbeat-6.6.2-linux-x86_64.tar.gz
tar xzvf heartbeat-6.6.2-linux-x86_64.tar.gz





nohup $JAVA_HOME/bin/java -Xms512m -Xmx512m -classpath $CLASSPATH:
$BINPATH/micro-web-1.0.0.jar org.springframework.boot.loader.JarLauncher 
--spring.config.location=$BINPATH/conf/bootstrap-dev.yml -Dlogback.configurationFile=$BINPATH/conf/logback.xml > micro-web.out  2>&1&





1.修改redis配置主机和端口
cp redis.properties /home/soft/microservice/microsrv-web-cloud/micro-alarm/conf

/home/soft/jdk1.8.0_40
/home/soft/microservice/microsrv-web-cloud/micro-alarm


/home/ap/appmon/jdk1.8.0_144

jdbc:oracle:thin:@60.205.208.22:21003:orcl
60.205.208.22:5001,60.205.208.22:5002,60.205.208.22:5003,60.205.208.22:5004,60.205.208.22:5005,60.205.208.22:5006
~~~



# 各个微服务对应阿里云服务器上的端口

~~~
alarm 21004
auth 24160
auth-server 24900
cloudview 24997
es 24156
es-data 24155
feign-server 24999
servicetracking 24521
tableview 24152
tranline 24996
trantrain 24151
web 24180
zuul 24130
mo 25000

--micro-tableview需要导入的jar
mvn install:install-file -Dfile=D:\tmp\micro-tableview-1.0.0\BOOT-INF\lib\appmon-baseline3-his-2.0.0.jar  -DgroupId=appmon.baseline3 -DartifactId=appmon-baseline3-his -Dversion=2.0.0 -Dpackaging=jar
mvn install:install-file -Dfile=D:\tmp\micro-tableview-1.0.0\BOOT-INF\lib\commons-jexl-3.1.jar  -DgroupId=org.apache.commons -DartifactId=commons-jexl3 -Dversion=3.1 -Dpackaging=jar


==================================
获取ES索引的Mapping结构



~~~
# =docker挂载

~~~
docker run -d  --privileged=true -p 24382:80 --net=47467db801c8 --ip 172.22.0.4 --name nginx3 \
-v /dev/shm/dockerVolumn/nginx/www:/usr/share/nginx/html \
-v /dev/shm/dockerVolumn/nginx/conf:/etc/nginx/nginx.conf \
-v /dev/shm/dockerVolumn/nginx/logs:/var/log/nginx nginx:release


docker安装oracle操作步骤：

安装环境查看centos版本
lsb_release -a
uname -r
用yum源安装
查看是否已经安装docker列表
yum list installed | grep docker
安装docker
yum -y install docker
启动docker
systemctl start docker
查看docker服务状态
systemctl status docker

下载jdk
 wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz
配置jdk环境变量
JAVA_HOME=/dev/shm/jdk1.8.0_40
JRE_HOME=$JAVA_HOME/jre
CLASS_PATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH PATH
source /etc/profile使配置生效

安装oracle
将linux.x64_11gR2_database_1of2.zip和linux.x64_11gR2_database_2of2.zip两个压缩包解压到home目录下
创建oracle网络
docker network create --subnet=172.20.0.0/16 oracle

启动oracle镜像进行安装 (-v后面的目录就是oracle解压的路径)
docker run --privileged --name oracle11g -p 1521:1521 -v /data/oracle:/install jaspeen/oracle-11g
docker run --privileged --name oracle --network oracle -p 21003:1521 -v /data/oracle:/install jaspeen/oracle-11g
docker run --privileged --name oracle --network oracle -p 21003:1521 -v /dev/shm:/install jaspeen/oracle-11g
docker run --privileged --name oracle --network oracle -p 21003:1521 -v /home:/install jaspeen/oracle-11g

配置oracle
docker exec -it oracle11g /bin/bash
su - oracle
sqlplus / as sysdba
SQL> alter user scott account unlock;
SQL> commit;
SQL> conn scott/tiger;

~~~