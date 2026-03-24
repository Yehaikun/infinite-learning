# Docker Compose 网络与存储

**Metadata**
- Date: 2026-03-24
- Topic: Docker Compose
- Tags: #docker #docker-compose #network #volume #存储 #学习路径03

---

## 1. Docker 网络基础

### 1.1 为什么需要网络

在微服务架构中，服务之间需要互相通信。比如：
- Spring Boot 应用需要连接 MySQL
- 应用需要访问 Redis 缓存
- 不同服务之间通过 RabbitMQ 通信

Docker 网络就是解决"容器之间如何通信"的问题。

### 1.2 Docker 网络驱动类型

Docker 提供了多种网络驱动：

| 驱动类型 | 说明 | 适用场景 |
|----------|------|----------|
| **bridge** | 桥接网络，默认类型 | 单主机多容器 |
| **host** | 主机网络，共享主机网络栈 | 性能敏感场景 |
| **overlay** | 覆盖网络，跨主机通信 | Docker Swarm |
| **macvlan** | MAC 地址虚拟化 | 需要直接暴露 IP |
| **none** | 无网络，容器完全隔离 | 隔离安全场景 |

### 1.3 Docker Compose 默认网络

当你运行 `docker compose up` 时，Docker Compose 会：

1. 自动创建一个 bridge 网络：`{项目名}_default`
2. 所有服务加入这个网络
3. 服务之间可以通过服务名互相访问

```bash
# 默认创建的网络
docker network ls

# 输出类似：
# NETWORK ID     NAME                 DRIVER    SCOPE
# abc123...      nginx-demo_default   bridge    local
```

---

## 2. Docker Compose 网络配置

### 2.1 基本网络配置

```yaml
version: "3.8"

services:
  web:
    image: nginx:1.25
    networks:
      - frontend
      - backend

  app:
    image: myapp
    networks:
      - backend

  db:
    image: mysql:8.4
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

**网络通信规则**：
- web 可以访问 frontend 和 backend 网络
- app 只能访问 backend 网络
- db 只能访问 backend 网络
- app 和 db 可以互相通信（在同一个 backend 网络）

### 2.2 指定网络模式

```yaml
services:
  # 使用默认 bridge 网络
  web:
    image: nginx:1.25

  # 使用主机网络（直接使用主机网络栈）
  proxy:
    image: nginx:1.25
    network_mode: host
```

### 2.3 网络高级配置

#### 2.3.1 自定义子网

```yaml
networks:
  backend:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1
```

#### 2.3.2 容器间通信

同一网络下的容器可以通过服务名直接访问：

```yaml
services:
  app:
    image: myapp
    networks:
      - backend

  mysql:
    image: mysql:8.4
    networks:
      - backend
```

在 app 容器内，访问 MySQL：

```bash
# 使用服务名 mysql 即可访问
# 连接地址：mysql:3306
# 不需要知道容器 IP

# 测试连通性
docker compose exec app ping mysql
docker compose exec app telnet mysql 3306
```

### 2.4 外部网络

如果需要容器加入已经存在的网络：

```yaml
networks:
  # 定义一个外部网络（已存在）
  existing-network:
    external: true

services:
  app:
    networks:
      - existing-network
```

### 2.5 网络别名

```yaml
services:
  mysql:
    image: mysql:8.4
    networks:
      backend:
        aliases:
          - database      # 别名
          - mysql-server  # 另一个别名
```

这样，在 backend 网络中，容器可以通过以下任何一个名称访问：
- `mysql`
- `database`
- `mysql-server`

---

## 3. 实战：微服务网络配置

### 3.1 场景说明

一个典型的微服务架构：
- **gateway**：API 网关，对外暴露
- **auth-service**：认证服务
- **order-service**：订单服务
- **product-service**：商品服务
- **mysql**：数据库，仅内部访问
- **redis**：缓存，仅内部访问
- **rabbitmq**：消息队列，仅内部访问

### 3.2 网络分层设计

```
┌─────────────────────────────────────────────────────────┐
│                      外网                                │
│                    (浏览器/APP)                          │
└─────────────────────┬───────────────────────────────────┘
                      │ 端口 80/443
                      ▼
              ┌────────────────┐
              │    gateway     │  ← frontend 网络
              └────────┬───────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
         ▼             ▼             ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│auth-service│ │order-service│ │product-service│
│            │ │            │ │            │
└─────┬──────┘ └─────┬──────┘ └─────┬──────┘
      │              │              │
      └──────────────┼──────────────┘
                     │  backend 网络
      ┌──────────────┼──────────────┐
      │              │              │
      ▼              ▼              ▼
  ┌────────┐   ┌────────┐   ┌────────┐
  │  MySQL │   │  Redis │   │RabbitMQ│
  └────────┘   └────────┘   └────────┘
```

### 3.3 docker-compose.yml

```yaml
version: "3.8"

services:
  # ==================== 前端服务 ====================
  
  # API 网关
  gateway:
    image: nginx:1.25-alpine
    container_name: gateway
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - frontend
    depends_on:
      - auth-service
      - order-service
      - product-service

  # ==================== 业务服务 ====================
  
  # 认证服务
  auth-service:
    build:
      context: ./auth-service
      dockerfile: Dockerfile
    container_name: auth-service
    restart: always
    environment:
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD}
    networks:
      - frontend
      - backend
    depends_on:
      redis:
        condition: service_healthy

  # 订单服务
  order-service:
    build:
      context: ./order-service
      dockerfile: Dockerfile
    container_name: order-service
    restart: always
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/${MYSQL_DATABASE}
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD}
    networks:
      - backend
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy

  # 商品服务
  product-service:
    build:
      context: ./product-service
      dockerfile: Dockerfile
    container_name: product-service
    restart: always
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/${MYSQL_DATABASE}
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD}
    networks:
      - backend
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy

  # ==================== 基础设施 ====================
  
  # MySQL 数据库
  mysql:
    image: mysql:8.4
    container_name: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - backend
    healthcheck:
      test: ["CMD", "mysqladmin", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  # Redis 缓存
  redis:
    image: redis:8.0-alpine
    container_name: redis
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # RabbitMQ 消息队列
  rabbitmq:
    image: rabbitmq:3.13-management-alpine
    container_name: rabbitmq
    restart: always
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    networks:
      - backend
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 15s
      timeout: 10s
      retries: 5

# ==================== 网络定义 ====================
networks:
  # 外部网络：网关和前端服务
  frontend:
    driver: bridge
    name: microservices-frontend
  
  # 内部网络：后端服务和基础设施
  backend:
    driver: bridge
    name: microservices-backend
    internal: false  # 是否禁用外部访问

# ==================== 卷定义 ====================
volumes:
  mysql-data:
  redis-data:
  rabbitmq-data:
```

---

## 4. Docker 存储基础

### 4.1 为什么需要存储

默认情况下，容器内的数据是临时的：
- 容器删除 = 数据丢失
- 容器重启 = 数据丢失

对于数据库、文件存储等需要持久化的场景，需要使用 Docker 卷。

### 4.2 存储类型

| 类型 | 说明 | 生命周期 | 用途 |
|------|------|---------|------|
| **匿名卷** | Docker 自动创建 | 随容器删除 | 临时数据 |
| **命名卷** | 自定义名称 | 独立于容器 | 持久化数据（推荐） |
| **绑定挂载** | 宿主机目录映射 | 手动管理 | 配置、代码 |
| **tmpfs** | 内存存储 | 随容器停止删除 | 敏感数据、缓存 |

---

## 5. Docker Compose 存储配置

### 5.1 命名卷（推荐）

```yaml
version: "3.8"

services:
  mysql:
    image: mysql:8.4
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

**特点**：
- Docker 自动管理存储位置
- 数据独立于容器生命周期
- 支持热备份

### 5.2 绑定挂载

```yaml
services:
  nginx:
    image: nginx:1.25
    volumes:
      # 相对路径（相对于 docker-compose.yml）
      - ./html:/usr/share/nginx/html
      
      # 绝对路径
      - /opt/config:/etc/nginx/conf.d
      
      # 只读挂载
      - ./config:/etc/nginx:ro
      
      # 单文件
      - ./nginx.conf:/etc/nginx/nginx.conf
```

### 5.3 tmpfs（内存存储）

```yaml
services:
  redis:
    image: redis:8.0
    tmpfs:
      - /tmp        # 内存存储
      - /run        # 内存存储
```

适用于：
- 敏感数据（不落盘）
- 临时缓存
- 性能敏感场景

### 5.4 只读挂载

```yaml
services:
  app:
    image: myapp
    volumes:
      # ro = read only
      - ./config:/app/config:ro
      
      # shared, slave, private 等挂载选项
      - ./data:/app/data:ro
```

### 5.5 多容器共享卷

```yaml
services:
  app1:
    image: myapp
    volumes:
      - shared-data:/data

  app2:
    image: myapp
    volumes:
      - shared-data:/data

volumes:
  shared-data:
```

---

## 6. 实战：MySQL 数据持久化

### 6.1 场景说明

MySQL 的数据目录是 `/var/lib/mysql`，需要持久化存储。

### 6.2 错误的做法

```yaml
# ❌ 错误：没有挂载卷，容器删除后数据丢失
services:
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: root123
```

### 6.3 正确的做法

```yaml
# ✅ 正确：使用命名卷持久化
services:
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: root123
    volumes:
      - mysql-data:/var/lib/mysql

volumes:
  mysql-data:
```

### 6.4 更好的做法：绑定初始化脚本

```yaml
services:
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: hubspace
    volumes:
      - mysql-data:/var/lib/mysql
      # 初始化脚本：容器首次启动时自动执行
      - ./init-db:/docker-entrypoint-initdb.d
      # 自定义配置
      - ./conf/my.cnf:/etc/mysql/conf.d/my.cnf:ro

volumes:
  mysql-data:
```

初始化脚本目录结构：

```
init-db/
├── 01-schema.sql      # 建表语句
├── 02-data.sql        # 初始数据
└── 03-procedures.sql # 存储过程
```

### 6.5 最完整的生产配置

```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: production-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      TZ: Asia/Shanghai
    ports:
      - "3306:3306"
    volumes:
      # 1. 数据目录（命名卷）
      - mysql-data:/var/lib/mysql
      
      # 2. 初始化脚本
      - ./init-db:/docker-entrypoint-initdb.d:ro
      
      # 3. 配置文件
      - ./conf/my.cnf:/etc/mysql/conf.d/my.cnf:ro
      
      # 4. 日志目录
      - ./logs:/var/log/mysql
    networks:
      - backend
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    command:
      # MySQL 启动参数
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --max-connections=500
      - --innodb-buffer-pool-size=256M

volumes:
  mysql-data:

networks:
  backend:
    driver: bridge
```

---

## 7. 存储高级话题

### 7.1 卷的备份与恢复

**备份**：

```bash
# 创建临时容器，备份卷数据
docker run --rm -v myapp_mysql-data:/data -v $(pwd):/backup alpine tar czf /backup/mysql-backup.tar.gz -C /data .

# 或者使用 docker compose
docker compose run --rm mysql tar czf /backup/mysql-backup.tar.gz -C /var/lib/mysql .
```

**恢复**：

```bash
# 停止服务
docker compose stop mysql

# 恢复数据
docker run --rm -v myapp_mysql-data:/data -v $(pwd):/backup alpine tar xzf /backup/mysql-backup.tar.gz -C /data

# 启动服务
docker compose start mysql
```

### 7.2 查看卷信息

```bash
# 查看所有卷
docker volume ls

# 查看卷详情
docker volume inspect myapp_mysql-data

# 查看卷使用情况
docker system df -v
```

### 7.3 卷的清理

```bash
# 删除未使用的卷（危险！数据会丢失）
docker volume prune

# 删除指定卷
docker volume rm myapp_mysql-data
```

---

## 8. 常见问题

### Q1: 容器无法连接到其他容器？

检查网络配置：

```bash
# 查看网络
docker network ls

# 查看网络详情
docker network inspect myapp_default

# 测试连通性
docker compose exec web ping mysql
```

常见原因：
- 不在同一网络
- 网络名称拼写错误
- 服务名拼写错误

### Q2: 服务可以访问外网但无法访问其他容器？

```yaml
# 检查网络配置
services:
  web:
    networks:
      - my-network

networks:
  my-network:
    driver: bridge
    name: myapp_my-network
```

或者可能是 DNS 问题：

```bash
# 在容器内测试 DNS
docker compose exec web nslookup mysql
docker compose exec web cat /etc/hosts
```

### Q3: 数据卷权限问题？

```yaml
# 方案1：指定用户
services:
  app:
    image: myapp
    user: "1000:1000"

# 方案2：修改宿主机目录权限
chmod -R 777 /path/to/host/dir

# 方案3：在容器内修改权限
docker compose exec app chown -R 1000:1000 /data
```

### Q4: 如何在不同主机之间共享数据？

Docker Compose 是单主机工具。跨主机共享数据需要：

1. **Docker Swarm**：分布式存储
2. **Kubernetes**：持久卷
3. **NFS**：网络文件系统
4. **Ceph/GlusterFS**：分布式存储

### Q5: 容器内的数据可以跨容器共享吗？

可以，使用命名卷：

```yaml
services:
  app1:
    volumes:
      - shared-data:/app/data
  
  app2:
    volumes:
      - shared-data:/app/data

volumes:
  shared-data:
```

### Q6: 如何查看卷的内容？

```bash
# 使用临时容器查看
docker run --rm -it -v myapp_mysql-data:/data alpine ls -la /data
```

---

## 9. 本章总结

### 9.1 网络核心要点

1. **默认网络**：Docker Compose 自动创建 bridge 网络
2. **服务发现**：同一网络内通过服务名访问
3. **网络隔离**：不同服务放在不同网络实现隔离
4. **分层设计**：frontend（外部）+ backend（内部）

### 9.2 存储核心要点

1. **命名卷**：推荐用于数据库等持久化数据
2. **绑定挂载**：用于配置文件、初始化脚本
3. **tmpfs**：用于敏感数据、临时缓存
4. **只读挂载**：用 `:ro` 标记

### 9.3 最佳实践

```
生产环境存储配置：
├── 数据目录 → 命名卷（mysql-data:/var/lib/mysql）
├── 初始化脚本 → 绑定挂载（./init-db:/docker-entrypoint-initdb.d）
├── 配置文件 → 绑定挂载（./conf:/etc/mysql/conf.d:ro）
└── 日志目录 → 绑定挂载（./logs:/var/log/mysql）
```

### 9.4 下篇预告

下篇我们将讲解 **Docker Compose 实战与最佳实践**，包括：
- 完整的项目实战
- 开发/测试/生产环境配置
- 常见问题排查
- 性能优化

---

## 10. 课后问题

1. **问题**：如何让 MySQL 容器只能被应用容器访问，而不能从外部直接访问？
   **提示**：考虑网络隔离配置。

2. **问题**：命名卷和绑定挂载的区别是什么？
   **提示**：从管理方式、生命周期、适用场景考虑。

3. **问题**：如果 MySQL 数据卷损坏了，如何恢复？
   **提示**：考虑备份和恢复策略。

---

## 11. 附录：常用网络命令

```bash
# 查看所有网络
docker network ls

# 查看网络详情
docker network inspect 网络名

# 创建网络
docker network create my-network

# 删除网络
docker network rm my-network

# 容器连接到网络
docker network connect my-network 容器名

# 容器断开网络
docker network disconnect my-network 容器名

# 测试网络连通性
docker compose exec 服务名 ping 目标服务名
```

## 12. 附录：常用卷命令

```bash
# 查看所有卷
docker volume ls

# 查看卷详情
docker volume inspect 卷名

# 创建卷
docker volume create my-volume

# 删除未使用的卷
docker volume prune

# 删除指定卷
docker volume rm 卷名
```

## 13. 实战：多层网络架构

### 13.1 场景：安全隔离的微服务架构

一个金融系统，需要严格的网络隔离：
- DMZ 区：只能接收外部请求
- 应用区：运行核心业务
- 数据区：存放敏感数据

```
┌─────────────────────────────────────────────────────────┐
│                     外部网络                             │
└─────────────────────────┬───────────────────────────────┘
                          │ 端口 80/443
                          ▼
              ┌─────────────────────┐
              │      nginx          │  ← DMZ 网络
              └──────────┬──────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
  ┌────────────┐ ┌────────────┐ ┌────────────┐
  │   auth     │ │   order    │ │  product   │  ← 应用网络
  │  service   │ │  service   │ │  service   │
  └────────────┘ └────────────┘ └────────────┘
         │               │               │
         └───────────────┼───────────────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │  MySQL   │   │  Redis   │   │RabbitMQ  │  ← 数据网络
   └──────────┘   └──────────┘   └──────────┘
```

### 13.2 docker-compose.yml

```yaml
version: "3.8"

services:
  # ==================== DMZ 区 ====================
  nginx:
    image: nginx:1.25-alpine
    container_name: dmz-nginx
    restart: always
    ports:
      - "80:80"
      - "443:443"
    networks:
      - dmz
    depends_on:
      - auth-service
      - order-service

  # ==================== 应用区 ====================
  auth-service:
    image: auth-service:v1
    container_name: app-auth
    restart: always
    networks:
      - dmz
      - application
    depends_on:
      redis:
        condition: service_healthy

  order-service:
    image: order-service:v1
    container_name: app-order
    restart: always
    networks:
      - application
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy

  product-service:
    image: product-service:v1
    container_name: app-product
    restart: always
    networks:
      - application
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy

  # ==================== 数据区 ====================
  mysql:
    image: mysql:8.4
    container_name: db-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - data
    healthcheck:
      test: ["CMD", "mysqladmin", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    # 不暴露端口到宿主机，安全的做法
    # ports:
    #   - "3306:3306"

  redis:
    image: redis:8.0-alpine
    container_name: cache-redis
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis-data:/data
    networks:
      - data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  rabbitmq:
    image: rabbitmq:3.13-management-alpine
    container_name: mq-rabbitmq
    restart: always
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    # RabbitMQ 管理界面不需要暴露到生产环境
    # ports:
    #   - "5672:5672"
    #   - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    networks:
      - data
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 15s
      timeout: 10s
      retries: 5

# ==================== 网络定义 ====================
networks:
  # DMZ 区：外部可见
  dmz:
    driver: bridge
    name: secure-dmz
    ipam:
      driver: default
      config:
        - subnet: 172.20.1.0/24

  # 应用区：内部通信
  application:
    driver: bridge
    name: secure-application
    ipam:
      driver: default
      config:
        - subnet: 172.20.2.0/24

  # 数据区：完全隔离
  data:
    driver: bridge
    name: secure-data
    internal: true  # 禁用外部访问，完全隔离
    ipam:
      driver: default
      config:
        - subnet: 172.20.3.0/24

# ==================== 卷定义 ====================
volumes:
  mysql-data:
  redis-data:
  rabbitmq-data:
```

### 13.3 网络通信规则

| 服务 | dmz | application | data |
|------|-----|-------------|------|
| nginx | ✅ | ✅ | ❌ |
| auth-service | ✅ | ✅ | ❌ |
| order-service | ❌ | ✅ | ✅ |
| product-service | ❌ | ✅ | ✅ |
| mysql | ❌ | ❌ | ✅ |
| redis | ❌ | ❌ | ✅ |
| rabbitmq | ❌ | ❌ | ✅ |

---

## 14. 常见网络故障排查

### 14.1 容器无法解析服务名

```bash
# 1. 检查是否在同一网络
docker network inspect 容器所在网络

# 2. 检查 DNS 解析
docker compose exec 容器名 nslookup 服务名
docker compose exec 容器名 ping 服务名

# 3. 检查 /etc/hosts
docker compose exec 容器名 cat /etc/hosts
```

### 14.2 端口映射不生效

```bash
# 1. 检查端口是否冲突
sudo lsof -i :8080

# 2. 检查容器端口映射
docker port 容器名

# 3. 检查防火墙
sudo iptables -L -n
```

### 14.3 网络性能问题

```yaml
# 使用 host 网络提升性能
services:
  nginx:
    image: nginx:1.25
    network_mode: host
```

或者优化 MTU：

```yaml
networks:
  backend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.mtu: 1450
```

---

## 15. 存储性能优化

### 15.1 使用内存卷提升性能

```yaml
services:
  redis:
    image: redis:8.0-alpine
    # 使用内存卷
    tmpfs:
      - /data
    command: redis-server --appendonly yes
```

### 15.2 选择合适的存储驱动

```bash
# 查看当前存储驱动
docker info | grep "Storage Driver"

# 推荐生产环境使用 overlay2
```

### 15.3 卷的 I/O 优化

```yaml
services:
  mysql:
    image: mysql:8.4
    volumes:
      # 使用绑定挂载指定特定存储
      - /fast-storage/mysql-data:/var/lib/mysql
    # 禁用日志（生产环境谨慎）
    # command: --skip-log-bin
```

---

## 16. 参考资料

- [Docker 网络文档](https://docs.docker.com/compose/networking/)
- [Docker 卷文档](https://docs.docker.com/compose/volumes/)
- [Docker 网络驱动](https://docs.docker.com/network/)

