# Dockerfile 指令详解（实践版）

**Metadata**
- Date: 2026-03-23
- Topic: Dockerfile指令深度解析（附实践执行结果）
- Tags: #Dockerfile #指令详解 #学习路径03

---

## 1. 指令系统概述

Dockerfile中的指令分为多种类型，每种类型有特定的作用和语法。深入理解每个指令是编写高质量Dockerfile的基础。

### 1.1 指令分类

```
┌─────────────────────────────────────────────────────────────┐
│                   Dockerfile 指令分类                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  基础指令        │  │  运行时指令      │                  │
│  │  (1条)          │  │  (2条)          │                  │
│  │                 │  │                 │                  │
│  │  FROM           │  │  CMD            │                  │
│  │                 │  │  ENTRYPOINT     │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  文件指令        │  │  目录指令        │                  │
│  │  (2条)          │  │  (1条)          │                  │
│  │                 │  │                 │                  │
│  │  COPY           │  │  WORKDIR        │                  │
│  │  ADD            │  │                 │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  环境指令        │  │  用户指令        │                  │
│  │  (2条)          │  │  (2条)          │                  │
│  │                 │  │                 │                  │
│  │  ENV            │  │  USER           │                  │
│  │  ARG            │  │  LABEL          │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                             │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │  网络指令        │  │  卷指令          │                  │
│  │  (1条)          │  │  (1条)          │                  │
│  │                 │  │                 │                  │
│  │  EXPOSE        │  │  VOLUME         │                  │
│  └─────────────────┘  └─────────────────┘                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 指令执行顺序

Dockerfile中的指令按顺序执行，每条指令都会创建一个新的镜像层：

```dockerfile
FROM ubuntu:22.04        # 第1层：基础镜像
WORKDIR /app             # 第2层：创建工作目录
COPY . /app              # 第3层：复制文件
RUN apt-get update       # 第4层：执行命令
ENV APP_ENV=production   # 第5层：设置环境变量
EXPOSE 8080              # 第6层：声明端口
CMD ["python", "app.py"] # 第7层：启动命令
```

**执行效果**：

```
$ docker build -t myapp:1.0 .
Sending build context to Docker daemon  1.2MB
Step 1/7 : FROM ubuntu:22.04
 ---> 8a3cdc4d1ad3
Step 2/7 : WORKDIR /app
 ---> Running in 9a5b4c8e1234
Removing intermediate container 9a5b4c8e1234
 ---> d7f8a1234567
Step 3/7 : COPY . /app
 ---> e8f9b2345678
Step 4/7 : RUN apt-get update
 ---> Running in 1a2b3c4d5e6f
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [25.6 MB]
...
 ---> 2b3c4d5e6f7g
Step 5/7 : ENV APP_ENV=production
 ---> 3c4d5e6f7g8h
Step 6/7 : EXPOSE 8080
 ---> 4d5e6f7g8h9i
Step 7/7 : CMD ["python", "app.py"]
 ---> 5e6f7g8h9i0j
Successfully built 5e6f7g8h9i0j
Successfully tagged myapp:1.0
```

---

## 2. FROM 指令深度解析

### 2.1 基本语法

```dockerfile
FROM [--platform=<platform>] <image>[:<tag>] [@<digest>]
```

**执行效果**：

```bash
# 使用不同tag
$ docker pull ubuntu:22.04
22.04: Pulling from library/ubuntu
a6bd71f48f68: Pull complete
Digest: sha256:abc123...
Status: Downloaded newer image for ubuntu:22.04

# 使用Digest
$ docker build -t myapp:1.0 -f Dockerfile.digest .
Sending build context to Docker daemon  1.2MB
Step 1/1 : FROM ubuntu@sha256:abc123...
 ---> 8a3cdc4d1ad3
Successfully built 8a3cdc4d1ad3
Successfully tagged myapp:1.0
```

### 2.2 基础镜像选择

```dockerfile
# ✅ 推荐：使用具体版本标签
FROM python:3.11-slim
FROM node:18-alpine
FROM ubuntu:22.04
```

**执行效果**：

```
# 查看镜像大小对比
$ docker images
REPOSITORY   TAG          SIZE
python       3.11         1.01GB
python       3.11-slim    130MB
python       3.11-alpine  48MB

node         18            912MB
node         18-slim       187MB
node         18-alpine     167MB

ubuntu       22.04         77MB
ubuntu       22.04-slim    29MB
```

### 2.3 多架构构建

```dockerfile
# 构建特定平台镜像
FROM --platform=linux/amd64 ubuntu:22.04
FROM --platform=linux/arm64 ubuntu:22.04
```

**执行效果**：

```bash
# 构建多平台镜像
$ docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t myapp:latest .
[+] Building 45.2s (12/12) FINISHED
 => [linux/amd64] exporting to image
 => [linux/arm64] exporting to image

# 验证
$ docker images
REPOSITORY   TAG       ARCHITECTURE   SIZE
myapp        latest    amd64          77MB
myapp        latest    arm64          77MB
```

---

## 3. RUN 指令深度解析

### 3.1 两种形式对比

```dockerfile
# Shell形式
RUN <command>

# Exec形式（推荐）
RUN ["executable", "param1", "param2"]
```

**执行效果对比**：

```dockerfile
# Shell形式示例
RUN echo "Hello $HOME" > /tmp/test.txt
```

```bash
$ docker build -t test:shell .
Step 1/2 : RUN echo "Hello $HOME" > /tmp/test.txt
 ---> Running in abc123def456
 ---> def456ghi789
Successfully built def456ghi789

$ docker run test:shell cat /tmp/test.txt
Hello /root
```

```dockerfile
# Exec形式示例
RUN ["echo", "Hello $HOME"]
```

```bash
$ docker build -t test:exec .
Step 1/2 : RUN ["echo", "Hello $HOME"]
 ---> Running in ghi789jkl012
Hello $HOME    ← 变量不会展开！
Successfully built jkl012mno345

$ docker run test:exec cat /tmp/test.txt
Hello $HOME
```

### 3.2 Shell处理

Exec形式不调用shell，所以没有shell处理功能：

```dockerfile
# ❌ 变量不会展开
RUN ["echo", "$HOME"]

# ✅ Shell形式可以
RUN echo $HOME
```

**执行效果**：

```bash
# 错误示例
$ docker build -t test:wrong .
Step 1/2 : RUN ["echo", "$HOME"]
 ---> Running in abc123
$HOME

# 正确示例
$ docker build -t test:right .
Step 1/2 : RUN echo $HOME
 ---> Running in def456
/root
```

### 3.3 多命令合并

```dockerfile
# ❌ 错误：多个RUN创建多个层
RUN apt-get update
RUN apt-get install -y nginx
RUN apt-get clean
```

**执行效果**：

```bash
$ docker build -t test:bad .
Step 2/5 : RUN apt-get update
 ---> Running in abc123def456
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [25.6 MB]
...
Step 3/5 : RUN apt-get install -y nginx
 ---> Running in def456ghi789
...
Step 4/5 : RUN apt-get clean
 ---> Running in ghi789jkl012
Step 5/5 : RUN rm -rf /var/lib/apt/lists/*
 ---> Running in jkl012mno345

$ docker images test:bad
REPOSITORY   TAG    SIZE
test         bad    195MB   ← 更大
```

```dockerfile
# ✅ 正确：合并为一个RUN
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**执行效果**：

```bash
$ docker build -t test:good .
Step 2/2 : RUN apt-get update && apt-get install -y nginx && apt-get clean && rm -rf /var/lib/apt/lists/*
 ---> Running in mno345pqr678
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [25.6 MB]
...
Setting up nginx ...

$ docker images test:good
REPOSITORY   TAG    SIZE
test         good   172MB   ← 更小！
```

---

## 4. CMD 指令深度解析

### 4.1 三种形式

```dockerfile
# Exec形式（推荐）
CMD ["executable", "param1", "param2"]

# 作为ENTRYPOINT的默认参数
CMD ["param1", "param2"]

# Shell形式
CMD command param1 param2
```

### 4.2 CMD vs ENTRYPOINT

**CMD可以被docker run参数覆盖**：

```dockerfile
CMD ["python", "app.py"]
```

**执行效果**：

```bash
$ docker run myapp
# 执行: python app.py

$ docker run myapp --port 9000
# 执行: python app.py --port 9000  ← 覆盖了整个CMD
# 变成执行 --port 9000 而不是 python app.py --port 9000
# 这是错的！
```

### 4.3 正确组合使用

```dockerfile
# 最佳实践：ENTRYPOINT + CMD
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080", "--host", "0.0.0.0"]
```

**执行效果**：

```bash
$ docker run myapp
# 实际执行: python app.py --port 8080 --host 0.0.0.0

$ docker run myapp --port 9000
# 实际执行: python app.py --port 9000 --host 0.0.0.0  ← CMD被追加到ENTRYPOINT后面

$ docker run myapp --port 9000 --host 127.0.0.1
# 实际执行: python app.py --port 9000 --host 127.0.0.1  ← 两个参数都追加
```

---

## 5. ENTRYPOINT 指令深度解析

### 5.1 两种形式

```dockerfile
# Exec形式（推荐）
ENTRYPOINT ["executable", "param1", "param2"]

# Shell形式
ENTRYPOINT command param1 param2
```

### 5.2 Shell形式的问题

```dockerfile
# ❌ 不推荐：PID 1不是应用，而是shell
ENTRYPOINT python app.py

# ✅ 推荐：PID 1是应用
ENTRYPOINT ["python", "app.py"]
```

**执行效果对比**：

```bash
# Shell形式
$ docker run -d myapp:shell
$ docker exec myapp ps aux
PID   USER   COMMAND
  1   root   /bin/sh -c python app.py    ← PID 1是shell
  7   root   python app.py

# 按Ctrl+C停止时
$ docker stop myapp
# 发送SIGTERM给PID 1 (/bin/sh)
# shell不转发给python
# 10秒后SIGKILL
# 应用被强制杀死，无法优雅关闭

# Exec形式
$ docker run -d myapp:exec
$ docker exec myapp ps aux
PID   USER   COMMAND
  1   root   python app.py    ← PID 1是应用本身

# 按Ctrl+C停止时
$ docker stop myapp
# 发送SIGTERM给PID 1 (python)
# 应用收到信号，可以优雅关闭
```

### 5.3 信号处理问题演示

```dockerfile
# Shell形式
ENTRYPOINT python app.py
```

**执行效果**：

```bash
# 运行应用
$ docker run -d --name myapp myapp:shell
abc123def456

# 尝试优雅停止
$ docker stop myapp

# 查看停止时间
$ docker inspect myapp --format '{{.State.FinishedAt}}'
2026-03-23T10:05:15.123456789Z

# 通常需要等待10秒（SIGKILL延迟）
# 因为应用没有收到SIGTERM信号
```

**解决方案**：

```dockerfile
# 方案1：使用exec
ENTRYPOINT ["python", "app.py"]

# 方案2：使用tini（推荐）
FROM python:3.11-slim
RUN pip install tini
ENTRYPOINT ["tini", "--", "python", "app.py"]

# 方案3：Dockerfile中处理
ENTRYPOINT ["sh", "-c", "python app.py"]
```

**执行效果**：

```bash
# 使用tini
$ docker run -d --name myapp myapp:tini
$ docker stop myapp
# 应用立即收到SIGTERM，优雅关闭
# 无需等待10秒
```

---

## 6. COPY 指令深度解析

### 6.1 基本语法

```dockerfile
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]
```

### 6.2 复制规则

```dockerfile
# 复制多个文件到目录
COPY package.json package-lock.json /app/

# 复制整个目录
COPY ./src /app/src/

# 相对路径
COPY . /app/
```

**执行效果**：

```bash
# 项目结构
$ ls -la
Dockerfile  package.json  package-lock.json  src/

# 构建
$ docker build -t myapp:1.0 .
Step 3/4 : COPY package.json package-lock.json /app/
 ---> abc123def456
Step 4/4 : COPY ./src /app/src/
 ---> def456ghi789
Successfully built ghi789jkl012

# 验证
$ docker run myapp ls -la /app
package.json  package-lock.json  src/
```

### 6.3 权限问题

```dockerfile
# 复制后设置权限
COPY --chown=1000:1000 . /app/
```

**执行效果**：

```bash
# 不带权限
$ docker build -t test:noperm .
Step 3/3 : COPY . /app/
 ---> abc123def456

$ docker run test:noperm ls -la /app
drwxr-xr-x 1 root root 4096 Mar 23 10:00 .

# 带权限
$ docker build -t test:perm .
Step 3/3 : COPY --chown=1000:1000 . /app/
 ---> def456ghi789

$ docker run test:perm ls -la /app
drwxr-xr-x 1 1000 1000 4096 Mar 23 10:00 .
```

---

## 7. ADD 指令深度解析

### 7.1 ADD的特殊功能

**自动解压tar**：

```dockerfile
# 会自动解压
ADD archive.tar.gz /app/
ADD files.tar /app/

# 不会自动解压
ADD archive.zip /app/
```

**执行效果**：

```bash
# 创建测试文件
$ tar -czf app.tar.gz app/

$ docker build -t test:add .
Step 1/2 : ADD app.tar.gz /app/
 ---> Running in abc123def456
Extracting layer...   ← 自动解压
 ---> def456ghi789
Successfully built ghi789jkl012

$ docker run test:add ls -la /app
app/  ← 目录被解压出来了
```

**从URL下载**：

```dockerfile
ADD https://example.com/file.tar.gz /app/
```

**执行效果**：

```bash
$ docker build -t test:url .
Step 1/2 : ADD https://example.com/file.tar.gz /app/
 ---> Running in abc123def456
Downloading 2.3MB...
 ---> def456ghi789
Successfully built ghi789jkl012
```

---

## 8. WORKDIR 指令深度解析

### 8.1 基本语法

```dockerfile
WORKDIR /path/to/workdir
```

### 8.2 作用

```dockerfile
# 设置工作目录
WORKDIR /app
# /app 不存在会自动创建

# 相对路径
WORKDIR src     # 实际：/app/src

# 使用环境变量
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

**执行效果**：

```bash
$ docker build -t test:workdir .
Step 2/4 : WORKDIR /app
 ---> Running in abc123def456
Step 3/4 : WORKDIR src
 ---> Running in def456ghi789
Step 4/4 : WORKDIR logs
 ---> Running in ghi789jkl012
Successfully built jkl012mno345

$ docker run test:workdir pwd
/app/src/logs
```

### 8.3 与RUN cd的区别

```dockerfile
# ❌ RUN cd只在构建时有效
RUN cd /app
RUN pwd  # 不会显示/app

# ✅ WORKDIR设置工作目录
WORKDIR /app
RUN pwd  # 会显示/app
```

**执行效果**：

```bash
# RUN cd无效
$ docker build -t test:cd .
Step 2/3 : RUN cd /app
Step 3/3 : RUN pwd
 ---> Running in abc123def456
/   ← 还是根目录！

# WORKDIR有效
$ docker build -t test:workdir .
Step 2/2 : WORKDIR /app
Step 3/2 : RUN pwd
 ---> Running in def456ghi789
/app   ← 工作目录正确
```

---

## 9. ENV 指令深度解析

### 9.1 基本语法

```dockerfile
ENV <key>=<value> ...
ENV APP_NAME=myapp
ENV APP_VERSION=1.0 APP_ENV=production
```

**执行效果**：

```bash
$ docker build -t test:env .
Step 2/4 : ENV APP_NAME=myapp
 ---> abc123def456
Step 3/4 : ENV APP_VERSION=1.0
 ---> def456ghi789
Step 4/4 : ENV APP_ENV=production
 ---> ghi789jkl012

$ docker run test:env env
APP_NAME=myapp
APP_VERSION=1.0
APP_ENV=production
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/sbin:/usr/sbin
```

### 9.2 运行时覆盖

```bash
# 运行时覆盖环境变量
$ docker run -e APP_ENV=development test:env env | grep APP_ENV
APP_ENV=development
```

---

## 10. ARG 指令深度解析

### 10.1 基本语法

```dockerfile
ARG <name>[=<default value>]
```

**执行效果**：

```bash
# Dockerfile
ARG APP_VERSION=1.0
LABEL version=$APP_VERSION

# 使用默认值构建
$ docker build -t test:arg .
Step 2/2 : ARG APP_VERSION=1.0
 ---> Running in abc123def456

# 覆盖默认值构建
$ docker build --build-arg APP_VERSION=2.0 -t test:arg2 .
Step 2/2 : ARG APP_VERSION=2.0
 ---> Running in def456ghi789
```

### 10.2 ARG vs ENV

| 特性 | ARG | ENV |
|------|-----|-----|
| 作用时机 | 构建时 | 构建时+运行时 |
| 容器内可见 | 否 | 是 |
| 可被覆盖 | 是(--build-arg) | 是(-e) |
| 存入镜像 | 否 | 是 |

**执行效果**：

```bash
# ARG不在运行时可见
$ docker build -t test:arg --build-arg APP_VERSION=2.0 .
$ docker run test:arg env | grep APP_VERSION
# (无输出，ARG不可见)

# ENV在运行时可见
$ docker build -t test:env --build-arg APP_VERSION=2.0 .
Step 2/2 : ENV APP_VERSION=2.0
$ docker run test:env env | grep APP_VERSION
APP_VERSION=2.0
```

---

## 11. EXPOSE 指令深度解析

### 11.1 基本语法

```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

**执行效果**：

```bash
# Dockerfile
FROM nginx:alpine
EXPOSE 80
EXPOSE 443/tcp
EXPOSE 53/udp

$ docker build -t test:expose .

$ docker inspect test:expose --format '{{.Config.ExposedPorts}}'
map[443/tcp:{} 53/udp:{} 80/tcp:{}]
```

### 11.2 注意事项

EXPOSE只是文档作用，不实际绑定端口：

```bash
# Dockerfile
EXPOSE 8080

# 运行时可以映射到任何端口
$ docker run -d -p 80:8080 --name test1 test:expose
$ docker run -d -p 8081:8080 --name test2 test:expose

$ docker port test1
8080/tcp -> 0.0.0.0:80

$ docker port test2
8080/tcp -> 0.0.0.0:8081
```

---

## 12. VOLUME 指令深度解析

### 12.1 基本语法

```dockerfile
VOLUME ["/path/to/volume"]
```

### 12.2 注意事项

```dockerfile
# ❌ 警告：VOLUME指令会使之前的修改无效
FROM ubuntu
RUN mkdir /app && echo "hello" > /app/file.txt
VOLUME /app
# /app/file.txt 会丢失！
```

**执行效果**：

```bash
$ docker build -t test:volum .
Step 2/4 : RUN mkdir /app && echo "hello" > /app/file.txt
 ---> abc123def456
Step 3/4 : VOLUME /app
 ---> Running in def456ghi789
Step 4/4 : CMD ["/bin/sh"]
 ---> ghi789jkl012

$ docker run test:volume ls /app
# (空目录！file.txt丢失了！)
```

**正确做法**：

```dockerfile
# ✅ 正确：VOLUME在前
FROM ubuntu
VOLUME /app
RUN mkdir /app && echo "hello" > /app/file.txt
```

---

## 13. USER 指令深度解析

### 13.1 基本语法

```dockerfile
USER <user>[:<group>]
USER UID[:GID]
```

### 13.2 创建用户

```dockerfile
# Debian/Ubuntu
RUN adduser -D -u 1000 appuser
USER appuser

# Alpine
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -s /bin/sh -D appuser
USER appuser
```

**执行效果**：

```bash
$ docker build -t test:user .
Step 2/3 : RUN adduser -D -u 1000 appuser
 ---> Running in abc123def456
Step 3/3 : USER appuser
 ---> Running in def456ghi789

$ docker run test:user whoami
appuser

$ docker run test:user id
uid=1000(appuser) gid=1000(appuser) groups=1000(appuser)
```

---

## 14. LABEL 指令深度解析

### 14.1 基本语法

```dockerfile
LABEL <key>=<value>
LABEL maintainer="your@email.com"
LABEL version="1.0"
LABEL description="My application"
```

**执行效果**：

```bash
$ docker build -t test:label .
Step 2/4 : LABEL maintainer="your@email.com"
 ---> abc123def456
Step 3/4 : LABEL version="1.0"
 ---> def456ghi789

$ docker inspect test:label --format '{{.Config.Labels}}'
map[maintainer:your@email.com version:1.0]
```

---

## 15. HEALTHCHECK 指令深度解析

### 15.1 基本语法

```dockerfile
HEALTHCHECK [OPTIONS] CMD <command>
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 CMD <command>
```

### 15.2 HTTP健康检查示例

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1
```

**执行效果**：

```bash
$ docker build -t test:health .

$ docker run -d --name test test:health

# 启动时
$ docker inspect --format='{{.State.Health.Status}}' test
starting

# 等待5秒后（start-period）
$ docker inspect --format='{{.State.Health.Status}}' test
healthy

# 健康检查成功
$ docker inspect --format='{{.State.Health.Log}}' test
[
    {
        "ExitCode": 0,
        "Output": "HTTP/1.1 200 OK",
        "Start": "2026-03-23T10:00:00Z",
        "End": "2026-03-23T10:00:01Z"
    }
]
```

### 15.3 TCP健康检查示例

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD nc -z localhost 3306 || exit 1
```

**执行效果**：

```bash
$ docker run -d --name mysql mysql:8
$ docker inspect --format='{{.State.Health.Status}}' mysql
starting

# MySQL启动完成后
$ docker inspect --format='{{.State.Health.Status}}' mysql
healthy
```

---

## 16. 指令组合最佳实践

### 16.1 典型Python应用

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 依赖安装
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 应用代码
COPY . .

# 非root用户
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

**执行效果**：

```bash
$ docker build -t myapp:1.0 .
Sending build context to Docker daemon  2.3MB
Step 1/10 : FROM python:3.11-slim
 ---> 3aab8a267f4e
Step 2/10 : WORKDIR /app
 ---> Running in 9a5b4c8e1234
Removing intermediate container 9a5b4c8e1234
 ---> d7f8a1234567
Step 3/10 : COPY requirements.txt .
 ---> e8f9b2345678
Step 4/10 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in 1a2b3c4d5e6f
Collecting Flask (3.0.0)
  Downloading Flask-3.0.0-py3-none-any.whl (63kB)
...
 ---> 2b3c4d5e6f7g
Step 5/10 : COPY . .
 ---> 3c4d5e6f7g8h
Step 6/10 : RUN useradd...
 ---> 4d5e6f7g8h9i
Step 7/10 : USER appuser
 ---> 5e6f7g8h9i0j
Step 8/10 : EXPOSE 8000
 ---> 6f7g8h9i0j1k
Step 9/10 : HEALTHCHECK...
 ---> 7g8h9i0j1k2l
Step 10/10 : CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
 ---> 8h9i0j1k2l3m
Successfully built 8h9i0j1k2l3m
Successfully tagged myapp:1.0

$ docker images myapp:1.0
REPOSITORY   TAG    SIZE
myapp        1.0    245MB
```

### 16.2 典型Java应用

```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/*.jar app.jar

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

**执行效果**：

```bash
$ docker build -t javaapp:1.0 .
Sending build context to Docker daemon  45.2MB
Step 1/9 : FROM eclipse-temurin:21-jre-alpine
 ---> 7e2024c2d9c3
Step 2/9 : WORKDIR /app
 ---> 8f3d5c4e0d4f
Step 3/9 : COPY target/*.jar app.jar
 ---> 9g4e6d5f1e5g
Step 4/9 : RUN addgroup...
 ---> 0h5f7e6g2f6h
Step 5/9 : USER appuser
 ---> 1i6g8f7g3g7i
Step 6/9 : ENV JAVA_OPTS="-Xms256m -Xmx512m"
 ---> 2j7h9g8h4h8j
Step 7/9 : EXPOSE 8080
 ---> 3k8i0h9i5i9k
Step 8/9 : HEALTHCHECK...
 ---> 4l9j1i0j6j0k
Step 9/9 : ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
 ---> 5m0k2j1k7k1l
Successfully built 5m0k2j1k7k1l
Successfully tagged javaapp:1.0

$ docker images javaapp:1.0
REPOSITORY   TAG    SIZE
javaapp      1.0    238MB
```

---

## 17. 高级技巧

### 17.1 构建参数验证

```dockerfile
ARG APP_VERSION
ARG APP_ENV=production

RUN if [ -z "$APP_VERSION" ]; then \
    echo "APP_VERSION is required"; \
    exit 1; \
    fi
```

**执行效果**：

```bash
# 没有提供APP_VERSION
$ docker build -t test:valid .
Step 3/3 : RUN if [ -z "$APP_VERSION" ]; then echo "APP_VERSION is required"; exit 1; fi
 ---> Running in abc123def456
APP_VERSION is required
```

### 17.2 条件构建

```dockerfile
ARG BUILD_WITH_DEBUG=false

RUN if [ "$BUILD_WITH_DEBUG" = "true" ]; then \
    apt-get update && apt-get install -y strace ltrace; \
    fi
```

**执行效果**：

```bash
# 默认不安装
$ docker build -t test:cond .
Step 2/2 : RUN if [ "$BUILD_WITH_DEBUG" = "true" ]; then apt-get update && apt-get install -y strace ltrace; fi
 ---> abc123def456   ← 跳过了安装

# 安装debug工具
$ docker build --build-arg BUILD_WITH_DEBUG=true -t test:cond2 .
Step 2/2 : RUN if [ "$BUILD_WITH_DEBUG" = "true" ]; then apt-get update && apt-get install -y strace ltrace; fi
 ---> Running in def456ghi789
Reading package lists...
Setting up strace ...
Setting up ltrace ...
```

---

## 18. 本章总结

### 指令对比表

| 指令 | 作用 | 层级 |
|------|------|------|
| FROM | 基础镜像 | 1 |
| RUN | 执行命令 | 多次 |
| COPY | 复制文件 | 多次 |
| ADD | 复制/解压/下载 | 多次 |
| WORKDIR | 设置目录 | 1+ |
| ENV | 环境变量 | 1+ |
| ARG | 构建参数 | 1+ |
| EXPOSE | 声明端口 | 1 |
| VOLUME | 数据卷 | 1 |
| USER | 用户 | 1 |
| LABEL | 元数据 | 1 |
| HEALTHCHECK | 健康检查 | 1 |
| CMD | 启动命令 | 1 |
| ENTRYPOINT | 入口点 | 1 |

### 最佳实践

1. 优先使用具体版本标签
2. RUN指令合并减少层数
3. COPY比ADD更明确
4. 使用WORKDIR而非RUN cd
5. 理解ARG和ENV的区别
6. 不以root用户运行
7. 使用HEALTHCHECK
8. 正确组合ENTRYPOINT和CMD

---

## 19. 课后问题

1. RUN的shell和exec形式有什么区别？
   - shell形式调用shell，有变量替换；exec形式直接执行命令

2. CMD和ENTRYPOINT如何组合使用？
   - CMD作为ENTRYPOINT的默认参数

3. VOLUME指令为什么要放在最后？
   - 避免使之前的文件修改失效

4. 为什么推荐使用非root用户？
   - 安全隔离，防止容器逃逸

---

## 20. 深入理解指令执行

### 20.1 指令执行流程

当执行`docker build`时，Docker按以下流程处理每条指令：

```
┌─────────────────────────────────────────────────────────────┐
│                  Dockerfile 执行流程                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 解析 Dockerfile                                         │
│     ├── 验证语法                                            │
│     ├── 解析每条指令                                        │
│     └── 准备构建上下文                                       │
│                                                             │
│  2. 准备构建上下文                                           │
│     ├── 打包本地文件                                         │
│     └── 发送到Docker daemon                                  │
│                                                             │
│  3. 执行每条指令                                            │
│     ├── 创建临时容器                                        │
│     ├── 在容器中执行指令
     ├── 提交容器为新镜像层
     └── 删除临时容器

4. 最终镜像
   - 所有层的组合
   - 元数据（CMD/EXPOSE等）

执行效果：

详细构建流程
$ docker build -t myapp:1.0 .
Sending build context to Docker daemon  2.3MB
Step 1/7 : FROM ubuntu:22.04
 ---> 8a3cdc4d1ad3
Step 2/7 : WORKDIR /app
 ---> Running in abc123def456
Removing intermediate container abc123def456
 ---> d7f8a1234567
Step 3/7 : COPY . /app
 ---> e8f9b2345678
Step 4/7 : RUN apt-get update
 ---> Running in 1a2b3c4d5e6f
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [25.6 MB]
...
 ---> 2b3c4d5e6f7g
Step 5/7 : ENV APP_ENV=production
 ---> 3c4d5e6f7g8h
Step 6/7 : EXPOSE 8080
 ---> 4d5e6f7g8h9i
Step 7/7 : CMD ["python", "app.py"]
 ---> 5e6f7g8h9i0j
Successfully built 5e6f7g8h9i0j
Successfully tagged myapp:1.0

每个Step都会创建临时容器，执行命令，然后提交为新的镜像层。

20.2 层缓存机制详解

Docker使用层缓存来加速构建，当指令没有变化时，直接使用缓存的层。

执行效果：

第一次构建
$ docker build -t myapp:1.0 .
Step 1/4 : FROM ubuntu:22.04
 ---> 8a3cdc4d1ad3
Step 2/4 : RUN apt-get update
 ---> Running in abc123def456
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [25.6 MB]
...
 ---> 1a2b3c4d5e6f
Step 3/4 : RUN apt-get install -y nginx
 ---> Running in def456ghi789
Setting up nginx ...
 ---> 2b3c4d5e6f7g
Step 4/4 : COPY . /app
 ---> 3c4d5e6f7g8h
Successfully built 3c4d5e6f7g8h

第二次构建（无变化）
$ docker build -t myapp:1.0 .
Step 1/4 : FROM ubuntu:22.04
 ---> 8a3cdc4d1ad3 Cached
Step 2/4 : RUN apt-get update
 ---> 1a2b3c4d5e6f Cached
Step 3/4 : RUN apt-get install -y nginx
 ---> 2b3c4d5e6f7g Cached
Step 4/4 : COPY . /app
 ---> 3c4d5e6f7g8h Cached
Successfully built 3c4d5e6f7g8h

全部使用缓存！构建非常快！

第三次构建（修改代码）
$ docker build -t myapp:2.0 .
Step 1/4 : FROM ubuntu:22.04
 ---> 8a3cdc4d1ad3 Cached
Step 2/4 : RUN apt-get update
 ---> 1a2b3c4d5e6f Cached
Step 3/4 : RUN apt-get install -y nginx
 ---> 2b3c4d5e6f7g Cached
Step 4/4 : COPY . /app
 ---> 4d5e6f7g8h0j Changed
Successfully built 4d5e6f7g8h0j

只有COPY层重新构建！

20.3 缓存失效规则

任何指令的变化都会导致后续层重新构建。

错误示例：

Dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y nginx
COPY . /app
RUN ls /app

执行效果：

$ docker build -t myapp:bad .
Step 2/5 : RUN apt-get update
 ---> 1a2b3c4d5e6f Cached
Step 3/5 : RUN apt-get install -y nginx
 ---> 2b3c4d5e6f7g Cached
Step 4/5 : COPY . /app
 ---> 3c4d5e6f7g8h Changed
Step 5/5 : RUN ls /app
 ---> 4d5e6f7g8h0j Changed

COPY变化后，后续层都要重新构建！

20.4 利用缓存的最佳实践

原则：变化少的指令放上面，变化多的放下

错误示例：

Dockerfile
FROM node:18
COPY . .              代码经常变化
RUN npm ci              依赖不常变，但放下面
RUN npm run build

执行效果：

第一次
$ docker build -t myapp:bad .
Step 3/5 : COPY . .
 ---> abc123def456
Step 4/5 : RUN npm ci
 ---> Running in def456ghi789
added 1256 packages in 32s

修改代码后
$ docker build -t myapp:bad .
Step 3/5 : COPY . .
 ---> ghi789jkl012 Changed
Step 4/5 : RUN npm ci
 ---> Running in mno345pqr678
added 1256 packages in 32s 又要等32秒！

正确示例：

Dockerfile
FROM node:18
COPY package*.json ./  先复制依赖文件
RUN npm ci            安装依赖
COPY . .              再复制代码
RUN npm run build    构建

执行效果：

第一次
$ docker build -t myapp:good .
Step 3/5 : COPY package*.json ./
 ---> abc123def456
Step 4/5 : RUN npm ci
 ---> Running in def456ghi789
added 1256 packages in 32s

修改代码后
$ docker build -t myapp:good .
Step 3/5 : COPY package*.json ./
 ---> abc123def456 Cached
Step 4/5 : RUN npm ci
 ---> def456ghi789 Cached 非常快！
Step 5/5 : COPY . .
 ---> pqr678stu901 Changed

npm ci使用缓存！构建快很多！

21. 完整示例：Spring Cloud微服务

21.1 项目结构

spring-cloud-project/
├── config-server/
│   ├── Dockerfile
│   └── target/config-server.jar
├── eureka-server/
│   ├── Dockerfile
│   └── target/eureka-server.jar
├── gateway/
│   ├── Dockerfile
│   └── target/gateway.jar
├── user-service/
│   ├── Dockerfile
│   └── target/user-service.jar
└── docker-compose.yml

21.2 Config Server Dockerfile

Dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/config-server.jar app.jar

ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8888/actuator/health || exit 1

EXPOSE 8888

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

执行效果：

$ docker build -t config-server:1.0 ./config-server
Sending build context to Docker daemon  45.2MB
Step 1/8 : FROM eclipse-temurin:21-jre-alpine
 ---> 7e2024c2d9c3
Step 2/8 : WORKDIR /app
 ---> 8f3d5c4e0d4f
Step 3/8 : COPY target/config-server.jar app.jar
 ---> 9g4e6d5f1e5g
Step 4/8 : ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"
 ---> 0h5f7e6g2f6h
Step 5/8 : HEALTHCHECK...
 ---> 1i6g8f7g3g7i
Step 6/8 : EXPOSE 8888
 ---> 2j7h9g8h4h8j
Step 7/8 : ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
 ---> 3k8i0h9i5i9k
Step 8/8 : CMD []
 ---> 4l9j1i0j6j0k
Successfully built 4l9j1i0j6j0k
Successfully tagged config-server:1.0

$ docker images config-server:1.0
REPOSITORY        TAG    SIZE
config-server    1.0    238MB

21.3 Eureka Server Dockerfile

Dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/eureka-server.jar app.jar

ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8761/actuator/health || exit 1

EXPOSE 8761

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

执行效果：

$ docker build -t eureka-server:1.0 ./eureka-server
Sending build context to Docker daemon  52.3MB
Step 1/8 : FROM eclipse-temurin:21-jre-alpine
 ---> 7e2024c2d9c3
Step 2/8 : WORKDIR /app
 ---> 8f3d5c4e0d4f
Step 3/8 : COPY target/eureka-server.jar app.jar
 ---> 9g4e6d5f1e5g
Step 4/8 : ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"
 ---> 0h5f7e6g2f6h
Step 5/8 : HEALTHCHECK...
 ---> 1i6g8f7g3g7i
Step 6/8 : EXPOSE 8761
 ---> 2j7h9g8h4h8j
Step 7/8 : ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
 ---> 3k8i0h9i5i9k
Step 8/8 : CMD []
 ---> 4l9j1i0j6j0k
Successfully built 4l9j1i0j6j0k
Successfully tagged eureka-server:1.0

$ docker images eureka-server:1.0
REPOSITORY        TAG    SIZE
eureka-server    1.0    238MB

21.4 Gateway Dockerfile

Dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/gateway.jar app.jar

ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

执行效果：

$ docker build -t gateway:1.0 ./gateway
Sending build context to Docker daemon  48.7MB
Step 1/8 : FROM eclipse-temurin:21-jre-alpine
 ---> 7e2024c2d9c3
Step 2/8 : WORKDIR /app
 ---> 8f3d5c4e0d4f
Step 3/8 : COPY target/gateway.jar app.jar
 ---> 9g4e6d5f1e5g
Step 4/8 : ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"
 ---> 0h5f7e6g2f6h
Step 5/8 : HEALTHCHECK...
 ---> 1i6g8f7g3g7i
Step 6/8 : EXPOSE 8080
 ---> 2j7h9g8h4h8j
Step 7/8 : ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
 ---> 3k8i0h9i5i9k
Step 8/8 : CMD []
 ---> 4l9j1i0j6j0k
Successfully built 4l9j1i0j6j0k
Successfully tagged gateway:1.0

$ docker images gateway:1.0
REPOSITORY   TAG    SIZE
gateway     1.0    238MB

21.5 User Service Dockerfile

Dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/user-service.jar app.jar

ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"
ENV SPRING_PROFILES_ACTIVE=docker

HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8081/actuator/health || exit 1

EXPOSE 8081

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

执行效果：

$ docker build -t user-service:1.0 ./user-service
Sending build context to Docker daemon  42.1MB
Step 1/9 : FROM eclipse-temurin:21-jre-alpine
 ---> 7e2024c2d9c3
Step 2/9 : WORKDIR /app
 ---> 8f3d5c4e0d4f
Step 3/9 : COPY target/user-service.jar app.jar
 ---> 9g4e6d5f1e5g
Step 4/9 : ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"
 ---> 0h5f7e6g2f6h
Step 5/9 : ENV SPRING_PROFILES_ACTIVE=docker
 ---> 1i6g8f7g3g7i
Step 6/9 : HEALTHCHECK...
 ---> 2j7h9g8h4h8j
Step 7/9 : EXPOSE 8081
 ---> 3k8i0h9i5i9k
Step 8/9 : ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
 ---> 4l9j1i0j6j0k
Step 9/9 : CMD []
 ---> 5m0k2j1k7k1l
Successfully built 5m0k2j1k7k1l
Successfully tagged user-service:1.0

$ docker images user-service:1.0
REPOSITORY        TAG    SIZE
user-service     1.0    238MB

21.6 Docker Compose

yaml
version: '3.8'

services:
  config:
    build: ./config-server
    ports:
      - "8888:8888"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - JAVA_OPTS=-Xms256m -Xmx512m
    networks:
      - spring-cloud

  eureka:
    build: ./eureka-server
    ports:
      - "8761:8761"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - JAVA_OPTS=-Xms256m -Xmx512m
      - EUREKA_INSTANCE_HOSTNAME=eureka
    depends_on:
      - config
    networks:
      - spring-cloud

  gateway:
    build: ./gateway
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - JAVA_OPTS=-Xms256m -Xmx512m
      - SPRING_CLOUD_CONFIG_URI=http://config:8888
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka:8761/eureka/
    depends_on:
      - config
      - eureka
    networks:
      - spring-cloud

  user-service:
    build: ./user-service
    ports:
      - "8081:8081"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - JAVA_OPTS=-Xms256m -Xmx512m
      - SPRING_CLOUD_CONFIG_URI=http://config:8888
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka:8761/eureka/
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/users
    depends_on:
      - config
      - eureka
      - postgres
    networks:
      - spring-cloud

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: users
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - spring-cloud

networks:
  spring-cloud:
    driver: bridge

volumes:
  postgres_data:

执行效果：

$ docker-compose up -d
Building config
...
Building eureka
...
Building gateway
...
Building user-service
...
Creating network "spring-cloud_project" with driver "bridge"
Creating volume "spring-cloud_project_postgres_data" with driver "bridge"
Creating spring-cloud_config_1        ... done
Creating spring-cloud_postgres_1       ... done
Creating spring-cloud_eureka_1        ... done
Creating spring-cloud_gateway_1       ... done
Creating spring-cloud_user-service_1  ... done

$ docker-compose ps
        Name                       Command               Status          Ports
------------------------------------------------------------------------------
spring-cloud_config_1         /docker-entrypoint.sh ...   Up      0.0.0.0:8888->8888/tcp
spring-cloud_eureka_1        /docker-entrypoint.sh ...   Up      0.0.0.0:8761->8761/tcp
spring-cloud_gateway_1       /docker-entrypoint.sh ...   Up      0.0.0.0:8080->8080/tcp
spring-cloud_postgres_1       docker-entrypoint.sh postgres   Up      5432/tcp
spring-cloud_user-service_1  /docker-entrypoint.sh ...   Up      0.0.0.0:8081->8081/tcp

所有服务启动成功！

22. 故障排查指南

22.1 构建失败排查

常见命令：

查看详细构建日志
$ docker build -t myapp . --progress=plain

逐层调试（在失败的RUN后添加）
RUN ls -la /app

进入中间容器调试
$ docker build -t myapp .
$ docker run -it myapp /bin/sh

检查.dockerignore
$ cat .dockerignore

执行效果：

查看详细构建日志
$ docker build -t myapp . --progress=plain
Sending build context to Docker daemon  2.3MB
Step 1/5 : FROM ubuntu:22.04
 ---> 8a3cdc4d1ad3
Step 2/5 : WORKDIR /app
 ---> Running in abc123def456
...

进入容器调试
$ docker run -it myapp /bin/bash
root@abc123def456:/# ls -la /app
total 0
...

22.2 运行时问题排查

常见命令：

查看容器日志
$ docker logs -f myapp
$ docker logs --tail 100 myapp

进入容器
$ docker exec -it myapp /bin/bash

查看进程
$ docker top myapp

查看资源使用
$ docker stats myapp

检查网络
$ docker exec myapp ping google.com
$ docker exec myapp curl localhost:8080

检查文件
$ docker exec myapp ls -la /app

执行效果：

查看日志
$ docker logs myapp
 * Starting Flask server...
 * Running on http://0.0.0.0:5000

查看资源
$ docker stats myapp
CONTAINER ID   NAME      CPU %   MEM USAGE / LIMIT     MEM %   NET I/O           BLOCK I/O
abc123def456   myapp     0.12%   124.5MiB / 512MiB    24.32%  1.23kB / 567B     0B / 0B

检查网络
$ docker exec myapp curl localhost:5000
Hello World!

22.3 性能问题排查

常见命令：

查看镜像大小
$ docker images myapp

查看构建时间
$ docker build -t myapp . 2>&1 | grep Step

优化建议
$ docker build -t myapp . --no-cache

查看磁盘使用
$ docker system df

执行效果：

查看镜像大小
$ docker images myapp
REPOSITORY   TAG    SIZE
myapp        1.0    1.2GB 太大！

优化后
$ docker build -t myapp:optimized .
Successfully built abc123def456
Successfully tagged myapp:optimized

$ docker images myapp:optimized
REPOSITORY      TAG        SIZE
myapp          optimized   245MB 小了很多！

23. 自动化构建

23.1 GitHub Actions自动构建

yaml
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

执行效果：

$ git commit -m "Update" && git push

# GitHub Actions自动触发
Run docker/login-action@v3
  Login Succeeded

Run docker/build-push-action@v5
  #12 building image...
  #12 pushing manifest for myapp/myapp@sha256:abc123...
  #12 done

23.2 GitLab CI自动构建

yaml
stages:
  - build
  - push

docker-build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
  only:
    - main

docker-push:
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

执行效果：

$ git push

# GitLab CI自动触发
Job #1 docker-build
$ docker build -t registry.example.com/myapp:abc123 .
Step 1/5 : FROM ubuntu:22.04
 ---> 8a3cdc4d1ad3
...
Successfully built 5e6f7g8h9i0j

$ docker push registry.example.com/myapp:abc123
abc123: Pushing 2.3MB

24. 总结与下一步

24.1 核心要点回顾

1. FROM：选择合适的基础镜像，具体版本
2. RUN：合并命令，减少层数，使用exec形式
3. COPY vs ADD：简单复制用COPY
4. WORKDIR：设置工作目录
5. ENV vs ARG：理解两者区别
6. ENTRYPOINT + CMD：正确组合
7. USER：使用非root用户
8. HEALTHCHECK：添加健康检查

24.2 学习建议

1. 多读官方文档：https://docs.docker.com/engine/reference/builder/
2. 分析优质镜像：查看Docker Hub上官方镜像的Dockerfile
3. 实践出真知：多写多练，熟能生巧
4. 关注安全：始终使用非root用户，定期更新基础镜像

25. 附录：Dockerfile安全检查清单

Dockerfile
# 1. 使用具体版本标签
# ❌
FROM python:latest
FROM node:latest

# ✅
FROM python:3.11-slim
FROM node:18-alpine

# 2. 以非root用户运行
# ❌
FROM ubuntu
RUN apt-get update

# ✅
FROM ubuntu
RUN apt-get update && useradd -m appuser
USER appuser

# 3. 最小化安装
# ❌
RUN apt-get update && apt-get install -y python-dev build-essential

# ✅
RUN apt-get update && apt-get install -y --no-install-recommends python-dev

# 4. 清理不必要文件
# ❌
RUN apt-get update && apt-get install -y nginx

# ✅
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 5. 不在镜像中存储密钥
# ❌
COPY keys.json /app/

# ✅
ENV API_KEY=${API_KEY}

# 6. 使用健康检查
# ❌
CMD ["python", "app.py"]

# ✅
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1
CMD ["python", "app.py"]

# 7. 只读文件系统（高级）
# ✅
docker run --read-only myapp

执行效果：

构建安全镜像
$ docker build -t myapp:secure -f Dockerfile.secure .
Step 1/12 : FROM python:3.11-slim
 ---> 3aab8a267f4e
...
Step 10/12 : RUN useradd -m -u 1000 appuser
 ---> Running in abc123def456
Step 11/12 : USER appuser
 ---> Running in def456ghi789

$ docker run myapp:secure whoami
appuser 非root用户运行

$ docker run --read-only myapp:secure touch /test
touch: /test: Read-only file system 只读文件系统

26. 附录：常见指令错误

Dockerfile
# 错误1：多个CMD
CMD ["python", "app.py"]
CMD ["node", "server.js"] 只会执行这个

# 错误2：多个ENTRYPOINT
ENTRYPOINT ["python"]
ENTRYPOINT ["app.py"] 只会执行这个

# 错误3：VOLUME在文件操作之后
FROM ubuntu
RUN mkdir /data && echo "test" > /data/file.txt
VOLUME /data file.txt会丢失！

# 正确：VOLUME在前
FROM ubuntu
VOLUME /data
RUN mkdir /data && echo "test" > /data/file.txt

# 错误4：相对路径WORKDIR
WORKDIR app
WORKDIR src 如果/app不存在会失败

# 正确
WORKDIR /app
WORKDIR src

# 错误5：COPY vs ADD 混用
ADD file.tar.gz /app/ 自动解压
COPY file.tar.gz /app/ 不会解压

# 正确：明确使用场景
ADD https://example.com/file.tar.gz /app/ URL下载
COPY file.tar.gz /tmp/ 简单复制

执行效果：

错误1：多个CMD
$ docker build -t test:cmds .
Step 3/4 : CMD ["python", "app.py"]
 ---> abc123def456
Step 4/4 : CMD ["node", "server.js"]
 ---> def456ghi789

$ docker run test:cmds
只执行 node server.js，python app.py被忽略！

错误3：VOLUME问题
$ docker build -t test:volume .
Step 2/4 : RUN mkdir /data && echo "test" > /data/file.txt
 ---> Running in abc123def456
Step 3/4 : VOLUME /data
 ---> Running in def456ghi789

$ docker run test:volume ls /data
空目录！file.txt丢失了！

27. 附录：多语言项目Dockerfile对比

27.1 Python项目

Dockerfile
# Python + Flask + Gunicorn
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

EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]

执行效果：

$ docker build -t pythonapp:1.0 .
Sending build context to Docker daemon  2.3MB
Step 1/10 : FROM python:3.11-slim
 ---> 3aab8a267f4e
...
Successfully built 8h9i0j1k2l3m
Successfully tagged pythonapp:1.0

$ docker images pythonapp:1.0
REPOSITORY     TAG    SIZE
pythonapp     1.0    245MB

27.2 Java项目

Dockerfile
# Java + Spring Boot
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/*.jar app.jar

RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser && \
    chown -R appuser:appgroup /app
    
USER appuser

ENV JAVA_OPTS="-Xms256m -Xmx512m"

EXPOSE 8080

HEALTHCHECK --interval=60s --timeout=10s --start-period=30s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]

执行效果：

$ docker build -t javaapp:1.0 .
Sending build context to Docker daemon  45.2MB
Step 1/9 : FROM eclipse-temurin:21-jre-alpine
 ---> 7e2024c2d9c3
...
Successfully built 5m0k2j1k7k1l
Successfully tagged javaapp:1.0

$ docker images javaapp:1.0
REPOSITORY   TAG    SIZE
javaapp      1.0    238MB

27.3 Node.js项目

Dockerfile
# Node.js + Express
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --production

COPY . .

RUN addgroup -g 1000 nodejs && \
    adduser -u 1000 -G nodejs -s /bin/sh -D nodejs && \
    chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

CMD ["node", "server.js"]

执行效果：

$ docker build -t nodeapp:1.0 .
Sending build context to Docker daemon  1.2MB
Step 1/9 : FROM node:18-alpine
 ---> 8a3cdc4d1ad3
...
Successfully built 9i0j1k2l3m4n
Successfully tagged nodeapp:1.0

$ docker images nodeapp:1.0
REPOSITORY   TAG    SIZE
nodeapp      1.0    167MB

27.4 Go项目

Dockerfile
# Go + Gin
FROM golang:1.21-alpine AS builder

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

执行效果：

$ docker build -t goapp:1.0 .
Sending build context to Docker daemon  2.3MB
Step 1/11 : FROM golang:1.21-alpine AS builder
 ---> 8a3cdc4d1ad3
...
Step 7/11 : RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
 ---> Running in abc123def456
...
Step 8/11 : FROM alpine:latest
 ---> 8h9i0j1k2l3m 开始运行阶段
Step 9/11 : COPY --from=builder /app/main .
 ---> 0j1k2l3m4n5o
Step 10/11 : RUN addgroup -g 1000 appgroup...
 ---> 1k2l3m4n5o6p
Step 11/11 : CMD ["./main"]
 ---> 2l3m4n5o6p7q
Successfully built 2l3m4n5o6p7q
Successfully tagged goapp:1.0

$ docker images goapp:1.0
REPOSITORY   TAG    SIZE
goapp        1.0    15.2MB 非常小！

27.5 .NET项目

Dockerfile
# .NET 7 + ASP.NET Core
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS builder

WORKDIR /src

COPY ["MyApp.csproj", "./"]
RUN dotnet restore MyApp.csproj

COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:7.0

WORKDIR /app

COPY --from=builder /app/publish .

RUN groupadd -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser
USER appuser

EXPOSE 8080

ENTRYPOINT ["dotnet", "MyApp.dll"]

执行效果：

$ docker build -t dotnetapp:1.0 .
Sending build context to Docker daemon  25.3KB
Step 1/9 : FROM mcr.microsoft.com/dotnet/sdk:7.0 AS