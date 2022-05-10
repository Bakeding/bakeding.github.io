[centos7安装Docker详细步骤（无坑版教程） - 云+社区 - 腾讯云](https://cloud.tencent.com/developer/article/1701451)

# 基本操作

```text
$ service docker start/stop         #启动或停止docker命令
$ systemctl enable docker       #設置docker开机自启动
$ docker search nginx           #查看可用版本
$ docker images                     #查看本地镜像
$ docker ps -a                  #查看运行的容器
$ docker ps -a --no-trunc       #查看容器完整command的命令
$ docker pull ubuntu                #获取镜像
$ docker run -dit ubuntu /bin/bash  # 启动容器
                    参数说明： -i #交互式操作。
                                -t #终端。
                                -d #后台运行
                                -p #将容器内部使用的网络端口随机映射到我们使用的主机上。
                                -v /etc/localtime:/etc/localtime  #在 run 时挂载宿主时间配置
                                ubuntu: #ubuntu 镜像。
                                /bin/bash：#放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash
$ docker start 容器id/name            #启动已停止运行的容器
$ docker stop <容器 ID>               #停止一个容器
$ docker restart <容器 ID>            #重启一个容器
​
#在使用 -d 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：
        docker attach 容器id/name             #如果从这个容器退出，会导致容器的停止
        docker exec -it 容器id/name /bin/bash         #此退出容器终端，不会导致容器的停止
​
$ docker rm -f 容器id/name              #删除容器
$ docker rm $(docker ps -aq)            #删除docker所有容器
$ docker rmi -f 1e560fca3906            #删除镜像
$ docker container prune                #清理掉所有处于终止状态的容器
$ sudo docker stop $(sudo docker ps -a -q) //  stop停止所有容器
$ docker  rm $(sudo docker ps -a -q) //   remove删除所有容器
```

# **网络相关**

```text
#查看容器网络
$ docker container inspect mycentos7-bridge|grep -wA 30 "Networks"
$ docker container inspect mycentos7-bridge|grep "NetworkMode"
#网络别名
$ docker run -it  --name=mosquitto-test --privileged -p 9883:1883  -p 9001:9001  --network iot --network-alias mosquitto   -v /data/mosquitto/config/mosquitto.conf:/mosquitto/config/mosquitto.conf -v /data/mosquitto/data:/mosquitto/data -v /data/mosquitto/log:/mosquitto/log  eclipse-mosquitto                     -         -network ：指定网络
            --network-alias ：指定网络别名
            -v ：挂载目录  宿主目录：docker目录
            --privileged ：docker root用户获取真正root权利
        
#容器断开网络
$ docker network disconnect network container
#删除一个或多个网络
$ docker network rm network
#显示一个或多个网络的详细信息
$ docker network inspect network
#创建一个网络
$ docker network create network
#列出网络
$ docker network ls
#删除所有未使用的网络
$ docker network prune
​
#容器连接到网络
$ docker network connect [OPTIONS] network container                  
    [OPTIONS] :
            --alias  为容器添加网络范围的别名
            --ip 指定IP地址
            --ip6 指定IPv6地址
            -ip-range:指定ip范围
            --link  添加链接到另一个容器
            --link-local-ip   添加容器的链接本地地址
 #例子
$ docker network connect --alias 别名 --link anothercontainer:别名 network container 
$ docker network create --subnet 172.20.0.0/16 --ip-range 172.20.240.0/20 networkname
$ docker network create \
  --driver=bridge \
  --subnet=172.100.0.0/16 \
  --ip-range=172.100.1.0/24 \
  --gateway=172.100.1.254 \
  iot

$ docker network connect --ip 172.20.128.2 networkname container2
```

### 查询容器之间是否网络相通

[https://segmentfault.com/a/1190000039660549](https://segmentfault.com/a/1190000039660549)

```shell
nsenter [options] [program [arguments]]

options:
-t, --target pid：指定被进入命名空间的目标进程的pid
-m, --mount[=file]：进入mount命令空间。如果指定了file，则进入file的命令空间
-u, --uts[=file]：进入uts命令空间。如果指定了file，则进入file的命令空间
-i, --ipc[=file]：进入ipc命令空间。如果指定了file，则进入file的命令空间
-n, --net[=file]：进入net命令空间。如果指定了file，则进入file的命令空间
-p, --pid[=file]：进入pid命令空间。如果指定了file，则进入file的命令空间
-U, --user[=file]：进入user命令空间。如果指定了file，则进入file的命令空间
-G, --setgid gid：设置运行程序的gid
-S, --setuid uid：设置运行程序的uid
-r, --root[=directory]：设置根目录
-w, --wd[=directory]：设置工作目录
#如果没有给出program，则默认执行$SHELL。


#获取容器pid
docker inspect mysql-mqtt -f '{{.State.Pid}}'   #输出结果  23069

#执行nsenter命令   
nsenter -n -t23069    #这时候主机上的命令都可以使用，这个命令没有任何输出，通过ifconfig命令查看当前ip可以确定我们已经进入了容器的网络空间

#如果想退出当前网络空间，返回系统网络空间，输入exit就行
exit
```

## docker network create命令

`docker network create`命令用于创建一个新的网络连接。 `DRIVER`接受内置网络驱动程序的桥接或覆盖。如果安装了第三方或自己的自定义网络驱动程序，则可以在此处指定`DRIVER`。 如果不指定`--driver`选项，该命令将为您自动创建一个桥接网络。 当安装`Docker Engine`时，会自动创建桥接网络。 该网络对应于Engine传统依赖的`docker0`网桥。 当启动使用`docker run`运行新容器时，它将自动连接到此桥接网络。不能删除此默认网桥，但可以使用`network create`命令创建新的网络。

```shell
$ docker network create -d bridge my-bridge-network
```

准备覆盖网络后，只需在群集中选择一个Docker主机，并发出以下内容创建网络：

```shell
$ docker network create -d overlay my-multihost-network
```

**用法**

```shell
docker network create [OPTIONS] NETWORK
```

**选项**

| 名称，简写         | 默认      | 说明                     |
| ------------- | ------- | ---------------------- |
| --attachable  | false   | 启用手动容器安装               |
| --aux-address | map[]   | 网络驱动程序使用的辅助IPv4或IPv6地址 |
| --driver, -d  | bridge  | 驱动程序管理网络               |
| --gateway     |         | 用于主子网的IPv4或IPv6网关      |
| --internal    | false   | 限制对网络的外部访问             |
| --ip-range    |         | 从子范围分配容器ip             |
| --ipam-driver | default | IP地址管理驱动程序             |
| --ipam-opt    | map[]   | 设置IPAM驱动程序的具体选项        |
| --ipv6        | false   | 启用IPv6网络               |
| --label       |         | 在网络上设置元数据              |
| --opt, -o     | map[]   | 设置驱动程序特定选项             |
| --subnet      |         | 表示网段的CIDR格式的子网         |

**相关命令**

| 命令名称                      | 说明             |
| ------------------------- | -------------- |
| docker network connect    | 将容器连接到网络       |
| docker network create     | 创建一个网络         |
| docker network disconnect | 断开容器的网络        |
| docker network inspect    | 显示一个或多个网络的详细信息 |
| docker network ls         | 列出网络           |
| docker network prune      | 删除所有未使用的网络     |
| docker network rm         | 删除一个或多个网络      |

### 示例

**连接容器网络**

启动容器时，使用`--network`标志将其连接到网络。 此示例将`busybox`容器添加到`mynet`网络：

```shell
$ docker run -itd --network=mynet busybox
```

如果要在容器运行后将容器添加到网络，请使用`docker network connect`子命令。

**指定高级选项**

创建网络时，引擎默认为网络创建一个不重叠的子网。 该子网不是现有网络的细分。 它纯粹用于IP寻址目的。可以覆盖此默认值，并使用`--subnet`选项直接指定子网络值。 在桥接网络上，只能创建单个子网：

```shell
$ docker network create --driver=bridge --subnet=192.168.0.0/16 br0
```

另外，还可以指定`--gateway --ip-range`和`--aux-address`选项。

```shell
$ docker network create \
  --driver=bridge \
  --subnet=172.104.0.0/16 \
  --ip-range=172.104.0.0/24 \
  --gateway=172.104.0.1 \
  iot
```

如果省略`--gateway`标志，引擎将从首选池中选择一个。对于覆盖网络和支持它的网络驱动程序插件，可以创建多个子网络。

```shell
$ docker network create -d overlay \
  --subnet=192.168.0.0/16 \
  --subnet=192.170.0.0/16 \
  --gateway=192.168.0.100 \
  --gateway=192.170.0.100 \
  --ip-range=192.168.1.0/24 \
  --aux-address="my-router=192.168.1.5" --aux-address="my-switch=192.168.1.6" \
  --aux-address="my-printer=192.170.1.5" --aux-address="my-nas=192.170.1.6" \
  my-multihost-network
```

确保子网不重叠。如果重叠的话网络创建失败，并且引擎会返回错误。

**桥接驱动程序选项**

创建自定义网络时，默认的网络驱动程序(即bridge)具有可以传递的其他选项。

例如，使用`-o`或`--opt`选项在发布端口时指定IP地址绑定：

```shell
$ docker network create \
    -o "com.docker.network.bridge.host_binding_ipv4"="172.19.0.1" \
    simple-network
```

# **容器时间和宿主机同步**

```shell
# 在 run 时挂载宿主时间配置
-v /etc/localtime:/etc/localtime
​
# 复制宿主机 localtime 配置
$ docker cp /etc/localtime 容器ID:/etc/localtime
$ docker cp /usr/share/zoneinfo/Asia/Shanghai 容器ID:/etc/localtime
```

- **iptables配置docker服务端口访问限制**

[iptables配置docker服务端口访问限制_shay的博客-CSDN博客_iptables 开放docker端口](https://blog.csdn.net/zxxshaycormac/article/details/115450925)









