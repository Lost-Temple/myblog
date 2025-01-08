---
title: 手动部署KubeFate
abbrlink: 14503
date: 2023-06-09 09:39:10
tags:
---

# 依赖的软件环境
- k8s：我使用的是k3s
- ingress-nginx
- 拉取KubeFate的代码并编译生成kubefate可执行文件：
```shell
git clone https://github.com/FederatedAI/KubeFATE.git
cd kubefate
make
```
> 官方文档有推荐版本，我没去管它

# 创建k8s服务的帐号以及namespace等
在`KubeFATE/k8s-deploy`目录下，有个`rbac-config.yaml`文件，里面有定义了帐号，命名空间等内容：
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kube-fate
  labels:
    name: kube-fate
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kubefate-admin
  namespace: kube-fate
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubefate
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubefate-role
subjects:
  - kind: ServiceAccount
    name: kubefate-admin
    namespace: kube-fate
---
apiVersion: v1
kind: Secret
metadata:
  name: kubefate-secret
  namespace: kube-fate
type: Opaque
stringData:
  kubefateUsername: admin
  kubefatePassword: admin
  mariadbUsername: kubefate
  mariadbPassword: kubefate
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kubefate-role
  namespace: kube-fate
rules:
- apiGroups:
  - ""
  resources:
  - namespaces
  - configmaps
  - services
  - secrets
  - persistentvolumeclaims
  - serviceaccounts
  verbs:
  - get
  - list
  - create
  - delete
  - update
  - patch
- apiGroups:
  - ""
  resources:
  - pods
  - pods/log
  - nodes
  verbs:
  - get
  - list
- apiGroups:
  - apps
  resources:
  - deployments
  - statefulsets
  - deployments/status
  - statefulsets/status
  verbs:
  - get
  - list
  - create
  - delete
  - update
  - patch
- apiGroups:
  - networking.k8s.io
  resources:
  - ingresses
  verbs:
  - get
  - list
  - create
  - delete
  - update
  - patch
- apiGroups:
  - networking.istio.io
  resources:
  - gateways
  - virtualservices
  verbs:
  - get
  - create
  - delete
  - update
  - patch
- apiGroups:
  - rbac.authorization.k8s.io
  resources:
  - roles
  - rolebindings
  verbs:
  - get
  - create
  - delete
  - update
  - patch
- apiGroups:
    - batch
  resources:
    - jobs
  verbs:
    - get
    - list
    - create
    - delete
    - update
    - patch
```
> 如果需要修改用户名密码之类的，可以查看上述yaml中的配置自行修改

在`KubeFATE/k8s-deploy`目录下执行：
```shell
sudo kubectl apply -f ./rbac-config.yaml
```

# 准备域名及部署KubeFATE服务
因为KubeFATE服务向外暴露RESTful APIs，所以系统管理员需要为此准备域名的DNS解析。在我们的配置样例里，使用`example.com`作为域名（需要修改成真实域名，但是为了学习搭建就无所谓了）。同时，系统管理员需要为每个FATE集群创建一个`namespace`，譬如`party_id`为`9999`的FATE集群，创建一个`fate-9999`的`namespace`，然后进行配额控制。

```shell
kubectl apply -f ./kubefate.yaml
kubectl create namespace fate-9999
```

后续再执行：
```shell
sudo kubectl get ingress -A
```
可以看到返回类似于以下信息：
```shell
NAMESPACE   NAME       CLASS   HOSTS         ADDRESS       PORTS   AGE
kube-fate   kubefate   nginx   example.com   10.43.62.61   80      12m
```
> ADDRESS 10.43.62.61 添加至/etc/hosts中
```shell
sudo vim /etc/hosts
```
```
10.43.62.61 example.com
```

查看ingress-nginx命名空间下的服务：
```shell
sudo kubectl get svc -n ingress-nginx
```

可以看到返回类似于以下信息：
```shell
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller-admission   ClusterIP   10.43.169.125   <none>        443/TCP                      4d
ingress-nginx-controller             NodePort    10.43.62.61     <none>        80:31683/TCP,443:31292/TCP   4d
```

```shell
sudo kubectl get svc -n kube-fate
```
返回类似以下信息：
```shell
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
mariadb    ClusterIP   10.43.128.75   <none>        3306/TCP   33m
kubefate   ClusterIP   10.43.235.20   <none>        8080/TCP   33m
```

完成以上步骤后，docker容器的启动情况查看一下：
```shell
sudo docker ps 
```
返回类似于以下信息：
```shell
CONTAINER ID   IMAGE                        COMMAND                   CREATED          STATUS          PORTS     NAMES
5ece9858fd52   93aad7121ec0                 "/kubefate service"       11 minutes ago   Up 11 minutes             k8s_kubefate_kubefate-74897fc6d7-zrcqf_kube-fate_d221b8f5-b6f1-4861-9281-5561f01875f3_2
116bb9217b97   9a79847e85fb                 "docker-entrypoint.s…"   12 minutes ago   Up 12 minutes             k8s_mariadb_mariadb-7456b6c968-b9zlx_kube-fate_2b1ac004-4155-4b11-bb3f-afd91e0563d2_0
5d13492b8755   rancher/mirrored-pause:3.6   "/pause"                  12 minutes ago   Up 12 minutes             k8s_POD_mariadb-7456b6c968-b9zlx_kube-fate_2b1ac004-4155-4b11-bb3f-afd91e0563d2_0
05635a80430c   rancher/mirrored-pause:3.6   "/pause"                  12 minutes ago   Up 12 minutes             k8s_POD_kubefate-74897fc6d7-zrcqf_kube-fate_d221b8f5-b6f1-4861-9281-5561f01875f3_0
```
> 至此，FATE的后台服务已经部署完成

# 准备FATE的安装配置文件并部署FATE
当系统管理员成功部署了`KubeFATE`服务，并准备好新的FATE集群`namespace`，接下来就可以根据这些信息开始部署FATE集群。`config.yaml`是使用`KubeFATE`命令行的配置文件，包含`KubeFATE`服务的访问用户名密码，以及`log`配置，`域名`。

```yaml
log:
  level: info
user:
  username: admin
  password: admin

serviceurl: example.com
safeconnect: false
```

| 名字 | 种类 | 描述 |
| -- | -- | -- |
| log | scalars | 命令行的日志级别。设置成debug级别可以看到REST通信信息 |
| user | mappings | KubeFATE的认证用户名，密码 |
| serviceurl | scalars | KubeFATE服务使用的域名 |
| safeconnect | scalars | 是否使用https访问KubeFATE服务使用的域名;[参考](https://github.com/FederatedAI/KubeFATE/blob/master/docs/configurations/kubefate_service_tls_enable.md) |

样例里面包含了`cluster.yaml`, 是FATE的部署计划，更多自定义内容参考：[FATE Cluster Configuration Guild](https://github.com/FederatedAI/KubeFATE/blob/master/docs/configurations/FATE_cluster_configuration.md)

因为我是要部署`Spark`作为计算引擎；`Pulsar`作为通讯组件的架构；所以选择的是样例里面的`cluster-spark-pulsar.yaml`
> 为了避免由于网络原因拉取docker镜像失败，需要设置一下`cluster-spark-pulsar.yaml'中的`registry`

```
registry: "hub.c.163.com/federatedai"
```

使用`kubefate`来安装集群, 在`k8s-deploy`目录下的`bin`目录下（之前`make`编译生成）：
```shell
kubefate cluster install -f ./cluster-spark-pulsar.yaml
```
```
create job Success, job id=0f82371a-442c-4a50-aeb5-528e5a896d52
```
> 这一步需要保证/etc/hosts中已添加了example.com，我在这一步卡了很久，一直不成功。

根据出错信息尝试了一些，最终解决掉了，但具体是哪个操作解决了目前没去深究，以下是我做的处理：
- 关闭科学上网软件（在终端内关闭科学上网，clash我忘了有没关，最好也关一下，之前遇到过在/etc/hosts添加了域名，但是clash开着导致hosts中的域名无法解析的坑，具体原因也不明）
- 移除了docker的代理（在 /etc/systemd/system/docker.service.d/proxy.conf中我原先设置了代理），后来把它mv proxy.conf proxy.conf.bak了
- 关闭了系统的防火墙
- 重启docker,重启k3s

# 检查安装集群任务的状态
```shell
kubefate job describe 0f82371a-442c-4a50-aeb5-528e5a896d52
```
```
UUID     	0f82371a-442c-4a50-aeb5-528e5a896d52
StartTime	2023-06-09 12:14:38
EndTime  	2023-06-09 12:23:44
Duration 	9m6s
Status   	Success
Creator  	admin
ClusterId	fd3e2768-a84d-4732-9de6-87558d7bcd39
States   	- update job status to Running
         	- create Cluster in DB Success
         	- helm install Success
         	- checkout Cluster status [529]
         	- job run Success

SubJobs  	mysql                ModuleStatus: Available, SubJobStatus: Success, Duration:   5m4s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:19:43
         	nginx                ModuleStatus: Available, SubJobStatus: Success, Duration:   9m6s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:23:44
         	python               ModuleStatus: Available, SubJobStatus: Success, Duration:   7m5s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:21:43
         	spark-worker         ModuleStatus: Available, SubJobStatus: Success, Duration:  2m53s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:17:31
         	client               ModuleStatus: Available, SubJobStatus: Success, Duration:  2m45s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:17:23
         	datanode             ModuleStatus: Available, SubJobStatus: Success, Duration:  2m36s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:17:14
         	fateboard            ModuleStatus: Available, SubJobStatus: Success, Duration:  2m54s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:17:32
         	namenode             ModuleStatus: Available, SubJobStatus: Success, Duration:   2m5s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:16:43
         	pulsar               ModuleStatus: Available, SubJobStatus: Success, Duration:  4m24s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:19:02
         	spark-master         ModuleStatus: Available, SubJobStatus: Success, Duration:  2m45s, StartTime: 2023-06-09 12:14:38, EndTime: 2023-06-09 12:17:23
```
```shell
kubefate cluster describe fd3e2768-a84d-4732-9de6-87558d7bcd39
```
```
UUID        	fd3e2768-a84d-4732-9de6-87558d7bcd39
Name        	fate-9999
NameSpace   	fate-9999
ChartName   	fate
ChartVersion	v1.11.1
Revision    	1
Age         	27m
Status      	Running
Spec        	algorithm: Basic
            	chartName: fate
            	chartVersion: v1.11.1
            	computing: Spark
            	device: CPU
            	federation: Pulsar
            	imagePullSecrets:
            	- name: myregistrykey
            	ingressClassName: nginx
            	istio:
            	  enabled: false
            	modules:
            	- mysql
            	- python
            	- fateboard
            	- client
            	- spark
            	- hdfs
            	- nginx
            	- pulsar
            	name: fate-9999
            	namespace: fate-9999
            	partyId: 9999
            	persistence: false
            	podSecurityPolicy:
            	  enabled: false
            	pullPolicy: null
            	registry: ""
            	skippedKeys:
            	- route_table
            	storage: HDFS

Info        	dashboard:
            	- spark.example.com
            	- notebook.example.com
            	- fateboard.example.com
            	- pulsar.example.com
            	ip: 192.168.100.121
            	status:
            	  containers:
            	    client: Running
            	    datanode: Running
            	    fateboard: Running
            	    fateflow: Running
            	    mysql: Running
            	    namenode: Running
            	    nginx: Running
            	    pulsar: Running
            	    spark-master: Running
            	    spark-worker: Running
            	  deployments:
            	    fateboard: Available
            	    nginx: Available
            	    spark-master: Available
            	    spark-worker: Available
            	  statefulSets:
            	    client: Available
            	    datanode: Available
            	    mysql: Available
            	    namenode: Available
            	    pulsar: Available
            	    python: Available
```