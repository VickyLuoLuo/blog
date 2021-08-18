---
title: VirtualBox新建Centos虚机环境配置
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top:
tags:
  - tools
categories:
  - work
abbrlink: '130'
subtitle:
---

### 一、全局代理设置

修改 /etc/profile 文件,添加下面内容:

```
http_proxy=http://name:pwd@ip:port
https_proxy=http://name:pwd@ip:port
ftp_proxy=http://name:pwd@ip:port
export http_proxy
export https_proxy
export ftp_proxy
```

使全局配置生效

```
[root@lcoalhost]# source /etc/profile
```

若只针对某个用户而言,则修改 ~/.bash_profile 文件,添加相同内容;

修改完成后,注销重新登录即可。

### 二、yum代理设置

修改 /etc/yum.conf，添加下面内容:

```
http_proxy=http://name:pwd@ip:port
```

保存退出后，就可以使用yum轻松的安装软件了。

### 三、wget代理设置

修改/etc/wgetrc，添加下面内容:

```
http_proxy = http://name:pwd@ip:port
ftp_proxy = http://name:pwd@ip:port
```

### 四、网络配置

VirtualBox主机网络管理器中需要创建一张Host-Only网卡，并手动配置网卡地址及DHCP服务器。虚机启动时会根据dhcp服务器配置自动分配地址。

虚机设置中启用两张网卡，网卡一连接方式为桥接网卡，网卡二连接方式为仅主机（Host-Only）网络。

虚机中，安装net-tools依赖包：

```
[root@lcoalhost]# yum -y install net-tools
```

查看网卡是否启用（是否分配ip，是否能ping通主机）：

```
[root@lcoalhost]# ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.112.118.81  netmask 255.255.254.0  broadcast 10.112.119.255 
        inet6 fe80::f41f:d:f042:d8ae  prefixlen 64  scopeid 0x20<link> 
        ether 08:00:27:fb:62:61  txqueuelen 1000  (Ethernet) //ether对应第一张网卡的mac
        RX packets 220530  bytes 251839085 (240.1 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 42719  bytes 2891040 (2.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.103  netmask 255.255.255.0  broadcast 192.168.56.255 // inet对应dhcp地址池中的地址
        inet6 fe80::5ff8:c6c:90:9239  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:96:49:8d  txqueuelen 1000  (Ethernet) //ether对应第二张网卡的mac
        RX packets 62911  bytes 3926194 (3.7 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 117073  bytes 10861798 (10.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

### 五、git代理设置

```
[root@lcoalhost]# yun -y install git
[root@lcoalhost]# git config --global https.proxy http://name:pwd@ip:port
[root@lcoalhost]# git config --global http.proxy http://name:pwd@ip:port
```

### 六、docker 安装

1、Docker 要求 CentOS 系统的内核版本高于 3.10 ，查看本页面的前提条件来验证你的CentOS 版本是否支持 Docker 。

通过 uname -r 命令查看你当前的内核版本

```
 [root@lcoalhost]# uname -r
```

2、使用 root 权限登录 Centos。确保 yum 包更新到最新。

```
[root@lcoalhost]# sudo yum update
```


3、卸载旧版本(如果安装过旧版本的话)

```
[root@lcoalhost]#sudo yum remove docker  docker-common docker-selinux docker-engine
```

4、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
[root@lcoalhost]# sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

5、设置yum源

```
[root@lcoalhost]# sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

 重新设置阿里镜像网站（有时候也很慢）

```
[root@lcoalhost]# yum-config-manager --add-repo http://mirrors.aliyun.com/dockerce/linux/centos/docker-ce.repo
```

6、可以查看所有仓库中所有docker版本，并选择特定版本安装

```
[root@lcoalhost]# yum list docker-ce --showduplicates | sort -r
```

7、安装docker

```
[root@lcoalhost]# sudo yum install docker-ce  #由于repo中默认只开启stable仓库，故这里安装的是最新稳定版17.12.0
[root@lcoalhost]# sudo yum install <FQPN>  # 例如：sudo yum install docker-ce-17.12.0.ce
```

8、启动并加入开机启动

```
[root@lcoalhost]# sudo systemctl start docker
[root@lcoalhost]# sudo systemctl enable docker
```


9、验证安装是否成功(有client和service两部分表示docker安装启动都成功了)

```
[root@lcoalhost]# docker version
```

### 七、设置docker 代理

```
[root@lcoalhost]# mkdir -p /etc/systemd/system/docker.service.d
[root@lcoalhost]# cat > /etc/systemd/system/docker.service.d/https-proxy.conf << EOF
[Service]
Environment="http://name:pwd@ip:port" "HTTPS_PROXY=http://name:pwd@ip:port" 
EOF
[root@lcoalhost]# systemctl daemon-reload
[root@lcoalhost]# systemctl restart docker
[root@lcoalhost]# systemctl show --property=Environment docker
```

docker 镜像加速，docker客户端版本大于 1.10.0 ,可修改daemon配置文件/etc/docker/daemon.json来使用加速器

```
[root@lcoalhost]#sudo mkdir -p /etc/docker
[root@lcoalhost]#sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": ["https://4ld4cyzt.mirror.aliyuncs.com"]
}
EOF
[root@lcoalhost]#sudo systemctl daemon-reload
[root@lcoalhost]#sudo systemctl restart docker
```



### 八、docker中安装Mysql

拉取mysql镜像

```
[root@lcoalhost]# docker pull mysql:5.7
5.7: Pulling from library/mysql
123275d6e508: Already exists 
27cddf5c7140: Pull complete 
c17d442e14c9: Pull complete 
2eb72ffed068: Pull complete 
d4aa125eb616: Pull complete 
52560afb169c: Pull complete 
68190f37a1d2: Pull complete 
3fd1dc6e2990: Pull complete 
85a79b83df29: Pull complete 
35e0b437fe88: Pull complete 
992f6a10268c: Pull complete 
Digest: sha256:82b72085b2fcff073a6616b84c7c3bcbb36e2d13af838cec11a9ed1d0b183f5e
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

查看镜像

```
[root@lcoalhost]# docker images
REPOSITORY  TAG    IMAGE ID     CREATED       SIZE
mysql       5.7    f5829c0eee9e 2 hours ago   455MB
[root@lcoalhost]# 
```

启动mysql

```shell
sudo docker run -p 3306:3306 --name mysql \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root \
-d mysql:5.7
```

修改配置

```
[root@lcoalhost]# cd /mydata/mysql/conf
// 配置如下：
[root@lcoalhost conf]# cat my.cnf
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve

[root@lcoalhost conf]# docker restart mysql
mysql
[root@lcoalhost conf]# 
```

进入容器查看配置是否生效：

```shell
[root@lcoalhost conf]# docker exec -it mysql /bin/bash
root@b3a74e031bd7:/# whereis mysql
mysql: /usr/bin/mysql /usr/lib/mysql /etc/mysql /usr/share/mysql

root@b3a74e031bd7:/# ls /etc/mysql 
my.cnf
root@b3a74e031bd7:/# cat /etc/mysql/my.cnf 
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
root@b3a74e031bd7:/# 
```

设置mysql随docker启动

```
[root@lcoalhost ~]# docker update mysql --restart=always
mysql
```

### 九、docker中安装Redis

拉取redis镜像

```shell
[root@lcoalhost ~]# docker pull redis
Using default tag: latest
latest: Pulling from library/redis
123275d6e508: Already exists 
f2edbd6a658e: Pull complete 
66960bede47c: Pull complete 
79dc0b596c90: Pull complete 
de36df38e0b6: Pull complete 
602cd484ff92: Pull complete 
Digest: sha256:1d0b903e3770c2c3c79961b73a53e963f4fd4b2674c2c4911472e8a054cb5728
Status: Downloaded newer image for redis:latest
docker.io/library/redis:latest
```

启动

```shell
[root@lcoalhost ~]# mkdir -p /mydata/redis/conf
[root@lcoalhost ~]# touch /mydata/redis/conf/redis.conf
[root@lcoalhost ~]# echo "appendonly yes"  >> /mydata/redis/conf/redis.conf // 开启数据持久化
[root@lcoalhost ~]# docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data \
 -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
 -d redis redis-server /etc/redis/redis.conf
ce7ae709711986e3f90c9278b284fe6f51f1c1102ba05f3692f0e934ceca1565
```

 连接docker中的redis

```shell
[root@lcoalhost ~]# docker exec -it redis redis-cli
127.0.0.1:6379> set key1 v1
OK
127.0.0.1:6379> get key1
"v1"
```

设置redis随docker启动

```shell
[root@lcoalhost ~]# docker update redis --restart=always
redis
```



