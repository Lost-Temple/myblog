---
title: 怎么把K8S中部署的服务端口映射出来
tags:
  - 端口映射
  - k8s
categories:
  - 云原生
  - 基础设施
cover: 'https://s2.loli.net/2023/05/11/EUAPmihexjgcIH5.jpg'
abbrlink: 27436
date: 2023-06-30 17:56:12
---
1. 修改fate-10001命名空间下的mysql的端口映射模式
   - 先查看fate-10001下有哪些服务
   ```shell
   kubectl get svc -n fate-10001
   ```
    ```
    NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                               AGE
    spark-master         ClusterIP      None            <none>          8080/TCP,7077/TCP,6066/TCP            16d
    fateflow             ClusterIP      None            <none>          9360/TCP,9380/TCP                     16d
    spark-worker-1       ClusterIP      None            <none>          8081/TCP                              16d
    fateflow-client      ClusterIP      10.43.80.240    <none>          9360/TCP,9380/TCP                     16d
    datanode             ClusterIP      10.43.247.69    <none>          9000/TCP,9864/TCP                     16d
    frontend             NodePort       10.43.24.110    <none>          8080:31925/TCP,8443:31194/TCP         16d
    notebook             ClusterIP      10.43.41.5      <none>          20000/TCP                             16d
    fateboard            ClusterIP      10.43.3.44      <none>          8080/TCP                              16d
    namenode             ClusterIP      10.43.146.160   <none>          9000/TCP,9870/TCP                     16d
    pulsar               ClusterIP      10.43.42.170    <none>          6650/TCP,6651/TCP,8080/TCP,8081/TCP   16d
    postgres             ClusterIP      10.43.182.72    <none>          5432/TCP                              16d
    mysql                ClusterIP      10.43.42.233    <none>          3306/TCP                              16d
    nginx                NodePort       10.43.101.234   <none>          9300:31960/TCP,9310:32603/TCP         16d
    spark-client         ClusterIP      10.43.69.60     <none>          8080/TCP,7077/TCP,6066/TCP            16d
    site-portal-server   ClusterIP      10.43.237.196   <none>          8080/TCP,8443/TCP                     16d
    pulsar-public-tls    LoadBalancer   10.43.144.61    192.168.11.71   6651:32593/TCP                        16d
    ```
    > 注意msyql 的type 是 ClusterIP，这样就只能被k8s集群内部访问而不能被外部访问到，所以这里要把它改为NodePort模式

    - 修改mysql的type
    ```shell
    kubectl edit svc mysql -n fate-10001
    ```

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    annotations:
        meta.helm.sh/release-name: party10001
        meta.helm.sh/release-namespace: fate-10001
    creationTimestamp: "2023-06-13T05:46:12Z"
    labels:
        app.kubernetes.io/managed-by: Helm
        chart: fate
        cluster: fate
        fateMoudle: mysql
        heritage: Helm
        name: party10001
        owner: kubefate
        partyId: "10001"
        release: party10001
    name: mysql
    namespace: fate-10001
    resourceVersion: "33441"
    uid: 08d5e8fe-7853-4f85-b17c-9315afdd6055
    spec:
    clusterIP: 10.43.42.233
    clusterIPs:
    - 10.43.42.233
    internalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - name: tcp-mysql
        port: 3306
        protocol: TCP
        targetPort: 3306
    selector:
        fateMoudle: mysql
        name: party10001
        partyId: "10001"
    sessionAffinity: None
    type: ClusterIP
    status:
    loadBalancer: {}
    ```
    修改后的内容为：
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
    annotations:
        meta.helm.sh/release-name: party10001
        meta.helm.sh/release-namespace: fate-10001
    creationTimestamp: "2023-06-13T05:46:12Z"
    labels:
        app.kubernetes.io/managed-by: Helm
        chart: fate
        cluster: fate
        fateMoudle: mysql
        heritage: Helm
        name: party10001
        owner: kubefate
        partyId: "10001"
        release: party10001
    name: mysql
    namespace: fate-10001
    resourceVersion: "33441"
    uid: 08d5e8fe-7853-4f85-b17c-9315afdd6055
    spec:
    clusterIP: 10.43.42.233
    clusterIPs:
    - 10.43.42.233
    internalTrafficPolicy: Cluster
    externalTrafficPolicy: Cluster
    ipFamilies:
    - IPv4
    ipFamilyPolicy: SingleStack
    ports:
    - name: tcp-mysql
        port: 3306
        protocol: TCP
        targetPort: 3306
        nodePort: 31306
    selector:
        fateMoudle: mysql
        name: party10001
        partyId: "10001"
    sessionAffinity: None
    type: NodePort
    status:
    loadBalancer: {}
    ```