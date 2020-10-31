<h1><center>linux防火墙查看状态firewall,iptable</center></h1>
CentOS7 的防火墙配置跟以前版本有很大区别，CentOS7这个版本的防火墙默认使用的是firewall，与之前的版本Centos 6.x使用iptables不一样

# 一、iptables防火墙
* 1.基本操作

    查看防火墙状态
~~~json
service iptables status  
~~~
    停止防火墙
~~~json
service iptables stop  
~~~
	启动防火墙
~~~json
service iptables start  
~~~
	重启防火墙
~~~
service iptables restart  
~~~
	永久关闭防火墙
~~~
chkconfig iptables off  
~~~
	永久关闭后重启
~~~
chkconfig iptables on　　
~~~

* 2.开启80端口

~~~
vim /etc/sysconfig/iptables
加入如下代码
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
保存退出后重启防火墙
service iptables restart
~~~

# 二、firewall防火墙
* 1.查看firewall服务状态
~~~
systemctl status firewalld
出现Active: active (running)切高亮显示则表示是启动状态。
出现 Active: inactive (dead)灰色表示停止，看单词也行。
~~~
* 2.查看firewall的状态
~~~
firewall-cmd --state
~~~
* 3.开启、重启、关闭、firewalld.service服务
~~~
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
~~~
* 4.查看防火墙规则
~~~
firewall-cmd --list-all 
~~~
* 5.查询、开放、关闭端口
~~~
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
~~~
 

# CentOS7 默认使用firewalld防火墙，如果想换回iptables防火墙，可关闭firewalld并安装iptables。

* 1、关闭firewall：
~~~
停止firewall
systemctl stop firewalld.service
禁止firewall开机启动
systemctl disable firewalld.service
查看默认防火墙状态（关闭后显示notrunning，开启后显示running）
firewall-cmd --state
~~~
* 2.安装iptables-services
~~~
yum install iptables-services
~~~
* 3.修改防火墙配置文件

~~~
vi /etc/sysconfig/iptables
默认的文件为：
在修改之前使用telnet命令查看端口是否开放。
1.启动telnet。控制面板-->程序和功能-->打开或关闭windows功能-->勾选Telnet的两个选项。
2.打开cmd窗口，输入telnet，如果端口关闭或者无法连接，则显示不能打开到主机的链接，链接失败；端口打开的情况下，链接成功，则进入telnet页面（全黑的），证明端口可用。
（1）telnet IP 端口。
（2）telnet 域名 端口。
如果成功连接会进入的界面
连接失败
添加端口80、8080、3306、3690端口：
esc :wq! 退出保存修改。
注意：添加在端口22上面或者下面，不要放在最后，不然不起作用。
4.重启防火墙使配置生效
systemctl restart iptables.service
刚刚yum install iptables.service之后系统如果没有重启，iptables.service是找不到的，会报unit not fount。耽误时间的小坑！
设置防火墙开机启动:
systemctl enable iptables.service
~~~