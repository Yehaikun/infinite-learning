# infinite-learning

<div align="center">

[![Typing SVG](https://readme-typing-svg.herokuapp.com?font=Fira+Code&weight=600&size=32&pause=1000&center=true&vCenter=true&width=500&height=70&lines=登量不止，学习是一生的事动&color=gradient)](https://git.io/typing-svg)


[![GitHub stars](https://img.shields.io/github/stars/Yehaikun/infinite-learning)](https://github.com/Yehaikun/infinite-learning/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/Yehaikun/infinite-learning)](https://github.com/Yehaikun/infinite-learning/network)
[![License](https://img.shields.io/github/license/Yehaikun/infinite-learning)](https://github.com/Yehaikun/infinite-learning/blob/main/LICENSE)
[![更新](https://img.shields.io/badge/%E6%9B%B4%E6%96%B0-%E5%8D%88%E5%8A%A8%E6%9C%89%E6%96%B0-orange)](https://github.com/Yehaikun/infinite-learning/commits/main)

</div>

---

> 📝 个人技术学习笔记与知识沉淀仓库，记录各种技术栈的学习过程、实践经验和最佳实践。

---

## 📚 学习目录

### 🐳 容器化与虚拟化

| 分类 | 描述 | 状态 |
|------|------|------|
| [Docker](./docker/) | 容器引擎 | ✅ 已完成 |
| [Docker Compose](./docker/docker-compose/) | 多容器编排 | ✅ 已完成 |
| [Docker Swarm](./docker/swarm/) | 集群编排 | ✅ 已完成 |

#### Docker 学习笔记

```
docker/
├── dockerfile/              # Dockerfile 系列
│   ├── 01-Docker-概念与安装.md
│   ├── 02-Dockerfile基础.md
│   ├── 03-Dockerfile指令详解-实践版.md
│   ├── 04-构建上下文与多阶段构建-实践版.md
│   ├── Dockerfile-完整手册.md
│   └── Dockerfile-学习指引.md
│
└── docker-compose/          # Docker Compose 系列
    ├── 01-DockerCompose-概念与安装.md
    ├── 02-DockerCompose-YAML编写详解.md
    ├── 03-DockerCompose-网络与存储.md
    ├── 04-DockerCompose-实战与最佳实践.md
    └── docker-compose-learning-path.md
```

---

### ☁️ 云原生与微服务

| 分类 | 描述 | 状态 |
|------|------|------|
| [Kubernetes](./k8s/) | 容器编排 | 🚧 规划中 |
| [Spring Cloud](./spring-cloud/) | 微服务框架 | 🚧 规划中 |
| [Nacos](./nacos/) | 注册/配置中心 | 🚧 规划中 |

---

### 🛠️ 基础工具

| 分类 | 描述 | 状态 |
|------|------|------|
| [Git & GitHub](./tools/github/) | 版本控制 | 🚧 规划中 |
| [Linux](./linux/) | 操作系统 | 🚧 规划中 |
| [Nginx](./nginx/) | Web 服务器 | 🚧 规划中 |

---

### 💾 数据库与缓存

| 分类 | 描述 | 状态 |
|------|------|------|
| [MySQL](./database/mysql/) | 关系型数据库 | 🚧 规划中 |
| [Redis](./database/redis/) | 缓存数据库 | 🚧 规划中 |
| [MongoDB](./database/mongodb/) | 文档数据库 | 🚧 规划中 |

---

### 🌐 网络与安全

| 分类 | 描述 | 状态 |
|------|------|------|
| [计算机网络](./network/) | 网络基础 | 🚧 规划中 |
| [HTTP/HTTPS](./network/http/) | 协议 | 🚧 规划中 |
| [TLS/SSL](./network/tls/) | 安全传输 | 🚧 规划中 |

---

## 🚀 学习指引

### 推荐的入门路径

```
Docker 基础 ────→ Docker Compose ────→ Docker Swarm
     │                 │                  │
     ▼                 ▼                  ▼
  镜像构建         多容器管理           集群部署
```

### 各模块学习目标

| 模块 | 目标 |
|------|------|
| Docker | 掌握容器构建与运行 |
| Docker Compose | 熟练编排多容器应用 |
| Docker Swarm | 理解集群部署与运维 |

---

## 🛠️ 使用说明

### 克隆仓库

```bash
# HTTPS
git clone https://github.com/Yehaikun/infinite-learning.git

# SSH
git clone git@github.com:Yehaikun/infinite-learning.git
```

### 浏览笔记

推荐使用 VS Code + Markdown Preview 插件，或使用 Typora、Notion 等工具阅读。

---

## 📋 目录结构

```
infinite-learning/
│
├── docker/                    # Docker 系列
│   ├── dockerfile/
│   ├── docker-compose/
│   └── swarm/
│
├── k8s/                      # Kubernetes（规划中）
├── spring-cloud/             # Spring Cloud（规划中）
├── tools/                    # 工具教程（规划中）
│   └── github/
├── linux/                    # Linux（规划中）
├── database/                 # 数据库（规划中）
├── network/                  # 网络（规划中）
│
├── papers/                   # 论文笔记
├── notes/                    # 学习笔记
│   └── tutorials/
│
├── templates/                # 模板
├── backups/                  # 备份
│
├── LICENSE                   # 许可证
└── README.md                 # 本文件
```

---

## 🤝 贡献指南

这是**个人学习笔记仓库**，主要用于自我知识沉淀。

- ❌ 不对外开放贡献（PR）
- ✅ 欢迎 Star ⭐ 和 Fork
- ✅ 欢迎 Issue 交流学习心得
- ✅ 如果有错误，欢迎指出

> 💡 如果你对我正在学习的技术感兴趣，可以一起交流学习！

---

## 📅 更新日志

- **2024-03** - Docker 系列笔记整理完成
- 持续更新中...

---

## ⭐ 感谢支持

如果你觉得这个仓库对你有帮助，欢迎点个 ⭐star⭐！

<div align="center">

[![Stargazers over time](https://starchart.cc/Yehaikun/infinite-learning.svg)](https://github.com/Yehaikun/infinite-learning/stargazers)

---

Made with ❤️ by [Yehaikun](https://github.com/Yehaikun)

</div>
