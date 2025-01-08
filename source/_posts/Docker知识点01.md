---
title: Docker知识点01
tags:
  - swarm
  - docker
  - mtu
  - bridge
  - overlay
  - network
categories:
  - 云原生
  - 基础设施
cover: 'https://s2.loli.net/2024/02/20/fnPqBZJia8KLhEW.png'
abbrlink: 59017
date: 2024-02-19 09:16:00
---

# 名词

| 名词         | 说明                                                                                                                                                                                                                                            |
|:-----------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Bridge 网络  | 1. Bridge 网络是 Docker 默认创建的网络模式。当您在 Docker 中创建一个新的容器时，默认会将容器连接到名为 bridge 的网络中 <br> 2. Bridge 网络允许容器与宿主机以及同一主机上的其他容器进行通信，但默认情况下不允许容器之间跨主机通信 <br> 3. Bridge 网络使用 Linux 系统上的桥接技术，在宿主机上创建一个虚拟的网桥设备，用于连接容器和主机的物理网络接口 <br> 4. Bridge 网络适用于单个主机上的容器通信 |
| Overlay 网络 | 1. Overlay 网络用于跨多个 Docker 主机构建容器集群，并允许集群中的容器之间进行跨主机通信 <br> 2. Overlay 网络使用了 VxLAN 技术，在不同主机上创建虚拟的 Overlay 网络，容器可以通过该网络进行通信 <br> 3. Overlay 网络通常用于 Docker Swarm 集群或 Kubernetes 集群中，用于构建分布式应用和微服务架构 <br> 4. Overlay 网络适用于跨多个主机的容器通信              |
| MTU        | 最大传输单元MTU（Maximum Transmission Unit，MTU），是指网络能够传输的最大数据包大小，以字节为单位。MTU的大小决定了发送端一次能够发送报文的最大字节数。如果MTU超过了接收端所能够承受的最大值，或者是超过了发送路径上途经的某台设备所能够承受的最大值，就会造成报文分片甚至丢弃，加重网络传输的负担。如果太小，那实际传送的数据量就会过小，影响传输效率。                                                |

# Overlay网络

docker swarm在启动的过程中会创建两个默认的网络：docker_gwbridge和ingress.

- **docker_gwbridge**：通过这个网络，容器可以连接到宿主机。（它的driver就是bridge)
- **ingress**：由docker swarm创建的overlay网络，这个网络用于将服务暴露给外部访问，docker swarm就是通过它实现的routing mesh（将外部请求路由到不同主机的容器）。

例子如下：
```Shell
[centos@bd-2 ~]$ docker network ls
NETWORK ID     NAME                           DRIVER    SCOPE
69477a3560e9   bridge                         bridge    local
320cd9ae8475   centos_default                 bridge    local
d344065cc4cc   docker_gwbridge                bridge    local
da5d24e5d251   flow-admin-backend_admin-net   bridge    local
4d5b92424942   flow-admin-front_admin-net     bridge    local
qqfrntz0lrex   hadoop-com                     overlay   swarm
5cd8e1bc97a5   host                           host      local
mtysig457puz   ingress                        overlay   swarm
7ad98d08bd3c   none                           null      local  
```

# docker0的mtu
mtu查看
```Shell
ifconfig | grep mtu
```
如果宿主机的mtu比docker0的mtu还小，网络会存在问题，所以是需要修改docker0的mtu

我直接通过修改/etc/docker/daemon.json文件：
```JSON
{
    "log-driver":"json-file",
    "log-opts": {"max-size":"200m", "max-file":"3"},
    "live-restore": false,
    "mtu": 1450,
    "insecure-registries":["192.168.10.166:88","192.168.9.68","192.168.11.149:8888","192.168.11.168:8888"]
}
```

重启docker

```Shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```

重新查看mtu，如果docker中此时没有容器在运行，那会发现docker0的mtu还是原先的值。这里有坑：当docker中有容器在运行时，再去查看docker0的mtu就生效了。
强迫症可以修改daemon.json配置，重启docker服务后，再使用命令临时修改docker0的mtu：
```Shell
sudo ip link set docker0 mtu 1450
```

# Swarm集群搭建
初始化manager节点
```Shell
docker swarm init --advertise-addr 192.168.10.91:2377
```
正常情况输出类似以下的提示：
```Shell
Swarm initialized: current node (tg66scjfm2kgw5z7pb7vpnrye) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-4e22sso1h4dofwqzq25a6crgc9fzk4it6237jdd6ezuzqknsw5-79qgv3b9sc88wa8jk5etylpty 192.168.10.91:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

后续想要查看怎么加入到swarm集群，使用以下命令:
```Shell
# 获取Manager节点加入命令
docker swarm join-token manager

# 获取Worker节点加入命令
docker swarm join-token worker
```

可以到work节点上执行：
```Shell
docker swarm join --token SWMTKN-1-4e22sso1h4dofwqzq25a6crgc9fzk4it6237jdd6ezuzqknsw5-79qgv3b9sc88wa8jk5etylpty 192.168.10.91:2377

```

如果是多manager的话，可以在其它manager节点执行：
```Shell
docker swarm join --token SWMTKN-1-4e22sso1h4dofwqzq25a6crgc9fzk4it6237jdd6ezuzqknsw5-2ccx8qd17rc0ydci0kcw9qfy2 192.168.10.91:2377
```

> 这里有坑，就是docker的daemon.json中配置的mtu，对于swarm的overlay网络是不起作用的。

修改overlay网络的mtu和eth0,docker0的mtu保持一致
1. 在manager节点中先获取子网信息
    ```Shell
    docker network inspect -f '{{json .IPAM}}' docker_gwbridge
    ```
   返回：
    ```Shell
    {"Driver":"default","Options":null,"Config":[{"Subnet":"172.18.0.0/16","Gateway":"172.18.0.1"}]} 
    ```
2. manager节点退出swarm集群(自定义docker_gwbridge网络，则必须在将 Docker 主机加入 swarm 之前或暂时从 swarm 中移除后进行)
    ```Shell
    docker swarm leave --force
    ```
3. manager节点停掉docker服务
    ```Shell
    sudo systemctl stop docker.service
    ```
4. manager节点中删掉虚拟网卡docker_gwbridge
    ```Shell
    sudo ip link set docker_gwbridge down
    sudo ip link del dev docker_gwbridge
    ```
5. manager节点中启动docker
    ```Shell
    sudo systemctl start docker.service
    ```
6. manager节点中重建docker_gwbridge（这一步用到了第1步获取的子网信息以及设置我们要的mtu值）
    ```Shell
    docker network rm docker_gwbridge
    docker network create \
      --subnet 172.18.0.0/16 \
      --gateway 172.18.0.1 \
      --opt com.docker.network.bridge.name=docker_gwbridge \
      --opt com.docker.network.bridge.enable_icc=false \
      --opt com.docker.network.bridge.enable_ip_masquerade=true \
      --opt com.docker.network.driver.mtu=1450 \
      docker_gwbridge
    ```
   **再到work节点上执行相同的命令执行1~6步骤**完成docker_gwbridge网络的自定义创建

7. manager节点中查看ingress网络信息
   ```Shell
   docker network inspect -f '{{json .IPAM}}' ingress
   ```
   返回
   ```Shell
   {"Driver":"default","Options":null,"Config":[{"Subnet":"10.0.0.0/24","Gateway":"10.0.0.1"}]}
   ```
8. manager节点中删除ingress network
   ```Shell
   docker network rm ingress
   ```
   > 这一步会有个警告

   ```Shell
   WARNING! Before removing the routing-mesh network, make sure all the nodes in your swarm run the same docker engine version. Otherwise, removal may not be effective and functionality of newly create ingress networks will be impaired.
   ```
   > 所以swarm中的节点的docker版本最好保持一致

9. manager节点中重建ingress(记得使用之前查看的ingress网络中的子网以及网关，mtu自定义修改)
   ```Shell
   docker network create \
     --driver overlay \
     --ingress \
     --subnet=10.0.0.0/24 \
     --gateway=10.0.0.1 \
     --opt com.docker.network.driver.mtu=1450 \
     ingress
   ```

10. 然后就是其它的manager节点/worker节点的加入了
   > **注意：新机器在join到swarm之前，得先重建docker_gwbridge(mtu得保持一致）**

11. 验证mtu是否都按我们定义的修改了
      启动一个swarm service
   ```Shell
   docker service create -td --name busybox busybox
   ```
   查看mtu
   ```Shell
   ifconfig | grep mtu
   ```
   返回
   ```Shell
   docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
   docker_gwbridge: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
   eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
   lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
   veth21384b6: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
   veth41d39c5: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
   ```

