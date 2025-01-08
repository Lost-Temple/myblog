---
title: jenkins搭建
date: 2024-06-13 08:24:41
tags:
  - jenkins
categories:
  - 云原生
  - 基础设施
cover: 'https://s2.loli.net/2024/06/13/EjX26P3MvTCQoVl.jpg'
---

# 拉取jenkins新版

```
docker pull jenkins/jenkins:latest
```

> 最近国内docker仓库镜像似乎都已下架，如果使用国内镜像仓库，拉到的可能都是旧版本


# 运行jenkins容器


```shell
docker run -itd -p 8082:8080 -p 50000:50000 --name jenkins --privileged=true -v /home/centos/jenkins:/var/jenkins_home jenkins/jenkins:latest
```

# 安装常用插件

- [Command Agent Launcher Plugin版本107.v773860566e2e](https://plugins.jenkins.io/command-launcher)
- [Extended Choice Parameter Plugin版本382.v5697b_32134e8](https://plugins.jenkins.io/extended-choice-parameter)
- [Git Parameter Plug-In版本0.9.19](https://plugins.jenkins.io/git-parameter)
- [JavaMail API版本1.6.2-10](https://plugins.jenkins.io/javax-mail-api)
- [Localization: Chinese (Simplified)版本371.v23851f835d6b_](https://plugins.jenkins.io/localization-zh-cn)
- [Multiple SCMs plugin版本0.8](https://plugins.jenkins.io/multiple-scms)
- [Oracle Java SE Development Kit Installer Plugin版本73.vddf737284550](https://plugins.jenkins.io/jdk-tool)
- [Publish Over SSH版本1.25](https://plugins.jenkins.io/publish-over-ssh)
- [SSH Agent Plugin版本367.vf9076cd4ee21](https://plugins.jenkins.io/ssh-agent)
- [SSH Build Agents plugin版本2.968.v6f8823c91de4](https://plugins.jenkins.io/ssh-slaves)
- [SSH plugin版本2.6.1](https://plugins.jenkins.io/ssh)
- [SSH server版本3.330.vc866a_8389b_58](https://plugins.jenkins.io/sshd)

> 需要科学上网安装插件
