[TOC]
***
> 搭建方式：编译安装、一键安装包、yum安装  
> 常用的一键安装包：  
> https://lamp.sh/  
> https://lnmp.org/  
> https://oneinstack.com/  

***
查看已经安装的CentOS版本信息:  
> cat /etc/issue 查看版本

![image](https://note.youdao.com/yws/res/21051/A23E6569CE594D1C8EA51A1625EA52C5)  

# 一、LAMP安装
CentOS 6.8安装配置LAMP服务器(Apache+PHP5+MySQL)  
默认的yum对应的版本：  
1. 查看Apache版本：在终端下执行             httpd -v     （CentOS6.8 yum默认安装Apache/2.2.15，三方yum：安装Apache/2.2.15）
2. 查看MySQL版本：在终端下执行             mysql -V    （CentOS6.8 yum默认安装MySQL/5.1.73，三方yum：安装MySQL/5.5.55）
3. 查看PHP版本：在终端下执行                  php -v        （CentOS6.8 yum默认安装PHP/5.3.3，三方yum：安装PHP/5.4.45）
4. 查看PHP已安装的扩展：在终端下执行    php -m
参考网址：http://www.osyunwei.com/archives/8415.html

## 1.1 添加第三方yum源
CentOS默认yum源软件版本太低了，要安装最新版本的LAMP，这里使用第三方yum源
```
yum install -y lrzsz        #安装sz下载和rz工具
yum install -y unzip zip    #安装zip打包工具

wget http://www.atomicorp.com/installers/atomic     #下载，首先使用默认yum源安装wget命令 yum install wget
    注意：（如果这步有误 可直接执行yum install gcc gcc-c++ gcc-g77 flex bison autoconf automake bzip2-devel  zlib-     devel ncurses-devel libjpeg-devel libpng-devel libtiff-devel freetype-devel pam-devel openssl-devel libxml2-devel  gettext-devel  pcre-devel mysql-devel net-snmp-devel curl-devel perl-DBI
yum -y install make gcc g++ gcc-c++ libtool autoconf automake imake mysql-devel libxml2-devel expat-devel php-devel
yum install make autoconf libtool
）
sh ./atomic      #安装
yum clean all    #清除当前yum缓存
yum makecache    #缓存yum源中的软件包信息
yum repolist     #列出yum源中可用的软件包
```
## 1.2 安装Apache
```
yum -y install httpd        #根据提示，输入Y安装即可成功安装
/etc/init.d/httpd start     #启动Apache
    备注：（Apache启动之后会提示错误：
    httpd:httpd: Could not reliably determine the server's fully qualif domain name, using ::1 for ServerName）
    解决办法：
    vi /etc/httpd/conf/httpd.conf        #编辑
    ServerName www.example.com:80        #去掉前面的注释
    :wq!                                 #保存退出
chkconfig httpd on                   #设为开机启动
/etc/init.d/httpd restart            #重启Apache
```
## 1.3 安装Mysql
**1 安装MySQL** 
```
yum -y install mysql mysql-server      #询问是否要安装，输入Y即可自动安装,直到安装完成
/etc/init.d/mysqld start               #启动MySQL
chkconfig mysqld on                    #设为开机启动

cp /usr/share/mysql/my-huge.cnf /etc/my.cnf  #这个是大型数据库（通常选用此配置）,根据提示输入“y”回车即可
或者：
cp /usr/share/mysql/my-medium.cnf /etc/my.cnf      #这个是小型数据库（注意：如果/etc目录下面默认有一个my.cnf，直接覆盖即可）
```
**2 为root账户设置密码**  
```
mysql_secure_installation    #输入命令回车后，再一次回车，再根据提示输入Y,输入2次密码，回车,根据提示一路输入Y,最后出现：Thanks for using MySQL!
    备注：按照上面的操作，设置Mysql的root密码可以很简单。因为这里面不允许root账号远程访问。用户只能通过授权的用户账号才能远程访问Mysql数据库。
    说明：
    Remove anonymous users：删除匿名用户
    Disallow root login remotely：不允许root远程登录
    Remove test database and access to it：删除测试数据库并访问它
    Reload privilege tables now：现在重新加载权限表（也可以登录mysql后使用：flush privileges #重新加载权限表）
MySql密码设置完成，重新启动 MySQL：
/etc/init.d/mysqld restart     #重启
或者：
/etc/init.d/mysqld stop        #停止
/etc/init.d/mysqld start       #启动
```
## 1.4 安装PHP
**1 安装PHP**  
```
yum -y install php     #根据提示输入Y直到安装完成
```
**2 安装PHP组件，使PHP支持MySQL**
```
yum -y install php-mysql php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt
这里选择以上安装包进行安装
根据提示输入Y回车
/etc/init.d/mysqld restart     #重启MySql
/etc/init.d/httpd restart        #重启Apche
```
**说明**：
> 如果想要使用PHP5.3那么就默认系统中的yum源安装，如果想要PHP5.4那么就需要添加第三方yum源，按照1.1中的操作添加。  
如果想要自定义PHP版本，那么就需要如下操作：  
yum list php*    查看是否有自己需要安装的版本，如果没有就需要添加第三方yum源， 推荐安装webtatic、rpmforge，还有国内163的。

下面以webtatic举例：  
```
1.检查当前安装的PHP包
    yum list installed | grep php
    如果有安装的PHP包，先删除他们, 如:
    yum remove php.x86_64 php-cli.x86_64 php-common.x86_64
    完全移除当前PHP安装包以免起冲突
    yum remove php*
2.配置安装包源，安装Webtatic的yum源:
    # Centos 5.X
    rpm -Uvh http://mirror.webtatic.com/yum/el5/latest.rpm
    # CentOs 6.x（包括php5.5,5.6,7.0,7.1）
    rpm -Uvh http://mirror.webtatic.com/yum/el6/latest.rpm
    # CentOs 7.X（需要执行下面两行命令）
    rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
    rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
    如果想删除上面安装的包，重新安装
    rpm -qa | grep webstatic
    rpm -e  [上面搜索到的包即可]
    安装完成后可以使用yum repolist查看已经安装的源，也可以通过ls /etc/yum.repos.d/查看。 
3.执行安装php和相关插件
    安装5.6（已测试，没问题）
    yum -y install php56w.x86_64
    yum -y --enablerepo=webtatic install php56w-devel
    yum -y install php56w-gd.x86_64 php56w-ldap.x86_64 php56w-mbstring.x86_64 php56w-mcrypt.x86_64 php56w-mysql.x86_64 php56w-pdo.x86_64 php56w-opcache.x86_64
    安装php7.0
    yum install -y php70w.x86_64 php70w-cli.x86_64 php70w-common.x86_64 php70w-gd.x86_64 php70w-ldap.x86_64 php70w-mbstring.x86_64 php70w-mcrypt.x86_64 php70w-mysql.x86_64 php70w-pdo.x86_64 php70w-fpm
    安装php7.1
    yum install -y php71w-fpm php71w-opcache php71w-cli php71w-gd php71w-imap php71w-mysqlnd php71w-mbstring php71w-mcrypt php71w-pdo php71w-pecl-apcu php71w-pecl-mongodb php71w-pecl-redis php71w-pgsql php71w-xml php71w-xmlrpc php71w-devel mod_php71w

4.安装PHP FPM（LNMP环境中的nginx是不支持php的，需要通过fastcgi插件来处理有关php的请求。而php需要php-fpm这个组件提供该功能。在php5.3.3以前的版本php-fpm是以一个补丁包的形式存在的，而php5.3.3以后只需在编译安装时使用–enable-fpm加载该模块即可，无需另行安装。）
    yum -y install php56w-fpm
    #设置php-fpm开机启动（fpm是置于PHP和Nginx之间的一层应用，所以配置成服务开机自启。）
    chkconfig php-fpm on
    #启动php-fpm
    /etc/init.d/php-fpm start
注：如果想更换到php5.5版本, 直接把上面的56w换成55w就可以了
```
***
# 二、LAMP配置
**注意**：通常只修改加粗标注部分即可。  
## 2.1 Apache配置
```
vi /etc/httpd/conf/httpd.conf #编辑文件
```
> **ServerTokens OS**　  修改为：ServerTokens Prod （在出现错误页的时候不显示服务器操作系统的名称）   
> **ServerSignature On**　  修改为：ServerSignature Off （在错误页中不显示Apache的版本）  
> Options Indexes FollowSymLinks　  修改为：Options Includes ExecCGI 
> FollowSymLinks（允许服务器执行CGI及SSI，禁止列出目录）  
> #AddHandler cgi-script .cgi　 修改为：AddHandler cgi-script .cgi .pl （允许扩展名为.pl的CGI脚本运行）   
> AllowOverride None　  修改为：AllowOverride All （允许.htaccess）  
> AddDefaultCharset UTF-8　 修改为：AddDefaultCharset   
> GB2312　（添加GB2312为默认编码）  
> Options Indexes MultiViews FollowSymLinks  修改为 Options MultiViews  
> FollowSymLinks（不在浏览器上显示树状目录结构）  
> DirectoryIndex index.html index.html.var  修改为：DirectoryIndex   
> index.html index.htm Default.html Default.htm  
> index.php Default.php index.html.var （设置默认首页文件，增加index.php）  
> **KeepAlive Off**  修改为：KeepAlive On （允许程序性联机）  
> **MaxKeepAliveRequests 100**  修改为：MaxKeepAliveRequests 1000 （增加同时连接数）  

```
:wq! #保存退出
/etc/init.d/httpd restart #重启
rm -f /etc/httpd/conf.d/welcome.conf /var/www/error/noindex.html #删除默认测试页
```
## 2.2 PHP配置
```
vi /etc/php.ini #编辑
```
> **date.timezone =** # 把前面的分号去掉，改为date.timezone = PRC  
> disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd, posix_getegid,posix_geteuid,posix_getgid, posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid, posix_getppid,posix_getpwnam,posix_getpwuid, posix_getrlimit, posix_getsid,posix_getuid,posix_isatty, posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid, posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname # 列出PHP可以禁用的函数，如果某些程序需要用到这个函数，可以删除，取消禁用。    
> **expose_php = Off** #在 禁止显示php版本的信息  
magic_quotes_gpc = On # 打开magic_quotes_gpc来防止SQL注入（php5.4开始没有这项）    
> short_open_tag = ON #支持php短标签  
> open_basedir = .:/tmp/ # 设置表示允许访问当前目录(即PHP脚本文件所在之目录)和/tmp/目录,可以防止php木马跨站,如果改了之后安装程序有问题(例如：织梦内容管理系统)，可以注销此行，或者直接写上程序的目录/data/www.osyunwei.com/:/tmp/    --->可以不设置！

```
:wq! #保存退出
/etc/init.d/mysqld restart #重启MySql
/etc/init.d/httpd restart #重启Apche
```
## 2.3 测试
```
cd /var/www/html
vi index.php #输入下面内容
<?php
phpinfo();
?>
:wq! #保存退出
```
在客户端浏览器输入服务器IP地址，可以看到PHP相关的配置信息！
注意：阿里云服务器中要手动添加Apache的80端口，才能正常访问。
## 2.4 配置虚拟主机
```
vi /etc/httpd/conf/httpd.conf
shift + g 定位到文件末尾，在末尾加上：
Include /etc/httpd/conf/vhost.conf

编辑虚拟主机文件，添加下面的内容：
vi /etc/httpd/conf/vhost.conf
<VirtualHost *:80>
        ServerName www.cdliya.cn
        ServerAlias  cdliya.cn
        DocumentRoot /var/web
        <Directory />
          Options -indexes
          DirectoryIndex index.php index.html
          AllowOverride All
          Order allow,deny
          Allow from all
        </Directory>
        ErrorLog logs/cdliya.cn-error_log
        CustomLog logs/cdliya.cn-access_log common
</VirtualHost>
```
配置完后，要修改项目文件夹的权限才能正常访问。
```
chmod 777 -R /var/web #可读可写可执行（默认这样配置，也可以根据权限组来针对性的限制操作权限）
```
然后重启Apache服务，既可以正常访问。  
**注意**：  
> apache默认的程序目录是/var/www/html  
> apache配置文件:/etc/httpd/conf/httpd.conf  

权限设置：chown apache.apache -R /var/www/html  
## 2.5 Apache的优化
查看httpd的工作模式，通过apachectl -l 查看：  
![image](https://note.youdao.com/yws/res/9265/C8E0F15E23B04851968C2BD19DF61B9D)  
```
vi /etc/httpd/conf/httpd.conf  
```
![image](https://note.youdao.com/yws/res/9269/69D5BE17EEC14CF18212E0077338C516)  

MaxKeepAliveRequests 100  修改为：MaxKeepAliveRequests 1000 （增加同时连接数）以及其他配置如下:  
![image](https://note.youdao.com/yws/res/9275/91568A73E2454746A49D564E2EC5A2C2)   
设置项目目录：   
先在/var目录下面建立一个项目文件夹“wwwroot”或者“web”，然后修改Apache配置文件中项目目录  
```
vi /etc/httpd/conf/httpd.conf
```  
这两行，把其中的路径改为你自定义的网站目录，需要修改的这两行相隔不远，如：  
![image](https://note.youdao.com/yws/res/9283/37833B7FA77A410DAAACD27DBA65D1C5)  
![image](https://note.youdao.com/yws/res/9286/B2B0D82080EA4F339A8A44715F2EEEA2)  
重启apache，仅仅/bin/apachectl restart无效，需要先停止，再启动。  
执行：`service httpd restart `或者
`/etc/init.d/httpd stop`和
`/etc/init.d/httpd start`  
## 2.6 PHP的优化
```
vi /etc/php.ini
```
修改php的内存：  
![image](https://note.youdao.com/yws/res/9296/A9081E0E5A514F1EB998FAB499A6DE15)  
修改php的上传文件的大小：  
![image](https://note.youdao.com/yws/res/9300/AAE5A3A110D545D8891C603609D71B0B)  
![image](https://note.youdao.com/yws/res/9302/7EB6DCA88F3D4CA78E04276FED434B3C)  
![image](https://note.youdao.com/yws/res/9304/E0D8093B94CB483C975930CB6581573D)  
**注意**：  
> post_max_size 大于 upload_max_filesize为佳  

## 2.7 gzip模块开启（可以不用安装）
打开http.conf文件,首先开启gzip模块（地址：/etc/httpd/conf/httpd.conf）  
`LoadModule deflate_module modules/mod_deflate.so    （默认是开启的）`  
然后,配置何时使用gzip压缩，在http.conf末尾添加如下代码：  
```
<IfModule mod_deflate.c>
# 压缩等级 6
DeflateCompressionLevel 6
# 压缩类型 html、xml、php、css、js
SetOutputFilter DEFLATE
AddOutputFilterByType DEFLATE text/html text/plain text/xml application/x-javascript application/x-httpd-php
AddOutputFilter DEFLATE js css
</IfModule>
```
最后，重启apache进行测试。  
## 2.8 mysql优化
如果配置mysql时选择的：cp /usr/share/mysql/my-huge.cnf /etc/my.cnf  #这个是大型数据库（通常选用此配置）  
那么，只需要优化两个地方：  
`vi /etc/my.cnf`  
注释掉这行，不允许生成垃圾日志。 （有两处）   
![image](https://note.youdao.com/yws/res/9317/A45E23D792B242839F003C57478928A7)  
在对应位置加一行，允许最大连接量：  
`max_connections = 5000`  
![image](https://note.youdao.com/yws/res/9321/023EC298DA9C4806B011E6C652F39B71)  
```
/etc/init.d/mysqld restart    #重启MySql  
/etc/init.d/httpd restart        #重启Apche  
```
然后输入账号、密码登录mysql数据库：  
```mysql
mysql -uroot -ppassword
```
给数据库授权用户用于远程访问数据库，如下（创建数据库、授权、重载授权表）：  
```mysql
create database MyProject default character set utf8 collate utf8_general_ci;
grant all on MyProject .* to username@'localhost' identified by 'userpsd';
grant all on MyProject .* to username@'127.0.0.1' identified by 'userpsd';
grant all on MyProject .* to username@'%' identified by 'userpsd';
flush privileges;
use MyProject ; 
```
之后，如果阿里云服务器和防火墙3306端口开启正常，那么就可以远程工具操作数据库了。  
**注意**：  
> 账号和密码不能太简单，因为这个是用来远程访问数据库的。  

## 2.9 系统优化
针对于centos系统的优化，一条总的语句：  
```
echo 5000 > /proc/sys/net/core/somaxconn
echo 1 >  /proc/sys/net/ipv4/tcp_tw_recycle
echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse 
echo 0 > /proc/sys/net/ipv4/tcp_syncookies
ulimit -n 65535
```
# 三、配置防火墙
`vi /etc/sysconfig/iptables #注：也可以通过setup命令通过命令的方法配置防火墙`  
在编辑模式下，复制以下代码覆盖原文件：  
```
# Firewall configuration written by system-config-firewall
# Manual customization of this file is not recommended.
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```
`/etc/init.d/iptables restart  #重启防火墙使配置生效（如果失败，可能是上面代码格式问题）`  
关闭SELINUX  
`vi /etc/selinux/config`  
```
#SELINUX=enforcing #注释掉    （这句没找到）
#SELINUXTYPE=targeted #注释掉
SELINUX=disabled #增加

:wq! #保存退出
```
**注意**：  
> 如果没有权限，防火墙问题，网站还是不能访问。就：setenforce 0    阿里云没有这个情况。买的实体机会有这个问题    

# 四、卸载PHP
查看php版本命令：#php -v  
这个命令是删除不干净的：#yum remove php  
因为使用这个命令以后再用：#php -v    还是会看到有版本信息的
必须强制删除，首先查看机器上安装的所有php相关的rpm包：#rpm -qa|grep php  
提示如下  
```
#php-pdo-5.1.6-27.el5_5.3
#php-mysql-5.1.6-27.el5_5.3
#php-xml-5.1.6-27.el5_5.3
#php-cli-5.1.6-27.el5_5.3
#php-common-5.1.6-27.el5_5.3
#php-gd-5.1.6-27.el5_5.3
```
注意卸载要先卸载没有依赖的，pdo是mysql的依赖项；common是gd的依赖项；  
例如：
```
# rpm -e php-pdo-5.1.6-27.el5_5.3
error: Failed dependencies:
php-pdo is needed by (installed) 
php-mysql-5.1.6-27.el5_5.3.i386
```
按依赖顺序进行删除，所以正确的卸载顺序是：
```    
# rpm -e php-mysql-5.1.6-27.el5_5.3 
# rpm -e php-pdo-5.1.6-27.el5_5.3 
# rpm -e php-xml-5.1.6-27.el5_5.3 
# rpm -e php-cli-5.1.6-27.el5_5.3 
# rpm -e php-gd-5.1.6-27.el5_5.3 
# rpm -e php-common-5.1.6-27.el5_5.3 
```
再用# php -v查看版本信息已经没有提示。





