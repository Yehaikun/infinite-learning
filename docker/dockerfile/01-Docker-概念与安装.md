# Docker 概念与安装

**Metadata**
- Date: 2026-03-23
- Topic: Docker容器化技术
- Tags: #Docker #容器化 #DevOps #学习路径01

---

## 1. 什么是Docker

### 1.1 概念解释

Docker是一个开源的容器化平台，用于开发、部署和运行应用程序。它让你可以把应用程序及其所有依赖项打包到一个轻量级、可移植的容器中。

**一句话解释**：Docker就像"集装箱"，把应用程序和它的运行环境一起打包，这样应用在哪都能跑。

想象一下传统运输货物的方式：
- 以前：不同的货物需要不同的运输方式，货物之间可能相互影响
- 现在：使用标准集装箱，所有货物独立包装，统一运输

Docker做的事情就是这样：
- 应用程序代码 = 货物
- Docker容器 = 集装箱
- 运输船/火车 = 操作系统
- 目的地 = 服务器

### 1.2 为什么要用Docker

#### 传统部署方式的问题

在传统部署中，我们经常遇到"在我机器上能跑"的问题：

```
开发人员：在我电脑上能跑啊！
运维人员：服务器上怎么跑不起来？
```

**问题根源**：
1. **环境不一致**：开发环境、测试环境、生产环境配置不同
2. **依赖冲突**：不同项目需要不同版本的库
3. **部署复杂**：需要手动安装各种依赖
4. **资源浪费**：虚拟机太重，启动慢

让我详细解释一下这些问题：

**问题1：环境不一致**

开发人员的电脑可能是：
- macOS
- Python 3.10
- Node.js 18
- 本地MySQL 8.0

服务器上是：
- CentOS 7
- Python 3.8
- 没有Node.js
- MySQL 5.7

代码在开发环境能跑，在生产环境就可能出问题。

**问题2：依赖冲突**

假设你有两个项目：
- 项目A需要 Django 2.2
- 项目B需要 Django 4.0

在同一台服务器上安装这两个项目就会冲突。

**问题3：部署复杂**

传统部署流程：
```
1. 登录服务器
2. 安装操作系统依赖（gcc、make等）
3. 安装Python、Node.js等运行时
4. 克隆代码
5. 安装Python依赖（pip install -r requirements.txt）
6. 安装Node依赖（npm install）
7. 配置环境变量
8. 配置数据库连接
9. 配置Web服务器（Nginx）
10. 启动应用
11. 配置日志轮转
12. 配置监控
```

每台服务器都要重复这些步骤，而且可能因为版本差异导致问题。

**问题4：资源浪费**

传统虚拟机：
- 每个虚拟机需要完整的操作系统
- 占用大量磁盘空间（通常20-40GB）
- 启动慢（通常几分钟）
- 内存占用大

#### Docker如何解决问题

```
┌─────────────────────────────────────────────────────────────┐
│                      没有Docker之前                          │
├─────────────────────────────────────────────────────────────┤
│  应用代码 ──→ 拷贝到服务器 ──→ 安装Python ──→ 安装依赖 ──→ 运行 │
│              ↑                                                   │
│              └─ 每台服务器都要重复这些步骤                      │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      使用Docker之后                          │
├─────────────────────────────────────────────────────────────┤
│  应用代码 ──→ 打包成镜像 ──→ 一键部署到任意服务器              │
│              ↑                                                   │
│              └─ 镜像包含：代码 + Python + 所有依赖             │
└─────────────────────────────────────────────────────────────┘
```

使用Docker后：
- 镜像包含完整的运行环境
- 任何安装Docker的服务器都能运行
- 启动速度快（几秒钟）
- 资源占用小（通常几百MB）

---

## 2. Docker核心概念

### 2.1 镜像（Image）

**镜像**就是应用程序的"模板"或"快照"，包含了：
- 操作系统基础文件
- 应用程序代码
- 运行时环境（Python、Node.js、Java等）
- 依赖库
- 配置参数

**比喻**：
- 镜像就像是**类（Class）**
- 容器就像是**对象（Object）**
- 一个类可以创建多个对象，一个镜像可以运行多个容器

更具体的比喻：
- 镜像就像**食谱**（Recipe）
- 容器就像**按照食谱做出来的菜**（Dish）
- 有了食谱，可以做无数道菜

**镜像的结构**：

Docker镜像是由多个层（Layer）组成的，每一层代表一条指令。让我详细解释：

```
┌─────────────────────────────────────────┐
│           Docker镜像结构                  │
├─────────────────────────────────────────┤
│                                         │
│   ┌─────────────────────────────┐      │
│   │         容器层（ writable）  │      │  ← 最上层，可写
│   ├─────────────────────────────┤      │
│   │         Layer 5 (RUN)       │      │
│   ├─────────────────────────────┤      │
│   │         Layer 4 (COPY)     │      │
│   ├─────────────────────────────┤      │
│   │         Layer 3 (RUN)      │      │
│   ├─────────────────────────────┤      │
│   │         Layer 2 (COPY)    │      │
│   ├─────────────────────────────┤      │
│   │         Layer 1 (FROM)     │      │  ← 基础镜像层
│   └─────────────────────────────┘      │
│                                         │
└─────────────────────────────────────────┘
```

每一层都是只读的，只有最上面的容器层可以写入。这种分层结构的好处是：
1. 共享基础层，节省磁盘空间
2. 复用已有层，加速构建
3. 缓存构建结果

**镜像仓库**：
- Docker Hub：官方公共仓库（https://hub.docker.com）
- 阿里云容器镜像服务（https://cr.console.aliyun.com）
- 私有仓库

**常用镜像命令**：

```bash
# 搜索镜像（在Docker Hub搜索）
docker search nginx
# 输出：
# NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
# nginx                             Official build of Nginx.                        18980     [OK]       
# jwilder/nginx-proxy              Automated Nginx reverse proxy for docker …     2146                 [OK]

# 拉取镜像（不指定tag默认latest）
docker pull nginx:latest
docker pull nginx:1.25
docker pull openjdk:21-jdk-slim

# 拉取特定架构的镜像（Apple Silicon Mac用）
docker pull --platform linux/amd64 nginx
docker pull --platform linux/arm64 nginx

# 查看本地镜像
docker images
# 输出：
# REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
# nginx        latest    a6bd71f48f68   2 weeks ago    187MB
# openjdk      21        7e2024c2d9c3   3 weeks ago    470MB

# 查看镜像详细信息（JSON格式）
docker image inspect nginx:latest

# 只看镜像的某一部分
docker image inspect nginx:latest --format '{{.Os}}:{{.Architecture}}'
# 输出：linux:amd64

# 删除镜像
docker rmi nginx:latest

# 强制删除（即使容器在使用）
docker rmi -f nginx:latest

# 清理悬空镜像（没有被容器使用的镜像）
docker image prune

# 清理所有未使用的镜像
docker image prune -a

# 构建镜像
docker build -t myapp:1.0 .

# 给镜像打标签
docker tag myapp:1.0 myapp:latest

# 推送镜像到仓库
docker push username/myapp:1.0
```

### 2.2 容器（Container）

**容器**是镜像的运行实例，就像对象是类的实例一样。

**容器的特性**：
- 轻量级：共享宿主机内核，不需要虚拟化
- 独立：每个容器有自己的文件系统、网络、进程空间
- 可移植：可以在任何安装Docker的机器上运行
- 隔离：容器之间相互隔离，一个容器崩溃不影响其他容器

让我详细解释Docker容器与虚拟机的区别：

```
┌─────────────────────────────────────────────────────────────────┐
│                    虚拟机 vs Docker容器                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   虚拟机架构：                      Docker容器架构：            │
│   ┌──────────────────┐            ┌──────────────────┐         │
│   │    VM 1          │            │    Container 1   │         │
│   │ ┌────────────┐   │            │ ┌────────────┐   │         │
│   │ │  Guest OS  │   │            │ │   App      │   │         │
│   │ └────────────┘   │            │ └────────────┘   │         │
│   │ ┌────────────┐   │            └──────────────────┘         │
│   │ │  App + Lib │   │            ┌──────────────────┐         │
│   │ └────────────┘   │            │    Container 2   │         │
│   └──────────────────┘            │ ┌────────────┐   │         │
│   ┌──────────────────┐            │ │   App      │   │         │
│   │    VM 2          │            │ └────────────┘   │         │
│   │ ┌────────────┐   │            └──────────────────┘         │
│   │ │  Guest OS  │   │            ┌──────────────────┐         │
│   │ └────────────┘   │            │    Container 3   │         │
│   │ ┌────────────┐   │            │ ┌────────────┐   │         │
│   │ │  App + Lib │   │            │ │   App      │   │         │
│   │ └────────────┘   │            │ └────────────┘   │         │
│   └──────────────────┘            └──────────────────┘         │
│   ┌──────────────────┐                    │                     │
│   │    Hypervisor    │                    │                     │
│   │  (VMware/KVM)   │                    │                     │
│   └──────────────────┘                    │                     │
│           ↓                                ↓                     │
│   ┌─────────────────────────────────────────────┐               │
│   │              Host OS / Hardware            │               │
│   │         (Linux Kernel + Physical HW)        │               │
│   └─────────────────────────────────────────────┘               │
│                                                                 │
│   特点：                         特点：                          │
│   - 每个VM需要完整操作系统          - 容器共享宿主机内核          │
│   - 占用空间大（GB级）             - 占用空间小（MB级）          │
│   - 启动慢（分钟级）               - 启动快（秒级）              │
│   - 资源隔离强                     - 资源隔离较弱                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**常用容器命令**：

```bash
# 运行容器（最常用方式）
docker run -d -p 8080:80 --name mynginx nginx

# 参数详细解释：
# -d: 后台运行（detached），不阻塞终端
# -p 8080:80: 端口映射（主机端口:容器端口）
# --name mynginx: 给容器起名字（不指定会随机生成名字）
# nginx: 使用的镜像

# 交互式运行（-it）
docker run -it --name myubuntu ubuntu /bin/bash

# -i: 交互模式（保持STDIN打开）
# -t: 分配伪终端（pseudo-TTY）
# /bin/bash: 在容器内执行的命令

# 运行并自动删除（--rm）
docker run --rm -it ubuntu /bin/bash
# 退出容器后自动删除，适用于临时测试

# 运行并指定环境变量（-e）
docker run -d -e MYSQL_ROOT_PASSWORD=123456 --name mysql mysql:8

# 运行并挂载数据卷（-v）
docker run -d -v /home/lyjew/data:/app/data --name myapp myapp:1

# 查看运行中的容器
docker ps

# 查看所有容器（包括停止的）
docker ps -a

# 查看容器ID（只显示ID）
docker ps -q

# 查看所有容器ID（包括停止的）
docker ps -aq

# 停止容器
docker stop mynginx
# 或用容器ID
docker stop a1b2c3d4e5f6

# 强制停止容器（立即终止，不等待优雅关闭）
docker kill mynginx

# 启动已停止的容器
docker start mynginx

# 重启容器
docker restart mynginx

# 删除容器（必须先停止）
docker rm mynginx

# 强制删除运行中的容器
docker rm -f mynginx

# 删除所有停止的容器
docker container prune
# 或
docker rm $(docker ps -aq)

# 进入容器内部（exec）
docker exec -it mynginx /bin/bash
docker exec -it mynginx sh  # 某些容器可能没有bash，用sh

# 在容器内执行单个命令
docker exec mynginx ls -la /app

# 查看容器日志
docker logs mynginx

# 实时查看日志（-f 类似 tail -f）
docker logs -f mynginx

# 查看最近100行日志
docker logs --tail 100 mynginx

# 查看容器资源使用（CPU、内存、网络）
docker stats mynginx

# 实时查看所有容器资源
docker stats

# 只显示容器ID和名称
docker stats --no-stream

# 查看容器详细信息
docker inspect mynginx

# 查看容器的IP地址
docker inspect mynginx --format '{{.NetworkSettings.IPAddress}}'

# 查看容器的端口映射
docker port mynginx

# 查看容器内进程
docker top mynginx

# 复制文件到容器内
docker cp file.txt mynginx:/app/

# 从容器内复制文件出来
docker cp mynginx:/app/file.txt ./

# 暂停容器（暂停所有进程）
docker pause mynginx

# 恢复容器
docker unpause mynginx
```

### 2.3 仓库（Registry）

**仓库**是存储镜像的地方，就像代码仓库存储代码一样。

最常用的是**Docker Hub**：

```bash
# 登录Docker Hub
docker login
# 输出：
# Login Succeeded

# 推送镜像到Docker Hub
docker push username/image_name:tag

# 从Docker Hub拉取镜像
docker pull username/image_name:tag
```

---

## 3. Docker架构

### 3.1 Docker引擎组成

Docker使用客户端-服务器架构：

```
┌─────────────────────────────────────────────────────────────┐
│                        Docker架构                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────┐         ┌──────────────┐                     │
│   │  Docker  │────────▶│  Docker Host │                     │
│   │  Client  │  REST   │  (服务器)    │                     │
│   │   (CLI)  │   API   │              │                     │
│   └──────────┘         │  ┌────────┐  │                     │
│                        │  │ Daemon │  │                     │
│   用户输入命令          │  └────────┘  │                     │
│                        │              │                     │
│                        │  ┌────────┐  │                     │
│   ┌──────────┐         │  │Container│ │                     │
│   │  Docker  │────────▶│  └────────┘  │                     │
│   │ Registry │         │              │                     │
│   │  (Hub)   │         │  ┌────────┐  │                     │
│   └──────────┘         │  │ Image  │  │                     │
│                        │  └────────┘  │                     │
│                        └──────────────┘                     │
│                                                             │
│  客户端(CLI)     API请求         主机(运行容器)              │
└─────────────────────────────────────────────────────────────┘
```

**核心组件详解**：

1. **Docker Daemon（dockerd）**
   - 运行在主机上的后台服务
   - 负责管理镜像、容器、网络、数据卷等
   - 监听Docker API请求
   - 构建、运行、分发容器
   
   查看Docker Daemon状态：
   ```bash
   sudo systemctl status docker
   ```

2. **Docker Client（docker）**
   - Docker CLI，用户与Docker交互的工具
   - 发送命令给Docker Daemon
   - 可以与本地或远程的Docker Daemon通信

3. **Docker Registry（仓库）**
   - 存储镜像的地方
   - 默认是Docker Hub
   - 可以搭建私有仓库

### 3.2 Docker工作流程

```
1. 开发者编写Dockerfile
         ↓
2. docker build 构建镜像
         ↓
3. 镜像保存到本地仓库
         ↓
4. docker push 推送到远程仓库
         ↓
5. 服务器上 docker pull 拉取镜像
         ↓
6. docker run 运行容器
```

让我详细解释这个流程：

**步骤1：编写Dockerfile**

```dockerfile
# Dockerfile示例
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**步骤2：构建镜像**

```bash
docker build -t myapp:1.0 .
```

构建过程：
```
Sending build context to Docker daemon  15.36kB
Step 1/6 : FROM python:3.11-slim
 ---> 3aab8a267f4e
Step 2/6 : WORKDIR /app
 ---> Running in 9a5b4c8e1234
Removing intermediate container 9a5b4c8e1234
 ---> d7f8a1234567
Step 3/6 : COPY requirements.txt .
 ---> e8f9b2345678
...
Successfully built a1b2c3d4e5f6
Successfully tagged myapp:1.0
```

**步骤3-4：推送镜像**

```bash
# 先登录（如果推送到私有仓库）
docker login registry.example.com

# 打标签
docker tag myapp:1.0 registry.example.com/myapp:1.0

# 推送
docker push registry.example.com/myapp:1.0
```

**步骤5-6：拉取并运行**

```bash
# 服务器上拉取镜像
docker pull registry.example.com/myapp:1.0

# 运行容器
docker run -d -p 8080:8080 registry.example.com/myapp:1.0
```

---

## 4. Docker安装（Debian/Linux）

### 4.1 安装前准备

检查系统要求：
```bash
# 查看系统版本
cat /etc/os-release
# 输出：
# PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
# NAME="Debian GNU/Linux 13 (trixie)"
# VERSION_ID="13"
# ID=debian

# 查看内核版本（需要3.10以上）
uname -r
# 输出：6.12.74+deb13+1-amd64

# 检查是否支持Docker（64位Linux）
uname -m
# 输出：x86_64（说明是64位）
```

### 4.2 安装步骤

#### 方法一：使用apt安装（推荐）

```bash
# 1. 更新apt包索引
sudo apt update

# 2. 安装依赖包
sudo apt install -y ca-certificates curl gnupg lsb-release

# 3. 添加Docker官方GPG密钥
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 4. 添加Docker仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 更新apt包索引
sudo apt update

# 6. 安装Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**软件包解释**：
- `docker-ce`：Docker社区版（Community Edition）
- `docker-ce-cli`：Docker命令行工具
- `containerd.io`：容器运行时
- `docker-buildx-plugin`：扩展构建功能
- `docker-compose-plugin`：Docker Compose插件

#### 方法二：使用脚本安装（适合测试）

```bash
# 下载安装脚本
curl -fsSL https://get.docker.com -o get-docker.sh

# 运行脚本（自动检测系统并安装）
sudo sh get-docker.sh
```

这个脚本会自动：
- 检测Linux发行版
- 安装所需的依赖
- 配置Docker仓库
- 安装Docker
- 启动Docker服务

#### 方法三：离线安装（适合内网环境）

如果你在内网环境，无法访问互联网：

```bash
# 1. 在有网络的机器上下载DEB包
# 访问 https://download.docker.com/linux/debian/dists/ 选择你的版本
# 下载以下包：
# containerd.io_<version>_amd64.deb
# docker-ce_<version>_amd64.deb
# docker-ce-cli_<version>_amd64.deb

# 2. 拷贝到目标机器
scp *.deb user@target-server:/tmp/

# 3. 安装
sudo dpkg -i /tmp/*.deb

# 4. 如果有依赖问题
sudo apt-get install -f
```

### 4.3 验证安装

```bash
# 检查Docker版本
docker --version
# 输出：Docker version 27.5.1, build 117a5b5

# 查看Docker详细信息（包括存储驱动、网络等）
docker info

# 运行hello-world镜像测试
sudo docker run hello-world
```

输出类似：
```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.
```

### 4.4 配置Docker

#### 启动Docker服务
```bash
# 启动Docker
sudo systemctl start docker

# 设置开机自启（推荐）
sudo systemctl enable docker

# 查看Docker状态
sudo systemctl status docker
# 输出：
# ● docker.service - Docker Application Container Engine
#      Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
#      Active: active (running) since Mon 2026-03-23 10:00:00 CST; 1h ago
#    Main PID: 1234 (dockerd)
#       Tasks: 10
#      Memory: 120.0M
#         CPU: 500ms
#      CGroup: /system.slice/docker.service
#              └─1234 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

#### 配置Docker用户组（避免每次sudo）

```bash
# 创建docker用户组
sudo groupadd docker

# 将当前用户添加到docker组
sudo usermod -aG docker $USER

# 重新登录后生效，或执行：
newgrp docker

# 验证（不需要sudo了）
docker ps
```

**警告**：docker组成员有root权限，慎重添加用户。

#### 配置国内镜像加速

国内访问Docker Hub非常慢，需要配置镜像加速：

```bash
# 创建配置目录
sudo mkdir -p /etc/docker

# 创建配置文件
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me",
    "https://docker.rainbond.cc",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
EOF

# 重启Docker
sudo systemctl restart docker

# 验证配置
docker info | grep -A 10 "Registry Mirrors"
```

国内可用的镜像加速器：
| 加速器地址 | 提供商 |
|------------|--------|
| https://docker.1ms.run | 1MS |
| https://docker.xuanyuan.me | 玄元 |
| https://docker.mirrors.ustc.edu.cn | 中科大 |
| https://docker.rainbond.cc | Rainbond |
| https://mirror.baidubce.com | 百度云 |
| https://mirror.ccs.tencentyun.com | 腾讯云 |

#### 配置Docker日志

默认情况下，Docker容器日志会无限增长，需要配置日志轮转：

```bash
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": [
    "https://docker.1ms.run"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
EOF

sudo systemctl restart docker
```

参数解释：
- `max-size`：单个日志文件最大10MB
- `max-file`：最多保留3个日志文件

---

## 5. Docker基本操作

### 5.1 镜像操作

```bash
# 搜索镜像（在Docker Hub搜索）
docker search nginx
# 输出：
# NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
# nginx                             Official build of Nginx.                        18980     [OK]       
# jwilder/nginx-proxy              Automated Nginx reverse proxy for docker …     2146                 [OK]
# linuxserver/nginx                Nginx container, incl …                          190                 [OK]

# 搜索带过滤
docker search --stars=1000 nginx  # 1000星以上

# 拉取镜像（不指定tag则拉取latest）
docker pull nginx:latest
docker pull nginx:1.25
docker pull openjdk:21-jdk-slim

# 拉取特定架构的镜像
docker pull --platform linux/amd64 nginx
docker pull --platform linux/arm64 nginx  # Apple Silicon Mac

# 查看本地镜像
docker images
# 或
docker image ls
# 输出：
# REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
# nginx        latest    a6bd71f48f68   2 weeks ago    187MB
# openjdk      21        7e2024c2d9c3   3 weeks ago    470MB

# 查看镜像列表（只显示ID）
docker images -q

# 查看镜像详情
docker image inspect nginx:latest

# 查看镜像的构建历史
docker history nginx:latest

# 删除镜像
docker rmi nginx:latest

# 强制删除
docker rmi -f nginx:latest

# 删除所有镜像
docker rmi $(docker images -q)

# 清理悬空镜像（没有被容器使用的镜像）
docker image prune

# 清理所有未使用的镜像
docker image prune -a

# 构建镜像
docker build -t myapp:1.0 .

# 构建镜像（指定Dockerfile路径）
docker build -t myapp:1.0 -f /path/to/Dockerfile .

# 给镜像打标签
docker tag myapp:1.0 myapp:latest
docker tag myapp:1.0 docker.io/username/myapp:1.0

# 推送镜像到仓库
docker push username/myapp:1.0

# 拉取镜像
docker pull username/myapp:1.0
```

### 5.2 容器操作

```bash
# 运行容器（最常用方式）
docker run -d -p 8080:80 --name mynginx nginx

# 参数解释：
# -d: 后台运行（detached）
# -p 8080:80: 端口映射（主机端口:容器端口）
# --name mynginx: 给容器起名字
# nginx: 使用的镜像

# 交互式运行（-it）
docker run -it --name myubuntu ubuntu /bin/bash

# 运行并自动删除（--rm）
docker run --rm -it ubuntu /bin/bash

# 运行并指定环境变量（-e）
docker run -d -e MYSQL_ROOT_PASSWORD=123456 --name mysql mysql:8

# 运行并挂载数据卷（-v）
docker run -d -v /home/lyjew/data:/app/data --name myapp myapp:1

# 运行并指定网络
docker run -d --network host --name myapp myapp:1

# 运行并指定内存限制
docker run -d -m 512m --name myapp myapp:1

# 运行并指定CPU限制
docker run -d --cpus=0.5 --name myapp myapp:1

# 查看运行中的容器
docker ps

# 查看所有容器（包括停止的）
docker ps -a

# 查看容器ID（只显示ID）
docker ps -q

# 查看所有容器ID（包括停止的）
docker ps -aq

# 停止容器
docker stop mynginx

# 强制停止容器
docker kill mynginx

# 启动已停止的容器
docker start mynginx

# 重启容器
docker restart mynginx

# 删除容器（必须先停止）
docker rm mynginx

# 强制删除运行中的容器
docker rm -f mynginx

# 删除所有停止的容器
docker container prune
# 或
docker rm $(docker ps -aq)

# 进入容器内部
docker exec -it mynginx /bin/bash
docker exec -it mynginx sh  # 某些容器可能没有bash

# 在容器内执行单个命令
docker exec mynginx ls -la /app

# 查看容器日志
docker logs mynginx

# 实时查看日志（-f 类似 tail -f）
docker logs -f mynginx

# 查看最近100行日志
docker logs --tail 100 mynginx

# 查看容器日志（带时间戳）
docker logs -t mynginx

# 查看容器资源使用
docker stats mynginx

# 实时查看所有容器资源
docker stats

# 只显示容器ID和名称
docker stats --no-stream

# 查看容器详细信息
docker inspect mynginx

# 查看容器的IP地址
docker inspect mynginx --format '{{.NetworkSettings.IPAddress}}'

# 查看容器的端口映射
docker port mynginx

# 查看容器内进程
docker top mynginx

# 复制文件到容器内
docker cp file.txt mynginx:/app/

# 从容器内复制文件出来
docker cp mynginx:/app/file.txt ./

# 暂停容器
docker pause mynginx

# 恢复容器
docker unpause mynginx

# 重命名容器
docker rename mynginx newname

# 查看容器变化（与镜像对比）
docker diff mynginx

# 提交容器为镜像
docker commit mynginx myimage:1.0
```

### 5.3 端口映射详解

```bash
# 格式：-p 主机端口:容器端口

# 常见示例：
-p 8080:80      # 访问主机的8080端口 => 容器的80端口
-p 3306:3306    # MySQL端口
-p 5432:5432    # PostgreSQL端口
-p 6379:6379    # Redis端口
-p 9000:9000    # PHP-FPM或MinIO

# 随机分配主机端口（-P）
docker run -P nginx  # -P 会自动映射所有EXPOSE的端口

# 指定IP和端口
-p 127.0.0.1:8080:80  # 只允许本机访问8080端口

# UDP端口
-p 8080:80/udp

# 查看端口映射
docker port mynginx

# 详细示例：运行MySQL
docker run -d \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  --name mysql \
  mysql:8

# 详细示例：运行Redis
docker run -d \
  -p 6379:6379 \
  -e REDIS_PASSWORD=123456 \
  --name redis \
  redis:7 redis-server --requirepass 123456
```

### 5.4 数据卷操作

```bash
# 创建数据卷
docker volume create myvolume

# 查看数据卷
docker volume ls

# 查看数据卷详情
docker volume inspect myvolume

# 删除数据卷
docker volume rm myvolume

# 清理未使用的数据卷
docker volume prune

# 使用数据卷运行容器
docker run -v myvolume:/app/data nginx

# 绑定主机目录
docker run -v /home/lyjew/data:/app/data nginx

# 绑定主机目录（只读）
docker run -v /home/lyjew/data:/app/data:ro nginx

# 详细示例：运行MySQL并使用数据卷
docker run -d \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  --name mysql \
  mysql:8
```

### 5.5 网络操作

```bash
# 查看网络
docker network ls

# 创建网络
docker network create mynetwork

# 查看网络详情
docker network inspect mynetwork

# 删除网络
docker network rm mynetwork

# 运行容器并加入网络
docker run -d --network mynetwork --name container1 nginx

# 将运行中的容器加入网络
docker network connect mynetwork container2

# 从网络中断开容器
docker network disconnect mynetwork container2
```

---

## 6. 第一个Docker示例

### 6.1 运行Nginx

```bash
# 1. 拉取Nginx镜像
docker pull nginx:latest

# 2. 运行Nginx容器
docker run -d -p 8080:80 --name mynginx nginx

# 3. 访问测试
curl http://localhost:8080

# 4. 查看日志
docker logs mynginx

# 5. 停止并删除
docker stop mynginx
docker rm mynginx
```

### 6.2 运行Spring Boot应用

假设你有一个Spring Boot jar包：

```bash
# 1. 创建项目目录
mkdir -p ~/docker/springboot-app
cd ~/docker/springboot-app

# 2. 创建简单的Spring Boot Dockerfile
cat > Dockerfile << 'EOF'
# 基础镜像：使用OpenJDK 21
FROM openjdk:21-jdk-slim

# 设置工作目录
WORKDIR /app

# 复制jar包
COPY target/myapp.jar app.jar

# 暴露端口
EXPOSE 8080

# 启动命令
ENTRYPOINT ["java", "-jar", "app.jar"]
EOF

# 3. 构建镜像
docker build -t myapp:1.0 .

# 4. 运行容器
docker run -d -p 8080:8080 --name myapp myapp:1.0

# 5. 查看日志
docker logs -f myapp
```

### 6.3 运行Python应用

```bash
# 1. 创建项目目录
mkdir -p ~/docker/python-app
cd ~/docker/python-app

# 2. 创建Python应用
cat > app.py << 'EOF'
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return f'Hello from Docker! Running on {os.environ.get("HOSTNAME", "unknown")}'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# 3. 创建requirements.txt
cat > requirements.txt << 'EOF'
flask==3.0.0
EOF

# 4. 创建Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV FLASK_APP=app.py

EXPOSE 5000

CMD ["python", "app.py"]
EOF

# 5. 构建镜像
docker build -t python-app:1.0 .

# 6. 运行容器
docker run -d -p 5000:5000 --name python-app python-app:1.0

# 7. 测试
curl http://localhost:5000
```

---

## 7. Docker Hub使用

### 7.1 注册账号

1. 访问 https://hub.docker.com
2. 点击"Sign Up"注册
3. 验证邮箱

### 7.2 登录

```bash
docker login
# 输出：
# Login Succeeded
```

如果使用私有仓库：
```bash
docker login registry.example.com
```

### 7.3 推送镜像

```bash
# 1. 给镜像打标签
docker tag myapp:1.0 username/myapp:1.0

# 2. 推送到Docker Hub
docker push username/myapp:1.0

# 3. 在其他机器上拉取
docker pull username/myapp:1.0
```

### 7.4 自动构建

在Docker Hub上配置自动构建：
1. 登录Docker Hub
2. 点击"Create" → "Create Automated Build"
3. 关联GitHub/GitLab仓库
4. 设置Dockerfile路径
5. 每次推送代码，自动构建镜像

---

## 8. 常见问题与解决

### 8.1 Docker服务无法启动

```bash
# 查看错误日志
sudo journalctl -u docker -n 50

# 手动启动查看错误
sudo dockerd
```

常见错误及解决方法：

**错误1：Device or resource busy**

```
Error starting daemon: Error initializing network controller: Error initializing driver: error parsing gateway address: ""
```

解决方法：
```bash
# 清理网络
sudo systemctl stop docker
sudo rm -rf /var/lib/docker/network
sudo systemctl start docker
```

**错误2：failed to start daemon: pid file found**

```
failed to start daemon: pid file found, ensure docker is not running or delete /var/run/docker.pid
```

解决方法：
```bash
# 删除pid文件
sudo rm /var/run/docker.pid
sudo systemctl start docker
```

**错误3：Docker没有足够权限**

```
dial unix /var/run/docker.sock: connect: permission denied
```

解决方法：
```bash
# 将用户加入docker组
sudo usermod -aG docker $USER
# 或者使用sudo运行
```

### 8.2 权限问题

```bash
# 错误：permission denied while trying to connect to the Docker daemon
# 解决方案：将用户添加到docker组
sudo usermod -aG docker $USER
newgrp docker
```

如果还是不行，检查Docker socket权限：
```bash
ls -la /var/run/docker.sock
# 输出：srw-rw---- 1 root docker 0 Mar 23 10:00 /var/run/docker.sock

# 如果当前用户不在docker组，可以临时用sudo
```

### 8.3 镜像拉取慢

```bash
# 配置国内镜像加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me"
  ]
}
EOF

# 重启Docker
sudo systemctl restart docker
```

### 8.4 磁盘空间不足

```bash
# 查看Docker占用空间
docker system df

# 输出：
# TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
# Images          10        3         5.2GB     3.8GB (73%)
# Containers      5         2         123MB     45MB (36%)
# Local Volumes   3         1         892MB     0B (0%)
# Build Cache     0         0         0B        0B

# 清理停止的容器
docker container prune

# 清理悬空镜像
docker image prune

# 清理所有未使用的镜像、容器、网络
docker system prune -a

# 清理构建缓存
docker builder prune

# 清理所有未使用数据（镜像、容器、网络、数据卷、构建缓存）
docker system prune -a --volumes
```

### 8.5 容器内无法访问网络

```bash
# 检查Docker网络
docker network ls

# 检查容器网络
docker inspect container_name --format '{{.NetworkSettings.Networks}}'

# 创建新的网络
docker network create mynetwork

# 将容器加入网络
docker network connect mynetwork container_name

# 检查DNS
docker exec container_name cat /etc/resolv.conf
```

如果宿主机能上网但容器不行，可能是DNS问题：
```bash
# 方案1：指定DNS
docker run --dns 8.8.8.8 -it ubuntu bash

# 方案2：配置Docker daemon的DNS
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
EOF
sudo systemctl restart docker
```

### 8.6 容器无法相互通信

```bash
# 创建自定义网络（推荐）
docker network create mynetwork

# 运行容器时加入网络
docker run -d --network mynetwork --name container1 nginx
docker run -d --network mynetwork --name container2 redis

# 测试通信
docker exec container2 ping container1

# 如果还是不行，检查iptables
sudo iptables -L -n
```

### 8.7 端口冲突

```bash
# 查看端口占用
sudo netstat -tlnp | grep 8080

# 或用ss
sudo ss -tlnp | grep 8080

# 如果端口被占用，换一个端口
docker run -d -p 8081:80 --name nginx2 nginx
```

---

## 9. Docker进阶操作

### 9.1 容器资源限制

```bash
# 限制内存
docker run -m 512m --name myapp myapp:1

# 限制CPU（0.5表示50%）
docker run --cpus=0.5 --name myapp myapp:1

# 限制CPU核心（只能在特定核心上运行）
docker run --cpuset-cpus="0,1" --name myapp myapp:1

# 限制IO（读写速度）
docker run --device-read-bps /dev/sda:1mb --device-write-bps /dev/sda:1mb --name myapp myapp:1

# 查看容器资源限制
docker inspect container_name --format '{{.HostConfig.Memory}}'
```

### 9.2 容器健康检查

```bash
# 定义健康检查
docker run \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  --health-start-period=40s \
  --name mynginx \
  nginx

# 查看健康状态
docker inspect --format='{{.State.Health.Status}}' mynginx
```

### 9.3 容器数据共享

**方式1：数据卷**

```bash
# 创建命名数据卷
docker volume create mydata

# 运行容器时使用
docker run -v mydata:/app/data --name container1 ubuntu
docker run -v mydata:/data --name container2 ubuntu

# 两个容器可以共享同一数据卷
```

**方式2：绑定主机目录**

```bash
# 绑定主机目录
docker run -v /home/lyjew/shared:/shared --name container1 ubuntu

# 另一个容器也可以绑定同一目录
docker run -v /home/lyjew/shared:/data --name container2 ubuntu
```

### 9.4 容器通信

**方式1：使用网络**

```bash
# 创建网络
docker network create mynet

# 运行服务容器
docker run -d --network mynet --name redis redis:7

# 运行应用容器（可以引用redis作为hostname）
docker run -d --network mynet --name myapp -e REDIS_HOST=redis myapp:1
```

**方式2：使用link（已废弃）**

```bash
# 不推荐使用link，因为已被废弃
docker run -d --link redis:redis --name myapp myapp:1
```

### 9.5 容器批量操作

```bash
# 停止所有运行中的容器
docker stop $(docker ps -q)

# 删除所有停止的容器
docker rm $(docker ps -aq)

# 删除所有镜像
docker rmi $(docker images -q)

# 删除所有数据卷
docker volume prune

# 清理所有
docker system prune -a
```

---

## 10. Docker图形化管理

### 10.1 Portainer（推荐）

Portainer是一个轻量级的Docker图形化管理界面。

```bash
# 安装Portainer
docker volume create portainer_data

docker run -d \
  -p 9000:9000 \
  -p 8000:8000 \
  --name portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest

# 首次访问：http://your-server:9000
# 设置admin密码
# 选择"Local"连接本地Docker
```

### 10.2 Docker Desktop

如果你在Windows或Mac上工作，可以使用Docker Desktop：
- 下载地址：https://www.docker.com/products/docker-desktop
- 包含Docker引擎、Docker CLI、Docker Compose、Kubernetes
- 图形化界面，易于使用

---

## 11. Docker常用场景

### 11.1 运行MySQL

```bash
# 方式1：使用数据卷持久化
docker run -d \
  --name mysql8 \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  -e MYSQL_USER=appuser \
  -e MYSQL_PASSWORD=app123 \
  -v mysql-data:/var/lib/mysql \
  mysql:8

# 方式2：使用主机目录
docker run -d \
  --name mysql8 \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -v /home/lyjew/mysql/data:/var/lib/mysql \
  mysql:8

# 连接测试
docker exec -it mysql8 mysql -uroot -proot123
```

### 11.2 运行Redis

```bash
# 方式1：简单运行
docker run -d \
  --name redis7 \
  -p 6379:6379 \
  redis:7

# 方式2：带密码
docker run -d \
  --name redis7 \
  -p 6379:6379 \
  redis:7 redis-server --requirepass redis123

# 方式3：带持久化
docker run -d \
  --name redis7 \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:7 redis-server --requirepass redis123 --appendonly yes

# 连接测试
docker exec -it redis7 redis-cli -a redis123
```

### 11.3 运行MongoDB

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=admin123 \
  -v mongo-data:/data/db \
  mongo:7

# 连接
docker exec -it mongodb mongosh -u admin -p admin123
```

### 11.4 运行PostgreSQL

```bash
docker run -d \
  --name postgres15 \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=postgres123 \
  -e POSTGRES_USER=appuser \
  -e POSTGRES_DB=appdb \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

# 连接
docker exec -it postgres15 psql -U appuser -d appdb
```

### 11.5 运行Nginx反向代理

```bash
# 创建配置文件目录
mkdir -p ~/docker/nginx/conf.d

# 创建配置文件
cat > ~/docker/nginx/conf.d/default.conf << 'EOF'
server {
    listen 80;
    server_name example.com;
    
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
EOF

# 运行Nginx
docker run -d \
  --name nginx \
  -p 80:80 \
  -p 443:443 \
  -v ~/docker/nginx/conf.d:/etc/nginx/conf.d \
  -v ~/docker/nginx/html:/usr/share/nginx/html \
  nginx:latest

# 重新加载配置（不重启）
docker exec nginx nginx -s reload
```

### 11.6 运行Elasticsearch

```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  -v es-data:/usr/share/elasticsearch/data \
  elasticsearch:8.11.0

# 等待启动完成（约1-2分钟）
# 访问：http://localhost:9200
```

---

## 12. Docker最佳实践

### 12.1 镜像优化

1. **使用合适的基础镜像**
   - 优先使用官方镜像
   - 选择alpine等轻量级镜像
   - 指定具体版本号，不使用latest

2. **减少镜像层数**
   - 合并RUN指令
   - 清理不必要的文件

3. **利用构建缓存**
   - 把变化少的指令放前面
   - 把变化多的指令放后面

4. **使用.dockerignore**
   - 排除不需要的文件
   - 减少构建上下文大小

### 12.2 容器安全

1. **不以root用户运行**
   ```dockerfile
   RUN addgroup -g 1000 appuser && \
       adduser -u 1000 -G appuser -s /bin/sh -D appuser
   USER appuser
   ```

2. **限制容器权限**
   ```bash
   docker run --read-only --tmpfs /tmp myapp
   ```

3. **定期更新镜像**
   ```bash
   docker pull image:tag
   ```

### 12.3 日志管理

1. **配置日志轮转**
   ```json
   {
     "log-driver": "json-file",
     "log-opts": {
       "max-size": "10m",
       "max-file": "3"
     }
   }
   ```

2. **使用日志收集工具**
   - Fluentd
   - Logstash
   - Loki

---

## 13. 本章总结

### 核心概念

| 概念 | 说明 |
|------|------|
| **镜像（Image）** | 应用程序模板，包含代码+运行环境+依赖 |
| **容器（Container）** | 镜像的运行实例 |
| **仓库（Registry）** | 存储镜像的地方（Docker Hub） |
| **Dockerfile** | 构建镜像的配置文件 |
| **Docker Daemon** | Docker后台服务 |
| **Docker Client** | Docker命令行工具 |

### 常用命令

```bash
# 镜像操作
docker pull <镜像名>
docker images
docker rmi <镜像名>
docker build -t 名字:标签 .

# 容器操作
docker run -d -p 主机端口:容器端口 --name 名字 镜像名
docker ps
docker stop <容器名>
docker rm <容器名>
docker exec -it <容器名> /bin/bash
docker logs -f <容器名>

# 网络操作
docker network create <网络名>
docker network ls

# 数据卷操作
docker volume create <卷名>
docker volume ls
```

---

## 14. 课后问题

1. Docker和传统虚拟机的区别是什么？
   - 虚拟机需要完整操作系统，容器共享宿主机内核
   - 虚拟机占用空间大（GB级），容器占用空间小（MB级）
   - 虚拟机启动慢（分钟级），容器启动快（秒级）

2. 镜像和容器的关系是什么？
   - 镜像是模板，容器是实例
   - 一个镜像可以运行多个容器

3. 如何让Docker使用国内镜像加速？
   - 配置/etc/docker/daemon.json，添加registry-mirrors

4. 容器中的数据如何持久化？
   - 使用数据卷（docker volume）
   - 绑定主机目录（-v /host/path:/container/path）

---

## 15. Docker版本说明

Docker有两种版本：
- **Docker Engine (Docker CE)**：开源版本，免费使用
- **Docker Enterprise (Docker EE)**：企业版，提供商业支持

查看Docker版本：
```bash
# 查看Docker版本
docker --version
# Docker version 27.5.1, build 117a5b5

# 查看详细版本信息
docker version
# Client: Docker Engine - Community
#  Version:           27.5.1
#  API version:       1.47
#  Go version:        go1.22.10
# 
# Server: Docker Engine - Community
#  Version:           27.5.1
#  API version:       1.47 (minimum version 1.24)
#  Go version:        go1.22.10

# 查看所有版本信息
docker info
```

版本号格式：`主版本.次版本.修订号`
- 27.5.1：主版本27，次版本5，修订号1

---

## 16. Docker生态工具

Docker不仅仅是一个容器引擎，而是一个完整的生态系统：

### 16.1 核心工具

| 工具 | 说明 |
|------|------|
| Docker Engine | 核心容器引擎 |
| Docker CLI | 命令行工具 |
| Docker Compose | 多容器编排 |
| Docker Swarm | 容器编排工具（与Kubernetes竞争） |
| Docker Hub | 镜像仓库 |

### 16.2 相关工具

| 工具 | 说明 |
|------|------|
| Kubernetes | 容器编排平台 |
| Helm | Kubernetes包管理器 |
| Prometheus | 监控 |
| Grafana | 可视化 |
| Jaeger | 链路追踪 |
| Fluentd | 日志收集 |

### 16.3 替代方案

| 工具 | 说明 |
|------|------|
| Podman | Red Hat的容器引擎，兼容Docker |
| containerd | 容器运行时（Docker基于它） |
| CRI-O | Kubernetes容器运行时 |
| Kaniko | Kubernetes中构建镜像 |
| BuildKit | Docker的下一代构建工具 |

---

## 17. 实际项目中使用Docker

### 17.1 开发环境

```bash
# 使用Docker进行开发
# 1. 克隆代码
git clone https://github.com/yourproject/yourproject.git
cd yourproject

# 2. 使用docker-compose启动开发环境
docker-compose up -d

# 3. 查看服务状态
docker-compose ps

# 4. 查看日志
docker-compose logs -f

# 5. 停止服务
docker-compose down
```

### 17.2 CI/CD集成

在Jenkins中使用Docker：

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8.6-eclipse-temurin-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }
    }
}
```

在GitLab CI中使用Docker：

```yaml
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t myapp:$CI_COMMIT_SHA .
    - docker push myapp:$CI_COMMIT_SHA
```

### 17.3 生产环境部署

```bash
# 1. 拉取最新镜像
docker pull myapp:latest

# 2. 停止旧容器
docker stop myapp

# 3. 删除旧容器
docker rm myapp

# 4. 运行新容器
docker run -d \
  --name myapp \
  -p 8080:8080 \
  -e ENV=production \
  --restart unless-stopped \
  myapp:latest
```

使用脚本进行零 downtime 部署：

```bash
#!/bin/bash
IMAGE=$1
PORT=8080

# 拉取新镜像
docker pull $IMAGE

# 运行新容器（新端口）
docker run -d \
  -p $((PORT+1)):$PORT \
  --name myapp_new \
  $IMAGE

# 等待健康检查
sleep 10

# 切换流量（使用nginx或负载均衡器）
# 停止旧容器
docker stop myapp_old
docker rm myapp_old

# 重命名新容器
docker stop myapp_new
docker rm myapp
docker rename myapp_new myapp
docker start myapp
```

---

## 18. 下篇预告

下一章我们将学习 **Dockerfile基础**，学习如何编写Dockerfile来构建自定义镜像。

**下篇内容预告**：
- Dockerfile的基本结构
- FROM、RUN、COPY指令
- 编写第一个Dockerfile
- 构建并运行自定义镜像
- .dockerignore文件的作用

