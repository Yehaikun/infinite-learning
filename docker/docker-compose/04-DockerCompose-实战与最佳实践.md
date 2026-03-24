# Docker Compose 实战与最佳实践

**Metadata**
- Date: 2026-03-24
- Topic: Docker Compose
- Tags: #docker #docker-compose #实战 #最佳实践 #学习路径04

---

## 1. 完整实战：搭建 Spring Boot 微服务开发环境

### 1.1 场景说明

我们要搭建一个完整的微服务开发环境，包含：

- **MySQL 8.4**：主数据库
- **Redis 8.0**：缓存和 Session
- **RabbitMQ 3.13**：消息队列
- **Nacos 3.1.1**：注册中心和配置中心
- **Sentinel 1.8.8**：限流熔断
- **auth-server**：认证服务（Spring Boot）
- **register-server**：注册服务（Spring Boot）

### 1.2 项目结构

```
hubspace-backend/
├── docker-compose.yml
├── .env
├── init-db/
│   ├── 01-init.sql
│   └── 02-data.sql
├── conf/
│   ├── mysql/
│   │   └── my.cnf
│   ├── redis/
│   │   └── redis.conf
│   └── nginx/
│       └── nginx.conf
├── auth-server/
│   ├── Dockerfile
│   └── target/auth-server.jar
└── register-server/
    ├── Dockerfile
    └── target/register-server.jar
```

### 1.3 .env 文件

```bash
# 环境变量文件
# 不要提交到版本控制！

# MySQL 配置
MYSQL_ROOT_PASSWORD=RootPassword123!
MYSQL_DATABASE=hubspace
MYSQL_USER=hubspace
MYSQL_PASSWORD=Hubspace123!

# Redis 配置
REDIS_PASSWORD=RedisPass456!

# RabbitMQ 配置
RABBITMQ_USER=hubspace
RABBITMQ_PASSWORD=RabbitmqPass789!

# Nacos 配置
NACOS_USERNAME=nacos
NACOS_PASSWORD=Nacos123!

# 时区
TZ=Asia/Shanghai
```

### 1.4 docker-compose.yml 完整配置

```yaml
version: "3.8"

services:
  # ==================== 基础设施层 ====================

  # MySQL 数据库
  mysql:
    image: mysql:8.4
    container_name: hubspace-mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      TZ: ${TZ}
    ports:
      - "3306:3306"
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init-db:/docker-entrypoint-initdb.d:ro
      - ./conf/mysql/my.cnf:/etc/mysql/conf.d/my.cnf:ro
      - ./logs/mysql:/var/log/mysql
    networks:
      - backend
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-p${MYSQL_ROOT_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --max-connections=500
      - --innodb-buffer-pool-size=256M
      - --default-authentication-plugin=mysql_native_password

  # Redis 缓存
  redis:
    image: redis:8.0-alpine
    container_name: hubspace-redis
    restart: always
    command: >
      redis-server
      --requirepass ${REDIS_PASSWORD}
      --appendonly yes
      --loglevel notice
      --maxmemory 512mb
      --maxmemory-policy allkeys-lru
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
      - ./conf/redis/redis.conf:/usr/local/etc/redis/redis.conf:ro
    networks:
      - backend
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    tmpfs:
      - /tmp

  # RabbitMQ 消息队列
  rabbitmq:
    image: rabbitmq:3.13-management-alpine
    container_name: hubspace-rabbitmq
    restart: always
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
      RABBITMQ_DEFAULT_VHOST: /
      TZ: ${TZ}
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
      - ./logs/rabbitmq:/var/log/rabbitmq
    networks:
      - backend
    healthcheck:
      test: ["CMD", "rabbitmq-diagnostics", "ping"]
      interval: 15s
      timeout: 10s
      retries: 5
      start_period: 60s

  # Nacos 注册中心 + 配置中心
  nacos:
    image: nacos/nacos-server:v3.1.1
    container_name: hubspace-nacos
    restart: always
    environment:
      MODE: standalone
      PREFER_HOST_MODE: hostname
      SPRING_DATASOURCE_PLATFORM: mysql
      MYSQL_SERVICE_HOST: mysql
      MYSQL_SERVICE_PORT: 3306
      MYSQL_SERVICE_DB_NAME: ${MYSQL_DATABASE}
      MYSQL_SERVICE_USER: ${MYSQL_USER}
      MYSQL_SERVICE_PASSWORD: ${MYSQL_PASSWORD}
      NACOS_AUTH_ENABLE: true
      NACOS_AUTH_TOKEN: ${NACOS_AUTH_TOKEN:-SecretKey012345678901234567890123456789012345678901234567890123456789}
      NACOS_AUTH_IDENTITY_KEY: serverIdentity
      NACOS_AUTH_IDENTITY_VALUE: security
      JVM_XMS: 256m
      JVM_XMX: 512m
      JVM_XMN: 256m
      TZ: ${TZ}
    ports:
      - "8848:8848"
      - "9848:9848"
    volumes:
      - nacos-data:/home/nacos/data
      - ./logs/nacos:/home/nacos/logs
    networks:
      - backend
    depends_on:
      mysql:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8848/nacos/v1/console/health/readiness"]
      interval: 15s
      timeout: 10s
      retries: 10
      start_period: 60s

  # Sentinel 限流熔断
  sentinel:
    image: bladex/sentinel-dashboard:1.8.8
    container_name: hubspace-sentinel
    restart: always
    environment:
      JAVA_OPTS: >
        -Dserver.port=8080
        -Dcsp.sentinel.dashboard.server=localhost:8080
        -Dproject.name=sentinel-dashboard
        -Dsentinel.dashboard.auth.username=sentinel
        -Dsentinel.dashboard.auth.password=sentinel
      TZ: ${TZ}
    ports:
      - "8080:8080"
      - "8719:8719"
    volumes:
      - sentinel-data:/root/sentinel-data
      - ./logs/sentinel:/root/logs
    networks:
      - backend
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:8080/"]
      interval: 30s
      timeout: 10s
      retries: 3

  # ==================== 应用服务层 ====================

  # 认证服务
  auth-server:
    build:
      context: ./auth-server
      dockerfile: Dockerfile
      args:
        JAR_FILE: auth-server.jar
    container_name: hubspace-auth-server
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: docker
      # Nacos 配置
      SPRING_CLOUD_NACOS_DISCOVERY_SERVER_ADDR: nacos:8848
      SPRING_CLOUD_NACOS_CONFIG_SERVER_ADDR: nacos:8848
      SPRING_CLOUD_NACOS_USERNAME: ${NACOS_USERNAME}
      SPRING_CLOUD_NACOS_PASSWORD: ${NACOS_PASSWORD}
      # MySQL 配置
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/${MYSQL_DATABASE}?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
      SPRING_DATASOURCE_USERNAME: ${MYSQL_USER}
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD}
      # Redis 配置
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD}
      SPRING_REDIS_DATABASE: 0
      # RabbitMQ 配置
      SPRING_RABBITMQ_HOST: rabbitmq
      SPRING_RABBITMQ_USERNAME: ${RABBITMQ_USER}
      SPRING_RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD}
      # Sentinel 配置
      SPRING_CLOUD_SENTINEL_EAGER: "true"
      SPRING_CLOUD_SENTINEL_TRANSPORT_DASHBOARD: sentinel:8080
      # 应用配置
      SERVER_PORT: 8081
      JWT_SECRET: ${JWT_SECRET:-HubspaceSecretKey123456789012345678901234567890}
      JWT_EXPIRATION: 86400
      TZ: ${TZ}
    ports:
      - "8081:8081"
    volumes:
      - ./logs/auth-server:/var/logs
    networks:
      - backend
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      nacos:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8081/actuator/health"]
      interval: 15s
      timeout: 10s
      retries: 5
      start_period: 90s

  # 注册服务
  register-server:
    build:
      context: ./register-server
      dockerfile: Dockerfile
      args:
        JAR_FILE: register-server.jar
    container_name: hubspace-register-server
    restart: always
    environment:
      SPRING_PROFILES_ACTIVE: docker
      # Nacos 配置
      SPRING_CLOUD_NACOS_DISCOVERY_SERVER_ADDR: nacos:8848
      SPRING_CLOUD_NACOS_CONFIG_SERVER_ADDR: nacos:8848
      SPRING_CLOUD_NACOS_USERNAME: ${NACOS_USERNAME}
      SPRING_CLOUD_NACOS_PASSWORD: ${NACOS_PASSWORD}
      # MySQL 配置
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/${MYSQL_DATABASE}?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=Asia/Shanghai&allowPublicKeyRetrieval=true
      SPRING_DATASOURCE_USERNAME: ${MYSQL_USER}
      SPRING_DATASOURCE_PASSWORD: ${MYSQL_PASSWORD}
      # Redis 配置
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PASSWORD: ${REDIS_PASSWORD}
      SPRING_REDIS_DATABASE: 1
      # RabbitMQ 配置
      SPRING_RABBITMQ_HOST: rabbitmq
      SPRING_RABBITMQ_USERNAME: ${RABBITMQ_USER}
      SPRING_RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD}
      # Sentinel 配置
      SPRING_CLOUD_SENTINEL_EAGER: "true"
      SPRING_CLOUD_SENTINEL_TRANSPORT_DASHBOARD: sentinel:8080
      # 应用配置
      SERVER_PORT: 8082
      TZ: ${TZ}
    ports:
      - "8082:8082"
    volumes:
      - ./logs/register-server:/var/logs
    networks:
      - backend
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
      nacos:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8082/actuator/health"]
      interval: 15s
      timeout: 10s
      retries: 5
      start_period: 90s

  # ==================== 网关层 ====================

  # API 网关 (可选)
  gateway:
    image: nginx:1.25-alpine
    container_name: hubspace-gateway
    restart: always
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./conf/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./conf/nginx/conf.d:/etc/nginx/conf.d:ro
      - ./logs/nginx:/var/log/nginx
      - ./html:/usr/share/nginx/html:ro
    networks:
      - backend
    depends_on:
      - auth-server
      - register-server
    healthcheck:
      test: ["CMD", "nginx", "-t"]
      interval: 30s
      timeout: 10s
      retries: 3

# ==================== 网络定义 ====================
networks:
  backend:
    driver: bridge
    name: hubspace-backend
    ipam:
      driver: default
      config:
        - subnet: 172.25.0.0/16

# ==================== 卷定义 ====================
volumes:
  mysql-data:
    name: hubspace-mysql-data
  redis-data:
    name: hubspace-redis-data
  rabbitmq-data:
    name: hubspace-rabbitmq-data
  nacos-data:
    name: hubspace-nacos-data
  sentinel-data:
    name: hubspace-sentinel-data
```

---

## 2. 多环境配置管理

### 2.1 环境分类

常见的部署环境：

| 环境 | 用途 | 特点 |
|------|------|------|
| dev | 开发调试 | 日志详细、热部署 |
| test | 测试验证 | 接近生产 |
| staging | 预发布 | 与生产一致 |
| prod | 生产运行 | 性能优先、安全 |

### 2.2 多文件方案

```
docker-compose.yml          # 基础配置（公共部分）
docker-compose.dev.yml      # 开发环境
docker-compose.test.yml     # 测试环境
docker-compose.prod.yml    # 生产环境
docker-compose.override.yml # 本地覆盖（不提交）
```

**docker-compose.yml（基础配置）**：

```yaml
version: "3.8"

services:
  mysql:
    image: mysql:8.4
    volumes:
      - mysql-data:/var/lib/mysql

  redis:
    image: redis:8.0-alpine

  app:
    build: ./app
    depends_on:
      - mysql
      - redis

volumes:
  mysql-data:
```

**docker-compose.dev.yml（开发环境）**：

```yaml
version: "3.8"

services:
  app:
    environment:
      SPRING_PROFILES_ACTIVE: dev
      DEBUG: "true"
    ports:
      - "8081:8081"
    volumes:
      - ./app:/app  # 热挂载
```

**docker-compose.prod.yml（生产环境）**：

```yaml
version: "3.8"

services:
  app:
    environment:
      SPRING_PROFILES_ACTIVE: prod
    ports:
      - "8081:8081"  # 只内网暴露
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
```

### 2.3 使用方法

```bash
# 使用开发配置启动
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d

# 使用生产配置启动
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# 查看合并后的配置
docker compose -f docker-compose.yml -f docker-compose.prod.yml config
```

---

## 3. 最佳实践

### 3.1 镜像选择

```yaml
# ✅ 推荐：指定具体版本
image: mysql:8.4
image: redis:8.0-alpine
image: nginx:1.25-alpine

# ❌ 避免：使用 latest
image: mysql:latest
image: redis:latest
```

**为什么？**
- `latest` 标签不稳定，每次拉取可能不同
- 生产环境必须可重现
- Alpine 镜像更小，下载更快

### 3.2 健康检查

```yaml
# ✅ 推荐：所有服务都配置健康检查
services:
  mysql:
    healthcheck:
      test: ["CMD", "mysqladmin", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  redis:
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

# ❌ 避免：不配置健康检查
```

### 3.3 数据持久化

```yaml
# ✅ 推荐：使用命名卷
volumes:
  mysql-data:/var/lib/mysql
  redis-data:/data

volumes:
  mysql-data:
  redis-data:

# ❌ 避免：使用匿名卷（重启后数据丢失）
# ✅ 避免：直接挂载宿主机目录到数据库
```

### 3.4 重启策略

```yaml
# ✅ 推荐：生产环境使用 always
services:
  mysql:
    restart: always
  redis:
    restart: always
  app:
    restart: always

# 开发环境可以用 no 或 on-failure
```

### 3.5 日志配置

```yaml
# ✅ 推荐：配置日志轮转
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"

# ✅ 推荐：生产环境使用日志收集
services:
  app:
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://localhost:514"
        tag: "myapp"
```

### 3.6 安全建议

```yaml
# ✅ 推荐：使用非 root 用户
services:
  app:
    user: "1000:1000"

# ✅ 推荐：限制容器能力
services:
  app:
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE

# ✅ 推荐：只读文件系统
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
```

---

## 4. 常见问题排查

### 4.1 容器启动失败

```bash
# 1. 查看容器日志
docker compose logs 服务名

# 2. 查看容器状态
docker compose ps

# 3. 进入容器调试
docker compose exec 服务名 sh

# 4. 检查配置
docker compose config --services
```

**常见原因**：
- 端口冲突：`lsof -i :端口`
- 依赖未就绪：检查 depends_on 和 healthcheck
- 权限问题：检查卷挂载权限
- 环境变量错误：检查 .env 文件

### 4.2 服务无法通信

```bash
# 1. 检查网络
docker network ls
docker network inspect hubspace-backend

# 2. 测试连通性
docker compose exec app ping mysql

# 3. 检查 DNS
docker compose exec app nslookup mysql

# 4. 检查端口
docker compose exec app telnet mysql 3306
```

### 4.3 数据卷问题

```bash
# 1. 查看卷
docker volume ls

# 2. 检查卷内容
docker run --rm -v hubspace-mysql-data:/data alpine ls -la /data

# 3. 备份卷
docker run --rm -v hubspace-mysql-data:/data -v $(pwd):/backup alpine tar czf /backup/backup.tar.gz -C /data .

# 4. 恢复卷
docker compose down
docker run --rm -v hubspace-mysql-data:/data -v $(pwd):/backup alpine tar xzf /backup/backup.tar.gz -C /data
docker compose up -d
```

### 4.4 性能问题

```bash
# 1. 查看资源使用
docker stats

# 2. 查看容器资源限制
docker inspect 容器名 | grep -A 10 Memory

# 3. 监控磁盘 I/O
docker run --rm -v /var/lib/docker/volumes:/volumes alpine ls -la /volumes
```

---

## 5. CI/CD 集成

### 5.1 GitHub Actions 示例

```yaml
# .github/workflows/docker-compose.yml
name: Deploy with Docker Compose

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push images
        run: |
          docker compose build
          docker compose push

      - name: Deploy to server
        uses: appleboy/ssh-action@v0.1.0
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /path/to/project
            docker compose pull
            docker compose up -d
```

### 5.2 钩子脚本

```bash
# docker-compose.yml 中配置
services:
  app:
    hooks:
      pre-start: ./hooks/pre-start.sh
      post-start: ./hooks/post-start.sh
      pre-stop: ./hooks/pre-stop.sh
      post-stop: ./hooks/post-stop.sh
```

---

## 6. 监控与运维

### 6.1 使用 Docker Stats 监控

```bash
# 实时监控所有容器
docker stats

# 监控指定容器
docker stats hubspace-mysql hubspace-redis

# 监控并显示详细信息
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}\t{{.NetIO}}"
```

### 6.2 使用 cAdvisor 监控

```yaml
# docker-compose.yml
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8082:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    networks:
      - backend
```

### 6.3 日志聚合

```yaml
# 使用 Loki + Grafana
services:
  loki:
    image: grafana/loki:2.8.0
    ports:
      - "3100:3100"
    volumes:
      - ./loki-config.yml:/etc/loki/local-config.yaml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      GF_INSTALL_PLUGINS: grafana-loki-datasource
```

---

## 7. 高级技巧

### 7.1 使用 Init 容器

```yaml
services:
  app:
    image: myapp
    depends_on:
      db:
        condition: service_healthy

  # 初始化容器：等待数据库就绪后执行一次
  db-init:
    image: alpine:latest
    command: ["sh", "-c", "sleep 10 && echo 'DB ready'"]
    depends_on:
      db:
        condition: service_healthy
```

### 7.2 使用长服务名避免冲突

```yaml
services:
  mysql:
    container_name: mycompany-project-mysql
```

### 7.3 条件启动

```yaml
services:
  # 只有在 .env 中设置了 ENABLE_DEBUG=true 时才启动
  debug-tool:
    image: myapp/debug
    profiles:
      - debug
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M
```

### 7.4 健康检查依赖

```yaml
services:
  app:
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy

  # 即使 mysql 启动失败，app 也能启动（只是健康检查失败）
  # 使用 healthcheck 保证依赖服务真正就绪
```

---

## 8. 本章总结

### 8.1 核心要点

1. **完整实战**：从 0 搭建微服务开发环境
2. **多环境配置**：dev/test/staging/prod
3. **最佳实践**：镜像选择、健康检查、安全配置
4. **问题排查**：启动失败、通信问题、性能问题
5. **CI/CD**：GitHub Actions 集成

### 8.2 生产环境检查清单

- [ ] 使用具体版本标签，不使用 latest
- [ ] 所有服务配置健康检查
- [ ] 配置日志轮转
- [ ] 数据使用命名卷持久化
- [ ] 配置适当的重启策略
- [ ] 敏感信息使用 .env 或 secrets
- [ ] 配置资源限制
- [ ] 定期备份数据卷

### 8.3 下篇预告

学完 Docker Compose 后，你可以继续学习：

- **Docker Swarm**：集群编排
- **Kubernetes**：大规模容器编排
- **Helm**：Kubernetes 包管理
- **GitLab CI/CD**：持续集成与部署

---

## 9. 课后问题

1. **问题**：如果 MySQL 容器启动失败，如何快速排查？
   **提示**：从日志、端口、权限、配置等方面考虑。

2. **问题**：生产环境中如何保证数据安全？
   **提示**：考虑备份、加密、访问控制。

3. **问题**：如何实现零停机部署？
   **提示**：考虑滚动更新、健康检查、负载均衡。

---

## 10. 附录：完整命令参考

```bash
# ==================== 基础命令 ====================
docker compose up -d                  # 后台启动
docker compose down                   # 停止并删除
docker compose restart                 # 重启
docker compose pause                   # 暂停
docker compose unpause                 # 恢复

# ==================== 构建命令 ====================
docker compose build                   # 构建镜像
docker compose build --no-cache        # 不使用缓存重建
docker compose up -d --build          # 构建并启动

# ==================== 日志命令 ====================
docker compose logs -f                 # 实时日志
docker compose logs -f 服务名          # 指定服务日志
docker compose logs --tail=100        # 最近100行

# ==================== 调试命令 ====================
docker compose ps                      # 查看状态
docker compose exec 服务名 sh          # 进入容器
docker compose cp 服务名:/path ./local # 复制文件

# ==================== 运维命令 ====================
docker compose config                   # 验证配置
docker compose port 服务名 端口         # 查看端口映射
docker compose top                     # 查看进程
docker stats                           # 资源监控

# ==================== 扩展命令 ====================
docker compose scale 服务名=3          # 扩展副本（V1）
docker compose up -d --scale 服务名=3  # 扩展副本（V2）

# ==================== 清理命令 ====================
docker compose rm                      # 删除已停止的容器
docker compose down -v                 # 删除卷
docker compose down --images            # 删除镜像
docker system prune -a                  # 清理所有未使用资源
```

---

## 11. 参考资料

- [Docker Compose 官方文档](https://docs.docker.com/compose/)
- [Docker Compose 文件参考](https://docs.docker.com/compose/compose-file/)
- [Docker 健康检查](https://docs.docker.com/engine/reference/builder/#healthcheck)
- [Nacos 官方文档](https://nacos.io/zh-cn/docs/what-is-nacos.html)
- [Sentinel 官方文档](https://sentinelguard.io/zh-cn/docs/introduction.html)
