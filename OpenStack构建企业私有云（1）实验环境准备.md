# OpenStack构建企业私有云（1）实验环境准备
> https://docs.openstack.org/install-guide/
> https://www.unixhot.com/article/407

基于OpenStack构建企业私有云，能够帮助大家快速的部署两个节点的OpenStack集群。   请严格按照实验环境执行，如果你是新手，不按环境来，几乎不会成功，因为OpenStack依赖的组件太多了。   任何一个小错误都可能导致最终无法创建虚拟机。      
## 基础软件包安装 
### 安装EPEL仓库
	[root@linux-study1 ~]# yum install -y https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm
### 安装时间网络协议
#### 控制节点安装chrony
	[root@linux-study1 ~]# yum install -y chrony
#### 编辑/etc/chrony.conf
	[root@linux-study1 ~]# vim /etc/chrony.conf 
	# Allow NTP client access from local network.
	allow 192.168.1.0/24
#### 启动NTP并设置开机启动
	[root@linux-study1 ~]# systemctl enable chronyd
	[root@linux-study1 ~]# systemctl start chronyd 
#### 其他节点服务器
	[root@linux-study2 ~]# yum install -y chrony
#### 编辑/etc/chrony.conf
	[root@linux-study2 ~]# vim /etc/chrony.conf 
	# Use public servers from the pool.ntp.org project.
	# Please consider joining the pool (http://www.pool.ntp.org/join.html).
	server linux-study1 iburst
#### 启动NTP服务并设置开机启动
	[root@linux-study2 ~]# systemctl enable chronyd
	[root@linux-study2 ~]# systemctl start chronyd
#### 验证控制节点（linux-study1)NTP服务
	[root@linux-study1 ~]# chronyc sources
	210 Number of sources = 4
	MS Name/IP address         Stratum Poll Reach LastRx Last sample               
	===============================================================================
	^? ntp.wdc1.us.leaseweb.net      0   8     0     -     +0ns[   +0ns] +/-    0ns
	^? ntp8.flashdance.cx            2   7   120   338   -2621h[ -2621h] +/-  184ms
	^? xk-6-95-a8.bta.net.cn         2   7   200   728    -17ms[  -22ms] +/-   59ms
	^? stratum2-1.ntp.sea03.us.>     2   7     0   727  +7857us[+2918us] +/-  163ms
#### 验证其它所有节点（linux-study2等)NTP服务
	[root@linux-study2 ~]# chronyc sources          
	210 Number of sources = 1
	MS Name/IP address         Stratum Poll Reach LastRx Last sample               
	===============================================================================
	^? linux-study1                  0   6     0     -     +0ns[   +0ns] +/-    0ns
### 安装OpenStack仓库
	[root@linux-study1 ~]# yum install -y centos-release-openstack-queens
### 安装OpenStack客户端
	[root@linux-study1 ~]# yum install -y python-openstackclient
### 安装OpenStack SELinux管理包
	[root@linux-study1 ~]# yum install -y openstack-selinux
## MySQL数据库部署
### MySQ安装
	[root@linux-study1 ~]#  yum install -y mariadb mariadb-server python2-PyMySQL
### 修改MySQL配置文件
	[root@linux-study1 ~]# vim /etc/my.cnf.d/openstack.cnf
	[mysqld]
	bind-address = 192.168.1.11 #设置监听的IP地址
	default-storage-engine = innodb #设置默认的存储引擎
	innodb_file_per_table	# 使用独享表空间
	max_connections = 4096 # 设置MySQL的最大连接数，生产青根据实际情况设置
	collation-server = utf8_general_ci # 服务器的默认校验规则
	character-set-server = utf8 #服务器安装时指定的默认字符集设定
### 启动MySQL Server并设置开机启动
	[root@linux-study1 ~]# systemctl enable mariadb
	[root@linux-study1 ~]# systemctl start mariadb
### 进行数据库安全设置
	[root@linux-study1 ~]# 	mysql_secure_installation

### 数据库创建
	[root@linux-study1 ~]# mysql -u root -p
	MariaDB [(none)]> create database keystone;
	MariaDB [(none)]> grant all on keystone.* to 'keystone'@'localhost' identified by 'keystone';
	MariaDB [(none)]> grant all on keystone.* to 'keystone'@'%' identified by 'keystone';
	
	MariaDB [(none)]> create database glance; 
	MariaDB [(none)]> grant all on glance.* to 'glance'@'localhost' identified by 'glance';
	MariaDB [(none)]> grant all on glance.* to 'glance'@'%' identified by 'glance';

	MariaDB [(none)]> create database glance; 
	MariaDB [(none)]> grant all on glance.* to 'glance'@'localhost' identified by 'glance';
	MariaDB [(none)]> grant all on glance.* to 'glance'@'%' identified by 'glance';

	MariaDB [(none)]> create databse nova;
	MariaDB [(none)]> grant all on nova.* to 'nova'@'localhost' indentified by 'nova';
	MariaDB [(none)]> grant all on nova.* to 'nova'@'%' indentified by 'nova';
	
	MariaDB [(none)]> create databse nova_api;
	MariaDB [(none)]> grant all on nova.* to 'nova'@'localhost' indentified by 'nova';
	MariaDB [(none)]> grant all on nova.* to 'nova'@'%' indentified by 'nova';

	MariaDB [(none)]> create database neutron;  
	MariaDB [(none)]> grant all on neutron.* to 'neutron'@'localhost' identified by 'neutron';
	MariaDB [(none)]> grant all on neutron.* to 'neutron'@'%' identified by 'neutron';
## 消息代理RabbitMQ
### 安装RabbitMQ
	[root@linux-study1 ~]# yum install -y rabbitmq-server 
### 启动RabbitMQ并设置开机启动

	[root@linux-study1 ~]# systemctl enable rabbitmq-server
	[root@linux-study1 ~]# systemctl start rabbitmq-server
### 添加openstack用户
	[root@linux-study1 ~]# rabbitmqctl add_user openstack openstack
	Creating user "openstack"
### 给openstack用户配置写和读权限
	[root@linux-study1 ~]# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
	Setting permissions for user "openstack" in vhost "/"
### 启用Web监控插件
	[root@linux-study1 ~]# rabbitmq-plugins list
	[root@linux-study1 ~]# rabbitmq-plugins enable rabbitmq_management
	The following plugins have been enabled:
	  amqp_client
	  cowlib
	  cowboy
	  rabbitmq_web_dispatch
	  rabbitmq_management_agent
	  rabbitmq_management
	
	Applying plugin configuration to rabbit@linux-study1... started 6 plugins.
### RabbitMQ相关知识
#### 端口号：5672
#### web端口号： 15672

### 安装Memcached软件包
	[root@linux-study1 ~]# yum install -y memcached
### 启动Memcached服务，并且设置开机启动
	[root@linux-study1 ~]# systemctl enable memcached
	[root@linux-study1 ~]# systemctl start memcached

