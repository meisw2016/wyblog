<h1><center>docker搭建grafana</center></h1>

# docker启动grafana

~~~
docker run -d --network elk --ip 172.18.0.4 --name=grafana -h grafana -p 21005:3000 grafana/grafana:latest
初始账号密码：admin/admin
new password:123456
~~~

# docker安装node

~~~
docker pull node:latest

-- ROUTE -p add 172.23.0.0 mask 255.255.0.0 192.168.60.137

ROUTE -p add 172.17.0.0 mask 255.255.0.0 192.169.60.129
172.17.0.2:3000
~~~