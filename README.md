首先需要安装docker 已經安裝过的兄台，可以忽略第一步，执行第二步


####1.安装Docker

windows 安装

[参考](https://www.jianshu.com/p/27dc76f855f1)


linux

下载安装
````
curl -sSL https://get.docker.com/ | sh
````
设置开机自启
````
sudo systemctl enable docker.service

sudo service docker start|restart|stop
````

####2. 安装
````
git clone https://github.com/zhangjiahong123/docker-lnmp.git
cd docker-lnmp
chmod 777 ./redis/redis.log
chmod -R 777 ./redis/data
docker-compose up -d
````
站点根目录为 docker-lnmp/www

该版本是通过拉取官方已经制作好的各个服务的镜像，再通过Dockerfile相关命令根据自身需求做相应的调整。所以该方式构建迅速使用方便，因为是基于Alpine Linux所以占用空间很小。

ELK(+filebeat)

Elasticsearch、Logstash、Kibana、Filebeat一键搭建日志收集系统

````
cd docker-lnmp/elk
chmod +x elk.sh
./elk.sh
````

####3.测试
使用docker ps查看容器启动状态,若全部正常启动了则 通过访问127.0.0.1、127.0.0.1/index.php、127.0.0.1/db.php、127.0.0.1/redis.php 即可完成测试 (若想使用https则请修改nginx下的dockerfile，和nginx.conf按提示去掉注释即可，灵需要在ssl文件夹中加入自己的证书文件，本项目自带的是空的，需要自己替换，保持文件名一致)

1.进入容器内部
使用 docker exec
````
docker exec -it nginx /bin/sh
````
2.使用nsenter命令
````
# cd /tmp; curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf-; cd util-linux-2.24;
# ./configure --without-ncurses
# make nsenter && sudo cp nsenter /usr/local/bin
````
为了连接到容器，你还需要找到容器的第一个进程的 PID，可以通过下面的命令获取再执行。
````
PID=$(docker inspect --format "{{ .State.Pid }}" container_id)
# nsenter --target $PID --mount --uts --ipc --net --pid
````


####常见问题处理
redis启动失败问题 在v2版本中redis的启动用户为redis不是root,所以在宿主机中挂载的./redis/redis.log和./redis/data需要有写入权限。

    chmod 777 ./redis/redis.log
    chmod 777 ./redis/data
    
    
MYSQL连接失败问题 在v2版本中是最新的MySQL8,而该版本的密码认证方式为Caching_sha2_password,而低版本的php和mysql可视化工具可能不支持,可通过phpinfo里的mysqlnd的Loaded plugins查看是否支持该认证方式,否则需要修改为原来的认证方式mysql_native_password: select user,host,plugin,authentication_string from mysql.user; ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456'; FLUSH PRIVILEGES;

注意挂载目录的权限问题，不然容器成功启动几秒后立刻关闭，例：以下/data/run/mysql 目录没权限的情况下就会出现刚才那种情况 docker run --name mysql57 -d -p 3306:3306 -v /data/mysql:/var/lib/mysql -v /data/logs/mysql:/var/log/mysql -v /data/run/mysql:/var/run/mysqld -e MYSQL_ROOT_PASSWORD=123456 -it centos/mysql:v5.7

需要注意php.ini 中的目录对应 mysql 的配置的目录需要挂载才能获取文件内容，不然php连接mysql失败


  ```
  # php.ini
  mysql.default_socket = /data/run/mysql/mysqld.sock
  mysqli.default_socket = /data/run/mysql/mysqld.sock
  pdo_mysql.default_socket = /data/run/mysql/mysqld.sock
  
  # mysqld.cnf
  pid-file       = /var/run/mysqld/mysqld.pid
  socket         = /var/run/mysqld/mysqld.sock
  
  ```

使用php连接不上redis错误的
 ```
 $redis = new Redis;
 $rs = $redis->connect('127.0.0.1', 6379);
 ```
 
php连接不上，查看错误日志
```
 PHP Fatal error:  Uncaught RedisException: Redis server went away in /www/index.php:7
````
考虑到docker 之间的通信应该不可以用127.0.0.1 应该使用容器里面的ip，所以查看redis 容器的ip

```
 [root@localhost docker]# docker ps
 CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS                                      NAMES
 b5f7dcecff4c        docker_nginx                         "/usr/sbin/nginx -..."   4 seconds ago       Up 3 seconds        0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   nginx
 60fd2df36d0e        docker_php                           "/usr/local/php/sb..."   7 seconds ago       Up 5 seconds        9000/tcp                                   php
 7c7df6f8eb91        hub.c.163.com/library/mysql:latest   "docker-entrypoint..."   12 seconds ago      Up 11 seconds       3306/tcp                                   mysql
 a0ebd39f0f64        docker_redis                         "usr/local/redis/s..."   13 seconds ago      Up 12 seconds       6379/tcp                                   redis
````
注意测试的时候连接地址需要容器的ip或者容器名names，比如redis、mysql. 例如nginx配置php文件解析 fastcgi_pass php:9000; 例如php连接redis $redis = new Redis;$res = $redis->connect('redis', 6379);

因为容器ip是动态的，重启之后就会变化，所以可以创建静态ip


第一步：创建自定义网络

备注：这里选取了172.172.0.0网段，也可以指定其他任意空闲的网段
```
 docker network create --subnet=172.171.0.0/16 docker-at
 docker run --name redis326 --net docker-at --ip 172.171.0.20 -d -p 6379:6379  -v /data:/data -it centos/redis:v3.2.6
```
连接redis 就可以配置对应的ip地址了，连接成功

```
 $redis = new Redis;
 $rs = $redis->connect('172.171.0.20', 6379);
```

另外还有种可能phpredis连接不上redis，需要把redis.conf配置略作修改。
````
 bind 127.0.0.1
 改为：
 bind 0.0.0.0
````
启动docker web服务时 虚拟机端口转发 外部无法访问 一般出现在yum update的时候（WARNING: IPv4 forwarding is disabled. Networking will not work.）或者宿主机可以访问，但外部无法访问
```
vi /etc/sysctl.conf
或者
vi /usr/lib/sysctl.d/00-system.conf
添加如下代码：
    net.ipv4.ip_forward=1
```

重启network服务

systemctl restart network

查看是否修改成功

sysctl net.ipv4.ip_forward

如果返回为"net.ipv4.ip_forward = 1"则表示成功了

如果使用最新的MySQL8无法正常连接，由于最新版本的密码加密方式改变，导致无法远程连接。
```
# 修改密码加密方式
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
```