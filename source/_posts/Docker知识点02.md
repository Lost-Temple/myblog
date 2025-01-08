---
title: Docker知识点02
tags:
  - swarm
  - docker stack
  - overlay
  - hadoop
  - spark
categories:
  - 云原生
  - 基础设施
cover: 'https://s2.loli.net/2024/02/20/fnPqBZJia8KLhEW.png'
abbrlink: 59337
date: 2024-05-17 10:21:25
---

# 前言

需要搭建spark集群以及hadoop集群，并且使它间之间进行交互。目标：编写一个yaml，用于一键部署。本文先试验一个个部署，最终形成一个一键部署的方案。

# 实践

## hadoop集群单机部署试验

1. 拉取appache官方镜像

   ```she
   docker pull apache/hadoop
   ```

2. 在服务器本地目录创建所需文件

   - 创建`docker-compose.yaml`文件

     ```yaml
     version: "3"
     services:
        namenode:
           image: apache/hadoop:3
           networks:
             - hadoop_network
           hostname: namenode
           command: ["hdfs", "namenode"]
           ports:
             - 9870:9870
             - 8020:8020
           env_file:
             - ./config
           environment:
               ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
        datanode:
           image: apache/hadoop:3
           networks:
             - hadoop_network
           command: ["hdfs", "datanode"]
           env_file:
             - ./config
        resourcemanager:
           image: apache/hadoop:3
           networks:
             - hadoop_network
           hostname: resourcemanager
           command: ["yarn", "resourcemanager"]
           ports:
              - 8088:8088
           env_file:
             - ./config
           volumes:
             - ./test.sh:/opt/test.sh
        nodemanager:
           image: apache/hadoop:3
           networks:
             - hadoop_network
           command: ["yarn", "nodemanager"]
           env_file:
             - ./config
     networks:
       hadoop_network:
         name: hadoop-net
     ```
     
     > 自定义网络，这样定义是会自动创建这个网络的
     
     如果网络已经存在，那可以这样指定(在后续我们一键部署时这个很重要，需要让每个容器在同一网络内，`docker stack` 部署还需要这个网络为`overlay`网络)
     
     ```yaml
     networks:
       hadoop_network:
         external: true
         name: hadoop-net
     ```

   - 上述文件中有一个`config`文件用来设置一些环境变量，所以我们再创建一个`config`文件

     ```shell
     CORE-SITE.XML_fs.default.name=hdfs://namenode
     CORE-SITE.XML_fs.defaultFS=hdfs://namenode
     HDFS-SITE.XML_dfs.namenode.rpc-address=namenode:8020
     HDFS-SITE.XML_dfs.replication=1
     MAPRED-SITE.XML_mapreduce.framework.name=yarn
     MAPRED-SITE.XML_yarn.app.mapreduce.am.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
     MAPRED-SITE.XML_mapreduce.map.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
     MAPRED-SITE.XML_mapreduce.reduce.env=HADOOP_MAPRED_HOME=$HADOOP_HOME
     YARN-SITE.XML_yarn.resourcemanager.hostname=resourcemanager
     YARN-SITE.XML_yarn.nodemanager.pmem-check-enabled=false
     YARN-SITE.XML_yarn.nodemanager.delete.debug-delay-sec=600
     YARN-SITE.XML_yarn.nodemanager.vmem-check-enabled=false
     YARN-SITE.XML_yarn.nodemanager.aux-services=mapreduce_shuffle
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-applications=10000
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.maximum-am-resource-percent=0.1
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.resource-calculator=org.apache.hadoop.yarn.util.resource.DefaultResourceCalculator
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.queues=default
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.capacity=100
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.user-limit-factor=1
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.maximum-capacity=100
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.state=RUNNING
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_submit_applications=*
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.root.default.acl_administer_queue=*
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.node-locality-delay=40
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings=
     CAPACITY-SCHEDULER.XML_yarn.scheduler.capacity.queue-mappings-override.enable=false
     ```

     > 这里面没有指定`HADOOP_HOME`的值，部署的时候会有一个警告，但也不影响使用，后来我就在这个文件中，这个变量使用前，添加了一行`HADOOP_HOME=/opt/hadoop`

3. 检查所需文件

   ```shell
   [centos@maodaoming-1 hadoop]$ ll
   total 8
   -rw-rw-r--. 1 centos centos 1789 May 17 09:21 config
   -rw-rw-r--. 1 centos centos  984 May 17 11:04 docker-compose.yaml
   ```

# spark 集群单机部署

因为是搭建FATE 2.1.0 版本的spark环境，所以使用的是使用FATE-Builder打包出来的spark镜像，包含：

```shell
yunpcds/spark-master        2.1.0-release   115b928c3b3d   21 hours ago    4.68GB
yunpcds/spark-worker        2.1.0-release   371c1de5aabe   21 hours ago    4.68GB
```

1. 创建`docker-compose.yaml`文件

```yaml
version: "3"

services:
  spark-master:
    hostname: spark-master
    image: ${IMAGE_PREFIX}/spark-master:${IMAGE_TAG}
    container_name: spark-master
    restart: always
    ports:
      - "8080:8080"
      - "7077:7077"
      - "6066:6066"

  spark-slave:
    hostname: spark-slave
    image: ${IMAGE_PREFIX}/spark-worker:${IMAGE_TAG}
    container_name: spark-slave
    restart: always
    ports:
      - "8781:8081"
    volumes:
      - /data/projects/:/data/projects/
    environment:
      SERVICE_PRECONDITION: "spark-master:7077"
      EGGROLL_HOME: "/data/projects/fate/eggroll"
      FATE_PROJECT_BASE: "/data/projects/fate"
      VIRTUAL_ENV: "/data/projects/python/venv"
      PYTHONPATH: "/data/projects/fate/:/data/projects/fate/eggroll/python:/data/projects/fate/fate/python:/data/projects/fate/fateflow/python:/data/projects/fate/python"
      PATH: "/data/projects/python/venv/bin:/opt/hadoop-3.2.3/bin:/opt/hive/bin:/opt/spark-3.1.3-bin-hadoop3.2/bin:/opt/hadoop-3.2.3/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
networks:
  spark_network:
    name: spark-net
```

2. 变量设置文件:<font size=5>`.env`</font>文件

```shell
IMAGE_PREFIX=yunpcds
IMAGE_TAG=2.1.0-release
```

> 这里面的变量会被传入到`docker-compose.yaml`中去,上面的PYTHONPATH, PATH根据实际情况再调整；还有<font size=5>volumes</font>的映射，需要把fate的代码拷贝到宿主机的对应目录`/data/projects/`

3. 查看所需文件

   ```shell
   [centos@maodaoming-2 spark]$ ll -a
   total 8
   drwxrwxr-x. 2 centos centos   45 May 17 15:33 .
   drwxrwxr-x. 4 centos centos   33 May 17 14:38 ..
   -rw-rw-r--. 1 centos centos 1103 May 17 14:56 docker-compose.yaml
   -rw-rw-r--. 1 centos centos   45 May 17 14:39 .env
   ```

# 一键部署

把`docker-compose.yaml`合并成一份，使用docker stack部署：

```yaml
version: "3"
services:
  namenode:
    image: apache/hadoop:3
    hostname: namenode
    command: ["hdfs", "namenode"]
    networks:
      - bigdata-network
    ports:
      - target: 9870
        published: 9870
        protocol: tcp
        mode: host
      - target: 9000
        published: 9000
        protocol: tcp
        mode: host
    env_file:
      - ./config
    environment:
      ENSURE_NAMENODE_DIR: "/tmp/hadoop-root/dfs/name"
    deploy:
      endpoint_mode: dnsrr
      placement:
        constraints:
          - 'node.labels.nodename == node55'
  datanode:
    image: apache/hadoop:3
    command: ["hdfs", "datanode"]
    networks:
      - bigdata-network
    env_file:
      - ./config
    ports:
      - target: 1004
        published: 1004
        protocol: tcp
        mode: host
      - target: 1006
        published: 1006
        protocol: tcp
        mode: host
      - target: 9866
        published: 9864
        protocol: tcp
        mode: host
    deploy:
      endpoint_mode: dnsrr
      placement:
        constraints:
          - 'node.labels.nodename == node55'
  resourcemanager:
    image: apache/hadoop:3
    hostname: resourcemanager
    command: ["yarn", "resourcemanager"]
    networks:
      - bigdata-network
    ports:
      - 8088:8088
    env_file:
      - ./config
    deploy:
      placement:
        constraints:
          - 'node.labels.nodename == node55'
  nodemanager:
    image: apache/hadoop:3
    command: ["yarn", "nodemanager"]
    networks:
      - bigdata-network
    env_file:
      - ./config
    deploy:
      placement:
        constraints:
          - 'node.labels.nodename == node55'

  spark-master:
    hostname: spark-master
    image: ${IMAGE_PREFIX}/spark-master:${IMAGE_TAG}
    container_name: spark-master
    restart: always
    networks:
      - bigdata-network
    ports:
      - "8080:8080"
      - "7077:7077"
      - "6066:6066"
    deploy:
      placement:
        constraints:
          - 'node.role == manager'
          - 'node.labels.nodename == node91'
  spark-slave:
    hostname: spark-worker
    image: ${IMAGE_PREFIX}/spark-worker:${IMAGE_TAG}
    container_name: spark-worker
    restart: always
    networks:
      - bigdata-network
    ports:
      - "8781:8081"
    volumes:
      - /data/projects/:/data/projects/
    environment:
      SERVICE_PRECONDITION: "spark-master:7077"
      EGGROLL_HOME: "/data/projects/fate/eggroll"
      FATE_PROJECT_BASE: "/data/projects/fate"
      VIRTUAL_ENV: "/data/projects/python/venv"
      PYTHONPATH: "/data/projects/fate/:/data/projects/fate/eggroll/python:/data/projects/fate/fate/python:/data/projects/fate/fateflow/python:/data/projects/fate/python"
      PATH: "/data/projects/python/venv/bin:/opt/hadoop-3.2.3/bin:/opt/hive/bin:/opt/spark-3.1.3-bin-hadoop3.2/bin:/opt/hadoop-3.2.3/bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    deploy:
      placement:
        constraints:
          - 'node.role == manager'
          - 'node.labels.nodename == node91'
volumes:
  hadoop_namenode:
  hadoop_datanode1:
  hadoop_datanode2:
  hadoop_datanode3:
  hadoop_historyserver:
  kerberos_db:
  kerberos_keytab:
    driver_opts:
      type: nfs
      o: addr=192.168.11.74,nfsvers=4,minorversion=0,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport
      device: :/home/centos/nfs_share/volume/bigdata/keytab
  hive_metastore:
  mysql_data:
  linkis_log:
  linkis_runtime:
networks:
  bigdata-network:
    external: true
    name: bigdata-network

```

> 注意：有volumes映射的一定要注意，如果路径不存在就启动失败

启动脚本编写：

```shell
docker stack rm bigdata

sleep 1s

echo "请注意清理nodedata的volume!!!"

[ -f .env ] && export $(sed '/^#/d' .env)
docker stack deploy -c docker-stack-deploy.yaml bigdata
```

启动后检查是否成功：

```shell
[centos@maodaoming-2 flow-spark-v2-env]$ docker service ls

gzpqsvib86y6   bigdata_datanode          replicated   1/1        apache/hadoop:3
proujh710xqf   bigdata_namenode          replicated   1/1        apache/hadoop:3
fyok561q2qiv   bigdata_nodemanager       replicated   1/1        apache/hadoop:3
72gzt56c9yrp   bigdata_resourcemanager   replicated   0/1        apache/hadoop:3                      *:8088->8088/tcp
opo2n1jaa0bj   bigdata_spark-master      replicated   1/1        yunpcds/spark-master:2.1.0-release   *:6066->6066/tcp, *:7077->7077/tcp, *:8080->8080/tcp
rsxhqoanjd3y   bigdata_spark-worker      replicated   1/1        yunpcds/spark-worker:2.1.0-release   *:8781->8081/tcp
chwttx82rp8u   busybox                   replicated   1/1        busybox:latest
```

可以查看到`bigdata_resourcemanager`是启动失败的。可以用以下命令查看日志：

```shell
[centos@maodaoming-2 flow-spark-v2-env]$ docker service ps  --no-trunc bigdata_resourcemanager

z8i8jbqpzz8zbmkh0w9k8i91o   bigdata_resourcemanager.1       apache/hadoop:3@sha256:af361b20bec0dfb13f03279328572ba764926e918c4fe716e197b8be2b08e37f   maodaoming-1.novalocal   Ready           Rejected 1 second ago     "invalid mount config for type "bind": bind source path does not exist: /home/centos/flow-spark-v2-env/test.sh"
spmxmabdm0gikj33y4bxoot51    \_ bigdata_resourcemanager.1   apache/hadoop:3@sha256:af361b20bec0dfb13f03279328572ba764926e918c4fe716e197b8be2b08e37f   maodaoming-1.novalocal   Shutdown        Rejected 6 seconds ago    "invalid mount config for type "bind": bind source path does not exist: /home/centos/flow-spark-v2-env/test.sh"
zo68mawn5flwwejynq54iwus0    \_ bigdata_resourcemanager.1   apache/hadoop:3@sha256:af361b20bec0dfb13f03279328572ba764926e918c4fe716e197b8be2b08e37f   maodaoming-1.novalocal   Shutdown        Rejected 11 seconds ago   "invalid mount config for type "bind": bind source path does not exist: /home/centos/flow-spark-v2-env/test.sh"
60ew6r1iqf9wppbc3dojq0566    \_ bigdata_resourcemanager.1   apache/hadoop:3@sha256:af361b20bec0dfb13f03279328572ba764926e918c4fe716e197b8be2b08e37f   maodaoming-1.novalocal   Shutdown        Rejected 17 seconds ago   "invalid mount config for type "bind": bind source path does not exist: /home/centos/flow-spark-v2-env/test.sh"
nk9rzlaqqw2e1rr4ali2634mp    \_ bigdata_resourcemanager.1   apache/hadoop:3@sha256:af361b20bec0dfb13f03279328572ba764926e918c4fe716e197b8be2b08e37f   maodaoming-1.novalocal   Shutdown        Rejected 21 seconds ago   "invalid mount config for type "bind": bind source path does not exist: /home/centos/flow-spark-v2-env/test.sh"
```

