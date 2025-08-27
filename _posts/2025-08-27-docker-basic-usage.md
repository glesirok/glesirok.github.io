---
title: docker基础使用
date: 2025-08-27 17:30 +0800
categories:
  - IT
  - Tutorial
tags:
  - Docker
comments: true
---
# 安装 for debain

## 配置docker apt仓库
[链接](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

## 安装

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 验证
```bash
sudo docker run hello-world
```

# 使用
## 查看镜像

```bash
docker image ls
```

## 查看容器

```bash
docker ps -a
```

## 拉去镜像示例

```bash
docker pull mysql:latest
```

## 删除镜像

```bash
docker rmi 镜像名
```

## 端口映射

-p 端口1:端口2， 其中端口1是主机端口，而2是容器端口

## 挂载

**容器的挂载类型：**

- `mount`：将宿主机的指定目录挂载到容器的指定目录，**以覆盖的形式挂载**（这也就意味着，容器指定目录下的内容也会随着消失）
- `volume`：**在宿主机的 Docker 存储目录下创建一个目录**，并挂载到容器的指定目录（并不会覆盖容器指定目录下的内容）

在有些时候，由于容器内的目录有着特殊作用，并不能以覆盖的形式进行挂载。但又想挂载到宿主机上，这时我们便可以使用 `volume` 类型的挂载方式。像我们上面所说的 `mount` 和 `volume` 都是支持以这两种类型的方式挂载，无非就是配置稍有不同。

两种命令使用 `mount` 类型挂载区别：当宿主机上指定的目录不存在时，我们使用 `volume` 挂载时，便会自动的在宿主机上创建出相应目录，而我们要是使用 `mount` 来挂载，便会输出 报错信息。

## 停止容器

```bash
docker stop [容器id]
```

## 删除容器

```bash
docker rm 容器ID
```

## 重启容器

```bash
docker start 容器ID :启动一个或多个已经被停止的容器

docker stop 容器ID :停止一个运行中的容器

docker restart 容器ID :重启容器

docker run --restart=always 在启动容器时，只要加上参数`--restart=always`就可以实现自动重启了
```

## 容器日志可以通过Docker的容器日志获得：

```bash
docker logs some-mysql
```

## 通过Docker命令进入Mysql容器内部,mysql是容器名，可以用序号代替

```bash
docker exec -it mysql /bin/bash
```

## 运行容器

`docker run -itd`解释

这三个参数(-i, -t, -d)是啥意思

|Options|Mean|
|---|---|
|-i|以交互模式运行容器，通常与 -t 同时使用；|
|-t|为容器重新分配一个伪输入终端，通常与 -i 同时使|
|-d|后台运行容器，并返回容器ID；|

大多数情况下，我们都是希望Docker能够在后台运行容器，而不是直接在宿主机上与之交互。如果不加-d选项，容器会在宿主机终端运行，并且把输出的结果（STDOUT）打印到宿主机上面。

使用-d选项后，容器会在后台运行，并输出容器ID，我们可以使用docker logs [containerID]来查看容器的输出结果。如果想跟容器进行交互，可以使用docker exec -it [container ID] /bin/bash来操作。

如果容器启动的进程是一个shell程序的话，我们通过-i选项就可以与之交互，保持容器持续运行。不指定-i，容器启动后就会自动退出。

如果只指定了-i选项，我们在宿主机终端输入任何字符都没有反应。这个问题，我们可以用-it来解决，所以我们接下来就介绍一下-t选项的作用。

简单来说，指定-t而不指定-i，意味着在容器里开启了一个伪终端，但是我们的输入并不会传递到伪终端的输入。

### 解释： 为什么不只用 -d

多数镜像可能执行 CMD ["/bin/bash"] 也就是说容器在后台启动成功后，执行了 COMMAND 命令后直接关闭了

# 注意

1. 一般情况下，安装了docker之后会创建docker组，只有root和docker组成员才能使用docker命令
    
2. 加入 docker 组会获得等同于 root 权限，docker 组权限的本质：
    - docker 组成员可以直接执行 docker 命令
    - 可以启动容器并挂载宿主机的任何目录
    - 可以在容器中以 root 身份运行命令
    
    例如，一个组内用户可以这样获取 root 权限 ：
```bash
    docker run -v /:/host -it ubuntu  # 挂载整个根目录  
```
    
    在容器内可以访问/修改宿主机的所有文件，或者直接在宿主机运行命令：
```bash
    docker run -it --pid=host ubuntu killall someprocess
```