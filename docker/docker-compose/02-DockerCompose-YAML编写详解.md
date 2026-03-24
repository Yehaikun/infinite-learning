# Docker Compose YAML 编写详解

**Metadata**
- Date: 2026-03-24
- Topic: Docker Compose
- Tags: #docker #docker-compose #yaml #配置文件 #学习路径02

---

## 1. YAML 语法基础

### 1.1 为什么用 YAML

YAML（YAML Ain't Markup Language）是一种人类友好的数据序列化格式。Docker Compose 选择 YAML 是因为：

1. **可读性强**：像写英文文档一样
2. **结构清晰**：通过缩进表示层级
3. **支持复杂类型**：数组、对象、嵌套都能表达

### 1.2 基本语法规则

#### 1.2.1 缩进规则

YAML 使用**空格**缩进，不能用 Tab：

```yaml
# ✅ 正确：使用空格缩进
services:
  nginx:
    image: nginx:1.25

# ❌ 错误：使用 Tab 缩进
services:
	nginx:
		image: nginx:1.25
```

缩进的空格数没有限制，但**同一层级必须一致**：

```yaml
# ✅ 一致缩进
services:
  nginx:
    image: nginx:1.25
    ports:
      - "8080:80"

# ❌ 缩进不一致（混用2空格和4空格）
services:
  nginx:
      image: nginx:1.25
    ports:
      - "8080:80"
```

**实战经验**：建议统一使用 **2个空格** 缩进，这是 Docker Compose 的惯例。

#### 1.2.2 键值对

```yaml
# 基本格式：key: value
image: nginx:1.25
port: 8080
enabled: true

# 字符串可以不用引号
name: nginx
name: "nginx"     # 效果相同
name: 'nginx'     # 效果相同

# 但特殊字符需要引号
message: "hello: world"   # 包含冒号，必须引号
message: 'hello "world"'  # 包含引号，外层用单引号

# 布尔值（true/false, yes/no, on/off 都可以）
debug: true
debug: yes
debug: on
```

#### 1.2.3 数组/列表

```yaml
# 行内写法
ports: ["8080:80", "8081:8081"]

# 多行写法（更常用）
ports:
  - "8080:80"
  - "8081:8081"
```

#### 1.2.4 对象/字典

```yaml
# 行内写法
environment: {MYSQL_ROOT_PASSWORD: root123}

# 多行写法（更常用）
environment:
  MYSQL_ROOT_PASSWORD: root123
  MYSQL_DATABASE: lyjew
```

#### 1.2.5 嵌套结构

```yaml
services:
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: root123
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
```

### 1.3 YAML 特殊语法

#### 1.3.1 锚点和引用

可以定义重复内容，然后用引用：

```yaml
# 定义基础配置
.base: &base
  image: nginx:1.25
  restart: always
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"

# 引用基础配置
services:
  web:
    <<: *base
    ports:
      - "8080:80"

  admin:
    <<: *base
    ports:
      - "8081:80"
```

这会展开为：

```yaml
services:
  web:
    image: nginx:1.25
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    ports:
      - "8080:80"

  admin:
    image: nginx:1.25
    restart: always
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
    ports:
      - "8081:80"
```

#### 1.3.2 多文档支持

一个 YAML 文件可以包含多个文档，用 `---` 分隔：

```yaml
# 文档1：开发环境配置
version: "3.8"
services:
  mysql:
    image: mysql:8.4

---
# 文档2：生产环境配置
version: "3.8"
services:
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: ${PROD_PASSWORD}
```

使用：

```bash
# 使用第一个文档
docker compose up -d

# 指定使用第二个文档
docker compose -f file.yml down
```

---

## 2. 核心配置项详解

### 2.1 version（版本声明）

```yaml
version: "3.8"
```

虽然 Docker Compose 现在推荐不写 version（V2 默认是 3.9），但为了兼容性，建议还是写上。

### 2.2 services（服务定义）

这是最核心的配置项，每个服务对应一个容器：

```yaml
services:
  mysql:        # 服务名（必须）
    image: mysql:8.4

  redis:
    image: redis:8.0

  auth-server:
    build: ./auth-server
```

### 2.3 image（指定镜像）

```yaml
# 使用官方镜像
services:
  mysql:
    image: mysql:8.4

  redis:
    image: redis:8.0-alpine  # Alpine 精简版，镜像更小

  nginx:
    image: nginx:latest

# 使用私有仓库镜像
services:
  myapp:
    image: registry.example.com/myapp:v1.0

# 不指定标签，默认使用 latest
services:
  nginx:
    image: nginx  # 等同于 nginx:latest
```

### 2.4 build（构建镜像）

```yaml
# 简单写法
services:
  auth-server:
    build: ./auth-server

# 详细写法
services:
  auth-server:
    build:
      context: ./auth-server     # 构建上下文（Dockerfile 所在目录）
      dockerfile: Dockerfile     # 指定 Dockerfile 文件名
      args:                      # 构建参数
        JAVA_VERSION: 21
        BUILD_ENV: production
      labels:                    # 镜像标签
        maintainer: "developer@hubspace.com"
      network: build             # 构建时的网络
```

**构建上下文（context）的概念**：

```
项目目录/
├── docker-compose.yml
└── auth-server/
    ├── Dockerfile
    ├── src/
    │   └── ...
    └── target/
        └── auth-server.jar
```

```yaml
services:
  auth-server:
    build:
      context: ./auth-server  # 相对路径，相对于 docker-compose.yml
      dockerfile: Dockerfile
```

- `context`：指定"哪里是构建的根目录"，COPY 和 ADD 命令只能复制这个目录下的文件
- `dockerfile`：指定使用哪个 Dockerfile

### 2.5 ports（端口映射）

```yaml
# 简单写法：宿主机端口:容器端口
services:
  nginx:
    ports:
      - "8080:80"

# 多个端口
services:
  nginx:
    ports:
      - "8080:80"
      - "8443:443"

# 指定 TCP/UDP（默认 TCP）
services:
  nginx:
    ports:
      - "8080:80"
      - "53:53/udp"       # UDP 端口
      - "3000-3005:3000-3005"  # 端口范围

# 详细写法（可以指定宿主机 IP）
services:
  nginx:
    ports:
      - "127.0.0.1:8080:80"     # 只绑定本机
      - "0.0.0.0:8081:80"       # 绑定所有网卡
      - "192.168.1.100:8082:80" # 绑定指定IP
```

**端口映射格式**：`[HOST:]CONTAINER[/PROTOCOL]`

- `8080:80` - 宿主机8080映射到容器80
- `:80` - 随机宿主机端口映射到容器80
- `8080:80/udp` - UDP 协议

### 2.6 environment（环境变量）

```yaml
# 简单写法
services:
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: lyjew

# 详细写法（可以指定具体值或从主机读取）
services:
  mysql:
    environment:
      - MYSQL_ROOT_PASSWORD=root123
      - MYSQL_DATABASE=lyjew
      - TZ=Asia/Shanghai

# 使用 .env 文件中的变量
# .env 文件内容：
# MYSQL_ROOT_PASSWORD=secret123

services:
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
```

### 2.7 volumes（卷挂载）

```yaml
# 命名卷（Docker 管理）
services:
  mysql:
    volumes:
      - mysql-data:/var/lib/mysql

# 绑定挂载（宿主机目录）
services:
  nginx:
    volumes:
      - ./html:/usr/share/nginx/html     # 相对路径（相对于 docker-compose.yml）
      - /opt/data:/data                  # 绝对路径

# 只读挂载
services:
  nginx:
    volumes:
      - ./config:/etc/nginx:ro           # ro = read only

# 多容器共享卷
services:
  app1:
    volumes:
      - shared-data:/data
  app2:
    volumes:
      - shared-data:/data

volumes:
  shared-data:
```

**卷的类型对比**：

| 类型 | 语法 | 生命周期 | 用途 |
|------|------|---------|------|
| 命名卷 | `mysql-data:/var/lib/mysql` | Docker 管理 | 数据库、日志 |
| 绑定挂载 | `./html:/usr/share/nginx/html` | 手动管理 | 配置、代码 |
| tmpfs | `tmpfs:/run` | 内存中 | 敏感数据、临时文件 |

### 2.8 networks（网络配置）

```yaml
services:
  web:
    networks:
      - frontend
      - backend

  api:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

**网络模式**：

| 模式 | 说明 | 示例 |
|------|------|------|
| bridge | 桥接网络（默认） | 容器间通信 |
| host | 主机网络 | 性能敏感场景 |
| none | 无网络 | 隔离容器 |
| service | 服务名作为主机名 | 同一服务内 |

### 2.9 depends_on（依赖关系）

```yaml
services:
  web:
    depends_on:
      - db
      - redis

  db:
    image: mysql:8.4

  redis:
    image: redis:8.0
```

**注意**：V3 版本的 `depends_on` 不保证启动顺序，只是声明依赖关系。如果需要等待依赖服务就绪，使用 V2.4 版本：

```yaml
version: "2.4"
services:
  web:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

  db:
    image: mysql:8.4
    healthcheck:
      test: ["CMD", "mysqladmin", "ping"]

  redis:
    image: redis:8.0
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
```

### 2.10 restart（重启策略）

```yaml
services:
  mysql:
    restart: always

  redis:
    restart: on-failure:3  # 失败时最多重试3次

  app:
    restart: unless-stopped  # 除非手动停止，否则一直重启
```

| 值 | 说明 |
|----|------|
| `no` | 不重启（默认） |
| `always` | 总是重启 |
| `on-failure` | 非0退出码时重启 |
| `on-failure:N` | 非0退出码时最多重启N次 |
| `unless-stopped` | 除非手动停止，否则重启 |

### 2.11 healthcheck（健康检查）

```yaml
services:
  mysql:
    image: mysql:8.4
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s      # 检查间隔
      timeout: 5s       # 单次检查超时
      retries: 5        # 重试次数
      start_period: 30s # 容器启动后多久开始检查
```

### 2.12 command（覆盖默认命令）

```yaml
# 覆盖默认命令
services:
  redis:
    command: redis-server --requirepass redis123

# 使用数组形式（推荐）
services:
  redis:
    command:
      - redis-server
      - --requirepass
      - redis123

# 覆盖 ENTRYPOINT
services:
  myapp:
    entrypoint: ["python", "app.py"]
    command: ["--debug"]
```

### 2.13 working_dir（工作目录）

```yaml
services:
  app:
    image: maven:3.9
    working_dir: /app
    volumes:
      - ./:/app
    command: mvn clean package
```

### 2.14 user（运行用户）

```yaml
services:
  app:
    image: nginx:1.25
    user: nginx  # 使用 nginx 用户运行
```

---

## 3. 高级配置

### 3.1 profiles（配置文件）

```yaml
services:
  web:
    image: nginx:1.25
    ports:
      - "8080:80"

  monitoring:
    image: prom/prometheus
    profiles: ["monitoring"]  # 只在指定 profile 时启动

  debugger:
    image: myapp/debugger
    profiles: ["debug"]
```

使用：

```bash
# 正常启动（只有 web）
docker compose up -d

# 包含 monitoring
docker compose --profile monitoring up -d

# 包含多个 profile
docker compose --profile monitoring --profile debug up -d
```

### 3.2 secrets（敏感信息）

```yaml
services:
  mysql:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_root_password
    secrets:
      - mysql_root_password

secrets:
  mysql_root_password:
    file: ./mysql_password.txt
```

或者使用 Docker Swarm 模式：

```yaml
services:
  mysql:
    image: mysql:8.4
    secrets:
      - db_password

secrets:
  db_password:
    external: true  # 来自 Docker Swarm
```

### 3.3 configs（配置）

```yaml
services:
  nginx:
    image: nginx:1.25
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf

configs:
  nginx_config:
    file: ./nginx.conf
```

### 3.4 deploy（部署配置）

```yaml
services:
  web:
    image: nginx:1.25
    deploy:
      replicas: 3          # 副本数
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
```

---

## 4. 完整示例：电商系统

### 4.1 场景说明

这是一个典型的电商系统，包含：
- MySQL：主数据库
- Redis：缓存、Session存储
- RabbitMQ：消息队列
- order-service：订单服务
- product-service：商品服务
- gateway：API网关

### 4.2 docker-compose.yml

```yaml
version: "3.8"

services:
  # ==================== 基础设施 ====================
  
  # MySQL 主数据库
  mysql:
    image: mysql:8.4
    container_name: ecommerce-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-root123}
      MYSQL_DATABASE: ecommerce
      MYSQL_USER: ecommerce
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-ecommerce123}
      TZ: Asia/Shanghai
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init-sql:/docker-entrypoint-initdb.d
      - ./conf/mysql.cnf:/etc/mysql/conf.d/my.cnf:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default-authentication-plugin=mysql_native_password

  # Redis 缓存
  redis:
    image: redis:8.0-alpine
    container_name: ecommerce-redis
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD:-redis123} --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # RabbitMQ 消息队列
  rabbitmq:
    image: rabbitmq:3.13-management-alpine
    container_name: ecommerce-rabbitmq
    restart: always
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER:-ecommerce}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD:-ecommerce123}
      RABBITMQ_DEFAULT_VHOST: /
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

  # ==================== 应用服务 ====================

  # 订单服务
  order-service:
    build:
      context: ./order-service
      dockerfile: Dockerfile
    container_name: ecommerce-order-service
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/ecommerce?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
      SPRING_DATASOURCE_USERNAME: ecommerce
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD:-ecommerce123}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD:-redis123}
      SPRING_RABBITMQ_HOST: rabbitmq
      SPRING_RABBITMQ_USERNAME: ${RABBITMQ_USER:-ecommerce}
      SPRING_RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD:-ecommerce123}
      SERVER_PORT: 8082
    ports:
      - "8082:8082"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8082/actuator/health"]
      interval: 15s
      timeout: 10s
      retries: 5
      start_period: 60s

  # 商品服务
  product-service:
    build:
      context: ./product-service
      dockerfile: Dockerfile
    container_name: ecommerce-product-service
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: docker
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/ecommerce?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
      SPRING_DATASOURCE_USERNAME: ecommerce
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD:-ecommerce123}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD:-redis123}
      SERVER_PORT: 8083
    ports:
      - "8083:8083"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8083/actuator/health"]
      interval: 15s
      timeout: 10s
      retries: 5
      start_period: 60s

  # API 网关
  gateway:
    image: nginx:1.25-alpine
    container_name: ecommerce-gateway
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./conf/conf.d:/etc/nginx/conf.d:ro
      - ./logs/nginx:/var/log/nginx
    depends_on:
      - order-service
      - product-service
    networks:
      - backend
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 10s
      retries: 3

# ==================== 网络定义 ====================
networks:
  backend:
    driver: bridge
    name: ecommerce-backend
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16

# ==================== 卷定义 ====================
volumes:
  mysql-data:
    name: ecommerce-mysql-data
  redis-data:
    name: ecommerce-redis-data
  rabbitmq-data:
    name: ecommerce-rabbitmq-data
```

### 4.3 .env 文件

```bash
# .env
MYSQL_ROOT_PASSWORD=root123
MYSQL_PASSWORD=ecommerce123
REDIS_PASSWORD=redis123
RABBITMQ_USER=ecommerce
RABBITMQ_PASSWORD=ecommerce123
```

---

## 5. YAML 编写最佳实践

### 5.1 缩进和格式

```yaml
# ✅ 推荐：2空格缩进，键值对齐
services:
  nginx:
    image: nginx:1.25
    ports:
      - "8080:80"

# ❌ 不推荐：4空格，格式混乱
services:
    nginx:
        image: nginx:1.25
        ports:
        - "8080:80"
```

### 5.2 使用锚点减少重复

```yaml
# 定义通用配置
x-common-env: &common-env
  TZ: Asia/Shanghai
  JAVA_OPTS: "-Xmx512m -Xms256m"

services:
  app1:
    image: myapp:v1
    environment:
      <<: *common-env
      APP_NAME: app1

  app2:
    image: myapp:v1
    environment:
      <<: *common-env
      APP_NAME: app2
```

### 5.3 使用 .env 文件管理敏感信息

```yaml
# docker-compose.yml
services:
  mysql:
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}

# .env
MYSQL_ROOT_PASSWORD=secret123

# 不提交 .env 到版本控制
echo ".env" >> .gitignore
```

### 5.4 版本控制最佳实践

```bash
# .gitignore
.env
*.log
*.pid
*.sock
/tmp/
/data/
```

---

## 6. 本章总结

### 6.1 核心要点

1. **YAML 语法**：缩进、键值对、数组、对象、锚点
2. **核心配置**：image、build、ports、environment、volumes、networks
3. **依赖管理**：depends_on 和 healthcheck
4. **高级特性**：profiles、secrets、configs、deploy

### 6.2 常用配置快速参考

```yaml
services:
  服务名:
    image: 镜像名:标签
    build: ./构建目录
    ports:
      - "宿主机端口:容器端口"
    environment:
      - 变量名=值
    volumes:
      - 宿主机路径:容器路径
    networks:
      - 网络名
    depends_on:
      - 依赖服务
    restart: always
    healthcheck:
      test: ["CMD", "命令"]
      interval: 10s
```

### 6.3 下篇预告

下篇我们将讲解 **Docker Compose 网络与存储**，包括：
- 网络模式详解
- 网络隔离与通信
- 卷的类型与使用
- 数据持久化最佳实践

---

## 7. 课后问题

1. **问题**：YAML 中缩进可以用 Tab 吗？为什么？
   **提示**：考虑 YAML 规范和 Docker Compose 的实际行为。

2. **问题**：如何在不同环境（开发、测试、生产）之间切换配置？
   **提示**：考虑 profiles 和多个 docker-compose 文件的使用。

3. **问题**：服务间的依赖关系用 depends_on 声明就够了，还需要 healthcheck 吗？
   **提示**：考虑服务启动和完全就绪的区别。

---

## 8. 附录：常见问题

### Q1: 端口冲突怎么办？

检查端口占用：

```bash
# 查看端口占用
sudo lsof -i :3306

# 或
sudo netstat -tlnp | grep 3306
```

解决方案：
- 修改 docker-compose.yml 中的端口映射
- 停止占用端口的其他服务

### Q2: 容器内无法访问宿主机服务？

```yaml
# 方案1：使用 host.docker.internal（Docker Desktop）
extra_hosts:
  - "host.docker.internal:host-gateway"

# 方案2：使用宿主机网络
network_mode: "host"

# 方案3：使用宿主机 IP（Linux）
# 替换 localhost 为宿主机内网 IP
```

### Q3: 如何查看容器 IP？

```bash
# 方法1：inspect
docker inspect 容器名 --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'

# 方法2：在容器内查看
docker compose exec 服务名 cat /etc/hosts
```

### Q4: YAML 文件有语法错误怎么办？

使用在线验证工具：
- https://www.yamllint.com/
- https://codebeautify.org/yaml-validator

或者使用 docker compose config 验证：

```bash
docker compose config  # 验证并输出合并后的配置
```

### Q5: 如何覆盖镜像的默认启动命令？

```yaml
services:
  mysql:
    image: mysql:8.4
    # 方式1：直接覆盖
    command: --default-authentication-plugin=mysql_native_password

  redis:
    image: redis:8.0
    # 方式2：数组形式（推荐）
    command:
      - redis-server
      - --requirepass
      - redis123
      - --appendonly
      - yes

  # 方式3：同时覆盖 ENTRYPOINT 和 COMMAND
  app:
    image: myapp:latest
    entrypoint: ["python"]
    command: ["app.py", "--debug"]
```

### Q6: 如何让多个服务使用同一个配置？

```yaml
# 使用 YAML 锚点
x-mysql-config: &mysql-config
  image: mysql:8.4
  environment:
    MYSQL_ROOT_PASSWORD: root123
  volumes:
    - mysql-data:/var/lib/mysql

services:
  db1:
    <<: *mysql-config
    container_name: db1

  db2:
    <<: *mysql-config
    container_name: db2
```

### Q7: 环境变量中没有设置值会怎样？

```yaml
environment:
  - VAR1           # 从当前环境继承
  - VAR2=value     # 显式设置
  - VAR3           # 未设置时为空
```

如果环境变量未设置，在容器中会是空字符串。

### Q8: 为什么 bind mount 的文件权限是 root？

这是 Docker 的安全机制。可以这样解决：

```yaml
services:
  app:
    image: alpine
    volumes:
      - ./data:/data
    # 使用 user 指定用户
    user: "1000:1000"  # 指定 UID:GID

# 或者在启动后修改权限
# docker compose exec app chown -R 1000:1000 /data
```

---

## 9. 实战练习：编写生产级配置

### 9.1 练习目标

编写一个包含以下要求的 docker-compose.yml：
1. MySQL + Redis + Spring Boot 应用
2. 使用 .env 管理敏感信息
3. 正确的健康检查
4. 数据持久化
5. 适当的日志配置

### 9.2 步骤

**步骤 1：创建项目目录**

```bash
mkdir -p ~/docker-projects/prod-config
cd ~/docker-projects/prod-config
```

**步骤 2：创建 .env 文件**

```bash
# .env
MYSQL_ROOT_PASSWORD=ComplexPassword123!
MYSQL_DATABASE=hubspace
REDIS_PASSWORD=RedisPass456!
TZ=Asia/Shanghai
```

**步骤 3：创建 docker-compose.yml**

```yaml
version: "3.8"

services:
  mysql:
    image: mysql:8.4
    container_name: hubspace-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      TZ: ${TZ:-Asia/Shanghai}
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init:/docker-entrypoint-initdb.d
    networks:
      - backend
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci

  redis:
    image: redis:8.0-alpine
    container_name: hubspace-redis
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes --loglevel notice
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: hubspace-app
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: ${SPRING_PROFILES_ACTIVE:-docker}
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/${MYSQL_DATABASE}?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD}
    ports:
      - "8081:8081"
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/actuator/health"]
      interval: 15s
      timeout: 10s
      retries: 5
      start_period: 60s
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"

networks:
  backend:
    driver: bridge

volumes:
  mysql-data:
  redis-data:
```

### 9.3 验证

```bash
# 验证配置
docker compose config

# 启动服务
docker compose up -d

# 查看状态
docker compose ps
```

---

## 10. 参考资料

- [Docker Compose 文件参考](https://docs.docker.com/compose/compose-file/)
- [YAML 规范](https://yaml.org/spec/1.2.2/)
- [YAML lint](https://www.yamllint.com/)
