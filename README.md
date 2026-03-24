# ♾️ Infinite Learning

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Tech Stack: Docker](https://img.shields.io/badge/Tech-Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![Status: Active](https://img.shields.io/badge/Status-Active-2ea44f)](https://github.com)
[![Update: Continuous](https://img.shields.io/badge/Update-Continuous-ff69b4)](https://github.com)

> 学无止境，行则将至。这里是我的个人技术学习笔记与知识沉淀仓库，记录着在技术海洋中探索的每一步。

---

## 📖 简介

本仓库旨在系统性地记录我学习各种技术栈的过程、实践经验和最佳实践。从基础概念到进阶用法，从理论笔记到实战代码，力求构建一份清晰、实用、可复用的知识库。

---

## 📚 学习目录

> 知识体系持续构建中，点击链接即可开始学习。

### 🐳 Docker

容器化技术的核心实践，分为两个子模块：

#### 1. Dockerfile 学习笔记

| 文档 | 描述 |
|:---|:---|
| [📝 01. Docker 概念与安装](docker/dockerfile/01-Docker-概念与安装.md) | Docker 基础概念、安装与配置 |
| [📝 02. Dockerfile 基础](docker/dockerfile/02-Dockerfile基础.md) | Dockerfile 基本语法与使用 |
| [📝 03. Dockerfile 指令详解 (实践版)](docker/dockerfile/03-Dockerfile指令详解-实践版.md) | 每条指令的详细用法与实践 |
| [📝 04. 构建上下文与多阶段构建 (实践版)](docker/dockerfile/04-构建上下文与多阶段构建-实践版.md) | 高级构建技巧与优化 |
| [📖 Dockerfile 完整手册](docker/dockerfile/Dockerfile-完整手册.md) | 全面参考手册 |
| [🗺️ Dockerfile 学习指引](docker/dockerfile/Dockerfile-学习指引.md) | 学习路线图与建议 |

#### 2. Docker Compose 学习笔记

| 文档 | 描述 |
|:---|:---|
| [📝 01. Docker Compose 概念与安装](docker/docker-compose/01-DockerCompose-概念与安装.md) | Compose 简介与安装 |
| [📝 02. Docker Compose YAML 编写详解](docker/docker-compose/02-DockerCompose-YAML编写详解.md) | YAML 配置全面解析 |
| [📝 03. Docker Compose 网络与存储](docker/docker-compose/03-DockerCompose-网络与存储.md) | 网络与数据卷配置 |
| [📝 04. Docker Compose 实战与最佳实践](docker/docker-compose/04-DockerCompose-实战与最佳实践.md) | 真实场景应用与优化 |
| [🗺️ Docker Compose 学习路径](docker/docker-compose/docker-compose-learning-path.md) | 完整学习路线 |

---

### 🚧 计划添加的技术领域

| 技术领域 | 状态 | 规划内容 |
|:---|:---:|:---|
| **☸️ Kubernetes** | 🚧 持续更新中 | 核心概念、Pod、Service、部署与运维 |
| **🐧 Linux** | 📅 规划中 | 基础命令、文件系统、网络配置、性能调优 |
| **🌐 网络** | 📅 规划中 | OSI 模型、TCP/IP、HTTP、负载均衡 |
| **🗄️ 数据库** | 📅 规划中 | MySQL/PostgreSQL 基础、索引优化、事务与锁 |
| **☁️ 云原生** | 📅 规划中 | 服务网格、Serverless、可观测性 |
| **🛠️ Git** | 📅 规划中 | 工作流、高级操作、问题排查 |

---

## 📖 学习指引

为了获得最佳的学习体验，建议按照以下路径进行：

### 建议学习顺序

1. **夯实基础**：从 **Docker** 开始，掌握容器化技术的核心概念和使用方法。
2. **分而治之**：
   - 先学习 **Dockerfile** 模块，了解如何构建自定义镜像
   - 再学习 **Docker Compose** 模块，学习如何编排和管理多容器应用
3. **迈向进阶**：在掌握 Docker 之后，继续学习后续添加的 **Kubernetes**、**Linux** 等进阶内容。

### 每个模块的学习目标

| 模块 | 学习目标 |
|:---|:---|
| **Dockerfile** | 能够编写生产级别的 Dockerfile，理解镜像分层、构建上下文和多阶段构建 |
| **Docker Compose** | 能够使用 YAML 文件定义复杂应用栈，熟练进行部署、调试和管理 |
| **Kubernetes** | 掌握容器编排核心概念，能够部署和管理分布式应用 |
| **Linux** | 熟悉常用命令和系统管理，具备问题排查能力 |

---

## 🛠️ 使用说明

### 克隆仓库

```bash
git clone https://github.com/[YourUsername]/infinite-learning.git
