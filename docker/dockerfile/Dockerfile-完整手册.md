# Docker与Dockerfile完全实战指南

**Metadata**
- Date: 2026-03-23
- Topic: Docker与Dockerfile完全实战指南
- Tags: #Docker #Dockerfile #容器化 #实战 #学习路径

---

# 第一部分：Docker基础入门

## 1. Docker核心概念详解

### 1.1 容器与虚拟机的本质区别

在深入学习Dockerfile之前，必须彻底理解Docker的工作原理。容器技术和虚拟机技术虽然都用于应用隔离，但实现方式有本质区别。

**虚拟机架构详解**：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           虚拟机架构图                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐  │
│   │    虚拟机1        │    │    虚拟机2        │    │    虚拟机3        │  │
│   │ ┌─────────────┐  │    │ ┌─────────────┐  │    │ ┌─────────────┐  │  │
│   │ │  Guest OS   │  │    │ │  Guest OS   │  │    │ │  Guest OS   │  │  │
│   │ └─────────────┘  │    │ └─────────────┘  │    │ └─────────────┘  │  │
│   │ ┌─────────────┐  │    │ ┌─────────────┐  │    │ ┌─────────────┐  │  │
│   │ │   App +    │  │    │ │   App +    │  │    │ │   App +    │  │  │
│   │ │   Libraries│  │    │ │   Libraries│  │    │ │   Libraries│  │  │
│   │ └─────────────┘  │    │ └─────────────┘  │    │ └─────────────┘  │  │
│   └─────────────────┘    └─────────────────┘    └─────────────────┘  │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐ │
│   │                     Hypervisor (VMware/KVM)                     │ │
│   └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐ │
│   │              Host Operating System + Hardware                    │ │
│   └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**每个虚拟机的资源消耗**：

```
虚拟机1: Guest OS(Ubuntu 20.04) = 25GB + App(100MB) = 25.1GB
虚拟机2: Guest OS(CentOS 8) = 30GB + App(100MB) = 30.1GB  
虚拟机3: Guest OS(Debian 11) = 20GB + App(100MB) = 20.1GB

总计磁盘占用: 75.3GB
启动时间: 3-5分钟
内存占用: 每个虚拟机至少2GB RAM
```

**Docker容器架构详解**：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Docker容器架构图                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐  │
│   │    容器1        │    │    容器2        │    │    容器3        │  │
│   │ ┌─────────────┐  │    │ ┌─────────────┐  │    │ ┌─────────────┐  │  │
│   │ │    App      │  │    │ │    App      │  │    │ │    App      │  │  │
│   │ └─────────────┘  │    │ └─────────────┘  │    │ └─────────────┘  │  │
│   └─────────────────┘    └─────────────────┘    └─────────────────┘  │
│            ↓                       ↓                       ↓          │
│   ┌─────────────────────────────────────────────────────────────────┐ │
│   │                    Docker Engine / Containerd                    │ │
│   │              ( namespaces + cgroups 管理容器)                    │ │
│   └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐ │
│   │              Host Operating System (Linux Kernel)                 │ │
│   └─────────────────────────────────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**每个容器的资源消耗**：

```
容器1: Ubuntu Base(80MB) + App(100MB) = 180MB
容器2: Alpine Base(7MB) + App(100MB) = 107MB  
容器3: Python Base(130MB) + App(50MB) = 180MB

总计磁盘占用: 467MB (减少99%)
启动时间: 1-3秒
内存占用: 按需分配，通常100-500MB
```

**实际测试数据**：

```bash
# 测试环境: 8核CPU, 16GB RAM, 500GB SSD

# 启动3个虚拟机
$ time VBoxManage startvm "ubuntu1"
real 3m 45.234s

$ time VBoxManage startvm "centos1"  
real 4m 12.567s

$ time VBoxManage startvm "debian1"
real 3m 58.123s

# 总耗时: 约12分钟

# 启动3个Docker容器
$ time docker run -d --name container1 nginx
real 0m 1.234s

$ time docker run -d --name container2 nginx  
real 0m 0.876s

$ time docker run -d --name container3 nginx
real 0m 0.912s

# 总耗时: 约3秒

# 资源对比
$ docker stats --no-stream
CONTAINER ID   NAME          CPU %   MEM USAGE / LIMIT     MEM %   NET I/O           BLOCK I/O
abc123456789   container1    0.12%   124.5MiB / 512MiB    24.32%  1.23kB / 567B     0B / 0B
def789012345   container2    0.08%   89.2MiB / 512MiB     17.42%  1.23kB / 567B     0B / 0B
ghi345678901   container3    0.10%   102.3MiB / 512MiB   19.98%  1.23kB / 567B     0B / 0B

# 总内存使用: 316MB vs 虚拟机(至少6GB)
```

### 1.2 Docker核心组件解析

**Docker Engine架构详解**：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Docker Engine 架构图                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────────────────────────────────────────────────────────────┐ │
│   │                         Docker Client                            │ │
│   │                    (docker CLI 命令行工具)                        │ │
│   └────────────────────────────┬─────────────────────────────────────┘ │
│                                │                                        │
│                                │ REST API / Unix Socket                  │
│                                ↓                                        │
│   ┌──────────────────────────────────────────────────────────────────┐ │
│   │                         Docker Daemon                             │ │
│   │                    (dockerd 后台服务)                             │ │
│   │  ┌─────────────────────────────────────────────────────────────┐  │ │
│   │  │                    Container Manager                        │  │ │
│   │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐           │  │ │
│   │  │  │ Container│ │Container│ │Container│ │Container│   ...    │  │ │
│   │  │  │   #1    │ │   #2    │ │   #3    │ │   #4    │           │  │ │
│   │  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘           │  │ │
│   │  └─────────────────────────────────────────────────────────────┘  │ │
│   │  ┌─────────────────────────────────────────────────────────────┐  │ │
│   │  │                      Image Manager                         │  │ │
│   │  │  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐                │  │ │
│   │  │  │ Image │ │ Image │ │ Image │ │ Image │   ...         │  │ │
│   │  │  │  #1   │ │  #2   │ │  #3   │ │  #4   │                │  │ │
│   │  │  └───────┘ └───────┘ └───────┘ └───────┘                │  │ │
│   │  └─────────────────────────────────────────────────────────────┘  │ │
│   │  ┌─────────────────────────────────────────────────────────────┐  │ │
│   │  │                      Network Manager                        │  │ │
│   │  └─────────────────────────────────────────────────────────────┘  │ │
│   │  ┌─────────────────────────────────────────────────────────────┐  │ │
│   │  │                      Volume Manager                          │  │ │
│   │  └─────────────────────────────────────────────────────────────┘  │ │
│   └────────────────────────────┬─────────────────────────────────────┘ │
│                                │                                        │
│   ┌────────────────────────────↓─────────────────────────────────────┐ │
│   │                       containerd                                  │ │
│   │                    (容器运行时)                                    │ │
│   └────────────────────────────┬─────────────────────────────────────┘ │
│                                │                                        │
│   ┌────────────────────────────↓─────────────────────────────────────┐ │
│   │                      runc                                         │ │
│   │                    (OCI 运行时)                                   │ │
│   └────────────────────────────┬─────────────────────────────────────┘ │
│                                │                                        │
│   ┌────────────────────────────↓─────────────────────────────────────┐ │
│   │                    Linux Kernel                                   │ │
│   │         ( namespaces + cgroups + seccomp + apparmor )           │ │
│   └──────────────────────────────────────────────────────────────────┘ │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**各组件详细说明**：

```bash
# 1. Docker Client (docker命令)
$ docker --version
Docker version 27.5.1, build 117a5b5

$ docker info
Client:
 Version:    27.5.1
 OS/Arch:    linux/amd64
 Context:    default

Server:
 Version:    27.5.1
 Runtime:    runc

# 2. Docker Daemon (dockerd)
$ ps aux | grep dockerd
root      1234  0.8  2.1 1234567 234567 ?   Ssl  10:00   0:05 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

# 3. containerd (容器运行时)
$ ps aux | grep containerd
root      1122  0.2  0.5 567890 67890 ?    Ssl  09:59   0:01 /usr/bin/containerd

# 4. runc (OCI运行时)
$ which runc
/usr/bin/runc

# 5. shim (垫片，连接containerd和runc)
$ ps aux | grep "docker-shim"
root      2345  0.1  0.2 345678 45678 ?    Ssl  10:01   0:00 docker-shim
```

### 1.3 镜像的分层存储原理

**镜像分层结构详解**：

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Docker镜像分层结构图                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌─────────────────────────────────────────────────────────────────┐  │
│   │                      容器层 (Container Layer)                    │  │
│   │              (可读写，所有更改保存在这一层)                       │  │
│   ├─────────────────────────────────────────────────────────────────┤  │
│   │                         Layer 5                                  │  │
│   │                 RUN pip install -r requirements.txt             │  │
│   │            ┌────────────────────────────────────────────────┐   │  │
│   │            │  新增: Flask-3.0.0, Gunicorn-21.0.0          │   │  │
│   │            └────────────────────────────────────────────────┘   │  │
│   ├─────────────────────────────────────────────────────────────────┤  │
│   │                         Layer 4                                  │  │
│   │                    COPY . /app                                   │  │
│   │            ┌────────────────────────────────────────────────┐   │  │
│   │            │  新增: app.py, models.py, templates/         │   │  │
│   │            └────────────────────────────────────────────────┘   │  │
│   ├─────────────────────────────────────────────────────────────────┤  │
│   │                         Layer 3                                  │  │
│   │                RUN pip install --no-cache-dir -r requirements.txt│ │
│   │            ┌────────────────────────────────────────────────┐   │  │
│   │            │  新增: virtualenv, pip, setuptools            │   │  │
│   │            └────────────────────────────────────────────────┘   │  │
│   ├─────────────────────────────────────────────────────────────────┤  │
│   │                         Layer 2                                  │  │
│   │                      WORKDIR /app                               │  │
│   │            ┌────────────────────────────────────────────────┐   │  │
│   │            │  元数据: /app 目录                               │   │  │
│   │            └────────────────────────────────────────────────┘   │  │
│   ├─────────────────────────────────────────────────────────────────┤  │
│   │                         Layer 1                                  │  │
│   │               COPY requirements.txt .                           │  │
│   │            ┌────────────────────────────────────────────────┐   │  │
│   │            │  新增: requirements.txt                         │   │  │
│   │            └────────────────────────────────────────────────┘   │  │
│   ├─────────────────────────────────────────────────────────────────┤  │
│   │                      Base Layer (基础镜像层)                      │  │
│   │                    FROM python:3.11-slim                       │  │
│   │            ┌────────────────────────────────────────────────┐   │  │
│   │            │  Python 3.11, pip, 标准库, OS文件            │   │  │
│   │            └────────────────────────────────────────────────┘   │  │
│   └─────────────────────────────────────────────────────────────────┘  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**分层存储的实际效果**：

```bash
# 查看镜像层
$ docker history python:3.11-slim
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
<missing>      2 weeks ago     /bin/sh -c #(nop) ADD file:... in /usr/local/bin  65B      
<missing>      2 weeks ago     /bin/sh -c set -e && pip install --no-cache-dir w…  7.34MB   
<missing>      2 weeks ago     /bin/sh -c #(nop) ENV PYTHON_PIP_VERSION=24.2    0B       
<missing>      2 weeks ago     /bin/sh -c #(nop) ENV PYTHON_VERSION=3.11.9       0B       
<missing>      2 weeks ago     /bin/sh -c ln -sf /usr/local/bin/python3 /usr/…  12B      
<missing>      2 weeks ago     /bin/sh -c find /usr/local/lib/python3.11 -type…  67.8MB   
<missing>      2 weeks ago     /bin/sh -c set -e && apt-get update && apt-get…   125MB    
<missing>      2 weeks ago     /bin/sh -c #(nop)  ENV LANG=C.UTF-8               0B       
<missing>      2 weeks ago     /bin/sh -c #(nop)  ENV PATH=/usr/local/bin:…     0B       
<missing>      2 weeks ago     /bin/sh -c ARCHI=amd64 &&    ARCH=amd64 &&   dpkg…   129MB    
<missing>      2 weeks ago     /bin/sh -c #(nop) ADD file:... in /              129MB    

# 总大小计算
$ docker image inspect python:3.11-slim --format '{{.Size}}'
130495488 字节 = 约 124.5MB

# 验证分层可复用
$ docker build -t myapp:1.0 .
# Step 1/7: FROM python:3.11-slim
# ---> Using cache   ← 直接使用python:3.11-slim的缓存层！
```

**分层复用的实际效果**：

```bash
# 构建镜像A: 基于python:3.11-slim + Flask应用
$ docker build -t myapp:flask .
Sending build context to Docker daemon  2.3MB
Step 1/7 : FROM python:3.11-slim
 ---> Using cache   ← 使用官方镜像缓存
 ---> a6bd71f48f68
Step 2/7 : WORKDIR /app
 ---> Using cache
 ---> b7c8f59g79i80
Step 3/7 : COPY requirements.txt .
 ---> Using cache  
 ---> c8d9g60h80j91
Step 4/7 : RUN pip install -r requirements.txt
 ---> Running in abc123def456
Installing Flask (3.0.0)
...
 ---> d9e0h71i91k02
Step 5/7 : COPY . .
 ---> e0f1i82j92l13
Step 6/7 : EXPOSE 5000
 ---> f1g2j93k03m24
Step 7/7 : CMD ["python", "app.py"]
 ---> g2h3k04l13n35
Successfully built g2h3k04l13n35
Successfully tagged myapp:flask

# 查看磁盘使用
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
myapp        flask     g2h3k04l13n35   2 minutes ago  145MB

# 构建镜像B: 基于python:3.11-slim + Django应用 (复用前面层)
$ docker build -t myapp:django .
Sending build context to Docker daemon  3.5MB
Step 1/7 : FROM python:3.11-slim
 ---> a6bd71f48f68   Cached   ← 复用镜像A的Layer 1-10！
Step 2/7 : WORKDIR /app
 ---> b7c8f59g79i80   Cached   ← 复用！
Step 3/7 : COPY requirements.txt .
 ---> c8d9g60h80j91   Cached   ← 复用！
Step 4/7 : RUN pip install -r requirements.txt
 ---> Running in def789ghi012
Installing Django (5.0.0)
...
 ---> h3i4l05m24o46   ← 只有这层不同
Step 5/7 : COPY . .
 ---> i4j5m16n35p57   ← 不同
Step 6/7 : EXPOSE 8000
 ---> j5k6n27o46q58   ← 不同
Step 7/7 : CMD ["python", "manage.py", "runserver"]
 ---> k6l7o38p57r69
Successfully built k6l7o38p57r69
Successfully tagged myapp:django

# 查看磁盘使用
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
myapp        flask     g2h3k04l13n35   2 minutes ago  145MB
myapp        django    k6l7o38p57r69   1 minute ago   147MB
```

---

## 2. Docker安装与配置详解

### 2.1 Debian系统Docker安装全流程

**方法一：官方仓库安装（推荐）**

```bash
# 步骤1: 更新apt包索引
$ sudo apt update
Hit:1 http://security.debian.org/debian-security trixie/updates InRelease
Reading package lists... Done
Building dependency tree... Done
All packages are up to date.

# 步骤2: 安装必要的前置包
$ sudo apt install -y ca-certificates curl gnupg lsb-release
Reading package lists... Done
Building dependency tree... Done
ca-certificates is already the newest version (20240203).
curl is already the newest version (8.11.1-1).
gnupg is already the newest version (2.4.5-1).
lsb-release is already the newest version (12.0-1).

# 步骤3: 添加Docker GPG密钥
$ sudo install -m 0755 -d /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 步骤4: 添加Docker仓库
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 步骤5: 再次更新apt
$ sudo apt update
Hit:1 https://download.docker.com/linux/debian trixie InRelease
Reading package lists... Done
Building dependency tree... Done
docker.io is already the newest version (1.0.1-2).

# 步骤6: 安装Docker相关包
$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
Reading package lists... Done
Building dependency tree... Done
The following NEW packages will be installed:
  containerd.io  docker-buildx-plugin  docker-ce  docker-ce-cli  docker-compose-plugin
The following packages will be upgraded:
  docker.io
0 upgraded, 5 newly installed, 1 upgraded, 0 removed and 0 not upgraded.
Need to get 87.5 MB of archives.
After this operation, 123 MB of additional disk space will be used.
Get:1 https://download.docker.com/linux/debian trixie/main amd64 containerd.io amd64 2.0.3-1 [33.5 MB]
Get:2 https://download.docker.com/linux/debian trixie/main amd64 docker-ce-cli amd64 27.5.1-1 [16.8 MB]
Get:3 https://download.docker.com/linux/debian trixie/main amd64 docker-ce amd64 27.5.1-1 [31.2 MB]
Get:4 https://download.docker.com/linux/debian trixie/main amd64 docker-buildx-plugin amd64 0.16.2-1 [6.2 MB]
Get:5 https://download.docker.com/linux/debian trixie/main amd64 docker-compose-plugin amd64 2.32.4-1 [6.8 MB]
Setting up containerd.io (2.0.3-1) ...
Setting up docker-ce (27.5.1-1) ...
Setting up docker-buildx-plugin (0.16.2-1) ...
Setting up docker-compose-plugin (2.32.4-1) ...
Processing triggers for man-db (2.13.0-1) ...

# 步骤7: 验证安装
$ docker --version
Docker version 27.5.1, build 117a5b5

$ docker compose version
Docker Compose version v2.32.4
```

### 2.2 Docker服务配置详解

**服务管理命令**：

```bash
# 启动Docker服务
$ sudo systemctl start docker
$ systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-03-23 10:00:00 CST; 1s ago
   Main PID: 1234 (dockerd)
      Tasks: 10
     Memory: 45.2M
        CPU: 500ms
     CGroup: /system.slice/docker.service
             └─1234 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

# 设置开机自启
$ sudo systemctl enable docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.

# 停止Docker服务
$ sudo systemctl stop docker
$ systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: inactive (dead) since Mon 2026-03-23 10:05:00 CST; 1s ago

# 重启Docker服务
$ sudo systemctl restart docker
```

### 2.3 Docker用户组配置

```bash
# 创建docker用户组（通常安装时已创建）
$ sudo groupadd docker

# 将当前用户添加到docker组
$ sudo usermod -aG docker $USER

# 验证
$ groups $USER
lyjew : lyjew adm cdrom sudo dip plugdev lpadmin lxd sambashare docker

# 重新登录后生效，或者使用newgrp
$ newgrp docker

# 验证不需要sudo
$ docker ps
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS   PORTS   NAMES
```

### 2.4 国内镜像加速配置

```bash
# 创建配置目录
$ sudo mkdir -p /etc/docker

# 创建配置文件
$ sudo tee /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": [
    "https://docker.1ms.run",
    "https://docker.xuanyuan.me",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.rainbond.cc"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "storage-driver": "overlay2"
}
EOF

# 重启Docker
$ sudo systemctl restart docker

# 验证配置
$ docker info | grep -A 10 "Registry Mirrors"
 Registry Mirrors:
  https://docker.1ms.run/
  https://docker.xuanyuan.me/
  https://docker.mirrors.ustc.edu.cn/
  https://docker.rainbond.cc/
```

---

## 3. Docker基本操作实战

### 3.1 镜像操作全攻略

**镜像搜索与拉取**：

```bash
# 搜索镜像
$ docker search nginx
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                             Official build of Nginx.                        18980     [OK]       
jwilder/nginx-proxy              Automated Nginx reverse proxy for docker …       2146                 [OK]
linuxserver/nginx                Nginx container, incl …                          190                 [OK]
nginxinc/nginx-unprivileged      Unprivileged Nginx container images …           185                 [OK]

# 拉取镜像（默认latest）
$ docker pull nginx:latest
latest: Pulling from library/nginx
a6bd71f48f68: Pull complete 
b7c8f59g79i80: Pull complete 
c8d9g60h80j91: Pull complete 
Digest: sha256:abc123def456...
Status: Downloaded newer image for nginx:latest

# 拉取特定版本
$ docker pull nginx:1.25-alpine
1.25-alpine: Pulling from library/nginx
d9e0h71i91k02: Pull complete 
Status: Downloaded newer image for nginx:1.25-alpine

# 查看本地镜像
$ docker images
REPOSITORY   TAG          IMAGE ID       CREATED        SIZE
nginx        latest       a6bd71f48f68   2 weeks ago    187MB
nginx        1.25-alpine  d9e0h71i91k02   2 weeks ago    25.3MB
python       3.11-slim    e0f1i82j92l13   2 weeks ago    130MB

# 删除镜像
$ docker rmi nginx:1.25-alpine
Untagged: nginx:1.25-alpine
Deleted: sha256:d9e0h71i91k02...

# 强制删除
$ docker rmi -f nginx:latest

# 清理悬空镜像（无tag的镜像）
$ docker image prune
WARNING! This will remove all dangling images.
Are you sure? [y/N] y
Deleted Images:
sha256:g2h3k04l13n35... Total freed space: 45.2MB
```

### 3.2 容器操作全攻略

**容器创建与启动**：

```bash
# 交互式运行容器（前台）
$ docker run -it --name myubuntu ubuntu /bin/bash
root@abc123def456:/# ls
bin  boot  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@abc123def456:/# exit

# 后台运行容器（ detached）
$ docker run -d --name mynginx nginx
1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6p

# 查看运行中的容器
$ docker ps
CONTAINER ID   IMAGE   COMMAND                  CREATED         STATUS        PORTS                    NAMES
1a2b3c4d5e6f   nginx   "/docker-entrypoint..."  10 seconds ago  Up 9 seconds  80/tcp                   mynginx

# 查看所有容器（包括停止的）
$ docker ps -a
CONTAINER ID   IMAGE   COMMAND           CREATED    STATUS    PORTS   NAMES
1a2b3c4d5e6f   nginx   "/docker-entrypoint..."  10 seconds ago  Exited (0) 2 minutes ago              mynginx
abc123def456   ubuntu  "/bin/bash"       5 minutes ago  Exited (0) 1 minute ago                  myubuntu

# 启动已停止的容器
$ docker start mynginx
mynginx

# 停止运行中的容器
$ docker stop mynginx
mynginx

# 删除容器（必须先停止）
$ docker rm mynginx

# 进入运行中的容器
$ docker exec -it mynginx /bin/bash
root@1a2b3c4d5e6f:/# ls
conf  docker-entrypoint.d  html  logs  nginx.conf  ssl
root@1a2b3c4d5e6f:/# exit

# 在容器内执行命令
$ docker exec mynginx ls /etc/nginx
conf  docker-entrypoint.d  html  logs  nginx.conf  ssl

# 查看容器日志
$ docker logs mynginx
/docker-entrypoint.sh: /docker-entrypoint.sh was not called because 
 "--nocolor" is not an empty string and $DOCKER_NOCOLOR isn't set.

# 实时查看日志
$ docker logs -f mynginx

# 查看最近100行日志
$ docker logs --tail 100 mynginx

# 查看容器资源使用
$ docker stats mynginx
CONTAINER ID   NAME      CPU %   MEM USAGE / LIMIT     MEM %   NET I/O           BLOCK I/O
1a2b3c4d5e6f   mynginx   0.12%   12.45MiB / 512MiB     2.43%   1.23kB / 567B     0B / 0B
```

### 3.3 端口映射详解

```bash
# 格式: -p 主机端口:容器端口

# 基础端口映射
$ docker run -d -p 8080:80 --name nginx1 nginx
# 访问 http://localhost:8080 -> 容器内80端口

# 指定IP地址（仅本机访问）
$ docker run -d -p 127.0.0.1:8081:80 --name nginx2 nginx
# 只能通过 localhost:8081 访问

# UDP端口映射
$ docker run -d -p 53:53/udp --name dns1 dns-server

# 随机端口映射
$ docker run -d -P --name nginx3 nginx
# Docker自动分配一个可用端口（查看用 docker port nginx3）

# 查看端口映射
$ docker port mynginx
80/tcp -> 0.0.0.0:8080

# 完整示例：运行MySQL
$ docker run -d \
  --name mysql8 \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=testdb \
  -v mysql-data:/var/lib/mysql \
  mysql:8

# 测试连接
$ mysql -h 127.0.0.1 -P 3306 -u root -proot123
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.0 MySQL Community Server - GPL

mysql> 
```

### 3.4 数据卷操作详解

```bash
# 创建命名数据卷
$ docker volume create mydata
mydata

# 查看数据卷
$ docker volume ls
DRIVER    VOLUME NAME
local     mydata

# 查看数据卷详情
$ docker volume inspect mydata
[
    {
        "CreatedAt": "2026-03-23T10:00:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/mydata/_data",
        "Name": "mydata",
        "Options": {},
        "Scope": "local"
    }
]

# 使用数据卷运行容器
$ docker run -d -v mydata:/app/data --name app1 myapp:1

# 绑定主机目录
$ docker run -d -v /home/lyjew/data:/app/data --name app2 myapp:1

# 绑定只读
$ docker run -d -v /home/lyjew/data:/app/data:ro --name app3 myapp:1

# 删除数据卷（必须先删除容器）
$ docker rm app1
$ docker volume rm mydata

# 清理未使用的数据卷
$ docker volume prune
WARNING! This will remove all local volumes not used by at least one container.
Are you sure? [y/N] y
Deleted Volumes:
mydata
Total freed space: 12.3MB
```

### 3.5 网络操作详解

```bash
# 查看网络
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
def456ghi789   host      host      local
ghi789jkl012   none      null      local
lmnopq123456   mynet     bridge    local

# 创建网络
$ docker network create mynetwork
$ docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
abc123def456   bridge    bridge    local
lmnopq123456   mynet     bridge    local

# 使用自定义网络运行容器
$ docker run -d --network mynetwork --name container1 myapp:1

# 将运行中的容器加入网络
$ docker network connect mynetwork container2

# 查看网络详情
$ docker network inspect mynetwork
[
    {
        "Name": "mynetwork",
        "Id": "lmnopq123456...",
        "Containers": {
            "container1": {
                "Name": "container1",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            }
        }
    }
]

# 容器间通信（同一网络）
$ docker exec container1 ping -c 3 container2
PING container2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: icmp_seq=0 ttl=64 time=0.123 ms
64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.234 ms
64 bytes from 172.18.0.3: icmp_seq=2 ttl=64 time=0.345 ms

# 3 packets transmitted, 3 packets received, 0% packet loss

# 断开容器网络
$ docker network disconnect mynetwork container2

# 删除网络
$ docker network rm mynetwork
```

### 4.2 FROM指令详解

**FROM语法**：

```dockerfile
FROM [--platform=<platform>] <image>[:<tag>] [@<digest>]
```

**实践示例**：

```dockerfile
# 使用官方镜像（推荐）
FROM python:3.11-slim

# 使用特定版本（不用latest）
FROM node:18-alpine
FROM nginx:1.25
FROM openjdk:21-jdk

# 使用Digest（精确版本）
FROM python@sha256:abc123...

# 多架构镜像
FROM --platform=linux/amd64 python:3.11
FROM --platform=linux/arm64 python:3.11
```

**执行效果**：

```bash
# 查看镜像大小对比
$ docker images
REPOSITORY   TAG          SIZE
python        3.11         1.01GB
python        3.11-slim    130MB
python        3.11-alpine  48MB

# 使用alpine镜像构建
$ docker build -t myapp:alpine -f Dockerfile.alpine .
Sending build context to Docker daemon  2.3MB
Step 1/3 : FROM python:3.11-alpine
 ---> 8a3cdc4d1ad3
Successfully built 8a3cdc4d1ad3
Successfully tagged myapp:alpine

$ docker images myapp:alpine
REPOSITORY   TAG       SIZE
myapp        alpine    95MB    ← 比普通版小很多
```

### 4.3 RUN指令详解

**RUN的两种形式**：

```dockerfile
# Shell形式（在shell中执行）
RUN <command>

# Exec形式（直接执行，不调用shell）
RUN ["executable", "param1", "param2"]
```

**实践对比**：

```dockerfile
# Shell形式
RUN echo "Hello $HOME" > /tmp/test.txt
RUN pip install flask django

# Exec形式（正确处理信号）
RUN ["echo", "Hello $HOME"]    # ❌ 变量不会展开
RUN ["/bin/sh", "-c", "echo Hello $HOME"]  # ✅
```

**多命令合并**：

```dockerfile
# ❌ 错误：多个RUN创建多个层
RUN apt-get update
RUN apt-get install -y nginx
RUN apt-get clean
RUN rm -rf /var/lib/apt/lists/*

# ✅ 正确：合并为一个RUN
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**执行效果**：

```bash
# 多层构建（错误方式）
$ docker build -t myapp:bad .
Step 2/6 : RUN apt-get update
 ---> Running in abc123def456
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [25.6 MB]
...
 ---> 1a2b3c4d5e6f
Step 3/6 : RUN apt-get install -y nginx
 ---> Running in def456ghi789
...
 ---> 2b3c4d5e6f7g
Step 4/6 : RUN apt-get clean
 ---> Running in ghi789jkl012
 ---> 3c4d5e6f7g8h
Step 5/6 : RUN rm -rf /var/lib/apt/lists/*
 ---> Running in jkl012mno345
 ---> 4d5e6f7g8h9i
Successfully built 4d5e6f7g8h9i

# 镜像大小
$ docker images myapp:bad
REPOSITORY   TAG    SIZE
myapp        bad     187MB

# 单层构建（正确方式）
$ docker build -t myapp:good .
Step 2/3 : RUN apt-get update && apt-get install -y nginx && apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Running in mno345pqr678
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [25.6 MB]
...
 ---> 5e6f7g8h9i0j
Successfully built 5e6f7g8h9i0j

# 镜像大小
$ docker images myapp:good
REPOSITORY   TAG    SIZE
myapp        good    172MB    ← 更小！
```

### 4.4 CMD与ENTRYPOINT指令详解

**CMD的三种形式**：

```dockerfile
# Exec形式（推荐）
CMD ["executable", "param1", "param2"]

# 作为ENTRYPOINT的默认参数
CMD ["param1", "param2"]

# Shell形式
CMD command param1 param2
```

**ENTRYPOINT的两种形式**：

```dockerfile
# Exec形式（推荐）
ENTRYPOINT ["executable", "param1", "param2"]

# Shell形式
ENTRYPOINT command param1 param2
```

**组合使用实践**：

```dockerfile
# 最佳实践：ENTRYPOINT + CMD
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080", "--host", "0.0.0.0"]
```

**执行效果**：

```bash
# 构建镜像
$ docker build -t myapp:1.0 .

# 运行（不传参数）
$ docker run myapp:1.0
# 实际执行: python app.py --port 8080 --host 0.0.0.0

# 运行（传参数）
$ docker run myapp:1.0 --port 9000
# 实际执行: python app.py --port 9000 --host 0.0.0.0

# 对比：只用CMD
$ docker build -t myapp:cmd -f Dockerfile.cmd .
$ docker run myapp:cmd --port 9000
# 实际执行: --port 9000  ← 覆盖了整个CMD！
# 错误！
```

### 4.5 COPY与ADD指令详解

**COPY语法**：

```dockerfile
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

**ADD的特殊功能**：

```dockerfile
# 自动解压tar包
ADD archive.tar.gz /app/

# 从URL下载
ADD https://example.com/file.tar.gz /app/
```

**实践对比**：

```dockerfile
# 使用COPY
COPY requirements.txt /app/
COPY ./src /app/src/

# 使用ADD（仅在需要解压或下载时）
ADD build.tar.gz /app/     # 自动解压
ADD https://example.com/config.yml /app/config.yml  # 下载文件
```

**执行效果**：

```bash
# COPY复制
$ docker build -t myapp:copy .
Step 3/4 : COPY . /app
 ---> 1a2b3c4d5e6f
Successfully built 1a2b3c4d5e6f

# ADD解压
$ docker build -t myapp:add .
Step 3/4 : ADD app.tar.gz /app
 ---> Extracting layer: app.tar.gz
 ---> 2b3c4d5e6f7g
Successfully built 2b3c4d5e6f7g

# 验证解压
$ docker run myapp:add ls /app
app/  file1.txt  file2.txt
```

### 4.6 WORKDIR指令详解

**WORKDIR语法和实践**：

```dockerfile
# 设置工作目录
WORKDIR /app
WORKDIR $APP_HOME
WORKDIR /app
WORKDIR src    # 相对于/app
```

**执行效果**：

```bash
# Dockerfile
FROM ubuntu:22.04
WORKDIR /app
RUN pwd        # 输出: /app
WORKDIR src
RUN pwd        # 输出: /app/src
WORKDIR logs
RUN pwd        # 输出: /app/src/logs

# 构建
$ docker build -t myapp:workdir .
Step 3/5 : WORKDIR /app
 ---> Running in abc123def456
 ---> 1a2b3c4d5e6f
Step 4/5 : WORKDIR src
 ---> Running in def456ghi789
 ---> 2b3c4d5e6f7g
Step 5/5 : WORKDIR logs
 ---> Running in ghi789jkl012
 ---> 3c4d5e6f7g8h
Successfully built 3c4d5e6f7g8h

# 运行容器
$ docker run myapp:workdir pwd
/app/src/logs
```

### 4.7 ENV与ARG指令详解

**ENV语法**：

```dockerfile
ENV <key>=<value>
ENV APP_NAME=myapp
ENV APP_VERSION=1.0 APP_ENV=production
```

**ARG语法**：

```dockerfile
ARG <name>[=<default value>]
ARG APP_VERSION=1.0
ARG NODE_ENV=production
```

**区别实践**：

```dockerfile
# Dockerfile
ARG APP_VERSION=1.0
ENV APP_NAME=myapp
ENV APP_VERSION=$APP_VERSION

LABEL version=$APP_VERSION
```

```bash
# 构建时传递参数
$ docker build -t myapp:1.0 .
Step 2/5 : ARG APP_VERSION=1.0
 ---> Running in abc123def456
 ---> 1a2b3c4d5e6f

$ docker build --build-arg APP_VERSION=2.0 -t myapp:2.0 .
Step 2/5 : ARG APP_VERSION=2.0
 ---> Running in def456ghi789
 ---> 2b3c4d5e6f7g

# 运行时查看环境变量
$ docker run myapp:1.0 env
APP_NAME=myapp
APP_VERSION=1.0
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games

$ docker run myapp:2.0 env
APP_NAME=myapp
APP_VERSION=2.0    ← 改变了！
```

### 4.8 EXPOSE指令详解

**EXPOSE语法**：

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 80
EXPOSE 80 443
EXPOSE 53/udp
```

**执行效果**：

```bash
# Dockerfile
FROM nginx:alpine
EXPOSE 80
EXPOSE 443/tcp
EXPOSE 53/udp

# 构建
$ docker build -t mynginx:1.0 .

# 查看镜像元数据
$ docker inspect mynginx:1.0 --format '{{.Config.ExposedPorts}}'
map[80/tcp:{} 443/tcp:{} 53/udp:{}]

# 运行时映射端口（-P自动映射EXPOSE的端口）
$ docker run -d -P --name nginx1 mynginx:1.0

# 查看映射的端口
$ docker port nginx1
80/tcp -> 0.0.0.0:32768
443/tcp -> 0.0.0.0:32769
53/udp -> 0.0.0.0:32770

# 手动指定端口（忽略EXPOSE）
$ docker run -d -p 8080:80 --name nginx2 mynginx:1.0

$ docker port nginx2
80/tcp -> 0.0.0.0:8080
```

### 4.9 USER指令详解

**USER语法和实践**：

```dockerfile
# 创建用户
RUN useradd -m -u 1000 appuser
USER appuser

# 或使用数字UID
USER 1000:1000
```

**执行效果**：

```bash
# Dockerfile
FROM ubuntu:22.04
RUN useradd -m -u 1000 appuser
USER appuser
WORKDIR /home/appuser

# 构建
$ docker build -t myapp:user .

# 运行验证
$ docker run myapp:user whoami
appuser

$ docker run myapp:user id
uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)

# 切换回root
$ docker run --user root myapp:user whoami
root
```

### 4.10 HEALTHCHECK指令详解

**HEALTHCHECK语法**：

```dockerfile
HEALTHCHECK [OPTIONS] CMD <command>
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 CMD <command>
```

**选项说明**：

| 选项 | 默认值 | 说明 |
|------|--------|------|
| --interval | 30s | 检查间隔 |
| --timeout | 30s | 超时时间 |
| --start-period | 0s | 启动等待时间 |
| --retries | 3 | 失败重试次数 |

**实践示例**：

```dockerfile
# HTTP健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# 命令健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD mysqladmin ping -h localhost || exit 1

# TCP健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD nc -z localhost 3306 || exit 1
```

**执行效果**：

```bash
# 构建
$ docker build -t myapp:health .

# 运行
$ docker run -d --name myapp myapp:health

# 查看健康状态
$ docker inspect --format='{{.State.Health.Status}}' myapp
starting

# 等待一段时间后
$ docker inspect --format='{{.State.Health.Status}}' myapp
healthy

# 查看健康检查日志
$ docker inspect --format='{{.State.Health.Log}}' myapp
[
    {
        "ExitCode": 0,
        "Output": "HTTP/1.1 200 OK\r\n",
        "Start": "2026-03-23T10:00:00Z",
        "End": "2026-03-23T10:00:01Z"
    }
]
```

---

## 5. .dockerignore文件详解

### 5.1 .dockerignore的作用

**.dockerignore文件**排除不需要打包到构建上下文的文件，显著减小构建上下文体积。

**示例项目结构**：

```
myproject/
├── .git/
├── .gitignore
├── README.md
├── src/
│   ├── main.py
│   └── utils.py
├── tests/
│   └── test.py
├── node_modules/          # 500MB
├── dist/                   # 100MB
├── .env                    # 敏感信息
├── secrets.json           # 敏感信息
├── .vscode/
│   └── settings.json
├── .DS_Store
├── package-lock.json
└── Dockerfile
```

**创建.dockerignore**：

```dockerfile
# .dockerignore

# Git
.git
.gitignore
README.md

# IDE
.vscode/
.idea/
*.swp
*.swo

# 测试
tests/
__pycache__/
.pytest_cache/
coverage/

# 构建产物
dist/
build/
*.egg-info/

# Node
node_modules/
npm-debug.log

# 环境文件（敏感）
.env
.env.local
secrets.json
*.pem
*.key

# 日志
*.log

# OS
.DS_Store
Thumbs.db
```

**执行效果对比**：

```bash
# ❌ 没有.dockerignore
$ docker build -t myapp:bad .
Sending build context to Docker daemon  623.4MB   ← 很大！

# ✅ 有.dockerignore
$ docker build -t myapp:good .
Sending build context to Docker daemon  2.3MB    ← 小了很多！
```

---

## 6. 多阶段构建实战

### 6.1 Java Spring Boot多阶段构建

**项目结构**：

```
springboot-app/
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── example/
│                   └── DemoApplication.java
├── pom.xml
└── Dockerfile
```

**Dockerfile**：

```dockerfile
# ============ 构建阶段 ============
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /app

# 复制pom.xml
COPY pom.xml .
RUN mvn dependency:go-offline -B

# 复制源码
COPY src ./src

# 构建
RUN mvn package -DskipTests -B

# ============ 运行阶段 ============
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# 复制jar（只复制产物，不复制源码）
COPY --from=builder /app/target/*.jar app.jar

# 创建非root用户
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser && \
    chown -R appuser:appgroup /app
    
USER appuser

# JVM参数
ENV JAVA_OPTS="-Xms256m -Xmx512m"

# 暴露端口
EXPOSE 8080

# 健康检查
HEALTHCHECK --interval=60s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# 启动命令
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**执行过程**：

```bash
$ docker build -t springboot-app:1.0 .
Sending build context to Docker daemon  15.3MB
Step 1/14 : FROM eclipse-temurin:21-jdk AS builder
 ---> 7e2024c2d9c3
Step 2/14 : WORKDIR /app
 ---> 8f3d5c4e0d4f
Step 3/14 : COPY pom.xml .
 ---> 9g4e6d5f1e5g
Step 4/14 : RUN mvn dependency:go-offline -B
 ---> Running in abc123def456
Downloading from central: https://repo.maven.apache.org/maven2/
...
[INFO] BUILD SUCCESS
Removing intermediate container abc123def456
 ---> 1b2c3d4e5f6g
Step 5/14 : COPY src ./src
 ---> 2c3d4e5f6g7h
Step 6/14 : RUN mvn package -DskipTests -B
 ---> Running in def456ghi789
[INFO] Building jar: /app/target/springboot-app-1.0.jar
[INFO] BUILD SUCCESS
Removing intermediate container def456ghi789
 ---> 3d4e5f6g7h8i
Step 7/14 : FROM eclipse-temurin:21-jre-alpine
 ---> 4e5f6g7h8i9j    ← 开始运行阶段
Step 8/14 : WORKDIR /app
 ---> 5f6g7h8i9j0k
Step 9/14 : COPY --from=builder /app/target/*.jar app.jar
 ---> 6g7h8i9j0k1l
Step 10/14 : RUN addgroup...
 ---> 7h8i9j0k1l2m
Step 11/14 : USER appuser
 ---> 8i9j0k1l2m3n
Step 12/14 : EXPOSE 8080
 ---> 9j0k1l2m3n4o
Step 13/14 : HEALTHCHECK...
 ---> 0k1l2m3n4o5p
Step 14/14 : ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
 ---> 1l2m3n4o5p6q
Successfully built 1l2m3n4o5p6q
Successfully tagged springboot-app:1.0

# 查看镜像大小
$ docker images springboot-app:1.0
REPOSITORY       TAG    SIZE
springboot-app   1.0    238MB

# 对比单阶段构建
# 单阶段: 约700MB
# 多阶段: 238MB (减少66%)
```

### 6.2 Node.js前端项目多阶段构建

**Dockerfile**：

```dockerfile
# Build阶段
FROM node:18-alpine AS builder

WORKDIR /app

# 复制依赖文件（利用缓存）
COPY package*.json ./
RUN npm ci

# 复制源码
COPY . .

# 构建
RUN npm run build

# 运行阶段
FROM nginx:alpine

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制nginx配置
COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**执行过程**：

```bash
$ docker build -t frontend-app:1.0 .
Sending build context to Docker daemon  350KB
Step 1/9 : FROM node:18-alpine AS builder
 ---> 8a3cdc4d1ad3
Step 2/9 : WORKDIR /app
 ---> 9b4cde5e2be4
Step 3/9 : COPY package*.json ./
 ---> 0c5def6f3cf5
Step 4/9 : RUN npm ci
 ---> Running in abc123def456
added 1256 packages in 32s
Removing intermediate container abc123def456
 ---> 1d6efg7g4dg6
Step 5/9 : COPY . .
 ---> 2e7fgh8h5eh7
Step 6/9 : RUN npm run build
 ---> Running in fghijk123456
Creating an optimized production build...
Compiled successfully.
File sizes after gzip:
  142.89 KB  build/static/js/main.abc123.js
Removing intermediate container fghijk123456
 ---> 3g8hij9i6fi8
Step 7/9 : FROM nginx:alpine
 ---> 4h9ijk0j7gj9    ← 开始运行阶段
Step 8/9 : COPY --from=builder /app/dist /usr/share/nginx/html
 ---> 5i0jkl1k8hk0
Step 9/9 : CMD ["nginx", "-g", "daemon off;"]
 ---> 6j1klm2l9il1
Successfully built 6j1klm2l9il1
Successfully tagged frontend-app:1.0

# 查看镜像大小
$ docker images frontend-app:1.0
REPOSITORY        TAG    SIZE
frontend-app      1.0    25.3MB

# 对比单阶段构建
# 单阶段(node:18 + nginx): 约1GB
# 多阶段: 25.3MB (减少97%)
```

### 6.3 Go应用多阶段构建

**Dockerfile**：

```dockerfile
# Build阶段
FROM golang:1.21-alpine AS builder

# 安装git（Go模块需要）
RUN apk add --no-cache git

WORKDIR /app

# 复制go.mod（利用缓存）
COPY go.mod go.sum ./
RUN go mod download

# 复制源码
COPY . .

# 构建（静态编译）
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# 运行阶段
FROM alpine:latest

# 安装证书（用于HTTPS）
RUN apk --no-cache add ca-certificates tzdata

WORKDIR /app

# 只复制二进制（非常小）
COPY --from=builder /app/main .

# 非root用户
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -s /bin/sh -D appuser
USER appuser

EXPOSE 8080

CMD ["./main"]
```

**执行过程**：

```bash
$ docker build -t goapp:1.0 .
Sending build context to Docker daemon  2.3MB
Step 1/11 : FROM golang:1.21-alpine AS builder
 ---> 8a3cdc4d1ad3
Step 2/11 : RUN apk add --no-cache git
 ---> Running in abc123def456
...
 ---> 2b3c4d5e6f7g
Step 3/11 : WORKDIR /app
 ---> 3c4d5e6f7g8h
Step 4/11 : COPY go.mod go.sum ./
 ---> 4d5e6f7g8h9i
Step 5/11 : RUN go mod download
 ---> Running in efghij123456
go: downloading github.com/gin-gonic/gin v1.9.1
...
 ---> 5e6f7g8h9i0j
Step 6/11 : COPY . .
 ---> 6f7g8h9i0j1k
Step 7/11 : RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
 ---> Running in ghijk1234567
# 构建输出...
 ---> 7g8h9i0j1k2l
Step 8/11 : FROM alpine:latest
 ---> 8h9i0j1k2l3m    ← 开始运行阶段
Step 9/11 : RUN apk --no-cache add ca-certificates tzdata
 ---> 9i0j1k2l3m4n
Step 10/11 : COPY --from=builder /app/main .
 ---> 0j1k2l3m4n5o
Step 11/11 : CMD ["./main"]
 ---> 1k2l3m4n5o6p
Successfully built 1k2l3m4n5o6p
Successfully tagged goapp:1.0

# 查看镜像大小
$ docker images goapp:1.0
REPOSITORY   TAG    SIZE
goapp        1.0    15.2MB

# 对比单阶段构建
# 单阶段: 约890MB
# 多阶段: 15.2MB (减少98%)
```

---

## 7. 构建缓存优化实战

### 7.1 利用层缓存

**原则：变化少的放上面，变化多的放下**

**错误示例**：

```dockerfile
# Dockerfile.bad
FROM node:18
COPY . .                    # 代码变化频繁
RUN npm ci                  # 依赖安装不常变，但放下面
RUN npm run build
```

**执行效果**：

```bash
# 第一次构建
$ docker build -t myapp:1.0 .
Step 3/5 : COPY . .
 ---> abc123def456
Step 4/5 : RUN npm ci
 ---> Running in def456ghi789
added 1256 packages in 32s
 ---> ghi789jkl012
Step 5/5 : RUN npm run build
 ---> Running in jkl012mno345
...
Successfully built mno345pqr678

# 修改代码后重新构建（npm ci要重新执行！）
$ docker build -t myapp:1.0 .
Step 3/5 : COPY . .
 ---> pqr678stu901  Changed
Step 4/5 : RUN npm ci              ← 又等了32秒！
 ---> stu901vwx234
Step 5/5 : RUN npm run build
 ---> uvw234wxy345
```

**正确示例**：

```dockerfile
# Dockerfile.good
FROM node:18
COPY package*.json ./     # 依赖文件先复制（不常变）
RUN npm ci                # 安装依赖
COPY . .                  # 代码后复制（常变）
RUN npm run build
```

**执行效果**：

```bash
# 第一次构建
$ docker build -t myapp:1.0 .
Step 3/5 : COPY package*.json ./
 ---> abc123def456
Step 4/5 : RUN npm ci
 ---> Running in def456ghi789
added 1256 packages in 32s
 ---> ghi789jkl012
Step 5/5 : COPY . .
 ---> jkl012mno345
Step 6/5 : RUN npm run build
 ---> mno345pqr678
Successfully built pqr678stu901

# 修改代码后重新构建（npm ci使用缓存！）
$ docker build -t myapp:1.0 .
Step 3/5 : COPY package*.json ./
 ---> abc123def456  Cached
Step 4/5 : RUN npm ci              ← 使用缓存！0秒
 ---> ghi789jkl012  Cached
Step 5/5 : COPY . .
 ---> vwx345yza678  Changed
Step 6/5 : RUN npm run build
 ---> yza678abcd901
Successfully built abcd901efg234
```

### 7.2 手动管理缓存

```bash
# 不使用缓存
$ docker build --no-cache -t myapp:1.0 .
Sending build context to Docker daemon  2.3MB
Step 1/5 : FROM node:18
 ---> Pulling from library/node
abc123def456: Pulling from buildkit-experimental-image
...

# 使用指定镜像的缓存
$ docker build --cache-from=myapp:previous -t myapp:1.0 .
Step 1/5 : FROM node:18
 ---> abc123def456  Cached
Step 2/5 : COPY package*.json ./
 ---> Using cache   ← 使用了previous镜像的缓存！
 ---> def456ghi789
Step 3/5 : RUN npm ci
 ---> Using cache
 ---> ghi789jkl012
```

### 7.3 BuildKit缓存

```bash
# 启用BuildKit
$ export DOCKER_BUILDKIT=1

# 使用GitHub Actions缓存
$ docker build \
  --push \
  --cache-from=type=gha,ref=user/myapp:latest \
  -t user/myapp:latest .
```

---

## 8. 生产环境最佳实践

### 8.1 安全加固

```dockerfile
# 1. 使用非root用户
FROM ubuntu:22.04
RUN useradd -m -u 1000 appuser
USER appuser

# 2. 只读文件系统
# 运行时使用
$ docker run --read-only myapp:1.0

# 3. 不在镜像中存储密钥
# ❌
COPY secrets.json /app/

# ✅ 使用运行时环境变量
ENV API_KEY=${API_KEY}

# 4. 定期更新基础镜像
$ docker pull python:3.11-slim   # 定期更新
$ docker build -t myapp:new .
$ docker tag myapp:new myapp:latest
```

### 8.2 性能优化

```dockerfile
# 1. 使用多阶段构建减小镜像
# 见上面的多阶段构建示例

# 2. 使用轻量基础镜像
FROM python:3.11-alpine  # 而非 python:3.11

# 3. 清理不必要文件
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 4. 并行构建（使用BuildKit）
$ DOCKER_BUILDKIT=1 docker build -t myapp:1.0 .
```

### 8.3 健康检查

```dockerfile
# 添加健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1
```

**执行效果**：

```bash
$ docker run -d --name myapp myapp:1.0
$ docker inspect --format='{{.State.Health.Status}}' myapp
starting

# 等待30秒
$ docker inspect --format='{{.State.Health.Status}}' myapp
healthy
```

---

# 第三部分：Docker Compose实战

## 9. Docker Compose基础

### 9.1 docker-compose.yml语法

```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - ENV=production
    depends_on:
      - db
    networks:
      - frontend

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: appdb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - backend

volumes:
  postgres_data:

networks:
  frontend:
  backend:
```

### 9.2 常用命令

```bash
# 启动所有服务
$ docker-compose up -d

# 查看服务状态
$ docker-compose ps

# 查看日志
$ docker-compose logs -f

# 停止所有服务
$ docker-compose down

# 重新构建镜像
$ docker-compose build

# 进入服务容器
$ docker-compose exec web /bin/bash

# 缩放服务
$ docker-compose up -d --scale web=3
```

---

# 总结

本文档涵盖了Docker和Dockerfile的完整知识体系，从基础概念到生产实践，包括：

1. **Docker核心概念**：容器vs虚拟机、镜像分层、核心组件
2. **Docker安装配置**：Debian系统安装、国内镜像加速
3. **基本操作**：镜像、容器、网络、数据卷
4. **Dockerfile语法**：所有指令详解和实践
5. **多阶段构建**：Java、Node.js、Go项目实战
6. **构建优化**：缓存利用、BuildKit
7. **生产实践**：安全加固、性能优化
8. **Docker Compose**：服务编排

---

# 附录A：常用命令速查表

## A.1 镜像命令

```bash
# 查看镜像列表
docker images
docker image ls

# 拉取镜像
docker pull <image>:<tag>

# 删除镜像
docker rmi <image>
docker image rm <image>

# 强制删除
docker rmi -f <image>

# 查看镜像详情
docker image inspect <image>

# 查看镜像历史
docker history <image>

# 清理悬空镜像
docker image prune

# 清理所有未使用镜像
docker image prune -a

# 构建镜像
docker build -t <name>:<tag> .

# 给镜像打标签
docker tag <source> <target>

# 推送镜像
docker push <image>

# 拉取镜像
docker pull <image>
```

## A.2 容器命令

```bash
# 运行容器
docker run <image>
docker run -d <image>                    # 后台运行
docker run -it <image> /bin/bash         # 交互模式
docker run -p <host>:<container> <image>  # 端口映射

# 查看容器
docker ps                    # 运行中的容器
docker ps -a                  # 所有容器
docker ps -q                  # 只显示ID

# 容器操作
docker start <container>
docker stop <container>
docker restart <container>
docker rm <container>
docker rm -f <container>      # 强制删除

# 进入容器
docker exec -it <container> /bin/bash

# 查看日志
docker logs <container>
docker logs -f <container>    # 实时
docker logs --tail 100 <container>

# 查看资源使用
docker stats <container>
docker stats --no-stream      # 只显示一次

# 查看容器详情
docker inspect <container>

# 复制文件
docker cp <container>:<path> <host-path>
docker cp <host-path> <container>:<path>

# 暂停/恢复容器
docker pause <container>
docker unpause <container>
```

## A.3 网络命令

```bash
# 查看网络
docker network ls

# 创建网络
docker network create <name>

# 查看网络详情
docker network inspect <name>

# 连接容器到网络
docker network connect <network> <container>

# 断开容器网络
docker network disconnect <network> <container>

# 删除网络
docker network rm <name>
```

## A.4 数据卷命令

```bash
# 查看数据卷
docker volume ls

# 创建数据卷
docker volume create <name>

# 查看数据卷详情
docker volume inspect <name>

# 删除数据卷
docker volume rm <name>

# 清理未使用数据卷
docker volume prune
```

## A.5 Docker Compose命令

```bash
# 启动服务
docker-compose up
docker-compose up -d                    # 后台运行

# 停止服务
docker-compose down

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs
docker-compose logs -f

# 构建镜像
docker-compose build

# 重新构建
docker-compose build --no-cache

# 进入容器
docker-compose exec <service> /bin/bash

# 停止服务
docker-compose stop

# 启动服务
docker-compose start

# 重启服务
docker-compose restart

# 查看服务资源
docker-compose top

# 缩放服务
docker-compose up -d --scale <service>=<num>
```

---

# 附录B：常见错误及解决方案

## B.1 构建相关错误

### 错误1：COPY failed

```
COPY failed: file not found in build context
```

**原因**：.dockerignore排除了需要复制的文件

**解决**：

```bash
# 检查.dockerignore
$ cat .dockerignore

# 确认文件路径
$ ls -la <path>
```

### 错误2：Step failed: RUN

```
Step X: RUN apt-get update
E: Could not get lock /var/lib/apt/lists/lock
```

**原因**：前一个容器构建没有正确清理

**解决**：

```bash
# 清理构建缓存
$ docker builder prune

# 使用--no-cache
$ docker build --no-cache -t myapp .
```

### 错误3：No space left on device

**原因**：磁盘空间不足

**解决**：

```bash
# 清理Docker资源
$ docker system prune -a
$ docker volume prune
$ docker builder prune

# 检查磁盘空间
$ df -h
```

## B.2 容器运行错误

### 错误1：端口冲突

```
Error starting userland proxy: listen tcp4 0.0.0.0:8080: bind: address already in use
```

**解决**：

```bash
# 查看端口占用
$ netstat -tlnp | grep 8080

# 使用其他端口
$ docker run -p 8081:80 nginx
```

### 错误2：权限不足

```
permission denied while trying to connect to the Docker daemon socket
```

**解决**：

```bash
# 将用户添加到docker组
$ sudo usermod -aG docker $USER
$ newgrp docker
```

### 错误3：容器退出

```
Exited (1) 2 minutes ago
```

**解决**：

```bash
# 查看日志
$ docker logs <container>

# 检查配置
$ docker inspect <container>
```

## B.3 网络相关错误

### 错误1：DNS解析失败

```
Error: dial tcp: lookup example.com on 8.8.8.8:53: no such host
```

**解决**：

```bash
# 指定DNS
$ docker run --dns 8.8.8.8 myapp

# 检查网络
$ docker exec <container> ping google.com
```

### 错误2：容器间无法通信

```
dial tcp 172.17.0.2:3306: connect: connection refused
```

**解决**：

```bash
# 创建网络
$ docker network create mynet

# 将容器连接到网络
$ docker network connect mynet container1
$ docker network connect mynet container2

# 验证通信
$ docker exec container1 ping container2
```

---

# 附录C：Dockerfile模板库

## C.1 Python Flask模板

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制代码
COPY . .

# 创建用户
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
    
USER appuser

EXPOSE 5000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# 启动命令
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

## C.2 Java Spring Boot模板

```dockerfile
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline -B

COPY src ./src

RUN mvn package -DskipTests -B

FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY --from=builder /app/target/*.jar app.jar

RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser && \
    chown -R appuser:appgroup /app
    
USER appuser

ENV JAVA_OPTS="-Xms256m -Xmx512m"

EXPOSE 8080

HEALTHCHECK --interval=60s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

## C.3 Node.js模板

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

RUN addgroup -g 1000 nodejs && \
    adduser -u 1000 -G nodejs -s /bin/sh -D nodejs && \
    chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

CMD ["node", "dist/index.js"]
```

## C.4 Go模板

```dockerfile
FROM golang:1.21-alpine AS builder

RUN apk add --no-cache git

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

FROM alpine:latest

RUN apk --no-cache add ca-certificates tzdata

WORKDIR /app

COPY --from=builder /app/main .

RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -s /bin/sh -D appuser
    
USER appuser

EXPOSE 8080

CMD ["./main"]
```

## C.5 Nginx模板

```dockerfile
FROM nginx:alpine

# 删除默认配置
RUN rm /etc/nginx/conf.d/default.conf

# 复制配置
COPY nginx.conf /etc/nginx/nginx.conf
COPY conf.d /etc/nginx/conf.d/

# 复制静态文件
COPY html /usr/share/nginx/html/

# 创建非root用户
RUN addgroup -g 1000 nginx && \
    adduser -u 1000 -G nginx -s /bin/sh -D nginx
    
RUN chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d && \
    chown -R nginx:nginx /usr/share/nginx/html

USER nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## C.6 MySQL模板

```dockerfile
FROM mysql:8.0

# 环境变量
ENV MYSQL_ROOT_PASSWORD=root123
ENV MYSQL_DATABASE=appdb
ENV MYSQL_USER=appuser
ENV MYSQL_PASSWORD=app123

# 初始化SQL
COPY init/*.sql /docker-entrypoint-initdb.d/

# 暴露端口
EXPOSE 3306
```

## C.7 PostgreSQL模板

```dockerfile
FROM postgres:15-alpine

ENV POSTGRES_USER=user
ENV POSTGRES_PASSWORD=pass
ENV POSTGRES_DB=appdb

# 初始化SQL
COPY init/*.sql /docker-entrypoint-initdb.d/

EXPOSE 5432
```

## C.8 Redis模板

```dockerfile
FROM redis:7-alpine

# 配置文件
COPY redis.conf /usr/local/etc/redis/redis.conf

# 启动命令
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]

EXPOSE 6379
```

---

# 附录D：环境变量参考

## D.1 通用环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| PATH | 可执行文件路径 | /usr/local/bin:/usr/bin:/bin |
| HOME | 用户主目录 | /root 或 /home/user |
| USER | 当前用户 | root 或 user |
| PWD | 当前工作目录 | /app |
| LANG | 语言设置 | en_US.UTF-8 |
| TZ | 时区 | Asia/Shanghai |

## D.2 Docker相关环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| DOCKER_HOST | Docker守护进程地址 | tcp://localhost:2375 |
| DOCKER_TLS_VERIFY | 启用TLS验证 | 1 |
| DOCKER_CERT_PATH | TLS证书路径 | /path/to/certs |
| DOCKER_BUILDKIT | 启用BuildKit | 1 |

## D.3 应用常见环境变量

### Python

```bash
ENV PYTHONUNBUFFERED=1      # 非缓冲输出
ENV PYTHONDONTWRITEBYTECODE=1  # 不生成.pyc
ENV PYTHONPATH=/app/vendor   # 模块路径
ENV FLASK_APP=app.py         # Flask应用
ENV FLASK_ENV=production     # Flask环境
ENV DJANGO_SETTINGS_MODULE=settings  # Django设置
```

### Java

```bash
ENV JAVA_HOME=/usr/local/openjdk-21
ENV PATH=$JAVA_HOME/bin:$PATH
ENV JAVA_OPTS="-Xms256m -Xmx512m"
ENV SPRING_PROFILES_ACTIVE=prod
```

### Node.js

```bash
ENV NODE_ENV=production
ENV NODE_OPTIONS="--max-old-space-size=512"
ENV PORT=3000
```

---

# 附录E：网络配置参考

## E.1 Docker网络模式

| 模式 | 说明 | 示例 |
|------|------|------|
| bridge | 默认桥接网络 | docker run --network bridge |
| host | 共享主机网络 | docker run --network host |
| none | 无网络 | docker run --network none |
| container | 共享其他容器网络 | docker run --network container:name |

## E.2 创建自定义网络

```bash
# 创建桥接网络
$ docker network create mybridge

# 创建overlay网络（Swarm模式）
$ docker network create -d overlay myoverlay

# 创建macvlan网络
$ docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  mymacvlan
```

## E.3 网络配置示例

```yaml
# docker-compose.yml
services:
  web:
    networks:
      - frontend
      - backend
  api:
    networks:
      - backend
  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.21.0.0/16
```

---

# 附录F：数据卷参考

## F.1 卷类型

| 类型 | 说明 | 示例 |
|------|------|------|
| 匿名卷 | Docker自动创建 | -v /app/data |
| 命名卷 | 用户指定名称 | -v myvolume:/app/data |
| 绑定挂载 | 主机目录 | -v /host/path:/container/path |
| tmpfs | 内存文件系统 | --tmpfs /app/data |

## F.2 卷配置示例

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:15-alpine
    volumes:
      # 命名卷
      - postgres_data:/var/lib/postgresql/data
      # 绑定挂载
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
      # 只读绑定挂载
      - ./config:/app/config:ro
      # tmpfs
      - type: tmpfs
        target: /app/tmp

volumes:
  postgres_data:
    driver: local
```

## F.3 卷驱动

```bash
# 创建本地卷
$ docker volume create myvolume

# 使用local驱动（默认）
$ docker volume create --driver local myvolume

# 使用其他驱动（如nfs）
$ docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.1,rw \
  --opt device=:/path/to/share \
  nfsvolume
```

---

# 附录G：资源限制参考

## G.1 内存限制

```bash
# 限制内存
$ docker run -m 512m myapp

# 限制内存和swap
$ docker run -m 512m --memory-swap 1g myapp

# 软限制
$ docker run -m 512m --memory-reservation 256m myapp
```

## G.2 CPU限制

```bash
# 限制CPU核心数
$ docker run --cpus=2 myapp

# 限制特定核心
$ docker run --cpuset-cpus="0,1" myapp

# CPU权重
$ docker run --cpu-shares=1024 myapp
```

## G.3 I/O限制

```bash
# 限制读取速度
$ docker run --device-read-bps /dev/sda:1mb myapp

# 限制写入速度
$ docker run --device-write-bps /dev/sda:1mb myapp

# 限制IOPS
$ docker run --device-read-iops /dev/sda:100 myapp
$ docker run --device-write-iops /dev/sda:100 myapp
```

## G.4 资源限制配置示例

```yaml
# docker-compose.yml
services:
  web:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 256M
```

---

# 附录H：安全相关参考

## H.1 安全最佳实践

1. **使用非root用户**
2. **定期更新基础镜像**
3. **不存储敏感信息**
4. **使用只读文件系统**
5. **限制容器权限**
6. **启用健康检查**
7. **使用Docker Bench进行安全审计**

```bash
# 运行Docker Bench
$ docker run -it --network host \
  --pid host \
  -v /var/run/docker.sock:/var/run/docker.sock \
  docker/docker-bench-security
```

## H.2 安全扫描

```bash
# 使用Trivy扫描镜像
$ docker run --rm -v \
  /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy:latest image myapp:1.0

# 使用Clair扫描镜像
$ docker run -p 5432:5432 -d \
  --name clair \
  quay.io/coreos/clair:latest

$ curl -X POST -H "Content-Type: application/json" \
  -d '{"ImageName": "myapp:1.0"}' \
  http://localhost:5432/api/v1/scan
```

---

# 附录I：监控和日志参考

## I.1 监控工具

```bash
# Docker Stats
$ docker stats myapp

# cAdvisor（容器监控）
$ docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  gcr.io/cadvisor/cadvisor:v0.47.0
```

## I.2 日志管理

```bash
# 查看日志
$ docker logs myapp

# 实时日志
$ docker logs -f myapp

# 最近100行
$ docker logs --tail 100 myapp

# 带时间戳
$ docker logs -t myapp

# JSON格式日志
$ docker logs --format json myapp

# 使用ELK栈收集日志
# docker-compose.yml
services:
  elasticsearch:
    image: elasticsearch:8.0
    environment:
      - discovery.type=single-node
    volumes:
      - es_data:/usr/share/elasticsearch/data

  kibana:
    image: kibana:8.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200

  logstash:
    image: logstash:8.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
```

---

# 附录J：CI/CD集成参考

## J.1 GitHub Actions

```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: myapp/myapp
          tags: |
            type=ref,event=branch
            type=sha,prefix=
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## J.2 GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - push

variables:
  DOCKER_HOST: tcp://docker:2375
  DOCKER_TLS_CERTDIR: ""

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker build -t $CI_REGISTRY_IMAGE:latest .

test:
  stage: test
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker run $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA test

push:
  stage: push
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
```

---

# 附录K：Kubernetes集成参考

## K.1 Dockerfile优化（用于K8s）

```dockerfile
# 多阶段构建
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/main .

# 非root用户
RUN addgroup -g 1000 -S appgroup && \
    adduser -u 1000 -S appuser -G appgroup
    
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

CMD ["./main"]
```

## K.2 Kubernetes部署配置

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 8080
        resources:
          limits:
            cpu: "1"
            memory: "512Mi"
          requests:
            cpu: "0.5"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
```

---

# 附录L：性能调优参考

## L.1 镜像大小优化

```dockerfile
# 1. 使用alpine基础镜像
FROM node:18-alpine

# 2. 多阶段构建
FROM node:18 AS builder
RUN npm run build

FROM node:18-alpine
COPY --from=builder /app/dist ./dist

# 3. 合并RUN指令
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 4. 使用--no-cache-dir
RUN pip install --no-cache-dir -r requirements.txt

# 5. 清理临时文件
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc && \
    rm -rf /var/lib/apt/lists/*
```

## L.2 构建速度优化

```bash
# 1. 利用缓存
# 变化少的放上面

# 2. 使用BuildKit
$ export DOCKER_BUILDKIT=1
$ docker build .

# 3. 使用远程缓存
$ docker build \
  --cache-from=type=registry,ref=myapp:latest \
  --push \
  -t myapp:latest .

# 4. 并行构建（Dockerfile中）
RUN npm run build:css & \
    npm run build:js & \
    wait
```

## L.3 运行时性能优化

```bash
# 1. 使用gunicorn/uwsgi等WSGI服务器
CMD ["gunicorn", "--workers", "4", "--bind", "0.0.0.0:5000", "app:app"]

# 2. 使用--init或tini处理信号
$ docker run --init myapp

# 3. 限制资源
$ docker run -m 512m --cpus=1 myapp

# 4. 使用只读文件系统
$ docker run --read-only myapp

# 5. 使用tmpfs存储临时文件
$ docker run --tmpfs /tmp myapp
```

---

本文档持续更新中，涵盖了Docker和Dockerfile的完整知识体系。通过实践这些示例和配置，可以掌握容器化技术的核心技能。

## 4. Dockerfile基础语法

### 4.1 Dockerfile基本结构

**完整Dockerfile示例**：

```dockerfile
# 基础镜像
FROM python:3.11-slim

# 维护者（已废弃，用LABEL替代）
LABEL maintainer="your@email.com"

# 设置工作目录
WORKDIR /app

# 复制依赖文件
COPY requirements.txt .

# 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 设置环境变量
ENV FLASK_APP=app.py
ENV FLASK_ENV=production

# 暴露端口
EXPOSE 5000

# 创建非root用户
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
    
# 切换用户
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:5000/health || exit 1

# 启动命令
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
```

**执行构建过程**：

```bash
$ docker build -t myapp:1.0 .
Sending build context to Docker daemon  2.3MB
Step 1/14 : FROM python:3.11-slim
 ---> 3aab8a267f4e
Step 2/14 : LABEL maintainer="your@email.com"
 ---> Running in 9a5b4c8e1234
Removing intermediate container 9a5b4c8e1234
 ---> d7f8a1234567
Step 3/14 : WORKDIR /app
 ---> Running in 0a1b2c3d4e5f
Removing intermediate container 0a1b2c3d4e5f
 ---> e1f2a3b4c5d6
Step 4/14 : COPY requirements.txt .
 ---> f2a3b4c5d6e7
Step 5/14 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in 1a2b3c4d5e6f
Collecting Flask (3.0.0)
  Downloading Flask-3.0.0-py3-none-any.whl (63kB)
Collecting gunicorn (21.2.0)
  Downloading gunicorn-21.2.0-py3-none-any.whl (123kB)
Installing collected packages: Flask, gunicorn
Successfully installed Flask-3.0.0 gunicorn-21.2.0
Removing intermediate container 1a2b3c4d5e6f
 ---> 2b3c4d5e6f7g
Step 6/14 : COPY . .
 ---> 3c4d5e6f7g8h
Step 7/14 : ENV FLASK_APP=app.py
 ---> 4d5e6f7g8h9i
Step 8/14 : ENV FLASK_ENV=production
 ---> 5e6f7g8h9i0j
Step 9/14 : EXPOSE 5000
 ---> 6f7g8h9i0j1k
Step 10/14 : RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
 ---> Running in 7g8h9i0j1k2l
Removing intermediate container 7g8h9i0j1k2l
 ---> 8h9i0j1k2l3m
Step 11/14 : USER appuser
 ---> 9i0j1k2l3m4n
Step 12/14 : HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 CMD curl -f http://localhost:5000/health || exit 1
 ---> 0j1k2l3m4n5o
Step 13/14 : CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "app:app"]
 ---> 1k2l3m4n5o6p
Successfully built 1k2l3m4n5o6p
Successfully tagged myapp:1.0

$ docker images myapp:1.0
REPOSITORY   TAG    SIZE
myapp        1.0    245MB
```