链接：https://pan.baidu.com/s/1u-A2cgHXTzuoOTrldH2Egw 提取码：ylx3  

一、LNMP软件所需要的软件包  
```
MySQL=http://dev.mysql.com/downloads/mysql/                mysql主程序包
PHP=http://php.net/downloads.php                           php主程序包
Nginx=http://nginx.org/en/download.html                    Nginx主程序包
libmcrypt=http://mcrypt.hellug.gr/index.html               libmcrypt加密算法扩展库，支持3DES等加密
或者：http://mcrypt.sourceforge.net/                        MCrypt/Libmcrypt development site (secure access)
pcre=http://pcre.org/                                      pcre是php的依赖包
```  

二、软件版本  
libmcrypt-2.5.8  
mysql-5.6.26  
nginx-1.8.0  
pcre-8.37  
php-5.6.13  

旧版本下载：http://mirrors.sohu.com/  

三、编译安装Nginx  

1、安装依赖  
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum groupinstall "Development Tools" "Development Libraries" -y
yum install gcc gcc-c++ autoconf automake zlib zlib-devel openssl openssl-devel pcre* pcre-devel  -y
```  

2、解压编译安装  
```
# tar xf pcre-8.37.tar.bz2 -C /usr/local/src/
# tar xvf nginx-1.8.0.tar.gz -C /usr/local/src/
# cd /usr/local/src/nginx-1.8.0
# ./configure --prefix=/usr/local/nginx --with-http_dav_module --with-http_stub_status_module --with-http_addition_module --with-http_sub_module --with-http_flv_module --with-http_mp4_module --with-pcre=/usr/local/src/pcre-8.37
# make –j 3 ; make install
# useradd -M -u 8001 -s /sbin/nologin nginx 
# ll /usr/local/nginx/
total 4
drwxr-xr-x. 2 root root 4096 Jul 30 21:44 conf   #Nginx相关配置文件
drwxr-xr-x. 2 root root   40 Jul 30 21:44 html   #网站根目录
drwxr-xr-x. 2 root root    6 Jul 30 21:44 logs   #日志文件
drwxr-xr-x. 2 root root   19 Jul 30 21:44 sbin   #Nginx启动脚本
```  

3、配置Nginx支持php文件  
```

#user  nobody;
worker_processes  1;

user nginx nginx;
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index index.php index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    location ~ \.php$ {
        root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html$fastcgi_script_name;  
        include        fastcgi_params;
    }
}
}
```  

4、启动nginx  
```
# /usr/local/nginx/sbin/nginx
# netstat -tlnp | grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      20177/nginx: master
```  
使用浏览器测试  
http://192.168.101.70/  


四、编译安装Mysql  

1、安装依赖  
```
yum install -y cmake ncurses-devel
```  

2、编译安装mysql  
```
# tar xf mysql-5.6.26.tar.gz -C /usr/local/src/ 
# cd /usr/local/src/mysql-5.6.26
# useradd -M -s /sbin/nologin mysql
# cmake  -DCMAKE_INSTALL_PREFIX=/usr/local/mysql  -DMYSQL_UNIX_ADDR=/tmp/mysql.sock  -DDEFAULT_CHARSET=utf8  -DDEFAULT_COLLATION=utf8_general_ci  -DWITH_EXTRA_CHARSETS=all  -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DENABLED_LOCAL_INFILE=1 -DMYSQL_DATADIR=/usr/local/mysql/data  -DMYSQL-USER=mysql
# make -j 2 ; make install
```

3、配置mysql  
```
配置属主属组
# chown -R mysql:mysql /usr/local/mysql/
拷贝配置文件
# cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
拷贝启动脚本
# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld

更改启动脚本中指定mysql位置
vim /etc/init.d/mysqld
basedir=
datadir=
#修改为
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data

开机启动
# chkconfig mysqld  on
```  

4、初始化数据库  
```
# /usr/local/mysql/scripts/mysql_install_db \
--defaults-file=/etc/my.cnf  \
--basedir=/usr/local/mysql/ \
--datadir=/usr/local/mysql/data/ \
--user=mysql
```  

5、命令软连接  
```
# ln -s /usr/local/mysql/bin/* /bin/
```  

6、启动  
```
# service mysqld  start
# netstat -tanlp |grep 3306
tcp6       0      0 :::3306                 :::*                    LISTEN      35090/mysqld
```  


7、可直接执行编译安装mysql脚本  
```
#!/bin/bash
yum remove  -y mysql mysql-server
clear
echo 'This shell will Auto Install Mysql5.6'
yum install -y cmake ncurses-devel
tar -xf mysql-5.6.26.tar.gz  -C  /usr/local/src && cd /usr/local/src/mysql-5.6.26
useradd -M -s /sbin/nologin mysql
mkdir /usr/local/mysql
cmake \
 -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
 -DDEFAULT_CHARSET=utf8 \
 -DDEFAULT_COLLATION=utf8_general_ci \
 -DWITH_EXTRA_CHARSETS=all \
 -DWITH_MYISAM_STORAGE_ENGINE=1\
 -DWITH_INNOBASE_STORAGE_ENGINE=1\
 -DWITH_MEMORY_STORAGE_ENGINE=1\
 -DWITH_READLINE=1\
 -DENABLED_LOCAL_INFILE=1\
 -DMYSQL_DATADIR=/usr/local/mysql/data \
 -DMYSQL-USER=mysql
make -j 3 && make  install
chown -R mysql:mysql  /usr/local/mysql
/usr/local/mysql/scripts/mysql_install_db --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
mv /etc/my.cnf  /etc/my.cnf.bak
cp -r /usr/local/mysql/support-file/my-default.cnf  /etc/my.cnf
sed -i '/^\[mysqld\]/adatadir = /usr/local/mysql/data' /etc/my.cnf
sed -i '/^\[mysqld\]/abasedir = /usr/local/mysql' /etc/my.cnf
cp -r /usr/local/mysql/support-files/mysql.server  /etc/init.d/mysqld
chmod  +x /etc/init.d/mysqld
echo "PATH=/usr/local/mysql/bin:$PATH" >>/etc/profile
service mysqld restart
echo
echo "install success"
source /etc/profile
echo "source /etc/profile" >>/etc/rc.local
service mysqld restart
echo "If you now running mysql and others commands,Please running: source /etc/profile"
```  

五、编译安装PHP  
http://php-fpm.org/download  

1、安装依赖  
```
# yum install php-pear -y 
```  

2、PHP添加libmcrypt拓展  
```
# tar xf libmcrypt-2.5.8.tar.bz2 -C /usr/local/src/
# cd /usr/local/src/libmcrypt-2.5.8/
# ./configure --prefix=/usr/local/libmcrypt
# make && make install
```  

3、除开上面的依赖解决之外，还需要安装图片，xml，字体支持基本库  
```
# yum install -y libxml2-devel libcurl-devel libjpeg-devel libpng-devel freetype freetype-devel libzip libzip-devel
```  

4、需要添加到库文件路径  
由于系统默认规定只在/lib、/lib64、/lib/lib64下面找库文件，所以我们需要手动添加进去。  
```
# vim /etc/ld.so.conf
include ld.so.conf.d/*.conf                    #此行原有
/usr/local/libmcrypt/lib                       #此行添加
/usr/local/mysql/lib                           #此行添加

# ldconfig
# echo 'ldconfig' >> /etc/rc.local
```  

5、编译安装php  
```
# tar xf php-5.6.13.tar.bz2 -C /usr/local/src/
# cd /usr/local/src/php-5.6.13
# ./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex  --enable-fpm --enable-mbstring --with-gd --enable-mysqlnd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-soap --with-gettext --with-mcrypt=/usr/local/libmcrypt
# make -j 3 && make install
```  

6、配置php和php-fpm  
```
拷贝PHP配置文件
# cp /usr/local/src/php-5.6.13/php.ini-production /usr/local/php/php.ini

PHP-FPM配置文件
# cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf

修改 /usr/local/php/etc/php-fpm.conf 运行用户和组改为nginx
# vim /usr/local/php/etc/php-fpm.conf
user = nginx
group = nginx

拷贝PHP-FPM启动脚本
# cp /usr/local/src/php-5.6.13/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
# chmod +x /etc/init.d/php-fpm

启动php
# /etc/init.d/php-fpm start

查看是否启动
# netstat -tnalp |grep 9000
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      56186/php-fpm: mast 

# ps aux |grep php
root      56186  0.0  0.3 218532  4964 ?        Ss   23:04   0:00 php-fpm: master process (/usr/local/php/etcphp-fpm.conf)
nginx     56187  0.0  0.2 218532  4504 ?        S    23:04   0:00 php-fpm: pool www
nginx     56188  0.0  0.2 218532  4504 ?        S    23:04   0:00 php-fpm: pool www
```  

7、测试LNMP的PHP支持  
```
# echo "<?php phpinfo(); ?>" > /usr/local/nginx/html/index.php
```  

浏览器访问  
http://192.168.101.70/  
