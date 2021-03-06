官网  
https://www.rabbitmq.com/  

准备环境  
```
yum install -y build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel m4 ncurses-devel tk tc xz
```  
erlang与rabbix版本对应关系，必选对应上，否则无法使用  
https://www.rabbitmq.com/which-erlang.html  

安装下载：  
```
wget www.rabbitmq.com/releases/erlang/erlang-18.3-1.el7.centos.x86_64.rpm
wget http://repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-5.el7.lux.x86_64.rpm
wget www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-3.6.5-1.noarch.rpm

按照顺序依次安装
#  rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
   1:erlang-18.3-1.el7.centos         ################################# [100%]
#  rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm 
warning: socat-1.7.3.2-5.el7.lux.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 53e4e7a9: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:socat-1.7.3.2-5.el7.lux          ################################# [100%]
#  rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm 
warning: rabbitmq-server-3.6.5-1.noarch.rpm: Header V4 RSA/SHA1 Signature, key ID 6026dfca: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:rabbitmq-server-3.6.5-1          ################################# [100%]
```  

修改配置文件  
```
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app
修改去掉<<>>
{loopback_users, [<<"guest">>]}
修改后
{loopback_users, ["guest"]}
```  

启动 rabbitmq-server start &  
停止 rabbitmqctl app_stop  
状态 rabbitmqctl status  

管理插件：rabbitmq-plugins enable rabbitmq_management  
访问地址：http://192.168.11.76:15672/  


配置方式：  
```
环境变量：/etc/rabbitmq/rabbitmq-env.conf
RABBITMQ_BASE: 数据库和日志文件,主要适用于windows主机，linux不常用
RABBITMA_CONFIGFILE：
RABBITMA_LOGS
RABBITMQ_NODE_IP_ADDRESS
RABBITMQ_NODE_PORT
RABBITMQ_PLUGIN_DIR
```  

配置文件：  
```
auth_mechanisms: 认机机制
default_user: guest
default_pass: guest
default_permission
dis_free_limit
heartbeat
hipe_compile
log_levels {none| error|waring|info}
tcp_listeners: 监听的地址和端口
ssl_listeners:基于ssl通信协议鉴定的地址和端口
vm_memory_hight_watermark
```  

运行时参数：  
```
rabbitmqctl命令：
命令行语法风格：使用下划线
rabbitctl add_user <username> <password>
rabbitctl set_user_tags <username> <tags>
list_users
```  
