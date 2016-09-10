---
title: Centos 6.6 mini amd64 VirtualBox5.0.8
date: 2016-09-09 18:00
---


### 0x01 网络配置
	
	虚拟机：桥接网络
	
	1. 配置/etc/sysconfig/network-script/ifcfg-eth0
		ONBOOT=yes
		BOOTPROTO=static
		IPADDR=192.168.1.111(因为通过无线路由器连接的)
		NETMASK=255.255.255.0
		GATEWAY=192.168.1.1
		
	2. 配置/etc/resolv.conf
		nameserver 8.8.8.8
		
	3. /etc/init.d/network restart
	
### 0x02 安装Apache

	1. yum install httpd
	2. /etc/init.d/httpd restart
		错误：Starting httpd: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain for ServerName
		解决方法：
			(1). vi /etc/httpd/conf/httpd.conf
			(2). 找到#ServerName www.example.com:80
			(3). 修改为ServerName localhost:80
	3. /etc/init.d/httpd restart
	4. chkconfig httpd on -- 开机启动httpd服务
	
### 0x03 安装Tomcat
	1. 安装java
		a. tar -xzvf jdk-1.7.0_80.tar.gz
		b. mv jdk1.7.0_80 /usr/local
		c. vim /etc/profile
			# 放在文件末尾
			PATH=/usr/local/jdk1.7.0_80/bin:$PATH
			export PATH
		d. source /etc/profile
		e. java -version
		
	2. 安装tomcat
		a. tar -xzvf apache-tomcat-x.x.x.tar.gz
		b. mv apache-tomcat-x tomcat
		c. mv tomcat /usr/local
		d. vim /usr/local/tomcat/bin/catalina.sh
			# 这些配置信息要放在文件开头第二行开始
			# chkconfig: 35 85 15 -- 这一行应该是开机启动服务
			# 寻找CLASSPATH=行
			CLASSPATH=/usr/local/jdk1.7.0_71/lib/tools.jar:/usr/local/jdk1.7.0_71/lib/dt.jar
			# 寻找CATALINE_HOME
			CATALINA_HOME=/usr/local/tomcat
			# 寻找JAVA_HOME
			JAVA_HOME=/usr/local/jdk1.7.0_71
			# 寻找JRE_HOME
			JRE_HOME=/usr/local/jdk1.7.0_71
		e. cp /usr/local/tomcat/bin/catalina.sh /etc/init.d/tomcat
		f. chmod +x /etc/init.d/tomcat
	
### 0x04 安装MySQL

	1. yum install mysql mysql-server
	
	2. /etc/init.d/mysqld start
	
	3. chkconfig mysqld on -- 开机启动mysqld服务
	
	4. cp /usr/share/mysql/my-medium.cnf /etc/my.cnf -- 若etc下存在，则覆盖
	
	5. /usr/bin/mysqladmin -u root password 'YourPassWord'
	
	6. /etc/init.d/mysqld restart
	
### 0x05 安装PHP
	1. yum install php
	
	2. yum install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt
	
	3. Centos6.6 版本，反正我安装的时候yum里面是找不到libmcrypt这个包，不要着急，等下会解决
	
### 0x06 配置phpMyAdmin(4.0.10.11)
	
> 错误：You don't have permission to access /phpMyAdmin on this server.
	
	解决方案：
		(1). vi /etc/selinux/config
		(2). 
			SELINUX=enforcing 加#注释
			SELINUXTYPE=targeted 加#注释
			SELINUX=disabled 增加这行
			
> 错误：缺少mcrypt扩展，请检查PHP配置
	
	解决方案：
		准备工作：
			a. 确定phpize和php-config
				whereis phpize 和 whereis php-config 可确定这2个包是否安装
			b. 若没有phpize，则 yum install php-devel
			c. 事先安装 yum install gcc，因为编译需要c编译器
		(1). http://cn.php.net/releases/ 网页下找到自己服务器的php版本，下载后tar解压（本人的是php5.3.3）
		(2). tar -zxvf php-5.3.3.tar.gz
		(3). cd php-5.3.3/ext/mcrypt/
		(4). phpize --> #运行这一条生成configure
		(5). ./configure --with-php-config=/usr/bin/php-config
		(6). make && make install
			若输出：Installing shared extensions:     /usr/lib64/php/modules/  --- 哒哒哒~安装成功
		(7). cd /etc/php.d
		(8). echo 'extension=mcrypt.so' > mcrypt.ini
		(9). 重启apache
	
> 错误：缺少短语密码，高级功能未开启
	
	解决方案：
		1. cd /var/www/html/phpMyAdmin/examples
		2. mysql -u root -p < create_tables.sql
		3. cp config.sample.inc.php config.inc.php
		4. vim config.inc.php 删除pma的注释，前面有3行左右可以保留注释
		5. 注销phpMyAdmin重新登陆

### 0x07 个人站点

	1. 文件配置：
		1. vi /etc/httpd/conf/httpd.conf
		2. 注释UserDir disable
		3. 取消注释UserDir public_html
		4. 取消注释<Directory /home/*/public_html> ----- </Directory>之间
	2. 网站搭建：
		1. cd ~
		2. mkdir public_html(这个是配置文件里表明的网站主目录，没改过，感觉可以改) -- (后来发现可以改)
		3. chmod 705 -R /home/$USERNAME
	
### 0x08 信息

	1. php
		PHP 5.3.3 (cli) (built: Jul  9 2015 17:39:00) 
		Copyright (c) 1997-2010 The PHP Group
		Zend Engine v2.3.0, Copyright (c) 1998-2010 Zend Technologies
	2. mysql
		mysql  Ver 14.14 Distrib 5.1.73, for redhat-linux-gnu (x86_64) using readline 5.1
