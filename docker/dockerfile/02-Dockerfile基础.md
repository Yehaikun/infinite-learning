# Dockerfile 基础

**Metadata**
- Date: 2026-03-23
- Topic: Dockerfile基础与语法
- Tags: #Dockerfile #镜像构建 #学习路径02

---

## 1. 什么是Dockerfile

### 1.1 概念解释

Dockerfile是一个文本文件，用于定义如何构建Docker镜像。它包含了一系列指令（Instruction），每条指令描述镜像的一层。

**一句话解释**：Dockerfile就是"食谱"，告诉Docker如何从零开始制作一个包含你的应用的"菜肴"（镜像）。

### 1.2 为什么需要Dockerfile

#### 手动创建镜像的问题

之前我们学习了如何从Docker Hub拉取镜像运行容器，但实际项目中需要构建自己的镜像：

```
手动方式（不可取）：
1. 运行基础镜像容器
2. 进入容器
3. 手动安装依赖、配置
4. 退出容器
5. docker commit 提交为镜像

问题：
- 不可重复
- 难以版本控制
- 无法自动化
- 过程不透明
```

```
使用Dockerfile（推荐）：
1. 编写Dockerfile
2. docker build 构建镜像
3. docker run 运行容器

优点：
- 可重复构建
- 版本控制
- 自动化
- 过程透明
```

---

## 2. Dockerfile基本结构

### 2.1 一个完整的Dockerfile示例

```dockerfile
# 1. 指定基础镜像
FROM python:3.11-slim

# 2. 设置维护者信息（可选，已废弃）
MAINTAINER Your Name <your@email.com>

# 3. 设置工作目录
WORKDIR /app

# 4. 复制依赖文件
COPY requirements.txt .

# 5. 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 6. 复制应用代码
COPY . .

# 7. 暴露端口
EXPOSE 5000

# 8. 设置环境变量
ENV FLASK_APP=app.py

# 9. 定义启动命令
CMD ["python", "app.py"]
```

### 2.2 Dockerfile执行顺序

Dockerfile中的指令按顺序执行，每条指令都会创建一个新的镜像层：

```
┌─────────────────────────────────────────────────────────────┐
│                   Dockerfile 执行流程                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   FROM python:3.11-slim        ──→ 基础镜像层               │
│          ↓                                                   │
│   WORKDIR /app                 ──→ 创建新层                  │
│          ↓                                                   │
│   COPY requirements.txt .     ──→ 创建新层                  │
│          ↓                                                   │
│   RUN pip install ...          ──→ 创建新层                  │
│          ↓                                                   │
│   COPY . .                    ──→ 创建新层                  │
│          ↓                                                   │
│   EXPOSE 5000                 ──→ 创建新层（仅元数据）       │
│          ↓                                                   │
│   CMD ["python", "app.py"]    ──→ 创建新层（仅元数据）       │
│          ↓                                                   │
│      ┌────────────┐                                        │
│      │  最终镜像   │                                        │
│      └────────────┘                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 核心指令详解

### 3.1 FROM指令

**作用**：指定基础镜像，是Dockerfile的第一条指令

**语法**：
```dockerfile
FROM <image>[:<tag>]
FROM <image>[@<digest>]
```

**示例**：
```dockerfile
# 使用官方Python镜像
FROM python:3.11-slim

# 使用特定版本
FROM node:18-alpine

# 使用AMD64架构
FROM --platform=linux/amd64 python:3.11

# 使用Digest（镜像的SHA256哈希）
FROM python@sha256:abc123...
```

**重要原则**：
1. 尽量使用官方镜像
2. 指定具体版本，不使用latest
3. 选择合适的基础镜像（alpine、slim等）

**常用基础镜像对比**：

| 镜像 | 大小 | 适用场景 |
|------|------|----------|
| ubuntu:latest | ~77MB | 通用，需要完整功能 |
| python:3.11 | ~1GB | Python应用 |
| python:3.11-slim | ~130MB | Python应用，生产推荐 |
| python:3.11-alpine | ~50MB | Python应用，轻量级 |
| node:18 | ~900MB | Node.js应用 |
| node:18-alpine | ~170MB | Node.js应用，轻量级 |
| openjdk:21 | ~450MB | Java应用 |
| openjdk:21-slim | ~230MB | Java应用 |
| openjdk:21-jdk | ~600MB | Java开发 |

### 3.2 RUN指令

**作用**：在镜像构建过程中执行命令

**语法**：
```dockerfile
# shell形式（在shell中执行）
RUN <command>

# exec形式（JSON数组，不调用shell）
RUN ["executable", "param1", "param2"]
```

**示例**：
```dockerfile
# shell形式
RUN apt-get update && apt-get install -y nginx

# exec形式（推荐）
RUN ["apt-get", "update"]
RUN ["apt-get", "install", "-y", "nginx"]

# 多个命令（&&连接）
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**最佳实践**：
```dockerfile
# ❌ 错误：分开执行会产生多层
RUN apt-get update
RUN apt-get install -y nginx

# ✅ 正确：合并执行，减少层数
RUN apt-get update && apt-get install -y nginx
```

### 3.3 COPY指令

**作用**：复制文件或目录到镜像中

**语法**：
```dockerfile
COPY <src>... <dest>
COPY ["<src>",... "<dest>"]  # 路径有空格时使用
```

**示例**：
```dockerfile
# 复制单个文件
COPY requirements.txt /app/

# 复制整个目录
COPY . /app/

# 复制并重命名
COPY config.yaml /app/config.yml

# 多个源文件
COPY package.json package-lock.json /app/
```

### 3.4 ADD指令

**作用**：类似于COPY，但功能更强大（能解压tar、下载URL）

**语法**：
```dockerfile
ADD <src>... <dest>
ADD ["<src>",... "<dest>"]
```

**示例**：
```dockerfile
# 解压tar包（自动解压）
ADD project.tar.gz /app/

# 从URL下载
ADD https://example.com/file.tar.gz /app/

# 复制本地tar并解压
ADD files.tar.gz /app/
```

**推荐**：优先使用COPY，因为：
- COPY语义更明确
- ADD的自动解压可能造成意外
- ADD下载功能可用其他方式替代

### 3.5 WORKDIR指令

**作用**：设置工作目录

**语法**：
```dockerfile
WORKDIR /path/to/workdir
WORKDIR /app  # 相对路径相对于前一个WORKDIR
```

**示例**：
```dockerfile
WORKDIR /app
WORKDIR src  # 实际路径：/app/src
WORKDIR logs # 实际路径：/app/src/logs

# 使用环境变量
WORKDIR $APP_HOME
```

**注意**：WORKDIR会自动创建目录

### 3.6 ENV指令

**作用**：设置环境变量

**语法**：
```dockerfile
ENV <key>=<value> ...
ENV APP_HOME=/app
ENV APP_NAME=myapp APP_VERSION=1.0
```

**示例**：
```dockerfile
# 设置单个变量
ENV NODE_ENV=production

# 设置多个变量
ENV APP_NAME=myapp \
    APP_VERSION=1.0 \
    PORT=8080

# 使用变量
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

### 3.7 EXPOSE指令

**作用**：声明容器监听的端口

**语法**：
```dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

**示例**：
```dockerfile
# 单个端口
EXPOSE 80

# 多个端口
EXPOSE 80 443

# 指定协议
EXPOSE 80/tcp
EXPOSE 53/udp
```

**注意**：
- EXPOSE只是声明，不实际映射端口
- 运行时用-p参数指定端口映射

### 3.8 USER指令

**作用**：设置运行容器的用户

**语法**：
```dockerfile
USER <user>[:<group>]
USER UID[:GID]
```

**示例**：
```dockerfile
# 创建用户并使用
RUN adduser -D -u 1000 appuser
USER appuser

# 直接使用数字UID
USER 1000
```

### 3.9 ARG指令

**作用**：定义构建时变量

**语法**：
```dockerfile
ARG <name>[=<default value>]
```

**示例**：
```dockerfile
# 定义变量
ARG APP_VERSION=1.0
ARG NODE_ENV=production

# 使用变量
RUN echo "Building version $APP_VERSION"

# 构建时传递
# docker build --build-arg APP_VERSION=2.0 .
```

### 3.10 VOLUME指令

**作用**：创建挂载点

**语法**：
```dockerfile
VOLUME ["/path/to/volume"]
VOLUME /app/data /app/logs
```

**示例**：
```dockerfile
# 创建匿名卷
VOLUME /app/data

# 命名卷
VOLUME data

# JSON数组形式
VOLUME ["/app/data", "/app/logs"]
```

### 3.11 ENTRYPOINT指令

**作用**：容器的入口点（启动命令）

**语法**：
```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]  # exec形式
ENTRYPOINT command param1 param2               # shell形式
```

**示例**：
```dockerfile
# exec形式（推荐）
ENTRYPOINT ["python", "app.py"]

# 带默认参数
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]

# shell形式（不推荐，信号处理有问题）
ENTRYPOINT python app.py
```

### 3.12 CMD指令

**作用**：定义容器的默认命令

**语法**：
```dockerfile
CMD ["executable", "param1", "param2"]  # exec形式
CMD ["param1", "param2"]                 # 作为ENTRYPOINT的参数
CMD command param1 param2                # shell形式
```

**示例**：
```dockerfile
# exec形式
CMD ["python", "app.py"]

# 作为ENTRYPOINT的参数
CMD ["--port", "8080"]

# shell形式
CMD python app.py
```

**CMD vs ENTRYPOINT**：
- CMD：可以被docker run参数覆盖
- ENTRYPOINT：docker run参数会追加到CMD之前

### 3.13 LABEL指令

**作用**：添加元数据

**语法**：
```dockerfile
LABEL <key>=<value>
LABEL maintainer="your@email.com"
LABEL version="1.0" description="My application"
```

**示例**：
```dockerfile
LABEL maintainer="your@email.com"
LABEL version="1.0"
LABEL description="A simple web application"
```

---

## 4. 第一个Dockerfile

### 4.1 创建项目结构

```bash
mkdir -p ~/docker/mywebapp
cd ~/docker/mywebapp

# 创建应用文件
cat > app.py << 'EOF'
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return f'Hello from Docker! Container ID: {os.environ.get("HOSTNAME", "unknown")}'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# 创建依赖文件
cat > requirements.txt << 'EOF'
flask==3.0.0
EOF
```

### 4.2 编写Dockerfile

```dockerfile
cat > Dockerfile << 'EOF'
# 1. 指定基础镜像
FROM python:3.11-slim

# 2. 设置工作目录
WORKDIR /app

# 3. 复制依赖文件
COPY requirements.txt .

# 4. 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 5. 复制应用代码
COPY . .

# 6. 暴露端口
EXPOSE 5000

# 7. 启动命令
CMD ["python", "app.py"]
EOF
```

### 4.3 构建镜像

```bash
# 构建镜像
docker build -t mywebapp:1.0 .

# 查看构建过程
docker build -t mywebapp:1.0 . --progress=plain

# 查看构建缓存
docker build -t mywebapp:1.0 . --no-cache  # 不使用缓存
```

构建输出：
```
Sending build context to Docker daemon  4.608kB
Step 1/7 : FROM python:3.11-slim
 ---> 3aab8a267f4e
Step 2/7 : WORKDIR /app
 ---> Running in 9a5b4c8e1234
Removing intermediate container 9a5b4c8e1234
 ---> d7f8a1234567
Step 3/7 : COPY requirements.txt .
 ---> e8f9b2345678
Step 4/7 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in 0a1b2c3d4e5f
Removing intermediate container 0a1b2c3d4e5f
 ---> f1a2b3c4d5e6
Step 5/7 : COPY . .
 ---> a1b2c3d4e5f6
Step 6/7 : EXPOSE 5000
 ---> Running in 1a2b3c4d5e6f
Removing intermediate container 1a2b3c4d5e6f
 ---> b2c3d4e5f6a1
Step 7/7 : CMD ["python", "app.py"]
 ---> Running in 2a3b4c5d6e7f
Removing intermediate container 2a3b4c5d6e7f
 ---> c3d4e5f6a1b2
Successfully built c3d4e5f6a1b2
Successfully tagged mywebapp:1.0
```

### 4.4 运行容器

```bash
# 后台运行
docker run -d -p 5000:5000 --name mywebapp mywebapp:1.0

# 查看日志
docker logs -f mywebapp

# 测试访问
curl http://localhost:5000

# 查看容器
docker ps
```

---

## 5. .dockerignore文件

### 5.1 作用

.dockerignore文件用于排除不需要打包到镜像中的文件，减小构建上下文。

### 5.2 语法

```
# 注释
*.md
!README.md
.git
node_modules
__pycache__
*.pyc
.env
```

### 5.3 示例

```bash
# 创建.dockerignore
cat > .dockerignore << 'EOF'
# Git
.git
.gitignore

# Python
__pycache__
*.pyc
*.pyo
*.pyd
.Python
*.so
*.egg
*.egg-info
dist
build

# Virtual environments
venv/
env/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Test
.pytest_cache
.coverage
htmlcov/

# Documentation
*.md
docs/

# Docker
Dockerfile
docker-compose.yml
.docker/

# Environment
.env
.env.local

# Logs
*.log

# OS
.DS_Store
Thumbs.db
EOF
```

---

## 6. 构建进阶

### 6.1 构建参数

```dockerfile
# Dockerfile
ARG APP_VERSION=1.0
ARG BUILD_DATE

LABEL version=$APP_VERSION
LABEL build-date=$BUILD_DATE

RUN echo "Building version $APP_VERSION"
```

构建：
```bash
docker build --build-arg APP_VERSION=2.0 -t myapp:2.0 .
docker build --build-arg BUILD_DATE=$(date -u +"%Y-%m-%dT%H:%M:%SZ") -t myapp:1.0 .
```

### 6.2 多阶段构建

```dockerfile
# 第一阶段：构建
FROM node:18 AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

# 第二阶段：运行
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./
RUN npm ci --production
CMD ["node", "dist/index.js"]
```

### 6.3 标签和版本

```bash
# 标签
docker build -t myapp:1.0 .
docker build -t myapp:latest .
docker build -t myapp:v1.0.0 .
docker build -t myapp:v1.0.0-beta .

# 多个标签
docker build -t myapp:1.0 -t myapp:latest .
```

---

## 7. 最佳实践

### 7.1 Dockerfile最佳实践

1. **选择合适的基础镜像**
   ```dockerfile
   # ❌ 不要使用latest
   FROM python:latest
   
   # ✅ 使用具体版本
   FROM python:3.11-slim
   ```

2. **减少镜像层数**
   ```dockerfile
   # ❌ 多个RUN指令产生多层
   RUN apt-get update
   RUN apt-get install -y nginx
   RUN apt-get clean
   
   # ✅ 一个RUN指令减少层数
   RUN apt-get update && \
       apt-get install -y nginx && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/*
   ```

3. **利用构建缓存**
   ```dockerfile
   # ❌ 变化频繁的文件放前面
   COPY . .
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   
   # ✅ 变化少的文件放前面
   COPY requirements.txt .
   RUN pip install -r requirements.txt
   COPY . .
   ```

4. **清理不必要的文件**
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y nginx && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/* && \
       rm -rf /tmp/* /var/tmp/*
   ```

5. **使用多阶段构建减小镜像**
   ```dockerfile
   # 大型镜像
   FROM golang:1.21
   WORKDIR /app
   COPY . .
   RUN go build -o main .
   
   # 轻量镜像
   FROM golang:1.21 AS builder
   WORKDIR /app
   COPY . .
   RUN go build -o main .
   
   FROM alpine:latest
   COPY --from=builder /app/main /app/main
   CMD ["/app/main"]
   ```

6. **设置正确的WORKDIR**
   ```dockerfile
   # ❌ 没有设置WORKDIR
   COPY . /
   CMD ["python", "app.py"]
   
   # ✅ 设置WORKDIR
   WORKDIR /app
   COPY . .
   CMD ["python", "app.py"]
   ```

7. **使用exec形式运行命令**
   ```dockerfile
   # ❌ shell形式
   CMD python app.py
   
   # ✅ exec形式
   CMD ["python", "app.py"]
   ```

8. **处理信号**
   ```dockerfile
   # ❌ 不正确处理SIGTERM
   CMD python app.py
   
   # ✅ 使用tini或正确处理信号
   CMD ["tini", "--", "python", "app.py"]
   ```

### 7.2 安全最佳实践

1. **不以root用户运行**
   ```dockerfile
   # 创建用户
   RUN addgroup -g 1000 appgroup && \
       adduser -u 1000 -G appgroup -s /bin/sh -D appuser
   
   USER appuser
   ```

2. **使用最小权限**
   ```dockerfile
   # ❌ 给全部权限
   RUN chmod -R 777 /app
   
   # ✅ 给最小权限
   RUN chmod -R 755 /app
   ```

3. **不要在镜像中存储密钥**
   ```dockerfile
   # ❌ 复制密钥文件
   COPY keys.json /app/
   
   # ✅ 使用运行时环境变量或密钥管理服务
   ENV API_KEY=${API_KEY}
   ```

---

## 8. 常见问题

### 8.1 构建失败

```bash
# 常见错误1：上下文太大
# 错误：Sending build context to Docker daemon 1.2GB
# 解决：添加.dockerignore，排除大文件

# 常见错误2：缓存不足
# 错误：No space left on device
# 解决：清理Docker构建缓存
docker builder prune

# 常见错误3：网络问题
# 错误：Get "https://registry-1.docker.io/v2/": dial tcp
# 解决：配置国内镜像加速
```

### 8.2 权限问题

```bash
# 错误：permission denied
# 解决：创建用户并切换
RUN adduser -D -u 1000 appuser
USER appuser
```

### 8.3 中文乱码

```dockerfile
# 解决：设置locale
ENV LANG=C.UTF-8 LC_ALL=C.UTF-8
```

---

## 9. 完整示例

### 9.1 Spring Boot应用Dockerfile

```dockerfile
# 基础镜像
FROM eclipse-temurin:21-jdk-alpine AS builder

# 设置工作目录
WORKDIR /app

# 复制pom文件
COPY pom.xml .

# 下载依赖（利用缓存）
RUN apk add --no-cache maven
RUN mvn dependency:go-offline -B

# 复制源码
COPY src ./src

# 构建
RUN mvn package -DskipTests -B

# 运行镜像
FROM eclipse-temurin:21-jre-alpine

# 设置工作目录
WORKDIR /app

# 从builder复制jar
COPY --from=builder /app/target/*.jar app.jar

# 暴露端口
EXPOSE 8080

# 环境变量
ENV JAVA_OPTS="-Xms256m -Xmx512m"

# 启动命令
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### 9.2 Node.js应用Dockerfile

```dockerfile
# 构建阶段
FROM node:18-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package.json package-lock.json ./

# 安装依赖
RUN npm ci --only=production

# 复制源码
COPY . .

# 构建
RUN npm run build

# 运行阶段
FROM node:18-alpine

WORKDIR /app

# 从builder复制产物
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

# 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# 暴露端口
EXPOSE 3000

# 启动
CMD ["node", "dist/main.js"]
```

### 9.3 Python应用Dockerfile

```dockerfile
FROM python:3.11-slim

# 设置工作目录
WORKDIR /app

# 安装依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装Python依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 创建非root用户
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# 启动命令
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:app"]
```

---

## 10. 本章总结

### 核心指令

| 指令 | 说明 |
|------|------|
| FROM | 指定基础镜像 |
| RUN | 执行命令 |
| COPY | 复制文件 |
| ADD | 复制文件（可解压/下载） |
| WORKDIR | 设置工作目录 |
| ENV | 设置环境变量 |
| EXPOSE | 声明端口 |
| USER | 设置用户 |
| ARG | 构建参数 |
| VOLUME | 创建挂载点 |
| ENTRYPOINT | 入口点 |
| CMD | 默认命令 |
| LABEL | 元数据 |

### Dockerfile编写原则

1. 优先使用官方镜像
2. 指定具体版本号
3. 减少镜像层数
4. 利用构建缓存
5. 清理不必要文件
6. 不以root用户运行
7. 使用exec形式
8. 使用.dockerignore

---

## 11. 课后问题

1. Dockerfile和镜像的区别是什么？
   - Dockerfile是构建脚本，镜像是构建结果

2. RUN、CMD、ENTRYPOINT的区别是什么？
   - RUN：构建时执行
   - CMD：运行时默认命令，可被覆盖
   - ENTRYPOINT：运行时入口点

3. 如何减小镜像大小？
   - 使用alpine镜像
   - 多阶段构建
   - 清理不必要的文件

---

## 12. Dockerfile深度解析

### 12.1 RUN指令的两种形式

RUN指令有两种执行形式，理解它们的区别很重要：

#### Shell形式

```dockerfile
RUN command param1 param2
```

这种方式会在shell中执行（默认是 `/bin/sh -c`）：

```dockerfile
RUN echo "Hello World" > /tmp/test.txt
RUN pip install flask django
```

**执行过程**：
1. Docker启动一个临时容器
2. 在容器中执行命令
3. 提交容器为新镜像层
4. 删除临时容器

#### Exec形式

```dockerfile
RUN ["executable", "param1", "param2"]
```

这种方式直接执行命令，不调用shell：

```dockerfile
RUN ["echo", "Hello World"]
RUN ["pip", "install", "flask", "django"]
```

**为什么重要**：exec形式不会进行shell处理，比如变量替换：

```dockerfile
# ❌ 变量不会生效
RUN ["echo", "$HOME"]

# ✅ shell形式才能用变量
RUN echo $HOME
```

**正确使用方式**：

```dockerfile
# 安装软件包（使用shell形式处理变量）
RUN apt-get update && \
    apt-get install -y nginx && \
    echo "export PATH=$PATH:/opt/nginx" >> /etc/profile

# 执行脚本（使用exec形式）
RUN ["/bin/bash", "-c", "echo hello"]
```

### 12.2 CMD与ENTRYPOINT组合

这两个指令经常一起使用，理解它们的交互很重要：

#### 三种组合方式

**方式1：只有CMD**

```dockerfile
CMD ["python", "app.py"]
```

运行：`docker run myapp` → 执行 `python app.py`
运行：`docker run myapp arg1` → 执行 `python app.py arg1`（arg1被忽略）

**方式2：只有ENTRYPOINT**

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

运行：`docker run myapp` → 执行 `python app.py`
运行：`docker run myapp arg1` → 执行 `python app.py arg1`（arg1作为参数）

**方式3：ENTRYPOINT + CMD（推荐）**

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]
```

运行：`docker run myapp` → 执行 `python app.py --port 8080`
运行：`docker run myapp --port 9000` → 执行 `python app.py --port 9000`

CMD的参数会追加到ENTRYPOINT后面！

#### 实际应用案例

```dockerfile
# Nginx示例
ENTRYPOINT ["nginx", "-g", "daemon off;"]
CMD ["-c", "/etc/nginx/nginx.conf"]

# Python应用示例
ENTRYPOINT ["python"]
CMD ["-m", "http.server", "8080"]

# 带环境变量的示例
ENV PORT=8080
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "$PORT"]
```

### 12.3 ARG vs ENV

这两个指令很容易混淆，让我详细解释：

#### ARG（构建参数）

```dockerfile
# 定义（默认值可选）
ARG APP_VERSION=1.0
ARG NODE_ENV=production

# 使用（在构建时）
RUN echo "Building $APP_VERSION"
RUN npm run build:$NODE_ENV
```

**特点**：
- 仅在构建时有效
- 可以被 `docker build --build-arg` 覆盖
- 不会保存在镜像中
- 用于构建时的变量传递

**构建时传递**：
```bash
docker build --build-arg APP_VERSION=2.0 -t myapp:2.0 .
```

#### ENV（环境变量）

```dockerfile
# 定义
ENV APP_VERSION=1.0
ENV NODE_ENV=production

# 使用（在构建和运行时都有效）
RUN echo "Running in $NODE_ENV"
CMD ["python", "app.py"]
```

**特点**：
- 构建时和运行时都有效
- 容器启动后仍然存在
- 会保存在镜像中
- 可以被运行时参数覆盖

**运行时覆盖**：
```bash
docker run -e NODE_ENV=development myapp
```

#### 对比总结

| 特性 | ARG | ENV |
|------|-----|-----|
| 作用时机 | 构建时 | 构建时+运行时 |
| 是否存入镜像 | 否 | 是 |
| 运行时覆盖 | 否 | 是 |
| 用途 | 构建参数 | 环境配置 |

### 12.4 镜像层和缓存机制

#### 镜像层原理

Docker镜像是分层存储的，每条指令创建一个新层：

```dockerfile
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y nginx
RUN echo "hello" > /tmp/test.txt
```

```
┌─────────────────────────────────────────┐
│              镜像层结构                    │
├─────────────────────────────────────────┤
│                                         │
│  Layer 4: echo hello > test.txt         │
│  ┌───────────────────────────────────┐  │
│  │ Layer 3: apt-get install nginx    │  │
│  │ ┌───────────────────────────────┐ │  │
│  │ │ Layer 2: apt-get update       │ │  │
│  │ │ ┌───────────────────────────┐ │ │  │
│  │ │ │ Layer 1: FROM ubuntu      │ │ │  │
│  │ │ └───────────────────────────┘ │ │  │
│  │ └───────────────────────────────┘ │  │
│  └───────────────────────────────────┘  │
│                                         │
└─────────────────────────────────────────┘
```

#### 缓存机制

Docker会复用之前的层来加速构建：

```
# 第一次构建
Step 1/4 : FROM ubuntu:22.04
 ---> abc123
Step 2/4 : RUN apt-get update
 ---> 执行并缓存为层 def456
Step 3/4 : RUN apt-get install -y nginx
 ---> 执行并缓存为层 ghi789
Step 4/4 : RUN echo "hello" > /tmp/test.txt
 ---> 执行并缓存为层 jkl012

# 第二次构建（如果文件没变）
Step 1/4 : FROM ubuntu:22.04
 ---> 使用缓存 abc123
Step 2/4 : RUN apt-get update
 ---> 使用缓存 def456
Step 3/4 : RUN apt-get install -y nginx
 ---> 使用缓存 ghi789
Step 4/4 : RUN echo "hello" > /tmp/test.txt
 ---> 使用缓存 jkl012
```

#### 破坏缓存

任何指令的变化都会导致后续层重新构建：

```
# 修改了第三步
Step 1/4 : FROM ubuntu:22.04
 ---> 使用缓存 abc123
Step 2/4 : RUN apt-get update
 ---> 使用缓存 def456
Step 3/4 : RUN apt-get install -y nginx vim  # 变化了！
 ---> 重新执行，不使用缓存 ghi789
Step 4/4 : RUN echo "hello" > /tmp/test.txt
 ---> 重新执行，不使用缓存 jkl012
```

#### 利用缓存的最佳实践

**原则**：变化少的指令放上面，变化多的放下面

```dockerfile
# ❌ 错误：代码经常变化，但依赖不经常变
COPY . .
RUN pip install -r requirements.txt

# ✅ 正确：依赖放上面，利用缓存
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

### 12.5 EXPOSE的真正作用

EXPOSE只是文档作用，告诉你这个容器会监听哪个端口：

```dockerfile
EXPOSE 8080
EXPOSE 443
```

**实际端口映射在运行时指定**：
```bash
# 不管EXPOSE写什么，运行时想映射哪个端口都可以
docker run -p 80:8080 myapp     # 把80映射到容器的8080
docker run -p 9000:8080 myapp    # 把9000映射到容器的8080
```

**但EXPOSE的好处**：
1. 文档作用：告诉使用者应该映射哪个端口
2. docker-compose会自动使用EXPOSE的端口
3. 某些工具会读取EXPOSE来建议端口

---

## 13. 实战练习

### 13.1 构建Java应用镜像

假设项目结构：
```
myjavaapp/
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── example/
│                   └── App.java
├── pom.xml
└── Dockerfile
```

**pom.xml**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>myjavaapp</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <name>My Java App</name>
    <properties>
        <maven.compiler.source>21</maven.compiler.source>
        <maven.compiler.target>21</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.0</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**Dockerfile（基础版）**:
```dockerfile
FROM eclipse-temurin:21-jdk

WORKDIR /app

COPY pom.xml .
COPY src ./src

RUN apt-get update && \
    apt-get install -y maven && \
    mvn clean package -DskipTests && \
    apt-get remove -y maven && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

EXPOSE 8080

CMD ["java", "-jar", "target/myjavaapp-1.0-SNAPSHOT.jar"]
```

**Dockerfile（优化版-多阶段构建）**:
```dockerfile
# ============ 阶段1: 构建 ============
FROM eclipse-temurin:21-jdk AS builder

WORKDIR /app

# 先复制pom，利用Maven缓存
COPY pom.xml .
RUN mvn dependency:go-offline -B

# 再复制源码
COPY src ./src

# 构建
RUN mvn clean package -DskipTests -B

# ============ 阶段2: 运行 ============
FROM eclipse-temurin:21-jre

WORKDIR /app

# 从builder复制jar
COPY --from=builder /app/target/myjavaapp-1.0-SNAPSHOT.jar app.jar

# 非root用户
RUN groupadd -r appgroup && \
    useradd -r -g appgroup appuser
    
USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 13.2 构建Golang应用镜像

**项目结构**:
```
mygoapp/
├── go.mod
├── main.go
└── Dockerfile
```

**main.go**:
```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello from Go!")
    })
    http.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
        w.WriteHeader(http.StatusOK)
        fmt.Fprint(w, "OK")
    })
    fmt.Println("Server starting on :8080")
    http.ListenAndServe(":8080", nil)
}
```

**go.mod**:
```go
module mygoapp

go 1.21
```

**Dockerfile**:
```dockerfile
# 构建阶段
FROM golang:1.21-alpine AS builder

WORKDIR /app

# 安装build dependencies
RUN apk add --no-cache git

# 复制go.mod（先复制，利用缓存）
COPY go.mod go.sum ./
RUN go mod download

# 复制源码
COPY . .

# 构建
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# 运行阶段
FROM alpine:latest

# 安装证书（用于HTTPS）
RUN apk --no-cache add ca-certificates tzdata && \
    apk --no-cache add --virtual=.builds-deps binutils

WORKDIR /app

# 从builder复制二进制
COPY --from=builder /app/main .

# 非root用户
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -s /bin/sh -D appuser
USER appuser

EXPOSE 8080

CMD ["./main"]
```

### 13.3 构建PHP应用镜像

**项目结构**:
```
myphpapp/
├── index.php
├── composer.json
└── Dockerfile
```

**index.php**:
```php
<?php
echo "Hello from PHP!\n";
echo "PHP Version: " . PHP_VERSION . "\n";

// 简单的路由
$uri = $_SERVER['REQUEST_URI'] ?? '/';
if ($uri === '/health') {
    http_response_code(200);
    echo "OK";
    exit;
}
?>
```

**composer.json**:
```json
{
    "name": "example/php-app",
    "description": "Simple PHP App",
    "require": {
        "php": ">=8.0"
    }
}
```

**Dockerfile**:
```dockerfile
FROM php:8.2-apache

# 安装扩展和工具
RUN docker-php-ext-install pdo pdo_mysql

# 启用Apache模块
RUN a2enmod rewrite headers

# 复制源码
COPY . /var/www/html/

# 设置权限
RUN chown -R www-data:www-data /var/www/html

# 暴露端口
EXPOSE 80

# 环境变量
ENV APACHE_DOCUMENT_ROOT=/var/www/html
```

或者使用更安全的非root版本：

```dockerfile
FROM php:8.2-fpm-alpine

# 安装nginx作为反向代理
RUN apk add --no-cache nginx

# 复制nginx配置
COPY nginx.conf /etc/nginx/http.d/default.conf

# 复制PHP配置
COPY php.ini /usr/local/etc/php/conf.d/app.ini

# 复制源码
COPY . /var/www/html

# 设置权限
RUN chown -R nginx:nginx /var/www/html

# 暴露端口
EXPOSE 8080

# 启动脚本
COPY start.sh /start.sh
RUN chmod +x /start.sh

CMD ["/start.sh"]
```

### 13.4 构建Nginx反向代理镜像

**项目结构**:
```
nginx-proxy/
├── Dockerfile
├── nginx.conf
├── conf.d/
│   └── default.conf
└── html/
    └── index.html
```

**nginx.conf**:
```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml text/javascript 
               application/json application/javascript application/xml+rss;
    
    include /etc/nginx/conf.d/*.conf;
}
```

**conf.d/default.conf**:
```nginx
server {
    listen 80;
    server_name localhost;
    
    root /usr/share/nginx/html;
    index index.html;
    
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    location /api {
        proxy_pass http://backend:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
}
```

**Dockerfile**:
```dockerfile
FROM nginx:1.25-alpine

# 删除默认配置
RUN rm /etc/nginx/conf.d/default.conf

# 复制配置
COPY nginx.conf /etc/nginx/nginx.conf
COPY conf.d /etc/nginx/conf.d/

# 复制静态文件
COPY html /usr/share/nginx/html/

# 创建非root用户
RUN addgroup -g 1000 -S nginx && \
    adduser -u 1000 -S nginx -G nginx
    
RUN chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    chown -R nginx:nginx /etc/nginx/conf.d && \
    chown -R nginx:nginx /usr/share/nginx/html

USER nginx

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 14. 进阶技巧

### 14.1 构建时变量注入

```dockerfile
ARG APP_NAME
ARG APP_VERSION
ARG BUILD_TIME

LABEL app.name=$APP_NAME
LABEL app.version=$APP_VERSION
LABEL build.time=$BUILD_TIME

RUN echo "Building $APP_NAME version $APP_VERSION at $BUILD_TIME"
```

```bash
docker build \
  --build-arg APP_NAME=myapp \
  --build-arg APP_VERSION=1.0.0 \
  --build-arg BUILD_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ") \
  -t myapp:1.0.0 .
```

### 14.2 条件构建

Dockerfile本身不支持if，但可以通过ARG实现：

```dockerfile
ARG ENABLE_DEBUG=false

RUN if [ "$ENABLE_DEBUG" = "true" ]; then \
    apt-get update && apt-get install -y strace; \
    fi
```

```bash
# 不安装strace
docker build -t myapp .

# 安装strace
docker build --build-arg ENABLE_DEBUG=true -t myapp:debug .
```

### 14.3 镜像健康检查

```dockerfile
FROM nginx:alpine

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost/ || exit 1

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**参数说明**：
- `--interval`：检查间隔（默认30s）
- `--timeout`：超时时间（默认30s）
- `--start-period`：启动等待时间（默认0s）
- `--retries`：重试次数（默认3）

### 14.4 构建日志分析

```bash
# 查看构建历史
docker history myapp:latest

# 输出：
# IMAGE          CREATED         CREATED BY                      SIZE      COMMENT
# abc123         10 seconds ago  CMD ["nginx" "-g" "daemon off;   0B        buildkit.export...
# def456         10 seconds ago  EXPOSE 80                       0B        buildkit.export...
# ghi789         15 seconds ago  RUN nginx -g ...                 4MB       buildkit.export...
# jkl012         30 seconds ago  COPY html /usr/share/...         12MB      buildkit.export...
# mno345         2 minutes ago   FROM nginx:alpine               25MB      buildkit.export...

# 查看镜像层详情
docker history --no-trunc myapp:latest

# 查看构建日志
docker build -t myapp . --progress=plain 2>&1 | tee build.log
```

---

## 15. 调试Dockerfile

### 15.1 构建失败调试

```bash
# 查看具体哪一步失败
docker build -t myapp . --progress=plain

# 在失败处停止
docker build -t myapp . --progress=plain --debug

# 调试模式
DOCKER_BUILDKIT=1 docker build -t myapp .
```

### 15.2 进入构建中间层

```dockerfile
# 在某一步暂停
FROM ubuntu:22.04
RUN sleep infinity
```

```bash
docker build -t debug .
docker run -it debug
```

### 15.3 查看构建缓存

```bash
# 查看缓存使用情况
docker build -t myapp . --progress=plain 2>&1 | grep -i "cache"

# 不使用缓存
docker build -t myapp . --no-cache

# 只使用特定步骤的缓存
docker build -t myapp . --cache-from=myapp:previous
```

---

## 16. 本章总结

### Dockerfile核心要点

1. **FROM**：选择合适的基础镜像
2. **RUN**：合并命令，减少层数
3. **COPY/ADD**：注意利用缓存
4. **WORKDIR**：设置工作目录
5. **ENV/ARG**：理解区别和用途
6. **EXPOSE**：文档作用
7. **USER**：安全运行
8. **ENTRYPOINT/CMD**：正确组合使用

### 构建优化要点

1. 变化少的指令放上面
2. 利用Maven/Gradle/npm缓存
3. 多阶段构建减小镜像
4. 清理不必要的文件
5. 使用.dockerignore

---

## 17. 课后问题

1. 如何让CMD的参数被ENTRYPOINT使用？
   - 使用JSON数组形式组合

2. ARG和ENV的区别是什么？
   - ARG仅构建时，ENV构建时+运行时

3. 如何减小Java镜像大小？
   - 多阶段构建，只复制jar

4. 为什么要用.dockerignore？
   - 减小构建上下文

---

## 18. 更多实战案例

### 18.1 Python + Gunicorn + Nginx 部署

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./static:/usr/share/nginx/html:ro
    depends_on:
      - web
    networks:
      - frontend

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass redis123
    volumes:
      - redis_data:/data

networks:
  frontend:

volumes:
  postgres_data:
  redis_data:
```

**项目结构**:
```
myproject/
├── Dockerfile
├── docker-compose.yml
├── nginx/
│   └── conf.d/
│       └── default.conf
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── models.py
│   └── schemas.py
├── requirements.txt
└── tests/
```

**Dockerfile**:
```dockerfile
FROM python:3.11-slim

# 安装系统依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# 复制依赖文件
COPY requirements.txt .

# 安装Python依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY app/ ./app/

# 创建日志目录
RUN mkdir -p /app/logs

# 设置权限
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
    
USER appuser

# 暴露端口
EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# 启动命令
CMD ["gunicorn", "app.main:app", "--bind", "0.0.0.0:8000", "--workers", "4", "--timeout", "120"]
```

**requirements.txt**:
```
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
psycopg2-binary==2.9.9
redis==5.0.1
pydantic==2.5.0
pydantic-settings==2.1.0
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
gunicorn==21.2.0
```

### 18.2 Node.js + React + Nginx 全栈部署

**项目结构**:
```
myproject/
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── index.js
│   │   └── routes.js
│   └── .env
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── nginx.conf
│   └── src/
│       ├── App.js
│       └── index.js
└── docker-compose.yml
```

**后端Dockerfile**:
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

# 安装依赖
COPY package.json package-lock.json ./
RUN npm ci

# 复制源码
COPY src ./src

# 构建
RUN npm run build

# 运行镜像
FROM node:18-alpine

WORKDIR /app

# 安装生产依赖
COPY package.json package-lock.json ./
RUN npm ci --production

# 从builder复制构建产物
COPY --from=builder /app/dist ./dist

# 创建用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
    
USER nodejs

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

**前端Dockerfile**:
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .
RUN npm run build

# Nginx镜像
FROM nginx:alpine

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制nginx配置
COPY nginx.conf /etc/nginx/nginx.conf

# 复制自定义配置
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/app
    depends_on:
      - db
    restart: unless-stopped

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres_data:
```

### 18.3 微服务架构Dockerfile

假设项目是一个Spring Cloud微服务：

**项目结构**:
```
microservices/
├── eureka-server/
│   ├── Dockerfile
│   └── target/eureka-server.jar
├── api-gateway/
│   ├── Dockerfile
│   └── target/api-gateway.jar
├── user-service/
│   ├── Dockerfile
│   └── target/user-service.jar
└── docker-compose.yml
```

**Eureka Server Dockerfile**:
```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/eureka-server.jar app.jar

# JVM参数
ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"

# 健康检查
HEALTHCHECK --interval=60s --timeout=10s --start-period=90s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8761/actuator/health || exit 1

EXPOSE 8761

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**API Gateway Dockerfile**:
```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/api-gateway.jar app.jar

ENV JAVA_OPTS="-Xms256m -Xmx512m"

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

**User Service Dockerfile**:
```dockerfile
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

COPY target/user-service.jar app.jar

ENV JAVA_OPTS="-Xms256m -Xmx512m -XX:+UseG1GC"

# 使用wait-for-it等待依赖服务
COPY wait-for-it.sh /wait-for-it.sh
RUN chmod +x /wait-for-it.sh

EXPOSE 8081

ENTRYPOINT ["/wait-for-it.sh", "eureka-server:8761", "--", "java", "$JAVA_OPTS", "-jar", "app.jar"]
```

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  eureka:
    build: ./eureka-server
    ports:
      - "8761:8761"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
    networks:
      - microservices

  gateway:
    build: ./api-gateway
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka:8761/eureka/
    depends_on:
      - eureka
    networks:
      - microservices

  user-service:
    build: ./user-service
    ports:
      - "8081:8081"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka:8761/eureka/
      - DATABASE_URL=jdbc:postgresql://postgres:5432/users
    depends_on:
      - eureka
      - postgres
    networks:
      - microservices

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: users
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - microservices

networks:
  microservices:

volumes:
  postgres_data:
```

---

## 19. Dockerfile调试技巧

### 19.1 构建过程查看

```bash
# 查看详细构建过程
docker build -t myapp . --progress=plain

# 查看构建耗时
docker build -t myapp . --progress=plain 2>&1 | tee build.log

# 使用BuildKit构建
DOCKER_BUILDKIT=1 docker build -t myapp .
```

### 19.2 构建后调试

```bash
# 进入运行中的容器
docker exec -it container_name /bin/sh

# 查看构建历史
docker history myapp:latest

# 查看镜像层
docker inspect myapp:latest

# 比较两个镜像
docker diff container_name
```

### 19.3 构建问题排查

```bash
# 问题：层缓存失效
# 解决：查看哪一步没有使用缓存

# 问题：构建太慢
# 解决：检查是否充分利用缓存

# 问题：镜像太大
# 解决：使用多阶段构建，清理不需要的文件

# 问题：权限错误
# 解决：检查USER指令是否正确设置
```

---

## 20. 总结与下一步

### 核心要点回顾

1. **Dockerfile是构建脚本**：定义如何从零创建镜像
2. **分层构建**：每条指令创建一层，层可以缓存复用
3. **指令组合**：ENTRYPOINT+CMD是最常用模式
4. **多阶段构建**：大幅减小镜像体积
5. **最佳实践**：选择合适镜像、减少层数、清理不必要文件

### 学习建议

1. **多练习**：从简单Dockerfile开始，逐步增加复杂度
2. **阅读官方文档**：https://docs.docker.com/engine/reference/builder/
3. **分析优质镜像**：查看官方镜像的Dockerfile
4. **实践微服务**：尝试构建完整的微服务架构

---

### 附录：常见错误及解决方案

**错误1：Step X: COPY failed: file not found**

```
COPY failed: file not found in build context
```

解决方案：
- 检查文件路径是否正确
- 确保文件在构建上下文内
- 检查.dockerignore是否误排

**错误2：Exited (1) - standard init linuxcapable**

```
standard init linuxcapable.1.0.0: /bin/sh: executable not found
```

解决方案：
- 基础镜像中没有shell
- 使用exec形式的CMD/ENTRYPOINT

**错误3：No space left on device**

解决方案：
- 清理Docker构建缓存：`docker builder prune`
- 删除不需要的镜像：`docker image prune -a`
- 减小构建上下文

**错误4：network not found**

```
network xxx declared as external, but could not be found
```

解决方案：
- 确保网络已创建：`docker network create xxx`
- 或使用docker-compose自动创建

---

## 21. 下篇预告

下一章我们将学习 **Dockerfile指令详解**，深入掌握每个指令的高级用法和最佳实践。

**下篇内容预告**：
- RUN指令的shell vs exec模式深度解析
- 多阶段构建的高级技巧  
- 构建缓存优化策略
- 安全加固最佳实践
- 实战案例分析