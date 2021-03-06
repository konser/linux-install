1、安装依赖
```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum groupinstall "Development Tools" "Development Libraries" -y
yum install gcc gcc-c++ openssl-devel
```  

2、安装apr和apr-util依赖
```
tar xf apr-1.5.2.tar.gz -C /usr/local/src/
tar xf apr-util-1.5.4.tar.bz2 -C /usr/local/src/
cd /usr/local/src/apr-1.5.2/
./configure --prefix=/usr/local/apr && make -j 2 && make install

cd /usr/local/src/apr-util-1.5.4/
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr && make -j 2 && make install
```  

3、Apache源码编译
```
tar xf httpd-2.4.27.tar.bz2 -C /usr/local/src/
cd /usr/local/src/httpd-2.4.27/
./configure --prefix=/usr/local/apache --sysconfdir=/etc/httpd --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-apr=/usr/local/apr --enable-deflate --with-apr-util=/usr/local/apr-util --enable-modules=most --enable-mpms-shared=all --with-mpm=event
make -j 2
make install
```  
注解：
```
--prefix=/usr/local/apache            #指定程序安装路径
--sysconfdir=/etc/httpd               #指定配置文件、或工作目录
--enable-so                           #开启基于DSO动态装载模块
--enable-ssl                          #开启支持ssl协议
--enable-cgi                          #开启cgi机制 
--enable-rewrite                      #开启支持URL重写
--with-zlib                           #zlib是网络上发送数据报文的通用压缩库的API，在apache调用压缩工具压缩发送数据时需要调用该库
--with-pcre                           #支持PCRE，把pcre包含进程序中，（此处没指定pcre程序所在路径，默认会在PATH环境下查找）
--with-apr=/usr/local/apr             #指定apr位置
--with-apr-util=/usr/local/apr-util   #指定apr-util
--enable-modeles=most                 #启动模块，all表示所有，most表示常用的
--enable-mpms-shared=all              #启动所有的MPM模块
--with-mpm=event                      #指定默认使用event模块
```  

4、查看配置文件：
```
# ls /etc/httpd/httpd.conf 
/etc/httpd/httpd.conf
```  

5、存放网站的根目录：  
```
# ls /usr/local/apache/htdocs/index.html 
/usr/local/apache/htdocs/index.html
```  

6、启动http  
1)配置apache可以开机启动并且可以使用systemctl命令启动apache服务器  
```
# vim /usr/lib/systemd/system/httpd.service
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)

[Service]
Type=forking
EnvironmentFile=/etc/httpd/httpd.conf
ExecStart=/usr/local/apache/bin/apachectl
ExecRestart=/usr/local/apache/bin/apachectl restart
ExecStop=/usr/local/apache/bin/apachectl stop
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```  

2)重新加载unit文件：
```
# systemctl daemon-reload
```  

3)设置开机自动启动：
```
# systemctl enable httpd
```  

4)启动apache：
```
# systemctl start httpd
```  

开始调优  
1、隐藏版本信息
```
# vim /etc/httpd/httpd.conf
# 将注释去掉
#Include /etc/httpd/extra/httpd-default.conf
Include /etc/httpd/extra/httpd-default.conf

# vim /etc/httpd/extra/httpd-default.conf
# 将Full改为On
ServerTokens Prod
ServerTokens On
如果这里不是off需要修改
ServerSignature Off

重启前测试
# curl -I 192.168.101.71
HTTP/1.1 200 OK
Date: Mon, 05 Aug 2019 15:46:43 GMT
Server: Apache/2.4.27 (Unix)    #有版本信息
Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
ETag: "2d-432a5e4a73a80"
Accept-Ranges: bytes
Content-Length: 45
Content-Type: text/html

# systemctl restart httpd

重启后测试
# curl -I 192.168.101.71
HTTP/1.1 200 OK
Date: Mon, 05 Aug 2019 15:46:51 GMT
Server: Apache                  #无版本信息
Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
ETag: "2d-432a5e4a73a80"
Accept-Ranges: bytes
Content-Length: 45
Content-Type: text/html
```  

2、彻底让版本等敏感信息消失  
```
1、编译前修改配置
# tar xf httpd-2.4.27.tar.bz2 -C /usr/local/src/
# cd /usr/local/src/httpd-2.4.27/
# vim include/ap_release.h
#define AP_SERVER_BASEVENDOR "Apache Software Foundation"   #服务的供应商名称
#define AP_SERVER_BASEPROJECT "Apache HTTP Server"          #服务的项目名称
#define AP_SERVER_BASEPRODUCT "Apache"                      #服务的产品名
#define AP_SERVER_MAJORVERSION_NUMBER 2                     #主要版本号
#define AP_SERVER_MINORVERSION_NUMBER 4                     #小版本号
#define AP_SERVER_PATCHLEVEL_NUMBER  6                      #补丁级别
#define AP_SERVER_DEVBUILD_BOOLEAN  0
修改为：
#define AP_SERVER_BASEVENDOR "web"
#define AP_SERVER_BASEPROJECT "web server"
#define AP_SERVER_BASEPRODUCT "web"

#define AP_SERVER_MAJORVERSION_NUMBER 8
#define AP_SERVER_MINORVERSION_NUMBER 1
#define AP_SERVER_PATCHLEVEL_NUMBER   2
#define AP_SERVER_DEVBUILD_BOOLEAN    3

2、修改完后需要重新编译安装
./configure --prefix=/usr/local/apache --sysconfdir=/etc/httpd --enable-so --enable-ssl --enable-cgi --enable-rewrite --with-zlib --with-pcre --with-apr=/usr/local/apr --enable-deflate --with-apr-util=/usr/local/apr-util --enable-modules=most --enable-mpms-shared=all --with-mpm=event
make -j 2
make install
```  

3、查看运行apache的默认用户  
通过更改apache的默认用户，可以提升apache的安全性。这样，即使apache服务被攻破，黑客拿到apache普通用户也不会对系统和其他应用造成破坏。  
```
# lsof -i:80
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd   60596   root    4u  IPv6  50114      0t0  TCP *:http (LISTEN)
httpd   60597 daemon    4u  IPv6  50114      0t0  TCP *:http (LISTEN)
httpd   60598 daemon    4u  IPv6  50114      0t0  TCP *:http (LISTEN)
httpd   60599 daemon    4u  IPv6  50114      0t0  TCP *:http (LISTEN)

# id daemon
uid=2(daemon) gid=2(daemon) groups=2(daemon)

1、创建apache用户，没有家目录，非登录用户
# useradd -M -s /sbin/nologin apache

# vim /etc/httpd/httpd.conf
User apache
Group apache

重启查看默认用户
# systemctl restart httpd
# lsof -i:80
COMMAND   PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd   60869   root    4u  IPv6  53228      0t0  TCP *:http (LISTEN)
httpd   60870 apache    4u  IPv6  53228      0t0  TCP *:http (LISTEN)
httpd   60871 apache    4u  IPv6  53228      0t0  TCP *:http (LISTEN)
httpd   60872 apache    4u  IPv6  53228      0t0  TCP *:http (LISTEN)

2、修改apache的工作目录为普通用户权限
# ll -sd /usr/local/apache/htdocs/
0 drwxr-xr-x. 2 root root 24 Jul  6  2017 /usr/local/apache/htdocs/

# chown apache. /usr/local/apache/htdocs/ -R

# ll /usr/local/apache/htdocs/
total 4
-rw-r--r--. 1 apache apache 45 Jun 11  2007 index.html


3、保护apache日志：设置好apache日志文件权限，所以不用修改
# ll -sd /usr/local/apache/logs/*
4 -rw-r--r--. 1 root root 1414 Aug  5 12:12 /usr/local/apache/logs/access_log
4 -rw-r--r--. 1 root root 3180 Aug  5 22:29 /usr/local/apache/logs/error_log
4 -rw-r--r--. 1 root root    6 Aug  5 22:29 /usr/local/apache/logs/httpd.pid
注：由于apache日志的记录是由apache的主进程进行操作的，而apache的主进程又是root用户启动的，所以这样不影响日志的输出。这也是日志记录的最安全的方法。
```  

4、设置错误页面-开启压缩和缓存功能
```
1、错误页面优雅化显示的实现方式主要有两种,下面我们主要以404错误为例：
方法：
# vim /etc/httpd/httpd.conf
# 在http工作目录内添加
<Directory "/usr/local/apache/htdocs">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
    ErrorDocument 404 /404.html         #添加此行
</Directory>

追加404错误页面，跳转页面
# echo "404 go to home" > /usr/local/apache/htdocs/404.html
重启后访问web不存在页面测试
# systemctl restart httpd
# curl 192.168.101.71/123
404 go to home

总结：ErrorDocument的命令格式如下：
ErrorDocument 错误代码 跳转到的页面或文件
另外这里需要注意，你若设置跳转到文件，必须要有这个文件才行。另外文件必须在站点目录内，不然会报错。
在跳转到文件的测试中，用全路径和别名路径进行测试，当把404错误页面跳转文件放到其他目录的时候，不报错，但是页面跳转不过去。



2、启用压缩模块mod_deflate
apache的压缩要用到mod_deflate模块，该模块提供了DEFLATE输出过滤器，允许服务器在将输出内容发送到客户端以前进行压缩，以节约带宽。它的核心思想就是把文件先在服务器进行压缩，然后再进行传输，这样可以显著减少文件传输的大小。当传输完毕后，客户端浏览器会重新对压缩过的内容进行解压缩。如果没特殊情况的话，所有的文本内容都应该能被gzip压缩，例如：html（php），js，css，xml，txt等。

mod_deflate模块检查及安装
# /usr/local/apache/bin/apachectl -M | grep deflate


# /usr/local/apache/bin/apachectl -M 
Loaded Modules:
 core_module (static)
 so_module (static)
 http_module (static)             #(static)弹出此种结果，则为编译安装时装的
 authn_file_module (shared)
 authn_core_module (shared)
 authz_host_module (shared)       #(shared)弹出此种结果，则为DSO方式安装的
 authz_groupfile_module (shared)
 authz_user_module (shared)
 authz_core_module (shared)
 access_compat_module (shared)
 auth_basic_module (shared)
 reqtimeout_module (shared)
 filter_module (shared)
 mime_module (shared)
 log_config_module (shared)
 env_module (shared)
 headers_module (shared)
 setenvif_module (shared)
 version_module (shared)
 mpm_event_module (shared)
 unixd_module (shared)
 status_module (shared)
 autoindex_module (shared)
 dir_module (shared)
 alias_module (shared)
 
 
 如果安装了，就可以直接进行压缩配置了，如果没有安装，下面为安装方法
a）编译时安装方法
编译的时候跟上--enable-deflate即可实现安装
b）DSO方式安装。
扩展：DSO： Dynamic shared object动态共享对象 。DSO模块可以在编译服务器之后编译，也可以用Apache扩展工具(apxs)编译并增加
使用DSO方式安装，/usr/local/apache/bin/apxs后跟的参数详解
-c  此选项表明需要执行编译操作。
-i  此选项表示需要执行安装操作，以安装一个或多个动态共享对象到服务器的modules目录。
-a  此选项自动增加一个LoadModule行到httpd.conf文件中，以激活此模块，或者，如果此行已经存在，则启用之。

开启模块,取消注释
查看该路径下是否有压缩模块，有直接开启
ls /usr/local/src/httpd-2.4.27/modules/filters/ |grep mod_deflate
# vim /etc/httpd/httpd.conf 
#LoadModule deflate_module modules/mod_deflate.so
开启此项
Include /etc/httpd/extra/httpd-default.conf

开始配置压缩功能，在全局模式下配置
在Listen下配置，粘贴配置

<ifmodule mod_deflate.c>
   DeflateCompressionLevel 9   
   SetOutputFilter DEFLATE  
   DeflateFilterNote Input instream 
   DeflateFilterNote Output outstream 
   DeflateFilterNote Ratio ratio  
   AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css application/javascript   
</ifmodule>
```  
- DeflateCompressionLevel 9 #压缩等级，越大效率越高，消耗CPU也越高。预设可以采用 6 这个数值，以维持耗用处理器效能与网页压缩质量的平衡。
- SetOutputFilter DEFLATE #启用压缩
- DeflateFilterNote Input instream #声明输入流的byte数量
- DeflateFilterNote Output outstream #声明输出流的byte数量
- DeflateFilterNote Ratio ratio #声明压缩的百分比

1）如果是虚拟主机，需要在<VirtualHost*:80></VirtualHost>中添加配置即可实现压缩  
2）图片和视频本身就是压缩格式，一般不需要压缩的。有些小图片和视频压缩后还会变大。  

总结：我们在企业生产环境中时，在启用mod_deflate时，一定要注意，对于太小的文件和某些格式的图片不要对它们进行压缩，有可能越压越大。  

扩展：AddOutputFilterByTypeDEFLATE后跟的所有的压缩文件类型，后期可以参照选择。  
text/plain text/html text/php text/xml text/css text/javascript  
application/xhtml+xml application/xml application/rss+xml application/atom_xml application/x-javascript application/x-httpd-php image/svg+xml image/gif image/png  image/jpe image/swf image/jpeg image/bmp  



5、mod_expires 设置网页缓存时间  
```
mod_expires模块检查及安装
# /usr/local/apache/bin/apachectl -M |grep expires

a）编译方式安装
编译的时候跟上--enable-expires即可实现安装

b）DSO方式安装，编译后，免编译安装模块
# cd /usr/local/src/httpd-2.4.27/modules/metadata/  #切到apache源码包mod_expires所在的目录下
# ls mod_expires.c 
以dso的方式编译安装到apache中
# /usr/local/apache/bin/apxs -c -i -a /usr/local/src/httpd-2.4.10/modules/metadata/mod_expires.c

# vim /etc/httpd/conf/httpd.conf
LoadModule expires_module     modules/mod_expires.so

重新加载配置
# systemctl restart httpd

再次检查是否安装模块
# /usr/local/apache/bin/apachectl -M |grep expires
 expires_module (shared)


缓存的用法有3种，分别问对全局，对目录，对虚拟主机。
1、对全局
对全局的配置就是在apache主配置文件httpd.conf的末尾加入如下参数即可
# vim /etc/httpd/httpd.conf  #在最后添加以下内容：
<IfModule mod_expires.c>  
ExpiresActive on
    ExpiresDefault "access plus 12 month"
    ExpiresByType text/html "access plus 12 months"
    ExpiresByType text/css "access plus 12 months"
    ExpiresByType image/gif "access plus 12 months"
    ExpiresByType image/jpeg "access plus  12 months"
    ExpiresByType image/jpg "access plus 12 months"
    ExpiresByType image/png "access plus 12 months"
    EXpiresByType application/x-shockwave-flash "access plus 12 months"
    EXpiresByType application/x-javascript "access plus 12 months"
ExpiresByType video/x-flv "access plus 12 months"
</IfModule>
重启服务：
# systemctl restart httpd

2、对目录
对目录的配置就是在apache主配置文件中<Directory></Directory>标签内，最后加入如下参数即可
# vim /usr/local/apache2.2-xuegod/conf/httpd.conf
<Directory "/usr/local/apache/htdocs">   
    <IfModule mod_expires.c>  
ExpiresActive on
    ExpiresDefault "access plus 12 month"
    ExpiresByType text/html "access plus 12 months"
    ExpiresByType text/css "access plus 12 months"
    ExpiresByType image/gif "access plus 12 months"
    ExpiresByType image/jpeg "access plus  12 months"
    ExpiresByType image/jpg "access plus 12 months"
    ExpiresByType image/png "access plus 12 months"
    EXpiresByType application/x-shockwave-flash "access plus 12 months"
    EXpiresByType application/x-javascript "access plus 12 months"
ExpiresByType video/x-flv "access plus 12 months"
</IfModule>
</Directory>

3、对虚拟主机
对虚拟主机的配置就是在apache的虚拟主机配置文件httpd-vhost.conf中添加如下参数即可
# vim /etc/httpd/httpd.conf
修改：
DocumentRoot "/usr/local/apache/htdocs"
改为：
# DocumentRoot "/usr/local/apache/htdocs"

修改：
467 # Include /etc/httpd/extra/httpd-vhosts.conf
改为：
Include /etc/httpd/extra/httpd-vhosts.conf

# vim /etc/httpd/extra/httpd-vhosts.conf
<VirtualHost *:80>
    ServerAdmin 888@qq.com
    DocumentRoot "/www/html"
    ServerName www.xuegod.cn
    ServerAlias xuegod.cn
    ErrorLog "logs/dummy-host.example.com-error_log"
    CustomLog "logs/dummy-host.example.com-access_log" common
<Directory "/www/html">
        Options None
        Require all granted
  </Directory>
<IfModule mod_expires.c>  
ExpiresActive on
    ExpiresDefault "access plus 12 month"
    ExpiresByType text/html "access plus 12 months"
    ExpiresByType text/css "access plus 12 months"
    ExpiresByType image/gif "access plus 12 months"
    ExpiresByType image/jpeg "access plus  12 months"
    ExpiresByType image/jpg "access plus 12 months"
    ExpiresByType image/png "access plus 12 months"
    EXpiresByType application/x-shockwave-flash "access plus 12 months"
    EXpiresByType application/x-javascript "access plus 12 months"
ExpiresByType video/x-flv "access plus 12 months"
</IfModule>
</VirtualHost>
```  
扩展：expires模块的语法  
expires模块用到了ExpiresDefault和EXpiresByType两个指令，下面是这两个指令的语法。  
ExpiresDefault “<base> [plus] {<num><type>}*”  
EXpiresByType type/encoding "<base> [plus] {<num><type>}"  
其中<base>的参数有3个：access，now（等价于‘access’），modification  
modification   [ˌmɒdɪfɪˈkeɪʃn]  改性，修正  
plus关键字是可选的  
plus [plʌs]  加上  
<num>必须是整数，确保可以atoi（）所接收。（atoi可以把字符串转换成长整型数）  
<type>参数类型：years，months，weeks，days，hours，minutes，seconds  


6、开启长连接功能  
```
# vim /etc/httpd/httpd.conf
改： #Include /etc/httpd/extra/httpd-default.conf
为：Include /etc/httpd/extra/httpd-default.conf
# vim /etc/httpd/extra/httpd-default.conf
修改：
KeepAlive On 
KeepAliveTimeout 5
```  

- MaxKeepAliveRequests #这个值一般不需要配置。
默认：100  
一个建立好的Keep-Alive连接，允许发送的请求的个数。一旦建立连接，要么就是个数达到了断开，要么就是等KeepAliveTimeout时间到了断开连接。  
MaxKeepAliveRequests指令限制了当启用KeepAlive时，每个连接允许的请求数量。如果将此值设为"0"，将不限制请求的数目。我们建议最好将此值设为一个比较大的值，以确保最优的服务器性能。"  
这个数字的设置，必须考虑在一个时间段内，同一个用户访问你的服务会发多少请求。要结合KeepAliveTimeout参数来考虑。  


7、查看当前使用的MPM模型,并修改
```
查看当前使用的MPM模型
# /usr/local/apache/bin/httpd -M | grep event
  mpm_event_module (shared)
查看模块：
     httpd -l      #查看MPM模块
     httpd -M      #查看DSO模块，由于mod_mpm_event.so，mod_mpm_prefork.so，mod_mpm_worker.so，此三个模块被做成DSO模块，所以使用httpd -M查看

查看当前使用的MPM模型
# /usr/local/apache/bin/httpd -V
Server version: Apache/2.4.27 (Unix)
Server built:   Aug  5 2019 12:04:06
Server's Module Magic Number: 20120211:68
Server loaded:  APR 1.5.2, APR-UTIL 1.5.2
Compiled using: APR 1.5.2, APR-UTIL 1.5.2
Architecture:   64-bit
Server MPM:     event                #当前使用的是event模型
  threaded:     yes (fixed thread count)
    forked:     yes (variable process count)
Server compiled with....

# vim /etc/httpd/httpd.conf 
#LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
#LoadModule mpm_worker_module modules/mod_mpm_worker.so

# systemctl restart httpd
[root@node03 ~]# /usr/local/apache/bin/httpd -V
Server version: Apache/2.4.27 (Unix)
Server built:   Aug  5 2019 12:04:06
Server's Module Magic Number: 20120211:68
Server loaded:  APR 1.5.2, APR-UTIL 1.5.2
Compiled using: APR 1.5.2, APR-UTIL 1.5.2
Architecture:   64-bit
Server MPM:     prefork           #当前使用的是prefork模型
  threaded:     no
    forked:     yes (variable process count)
```  
httpd2.4 新特性  
```
1）MPM支持在运行时装载
     指定启用：
             --enable-mpms-shared=all --with-mpm=event      //把所有支持的MPM都编译进来，但启用默认的是event
2) 支持event
3）异步读写
4) 在每模块及每目录上指定日志级别
5）每请求配置：<If> <Elseif>
6) 增强版的表达式分析器
7) 毫秒级的keepalive timeout ，使用ms指定为毫秒
8）支持主机名的虚拟主机不在需要NameVirtualHost指令
9) 支持使用自定义变量
新增一些模块：mod_proxy_fcgi（基于fcgi方式调用执行环境）
mod_ratelimit（用于做速率限定）
mod_request（对请求方法做限定）
mod_remoteip（对远端IP做限定）
对于基于IP的访问做了修改，不在使用order,allow,deny这些机制；而是统一使用require进行
```  
RHEL6/7系统自带的apache默认采用的是prefork进程模型；在编译apache源码时，如果不用--with-mpm显式指定某种MPM，prefork就是缺省的MPM  

8、对prefork模式进行优化。 修改apache 的httpd-mpm.conf 配置  
```
# vim /etc/httpd/httpd.conf
将注释去掉
#Include /etc/httpd/extra/httpd-mpm.conf
Include /etc/httpd/extra/httpd-mpm.conf


# vim /etc/httpd/extra/httpd-mpm.conf
将
<IfModule mpm_prefork_module>
    StartServers             5
    MinSpareServers          5
    MaxSpareServers         10
    MaxRequestWorkers      250
    MaxConnectionsPerChild   0
</IfModule>
修改为
<IfModule mpm_prefork_module>
ServerLimit 3000
StartServers 50
MinSpareServers 50
MaxSpareServers 100
MaxRequestWorkers    3000
MaxRequestsPerChild 1000
</IfModule>

#systemctl restart httpd
```  
- ServerLimit 是最大的进程数  
- MaxRequestWorkers 是最大的请求并发。    
- StartServers 50 启动时默认启动的 进程数
- MinSpareServers 指令设置空闲子进程的最小数量  
- MaxSpareServers 100 最大空闲进程  
- MaxRequestsPerChild指令设置每个子进程在其生存期内允许处理的最大请求数量


9、apache worker模拟性能优化  
```
# vim /etc/httpd/extra/httpd-mpm.conf
将
<IfModule mpm_worker_module>
    StartServers             3
    MinSpareThreads         75
    MaxSpareThreads        250
    ThreadsPerChild         25
    MaxRequestWorkers      400
    MaxConnectionsPerChild   0
</IfModule>
修改为
<IfModule mpm_worker_module>
    StartServers 2
    MaxRequestWorkers 150000 
    MinSpareThreads 25 
    MaxSpareThreads 75 
    ThreadsPerChild 25 
    MaxRequestsPerChild 0 
</IfModule>
```  
- StartServers 2 #最初建立的子进程
- MaxRequestWorkers 150000   # apache同时最多能支持150000个并发访问，超过的要进入队列等待
- MinSpareThreads 25    #基于整个服务器监视的最小空闲线程数
- MaxSpareThreads 75    #基于整个服务器监视的最大空闲线程数
- ThreadsPerChild 25    #每个子进程包含固定的线程数，此参数在worker模式中，是影响最大的参数，最大缺省值是64
- MaxRequestsPerChild 0    #每个子进程可以支持的请求数，这要设置为0，因为一个进程关闭，所有的线程也都关了。 

生产环境配置实例：
```
<IFModule mpm_worker_module>
StartServers          5
MaxRequestWorkers     9600
ServerLimit           64
MinSpareThreads       25
MaxSpareThreads       500
ThreadLimit           200
ThreadsPerChild       150
MaxRequestsPerChild   0
</IFModule>
```  
