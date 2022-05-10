# <u>Centos</u>

### 超级管理员

- ### 普通用户赋权超级权限
  
  - 进入超级用户模式。输入命令"chmod u+w /etc/sudoers"。 
  - 命令"vim /etc/sudoers",输入"a"进入编辑模式，找到这一 行："Allow root to ruan any commands anywhere"在【root ALL=(ALL) ALL】下面一行添加"xxx ALL=(ALL) ALL"(这里的xxx是你的用户名)，保存退出。 
  - 撤销sudoers文件的编写权限。输入命令"chmod u-w /etc/sudoers"

## 基本命令

> **env** 查看环境变量

- ### 磁盘查询

> **df -hl**：查看磁盘剩余空间
>
> **df -h**：查看每个根路径的分区大小
>
> **du -sh [目录名]**：返回该目录的大小
>
> **du -sm [文件夹]**：返回该文件夹总M数
>
> **du -h [目录名]**：查看指定文件夹下的所有文件大小（包含子文件夹）

## 时区

- timedatectl list-timezones  列出所有时区
- timedatectl set-local-rtc 1  将硬件时钟调整为与本地时钟一致, 0 为设置为 UTC 时间
- timedatectl set-timezone Asia/Shanghai  设置系统时区为上海
- cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 各版本Linux通用



### 防火墙操作 

- ##### 进程与状态相关

```shell
systemctl start firewalld.service            #启动防火墙  
systemctl stop firewalld.service             #停止防火墙  
systemctl status firewalld                   #查看防火墙状态
systemctl enable firewalld                  #设置防火墙随系统启动
systemctl disable firewalld                 #禁止防火墙随系统启动
firewall-cmd --state                         #查看防火墙状态  
firewall-cmd --reload                        #更新防火墙规则   
firewall-cmd --list-ports                    #查看所有打开的端口  
firewall-cmd --list-services                 #查看所有允许的服务  
firewall-cmd --get-services                  #获取所有支持的服务  
```

- ##### 区域相关

```shell
firewall-cmd --list-all-zones                    #查看所有区域信息  
firewall-cmd --get-active-zones                  #查看活动区域信息  
firewall-cmd --set-default-zone=public           #设置public为默认区域  
firewall-cmd --get-default-zone                  #查看默认区域信息  
```


​    
- ##### 接口相关

```shell
firewall-cmd --zone=public --add-interface=eth0         #将接口eth0加入区域public
firewall-cmd --zone=public --remove-interface=eth0       #从区域public中删除接口eth0  
firewall-cmd --zone=default --change-interface=eth0      #修改接口eth0所属区域为default  
firewall-cmd --get-zone-of-interface=eth0                #查看接口eth0所属区域  
```

- ##### 端口控制

```shell
firewall-cmd --query-port=8080/tcp                           # 查询端口是否开放
firewall-cmd --add-port=8080/tcp --permanent                 #永久添加8080端口例外(全局)
firewall-cmd --remove-port=8800/tcp --permanent             #永久删除8080端口例外(全局)
firewall-cmd --add-port=65001-65010/tcp --permanent         #永久增加65001-65010例外(全局)  
firewall-cmd  --zone=public --add-port=8080/tcp --permanent            #永久添加8080端口例外(区域public)
firewall-cmd  --zone=public --remove-port=8080/tcp --permanent         #永久删除8080端口例外(区域public)
firewall-cmd  --zone=public --add-port=65001-65010/tcp --permanent   #永久增加65001-65010例外(区域public) 
```


```shell
命令解析
    firewalld-cmd --zone=public --add-ports=80/tcp --permanent
    firwall-cmd：是Linux提供的操作firewall的一个工具（服务）命令
    --zone #作用域
    --add-port=8080/tcp #添加端口，格式为：端口/通讯协议 ；add表示添加，remove则对应移除
    --permanent #永久生效，没有此参数重启后失效
```


# <u>Docker操作</u>
## 基本操作

```shell
service docker start/stop  		#启动或停止docker命令
systemctl enable docker 		#設置docker开机自启动
docker search nginx 			#查看可用版本
docker images  					#查看本地镜像
docker ps -a					#查看运行的容器
docker pull ubuntu				#获取镜像
docker run -dit ubuntu /bin/bash  # 启动容器
					参数说明： -i #交互式操作。
                                -t #终端。
                                -d #后台运行
                                -p #将容器内部使用的网络端口随机映射到我们使用的主机上。
                                -v /etc/localtime:/etc/localtime  #在 run 时挂载宿主时间配置
                                ubuntu: #ubuntu 镜像。
                                /bin/bash：#放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash
docker start 容器id/name 			#启动已停止运行的容器
docker stop <容器 ID>				#停止一个容器
docker restart <容器 ID>			#重启一个容器

#在使用 -d 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：
        docker attach 容器id/name   	  		#如果从这个容器退出，会导致容器的停止
        docker exec -it 容器id/name /bin/bash 		#此退出容器终端，不会导致容器的停止

docker rm -f 容器id/name			  #删除容器
docker rm $(docker ps -aq)			#删除docker所有容器
docker rmi -f 1e560fca3906			#删除镜像
docker container prune				#清理掉所有处于终止状态的容器
```

- **网络相关**
```shell
#查看容器网络
	docker container inspect mycentos7-bridge|grep -wA 30 "Networks
	docker container inspect mycentos7-bridge|grep "NetworkMode"
#网络别名
	docker run -it  --name=mosquitto-test --privileged -p 9883:1883  -p 9001:9001  --network iot --network-alias mosquitto   -v /data/mosquitto/config/mosquitto.conf:/mosquitto/config/mosquitto.conf -v /data/mosquitto/data:/mosquitto/data -v /data/mosquitto/log:/mosquitto/log  eclipse-mosquitto                     -			-network ：指定网络
			--network-alias ：指定网络别名
			-v ：挂载目录  宿主目录：docker目录
			--privileged ：docker root用户获取真正root权利
		
#容器断开网络
	docker network disconnect network container
#删除一个或多个网络
	docker network rm network
#显示一个或多个网络的详细信息
	docker network inspect network
#创建一个网络
	docker network create network
#列出网络
	docker network ls
#删除所有未使用的网络
	docker network prune

#容器连接到网络
	docker network connect [OPTIONS] network container 				  
    [OPTIONS] :
            --alias  为容器添加网络范围的别名
            --ip 指定IP地址
            --ip6 指定IPv6地址
            -ip-range:指定ip范围
            --link  添加链接到另一个容器
            --link-local-ip   添加容器的链接本地地址
 #例子
    docker network connect --alias 别名 --link anothercontainer:别名 network container 
    docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 network
    docker network connect --ip 172.20.128.2 network container2
```

- **容器时间和宿主机同步**

```shell
# 在 run 时挂载宿主时间配置
-v /etc/localtime:/etc/localtime

# 复制宿主机 localtime 配置
docker cp /etc/localtime 容器ID:/etc/localtime
docker cp /usr/share/zoneinfo/Asia/Shanghai 容器ID:/etc/localtime
```

## Nginx

- **取最新版的** Nginx 镜像

```shell
docker pull nginx:latest
```

- 运行容器

```shell
docker run --name nginx-test -p 8080:80 -d nginx
```
参数说明：

```shell
--name nginx-test：容器名称。
-p 8080:80： 端口进行映射，将本地 8080 端口映射到容器内部的 80 端口。
-d nginx： 设置容器在在后台一直运行。
```

## MySQL
- 拉取 MySQL 镜像
```shell
docker pull mysql:latest
```
- 运行容器
```shell
docker run -itd --name mysql-test -p 5306:3306 --network=iot --network-alias=mysql -v /etc/mysql:/etc/mysql/conf.d -v /data/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456  mysql
```
参数说明：

```shell
-p 3306:3306 ：映射容器服务的 3306 端口到宿主机的 5306 端口，外部主机可以直接通过 宿主机ip:5306 访问到 MySQL 的服务。
MYSQL_ROOT_PASSWORD=123456：设置 MySQL 服务 root 用户的密码。
--character-set-server：设置数据库默认的编码
--collation-server：设置排序的编码
-e TZ="Asia/Shanghai"：时区设置
```
#### web容器连接mysql容器
```shell
docker run -d --name myweb --link mysqldb1:db1  -p 9999:8080 docker.io/tomcat:7.0.78
    -d:守护进程方式启动
    --name:web容器的别名
    --link mysqldb1:db1：mysqldb1是mysql容器的名称；db1是我们连接后起的别名，也就是在web容器中通过db1开头信息进行访问mysql容器的
    -p：将容器的8080端口映射到宿主机的9999端口
    docker.io/tomcat:7.0.78：docker pull下来的tomcat镜像
```
#### Navicat连接虚拟机中的MariaDB

[网络上解决方案](https://blog.csdn.net/constantdropping/article/details/83083321)

#### mysql挂载外部配置和数据目录

[网络上解决方案](https://www.cnblogs.com/qq931399960/p/11670625.html)

#### 设置远程登录(Navicat)

```shell
进入MySQL容器并登陆MySQL：  	docker exec -it mysql /bin/bash
进行授权远程连接授权：			 GRANT ALL ON *.* TO 'root'@'%';
刷新权限：					 flush privileges
注意,这时还不能远程访问** 因为Navicat只支持旧版本的加密,需要更改mysql的加密规则
更改加密规则：					ALTER USER 'root'@'localhost' IDENTIFIED BY 'password' PASSWORD EXPIRE NEVER;
更新root用户密码：				ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
刷新权限：					  flush privileges;
```

# <u>Grafana</u>

## Docker方式安装

```shell
docker run -d --name=grafana -p 3000:3000 grafana/grafana-enterprise:8.2.7
```
## Centos方式安装

```shell
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-8.2.7-1.x86_64.rpm
sudo yum install grafana-enterprise-8.2.7-1.x86_64.rpm
```

## grafana 连接另一个docker中的mysql数据源

```shell
sudo docker run -dit --name grafana --link mysql-test:mysql -p 5000:3000 grafana/grafana-enterprise    
```

**其中**：mysql-test:mysql表示把本地mysql-test容器在grafana中用mysql命名，在grafana中的连接地址为mysql:3306
					GRANT ALL ON *.* TO 'root'@'%';授权

# <u>Node-red</u>

## Docker方式安装
```shell
sudo docker run -it -p 5880:1880 -v /home/eason/workplace/.node-red:/data --name nodered-test nodered/node-red
```

> **流程图json文件存放目录**： /data/flow

# <u>EMQX</u>

## Docker方式安装
```shell
docker run -dit --name emqx-test -p 18083:18083 -p 1883:1883 -p 8083:8083 -p 8084:8084 --restart=always emqx/emqx
```

# Mosquitto

## Docker方式安装

```shell
docker run -dit  --name=mosquitto-test --privileged -p 9883:9883  -p 9001:9001  --network iot --network-alias mosquitto   -v /data/mosquitto/config/:/mosquitto/config/ -v /data/mosquitto/data:/mosquitto/data -v /data/mosquitto/log:/mosquitto/log  eclipse-mosquitto
```

先要在宿主机上创建文件夹/data/mosquitto/config    /data/mosquitto/data     /data/mosquitto/log

/data/mosquitto/config/mosquitto.conf配置文件：

```shell
persistence true
persistence_location /mosquitto/data
log_dest file /mosquitto/log/mosquitto.log
allow_anonymous true
listener 9883
```

# <u>Redis</u>

## Docker方式安装

```shell
docker run -itd --name redis-test -p 6379:6379 redis
```

**-p 6379:6379**：映射容器服务的 6379 端口到宿主机的 6379 端口。外部可以直接通过宿主机ip:6379 访问到 Redis 的服务。接着我们通过 redis-cli 连接测试使用 redis 服务。

# Node.js

## Docker方式安装

```shell
docker run -itd --name node-test node
```

