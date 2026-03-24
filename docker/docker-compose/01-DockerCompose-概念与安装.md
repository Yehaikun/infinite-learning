# Docker Compose 概念与安装

**Metadata**
- Date: 2026-03-24
- Topic: Docker Compose
- Tags: #docker #docker-compose #容器编排 #入门 #学习路径01

---

## 1. 为什么需要 Docker Compose

### 1.1 真实场景引入：微服务架构的痛点

想象一下，你正在开发一个典型的微服务项目「hubspace-backend」，包含以下服务：

- **auth-server**：认证服务，端口8081
- **register-server**：注册服务，端口8082
- **MySQL**：数据库，端口3306
- **Redis**：缓存，端口6379
- **RabbitMQ**：消息队列，端口5672/15672

在纯Docker时代，你需要手动管理这些容器：

```bash
# 1. 启动MySQL
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=root123 \
  -e MYSQL_DATABASE=lyjew \
  -p 3306:3306 \
  mysql:8.4

# 2. 启动Redis
docker run -d --name redis \
  -p 6379:6379 \
  redis:8.0 \
  redis-server --requirepass redis123

# 3. 启动RabbitMQ
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=hubspace \
  -e RABBITMQ_DEFAULT_PASS=hubspace123 \
  rabbitmq:3.13-management

# 4. 启动auth-server（假设已有镜像）
docker run -d --name auth-server \
  -p 8081:8081 \
  --link mysql:mysql \
  --link redis:redis \
  --link rabbitmq:rabbitmq \
  auth-server:latest

# 5. 启动register-server
docker run -d --name register-server \
  -p 8082:8082 \
  --link mysql:mysql \
  --link redis:redis \
  --link rabbitmq:rabbitmq \
  register-server:latest
```

**问题来了：**

1. **命令太长太繁琐**：每次启动都要复制粘贴一长串命令，容易出错
2. **依赖顺序问题**：必须先启动MySQL/Redis/RabbitMQ，再启动应用服务，忘记顺序会导致连接失败
3. **环境一致性问题**：不同机器上可能环境不同，这个机器能跑，那个机器不行
4. **清理麻烦**：需要一个个停止和删除容器，命令敲好几遍
5. **日志分散**：需要分别查看每个容器的日志，排查问题时非常不便
6. **扩缩容困难**：想启动多个实例需要手动复制命令，运维成本高
7. **配置难以复用**：换台机器又要重新敲一遍命令

### 1.2 Docker Compose 的诞生

Docker Compose 就是为了解决这些问题而生的。它的核心思想是：

> **用配置文件代替命令行，用声明式代替命令式**

你只需要编写一个 `docker-compose.yml` 文件，然后一行命令就能启动整个应用：

```bash
# 一键启动所有服务
docker-compose up -d

# 一键停止所有服务
docker-compose down

# 查看所有服务状态
docker-compose ps
```

这就是 Docker Compose 的本质：**用 YAML 配置文件来定义和运行多容器 Docker 应用**。

### 1.3 Docker Compose 之前 vs 之后对比

| 对比项 | Docker Compose 之前 | Docker Compose 之后 |
|--------|---------------------|---------------------|
| 启动应用 | 敲5+条命令 | 1条命令 |
| 停止应用 | 敲5+条命令 | 1条命令 |
| 配置管理 | 命令行参数，分散 | YAML文件，集中 |
| 环境迁移 | 重新敲命令 | 复制配置文件 |
| 服务依赖 | 手动控制顺序 | 自动处理 |
| 日志查看 | 多个命令 | 统一查看 |
| 扩缩容 | 手动复制 | 1条命令 |

---

## 2. Docker Compose 是什么

### 2.1 一句话解释

**Docker Compose 是一个工具，用于通过 YAML 配置文件来定义和运行多容器 Docker 应用。**

你可以理解为：
- Docker = 单个容器管理工具
- Docker Compose = 多容器应用编排工具

### 2.2 核心概念

Docker Compose 有三个核心概念，必须理解：

#### 1. 服务（Service）

在 Docker Compose 中，一个**服务**对应一个**容器**。

例如：
- MySQL 服务 = MySQL 容器
- Redis 服务 = Redis 容器
- auth-server 服务 = auth-server 容器

```yaml
services:
  mysql:          # 服务名（不是容器名！）
    image: mysql:8.4
    ports:
      - "3306:3306"
  
  redis:          # 服务名
    image: redis:8.0
    
  auth-server:    # 服务名
    build: ./auth-server
```

**重要**：服务名不是容器名。容器名的格式是 `{项目名}-{服务名}-{序号}`。

#### 2. 项目（Project）

一个 Docker Compose 文件就是一个**项目**。项目名默认是所在目录名，也可以通过 `-p` 参数指定。

```bash
# 项目名默认是目录名 "myapp"
docker-compose up

# 指定项目名为 "hubspace"
docker-compose -p hubspace up
```

项目名会作为以下内容的前缀：
- 网络名称：`{项目名}_default`
- 卷名称：`{项目名}_数据卷名`

#### 3. 网络（Network）

Docker Compose 会为每个项目创建一个默认网络，服务之间可以通过服务名互相通信。

```yaml
services:
  auth-server:
    build: ./auth-server
    # 在同一网络中，可以直接通过服务名访问
    # 例如：Spring配置中写 redis:6379 就能访问 Redis 容器
```

### 2.3 Docker Compose vs Docker 命令对比

| 操作 | 纯Docker命令 | Docker Compose |
|------|-------------|----------------|
| 启动单个容器 | `docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=root123 mysql:8.4` | `docker-compose up -d` |
| 停止容器 | `docker stop mysql && docker rm mysql` | `docker-compose down` |
| 查看状态 | `docker ps` | `docker-compose ps`| 查看日志 | `docker logs mysql` | `docker-compose logs mysql` |
| 进入容器 | `docker exec -it mysql bash` | `docker-compose exec mysql bash` |
| 缩放服务 | 手动运行多个容器 | `docker-compose up -d --scale auth-server=3` |
| 拉取镜像 | `docker pull mysql:8.4` | `docker-compose pull` |
| 查看资源 | `docker stats` | `docker-compose stats` |

### 2.4 Docker Compose 适用场景

**适合使用 Docker Compose 的场景：**

1. **本地开发环境**：一键启动完整的开发环境
2. **自动化测试**：快速搭建测试环境
3. **CI/CD 流水线**：构建和测试环境标准化
4. **小型生产部署**：不需要 Kubernetes 的简单应用

**不适合使用 Docker Compose 的场景：**

1. **大规模生产部署**：需要 Kubernetes 或 Docker Swarm
2. **需要高可用**：多机器负载均衡
3. **复杂的服务发现**：服务频繁上下线

---

## 3. Docker Compose 安装

### 3.1 Linux 安装（Debian/Ubuntu）

Docker Compose V2 是当前主流版本，V2 和 V1 语法兼容。V2 使用 `docker compose`（空格），V1 使用 `docker-compose`（横杠）。

**方法一：通过 Docker Desktop 安装（推荐用于桌面端）**

如果你使用 Docker Desktop for Linux，它已经内置了 Docker Compose：

```bash
# 验证安装
docker compose version
# Docker Compose version v2.24.0
```

**方法二：通过 apt 安装**

```bash
# 1. 更新包索引
sudo apt update

# 2. 安装依赖
sudo apt install -y ca-certificates curl gnupg

# 3. 添加 Docker 官方 GPG 密钥
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# 4. 添加 Docker 仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 安装 Docker Compose 插件
sudo apt update
sudo apt install -y docker-compose-plugin

# 6. 验证
docker compose version
```

**方法三：直接下载二进制文件**

```bash
# 1. 下载 Docker Compose V2
sudo curl -SL https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

# 2. 添加执行权限
sudo chmod +x /usr/local/bin/docker-compose

# 3. 验证（注意 V2 使用空格而不是横杠）
docker compose version
```

> ⚠️ 注意：Docker Compose V2 的命令是 `docker compose`（空格），不是 `docker-compose`（横杠）。V1 才是横杠。

### 3.2 验证安装

```bash
# 检查 Docker Compose 版本
docker compose version

# 预期输出类似：
# Docker Compose version v2.24.0

# 如果输出 command not found，说明安装失败
# 如果输出 Docker Compose version v1.x.x，说明安装的是 V1
```

如果输出类似 `Docker Compose version v2.24.0`，说明安装成功。

### 3.3 Docker Engine 要求

Docker Compose V2 需要 Docker Engine 20.10+。检查 Docker 版本：

```bash
docker --version
# Docker version 24.0.0, build 12345

# 或者
docker version
# Client: Docker Engine
#  Version:           24.0.0
# Server: Docker Engine
#  Version:           24.0.0
```

---

## 4. 快速入门：第一个 Docker Compose 项目

### 4.1 场景说明

让我们从最简单的例子开始：**运行一个 Nginx Web 服务器**。

这是最基础的入门案例，帮助你理解 Docker Compose 的工作流程。

### 4.2 创建项目目录

```bash
# 创建项目目录
mkdir -p ~/docker-projects/nginx-demo
cd ~/docker-projects/nginx-demo
```

### 4.3 创建 docker-compose.yml

```yaml
# docker-compose.yml
version: "3.8"  # Compose 文件格式版本

services:       # 定义服务（容器）
  nginx:        # 服务名
    image: nginx:1.25  # 使用 nginx 镜像
    ports:      # 端口映射
      - "8080:80"     # 宿主机8080 -> 容器80
    volumes:    # 卷挂载
      - "./html:/usr/share/nginx/html"  # 挂载HTML目录
```

### 4.4 启动服务

```bash
# 启动服务（-d 表示后台运行）
docker compose up -d

# 预期输出：
# [+] Running 2/2
#  ✔ Network nginx-demo_default        Created
#  ✔ Container nginx-demo-nginx-1      Started
```

**注意**：
- 项目名是目录名 `nginx-demo`
- 容器名是 `nginx-demo-nginx-1`
- 自动创建了网络 `nginx-demo_default`

### 4.5 验证运行

```bash
# 查看服务状态
docker compose ps

# 预期输出：
# NAME                IMAGE               COMMAND                  SERVICE             CREATED             STATUS
# nginx-demo-nginx-1  nginx:1.25          "/docker-entrypoint.…"   nginx               About a minute ago  Up About a minute
# nginx-demo-nginx-1  nginx:1.25          "/docker-entrypoint.…"   nginx               About a minute ago  Up About a minute (healthcheck)

# 测试访问
curl http://localhost:8080

# 预期输出（nginx默认页面）：
# <!DOCTYPE html>
# <html>
# <head>
# <title>Welcome to nginx!</title>
# ...
```

### 4.6 停止服务

```bash
# 停止并删除容器
docker compose down

# 预期输出：
# [+] Running 2/2
#  ✔ Container nginx-demo-nginx-1      Removed
#  ✔ Network nginx-demo_default        Removed
```

### 4.7 完整流程回顾

```
1. 创建目录
2. 编写 docker-compose.yml
3. 执行 docker compose up -d
4. 验证运行 docker compose ps
5. 测试访问 curl localhost:8080
6. 清理环境 docker compose down
```

这就是 Docker Compose 的基本工作流程，非常简单！

---

## 5. 进阶示例：MySQL + Redis + Spring Boot 应用

### 5.1 场景说明

这个示例展示了如何用 Docker Compose 启动一个完整的微服务开发环境：

- MySQL 8.4（数据库）
- Redis 8.0（缓存）
- RabbitMQ 3.13（消息队列）
- auth-server（Spring Boot 应用）

这是一个典型的 Java 微服务开发环境，也是你未来项目中会用到的配置。

### 5.2 项目结构

```
hubspace-backend/
├── docker-compose.yml
├── auth-server/
│   ├── Dockerfile
│   └── target/auth-server.jar
└── init.sql
```

### 5.3 docker-compose.yml 完整示例

```yaml
version: "3.8"

services:
  # MySQL 数据库
  mysql:
    image: mysql:8.4
    container_name: hubspace-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: lyjew
      MYSQL_USER: hubspace
      MYSQL_PASSWORD: hubspace123
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - hubspace-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-proot123"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  # Redis 缓存
  redis:
    image: redis:8.0
    container_name: hubspace-redis
    restart: always
    command: redis-server --requirepass redis123
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - hubspace-network
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # RabbitMQ 消息队列
  rabbitmq:
    image: rabbitmq:3.13-management
    container_name: hubspace-rabbitmq
    restart: always
    environment:
      RABBITMQ_DEFAULT_USER: hubspace
      RABBITMQ_DEFAULT_PASS: hubspace123
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    networks:
      - hubspace-network
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Spring Boot 应用 - auth-server
  auth-server:
    build:
      context: ./auth-server
      dockerfile: Dockerfile
    container_name: hubspace-auth-server
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/lyjew?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: root123
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: redis123
      SPRING_RABBITMQ_HOST: rabbitmq
      SPRING_RABBITMQ_USERNAME: hubspace
      SPRING_RABBITMQ_PASSWORD: hubspace123
    ports:
      - "8081:8081"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - hubspace-network

# 定义网络
networks:
  hubspace-network:
    driver: bridge

# 定义卷
volumes:
  mysql-data:
  redis-data:
  rabbitmq-data:
```

### 5.4 关键配置解释

#### 5.4.1 depends_on（服务依赖）

```yaml
depends_on:
  mysql:
    condition: service_healthy  # 等待 MySQL 健康检查通过
  redis:
    condition: service_healthy
  rabbitmq:
    condition: service_healthy
```

`depends_on` 确保服务启动顺序：MySQL → Redis → RabbitMQ → auth-server。

**注意**：
- V3 版本的 `depends_on` 不支持 `condition` 参数
- V2.4 版本支持 `condition`
- 如果使用 V3，需要在应用层处理重试（Spring Boot 的 `spring-boot-starter-actuator` 和 `spring-retry` 可以帮忙）

#### 5.4.2 healthcheck（健康检查）

```yaml
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-proot123"]
  interval: 10s      # 检查间隔
  timeout: 5s        # 超时时间
  retries: 5          # 重试次数
  start_period: 30s  # 启动等待时间（给服务初始化时间）
```

健康检查确保服务真正启动后才认为可用。

**各服务的健康检查命令**：

```yaml
# MySQL
healthcheck:
  test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-proot123"]

# Redis
healthcheck:
  test: ["CMD", "redis-cli", "--raw", "ping"]

# RabbitMQ
healthcheck:
  test: ["CMD", "rabbitmq-diagnostics", "ping"]

# Nginx
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost/"]

# 自定义健康检查
healthcheck:
  test: ["CMD", "wget", "--spider", "-q", "http://localhost:8081/actuator/health"]
```

#### 5.4.3 networks（网络隔离）

```yaml
networks:
  hubspace-network:
    driver: bridge
```

所有服务在同一个网络中，可以通过服务名互相访问。

**网络通信示例**：

```yaml
# auth-server 访问 MySQL
SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/lyjew  # 使用服务名 "mysql"

# auth-server 访问 Redis
SPRING_REDIS_HOST: redis  # 使用服务名 "redis"

# auth-server 访问 RabbitMQ
SPRING_RABBITMQ_HOST: rabbitmq  # 使用服务名 "rabbitmq"
```

#### 5.4.4 volumes（数据持久化）

```yaml
volumes:
  mysql-data:        # 命名卷，Docker 管理
  - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # 绑定挂载
```

**卷的类型**：

1. **命名卷**（Named Volume）：`mysql-data`
   - Docker 自动管理
   - 数据存储在 Docker 引擎的存储目录
   - 适合数据库等需要持久化的数据

2. **绑定挂载**（Bind Mount）：`./init.sql:/docker-entrypoint-initdb.d/init.sql`
   - 宿主机目录映射到容器
   - 适合配置文件、初始化脚本
   - 宿主机修改立即生效

3. **临时卷**（Tmpfs）：只存储在内存中，重启丢失
   - 适合临时数据

#### 5.4.5 restart（重启策略）

```yaml
restart: always  # 容器退出总是重启
```

| 值 | 说明 |
|----|------|
| `no` | 不自动重启（默认） |
| `always` | 总是重启 |
| `on-failure` | 退出码非0时重启 |
| `unless-stopped` | 除非手动停止，否则总是重启 |

生产环境推荐使用 `always` 或 `unless-stopped`。

#### 5.4.6 container_name（容器名）

```yaml
container_name: hubspace-mysql
```

默认情况下，容器名是 `{项目名}-{服务名}-{序号}`。使用 `container_name` 可以自定义容器名。

### 5.5 常用命令汇总

```bash
# 启动所有服务
docker compose up -d

# 启动并重新构建镜像
docker compose up -d --build

# 查看所有服务日志
docker compose logs -f

# 查看指定服务日志
docker compose logs -f auth-server

# 停止所有服务
docker compose stop

# 停止并删除容器、网络（保留卷）
docker compose down

# 停止并删除所有（包括卷）
docker compose down -v

# 查看服务状态
docker compose ps

# 进入服务容器
docker compose exec mysql bash

# 查看服务资源使用
docker stats

# 扩缩容（仅支持 docker-compose V1）
docker-compose up -d --scale auth-server=3

# V2 版本需要使用 deploy 配置
```

---

## 6. Docker Compose 文件版本说明

### 6.1 版本选择建议

| Compose 文件版本 | Docker Engine 版本 | 推荐场景 |
|-----------------|-------------------|---------|
| 3.9 | 20.10+ | **推荐**，最新功能 |
| 3.8 | 19.03+ | 推荐，功能全面 |
| 3.7 | 18.06+ | 兼容性好 |
| 3.3 | 1.13+ | 老项目兼容 |
| 2.4 | 17.12+ | 需要 depends_on condition |

> ⚠️ 注意：Docker Compose V2 的默认版本是 3.9。

### 6.2 版本差异对比

```yaml
# V2.4（支持 healthcheck 和 condition）
version: "2.4"
services:
  web:
    depends_on:
      db:
        condition: service_healthy

# V3.x（推荐，但 depends_on 不支持 condition）
version: "3.8"
services:
  web:
    depends_on:
      - db
    # 健康检查需要单独配置
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
```

### 6.3 版本选择建议

1. **新项目**：直接使用 V3.9
2. **需要健康检查等待**：使用 V2.4
3. **老项目**：保持原有版本不变

---

## 7. 本章总结

### 7.1 核心要点

1. **Docker Compose 解决的问题**：多容器应用的编排和管理，用配置文件代替繁琐的命令行
2. **核心概念**：
   - 服务（Service）= 容器
   - 项目（Project）= 一组相关联的服务
   - 网络（Network）= 服务间通信的基础
3. **核心命令**：
   - `docker compose up -d` 启动
   - `docker compose down` 停止
   - `docker compose ps` 查看状态
   - `docker compose logs` 查看日志
4. **配置文件**：使用 YAML 格式的 `docker-compose.yml`

### 7.2 理解要点

- Docker = 单容器管理
- Docker Compose = 多容器编排
- 一个服务 = 一个容器
- 服务间通过服务名（不是容器名）通信

### 7.3 下篇预告

下篇我们将深入讲解 **Docker Compose YAML 编写详解**，包括：
- YAML 语法基础
- 核心配置项详解（image、build、ports、environment 等）
- 高级配置（depends_on、networks、volumes 等）
- 完整的配置示例

---

## 8. 课后问题

1. **问题**：Docker Compose 和 Docker 命令的主要区别是什么？
   **提示**：从"声明式 vs 命令式"和"单容器 vs 多容器"角度思考。

2. **问题**：如果 MySQL 容器启动慢，Spring Boot 应用容器先启动了会怎样？
   **提示**：考虑服务启动顺序和依赖关系，以及如何保证服务按顺序启动。

3. **问题**：如何确保 MySQL 完全启动后再启动应用？
   **提示**：考虑 healthcheck 和 depends_on 的配合使用。

4. **问题**：服务名和容器名的区别是什么？
   **提示**：思考服务名在 YAML 中如何定义，容器名默认格式是什么。

5. **问题**：为什么 Spring 配置中可以使用 `mysql:3306` 而不是容器 IP？
   **提示**：考虑 Docker Compose 创建的网络和服务发现机制。

---

## 9. 附录：常见问题

### Q1: docker-compose 和 docker compose 有什么区别？

- `docker-compose`（带横杠）：V1 版本
- `docker compose`（空格）：V2 版本

推荐使用 V2，它集成在 Docker 桌面版中，命令更简洁。V2 是趋势，未来会完全取代 V1。

### Q2: 为什么服务之间可以通过服务名访问？

Docker Compose 会为每个项目创建一个默认的 bridge 网络，服务名会自动解析为容器 IP。这是 Docker 内置的服务发现机制，类似于 DNS。

```bash
# 在容器内部测试
docker compose exec auth-server ping mysql
# 能 ping 通，说明服务名解析正常工作
```

### Q3: 如何查看 Docker Compose 的版本？

```bash
# 查看 Compose 工具版本
docker compose version

# 查看 Compose 文件版本（在 YAML 中）
version: "3.8"
```

### Q4: docker-compose.yml 文件名可以改吗？

可以，但需要显式指定：

```bash
# 使用默认文件名 docker-compose.yml
docker compose up -d

# 指定自定义文件名
docker compose -f docker-compose.dev.yml up -d
docker compose -f docker-compose.prod.yml up -d
```

常见用法：
- `docker-compose.yml` - 默认
- `docker-compose.dev.yml` - 开发环境
- `docker-compose.prod.yml` - 生产环境
- `docker-compose.override.yml` - 本地覆盖配置

### Q5: 容器自动停止了怎么办？

```bash
# 查看容器停止原因
docker compose ps

# 查看详细日志
docker compose logs 服务名

# 查看容器详情
docker inspect 容器名
```

常见原因：
- 内存不足（OOM）
- 端口冲突
- 配置文件错误
- 健康检查失败

### Q6: 如何更新容器内的配置？

```bash
# 修改 docker-compose.yml 后重新创建容器
docker compose up -d

# 如果修改了环境变量
docker compose up -d --env .env.production
```

### Q7: 多台机器如何共享数据？

Docker Compose 默认只在单机运行。如果需要多机器共享：
- 使用 Docker Swarm
- 使用 Kubernetes
- 使用共享存储（NFS、CephFS）

### Q8: 如何让容器开机自启动？

有几种方式可以实现：

**方式一：Docker Compose 配置 restart 策略**

```yaml
services:
  mysql:
    image: mysql:8.4
    restart: always  # 开机自启
```

**方式二：使用 systemd 管理 Docker Compose**

创建服务文件 `/etc/systemd/system/docker-compose-app.service`：

```ini
[Unit]
Description=My Docker Compose Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
WorkingDirectory=/path/to/project
ExecStart=/usr/local/bin/docker compose up -d
ExecStop=/usr/local/bin/docker compose down
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

然后启用服务：

```bash
sudo systemctl enable docker-compose-app
sudo systemctl start docker-compose-app
```

### Q9: 容器内中文乱码怎么办？

```yaml
services:
  mysql:
    image: mysql:8.4
    environment:
      LANG: C.UTF-8
      LC_ALL: C.UTF-8
    volumes:
      - ./my.cnf:/etc/mysql/conf.d/my.cnf
```

创建 `my.cnf`：

```ini
[client]
default-character-set=utf8mb4

[mysqld]
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
```

### Q10: 容器内的时区不对怎么办？

```yaml
services:
  mysql:
    image: mysql:8.4
    environment:
      TZ: Asia/Shanghai
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /usr/share/zoneinfo/Asia/Shanghai:/etc/timezone:ro
```

---

## 10. 实战练习：搭建你的第一个微服务环境

### 10.1 练习目标

搭建一个完整的 Spring Boot + MySQL + Redis 开发环境，体验 Docker Compose 的完整流程。

### 10.2 前置要求

- 已安装 Docker Engine 20.10+
- 已安装 Docker Compose V2

### 10.3 练习步骤

**步骤 1：创建项目目录**

```bash
mkdir -p ~/docker-projects/spring-boot-dev
cd ~/docker-projects/spring-boot-dev
```

**步骤 2：创建 docker-compose.yml**

```yaml
version: "3.8"

services:
  # MySQL 数据库
  mysql:
    image: mysql:8.4
    container_name: dev-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: hubspace
      MYSQL_USER: dev
      MYSQL_PASSWORD: dev123
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - dev-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 缓存
  redis:
    image: redis:8.0
    container_name: dev-redis
    restart: always
    command: redis-server --requirepass redis123
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - dev-network
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  dev-network:
    driver: bridge

volumes:
  mysql-data:
  redis-data:
```

**步骤 3：启动服务**

```bash
docker compose up -d
```

**步骤 4：验证服务**

```bash
# 查看状态
docker compose ps

# 测试 MySQL
docker compose exec mysql mysql -uroot -proot123 -e "SHOW DATABASES;"

# 测试 Redis
docker compose exec redis redis-cli -a redis123 ping
```

**步骤 5：停止服务**

```bash
docker compose down
```

### 10.4 预期结果

- MySQL 可以通过 localhost:3306 访问
- Redis 可以通过 localhost:6379 访问
- 数据在容器重启后依然存在（使用命名卷）
- 服务之间可以互相通信

---

## 11. 参考资料

- [Docker Compose 官方文档](https://docs.docker.com/compose/)
- [Docker Compose 文件参考](https://docs.docker.com/compose/compose-file/)
- [Docker Compose 安装指南](https://docs.docker.com/compose/install/)

