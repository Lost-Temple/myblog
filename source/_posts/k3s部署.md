---
title: k3s部署
tags:
  - k3s
categories:
  - 云原生
  - 基础设施
cover: 'https://s2.loli.net/2023/05/11/EUAPmihexjgcIH5.jpg'
abbrlink: 41448
date: 2023-05-11 14:07:04
---

# 一、[官方文档](https://docs.k3s.io/zh/quick-start)

# 二、安装步骤
- 使用docker作为容器(默认为containerd)
- 设置docker拉取的http代理
- 安装K3S
- 使用ingress-nginx(默认是traefik)


## 使用docker作为容器
- 可以使用`Rancher`的一个[Docker安装脚本](https://github.com/rancher/install-docker)来安装Docker
- 也可以根据Docker官方文档手动[ubuntu下安装docker](https://docs.docker.com/engine/install/ubuntu/)
### 卸载旧版
```shell
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 设置`repository`

```shell
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

```shell
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

```shell
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### 安装`Docker Engine`

```shell
sudo apt-get update
```

```shell
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 设置docker拉取的http代理

```shell
sudo mkdir -p /etc/systemd/system/docker.service.d
cd /etc/systemd/system/docker.service.d
sudo vim proxy.conf
```
文件内容如下：
```
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890/"
Environment="HTTPS_PROXY=http://127.0.0.1:7890/"
Environment="NO_PROXY=localhost,127.0.0.1"
```

## 安装`K3S`
以Docker为容器
```shell
curl -sfL https://get.k3s.io | sh -s - --docker
```
> 如果服务器网络环境复杂，需要指定K3S服务节点的IP，则 `--node-ip=192.168.10.100`

## 使用ingress-nginx作为反向代理
之前在K3S搭建的环境中，默认是使用`traefik`的，在使用FedLCM部署FATE相关组件，里面使用了ingress-nginx，如果使用FedLCM去部署ingress-nginx，则会有一个坑：这个ingress-nginx在k3s中是部署成功了，但是FedLCM使用K8S的API去查询ingress-nginx是否部署完成返回的值是跟实际情况不符的。如果把`traefik`禁用了，就能避坑，这应该是K3S的一个bug(目前应该是解决了，后来我又部署过几次，没这个问题了)。

> 解决方法是：只需使用 `--disable=traefik` 启动 K3s Server，然后部署你的 Ingress 即可。

具体步骤：
- 安装kubectl
  ```shell
  # 下载kubectl
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  # 安装kubectl
  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
  ```
  ```shell
  mkdir -p ~/.kube
  sudo cat /etc/rancher/k3s/k3s.yaml
  # 在~/.kube/下创建一个`config`文件把k3s.yaml中的内容放入到`config`中
  ```
- 安装helm
  ```shell
  curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
  ```
- 安装 ingress-nginx
  ```shell
  helm upgrade --install ingress-nginx ingress-nginx  --repo https://kubernetes.github.io/ingress-nginx  --namespace ingress-nginx --create-namespace
  ```
- 查看 ingress-nginx controller信息
  ```shell
  kubectl get svc -n ingress-nginx ingress-nginx-controller
  ```
  ingress controller的服务类型有`NodePort`和`LoadBalancer`,这两种模式下，访问那些通过ingress反向代理的服务时，使用的IP地址是有区别的，`NodePort`模式下，没有`EXTERNAL-IP`

