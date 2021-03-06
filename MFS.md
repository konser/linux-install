mfsmaster
---
1、安装依赖  
``` yum install rpm-build gcc gcc-c++ fuse-devel zlib-devel -y ```  

2、创建mfs用户  
``` useradd -s /sbin/nologin mfs ```  

3、编译mfs  
```
wget https://github.com/moosefs/moosefs/archive/v3.0.105.tar.gz
cd moosefs
./configure --prefix=/opt/mfs --with-default-user=mfs --with-default-group=mfs
make && make install
```  

4、进入安装目录重命名配置文件  
```
cd /opt/mfs/etc/mfs
cp mfsmaster.cfg.sample mfsmaster.cfg
cp mfsexports.cfg.sample mfsexports.cfg
cp mfsmetalogger.cfg.sample mfsmetalogger.cfg
cp mfstopology.cfg.sample mfstopology.cfg
cd /opt/mfs/var/mfs
cp metadata.mfs.empty metadata.mfs
```  

5、更改属主属组  
``` chown -R mfs.mfs /opt/mfs/ ```  

6、启动  
``` sbin/sbin/mfsmaster start ```  

7、配置共享访问权限  
```
vim mfsexports.cfg
192.168.101.0/24         /       rw,alldirs,maproot=0,password=123456
```  

- / 表示 MooseFS 的根  
- . 表示 MFSMETA 文件系  

- ro 只读
- rw 读写
- alldirs 允许挂载任何指定的子目录 
- maproot 映射为 root 用户还是指定的用户
- password 指定客户端密码  


安装元数据mfsmetalogger
---
1、下载安装
```
wget https://github.com/moosefs/moosefs/archive/v3.0.105.tar.gz
tar xvf v3.0.105.tar.gz 
cd moosefs-3.0.105/
./configure --prefix=/opt/mfsmeta --with-default-user=mfs --with-default-group=mfs
make && make install
```  

2、拷贝配置文件  
```
cd /opt/mfsmeta/etc/mfs
cp mfsmetalogger.cfg.sample mfsmetalogger.cfg
vim mfsmetalogger.cfg
MASTER_HOST = 192.168.101.69
```  

3、更改属主属组  
``` chown -R mfs.mfs /opt/mfsmeta/ ```  

4、启动  
```
sbin/mfsmetalogger start
netstat -antpu | grep 9419
```  

存储Chunk Server  
---
1、下载安装
```
wget https://github.com/moosefs/moosefs/archive/v3.0.105.tar.gz
tar xvf v3.0.105.tar.gz 
cd moosefs-3.0.105/
./configure --prefix=/opt/mfsostor --with-default-user=mfs --with-default-group=mfs
make && make install
```  

2、修改配置文件  
```
cp mfschunkserver.cfg.sample mfschunkserver.cfg
vim  mfschunkserver.cfg
MASTER_HOST = 192.168.101.69
MASTER_PORT = 9420

cp mfshdd.cfg.sample mfshdd.cfg
/tmp     #填写要共享的目录路径

```  

3、更改属主属组  
``` chown -R mfs.mfs /opt/mfsostor ```  

4、启动  
```
sbin/mfschunkserver start
netstat -anput |grep 9422
```  


安装客户端  
---
1、下载安装
```
wget https://github.com/moosefs/moosefs/archive/v3.0.105.tar.gz
tar xvf v3.0.105.tar.gz 
cd moosefs-3.0.105/
./configure --prefix=/opt/mfsclient --with-default-user=mfs --with-default-group=mfs --enable-mfsmount
make && make install
```  

2、添加模块
```
# lsmod |grep fuse
# modprobe fuse
# lsmod |grep fuse
fuse                   91874  1 
```  

3、挂载  
```
#创建要挂载的目录
mkdir /opt/mount_t 
# ./mfsmount /opt/mount_t -H 192.168.101.69 -p  #master主机IP
mfsmaster accepted connection with parameters: read-write,restricted_ip,admin ; root mapped to root:root
```  

4、命令  
查看数据有几个备份  
```
# ./mfsgetgoal  /opt
/opt: 2              #两个备份
# ./mfsfileinfo /opt/fstab        #查看文件有几个备份
/opt/fstab: 
        chunk 0: 00000000000013C_00000001 / (id:316 ver:1)
               copy 1:192.168.101.70:9422 (status:VALID)
               copy 2:192.168.101.71:9422 (status:VALID)
设置目录有几个备份
./mfsgetgoal  -r 2 /opt

```  

在master节点配置MFS监控  
---

1、开启mfs监控  
```
/opt/mfs/sbin/mfscgiserv 
```  

2、浏览器访问  
http://192.168.101.69:9425  
