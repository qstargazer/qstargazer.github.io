---
title: QNAP安装blinko
description:
date: 2026-08-04
image: https://image-1258996033.cos.ap-shanghai.myqcloud.com/westlake
tags:
categories:
math: true
license:
hidden: false
comments: true
draft: false
toc: true
build:
  list: always
lastmod: 2026-08-04
---
# QNAP TS-464c2 使用 Docker 部署 Blinko 完整记录

> 在 QNAP NAS 上部署个人知识管理系统 Blinko，实现数据私有化存储。

## 背景

随着 AI 工具的发展，个人知识管理逐渐从传统笔记系统向 AI 增强知识库方向发展。

Blinko 是一个开源的个人知识管理工具，支持：

- 快速记录想法
- Markdown 笔记
- 标签管理
- AI 辅助能力
- 私有化部署

为了保证数据安全和长期可控，我计划将 Blinko 部署在自己的 NAS 上。

最终目标：

1. NAS 本地运行 Blinko
2. 数据持久化保存
3. 手机可以访问
4. 后续通过个人域名 + HTTPS 实现内外网统一访问


---

# 环境说明

## 硬件环境

NAS：

```
型号：
QNAP TS-464c2

CPU：
Intel x86_64

硬盘：
2 × HDD RAID1

SSD：
2 × NVMe RAID1
```

用途：

- Docker 服务运行
- 个人知识库
- 家庭数据存储


---

## 软件环境

NAS 系统：

```
QTS 5.2.9
```

Docker 环境：

```
Container Station

Docker Compose:
v2.29.1-qnap2
```


验证：

```bash
docker compose version
```

输出：

```
Docker Compose version v2.29.1-qnap2
```

说明 Docker 环境正常。


---

## 存储结构确认

通过：

```bash
cat /proc/mdstat
```

确认 RAID 状态：

```
md2 : active raid1
md1 : active raid1
```

说明：

- HDD 使用 RAID1
- NVMe 使用 RAID1

RAID1 可以保证：

- 单块硬盘损坏时数据仍然存在

但：

> RAID1 不是备份，仍然需要额外备份策略。


---

# 前期准备

## 安装 Container Station

打开 QNAP：

```
App Center
    ↓
Container Station
```

安装完成后验证：

```bash
docker ps
```

初始状态：

```
没有运行中的容器
```

说明 Docker 服务正常。


---

# Entware 安装过程（可选）

由于计划长期使用 NAS，同时希望具备一些 Linux 工具环境，因此安装 Entware。


## 初次安装问题

第一次安装 Entware 后执行：

```bash
opkg install zsh git tmux
```

出现：

```
verify_pkg_installable:
Only have 6000kb available on filesystem /opt
```

原因：

QNAP 默认：

```
/opt
```

位于系统分区。


查看：

```bash
df -h /
```

发现：

```
系统空间非常有限
```


---

## 问题分析

QNAP 系统结构：

```
系统盘
 |
 +-- QTS系统文件

数据盘
 |
 +-- 用户数据
 +-- Docker数据
 +-- QPKG应用
```

直接安装 Entware 会占用系统空间。

因此改用 QNAP QPKG 方式安装。


---

## 重新安装 Entware

通过 QNAP App Center 手动安装：

```
Entware QPKG
```

安装后：

```
/share/CACHEDEV1_DATA/.qpkg/Entware
```

映射：

```
/opt

↓

/share/CACHEDEV1_DATA/.qpkg/Entware
```


验证：

```bash
ls -la /opt
```

确认：

```
/opt -> Entware目录
```


测试：

```bash
/opt/bin/opkg update
```

成功：

```
Updated list of available packages
```


---

# 创建 Blinko 部署目录

为了方便管理，统一放在 Docker 数据目录：

```bash
mkdir -p /share/CACHEDEV1_DATA/docker/blinko

cd /share/CACHEDEV1_DATA/docker/blinko
```


最终目录：

```
docker
└── blinko
    |
    ├── docker-compose.yml
    ├── data
    └── postgres
```


其中：

- data：Blinko数据
- postgres：数据库文件


---

# 编写 Docker Compose

创建：

```bash
vim docker-compose.yml
```


内容：

```yaml
services:

  postgres:
    image: postgres:16-alpine
    container_name: blinko-postgres

    environment:
      POSTGRES_USER: blinko
      POSTGRES_PASSWORD: blinko_password
      POSTGRES_DB: blinko

    volumes:
      - ./postgres:/var/lib/postgresql/data

    restart:
      unless-stopped


  blinko:

    image: blinkospace/blinko:latest

    container_name: blinko

    depends_on:
      - postgres

    ports:
      - "1111:1111"

    environment:

      DATABASE_URL:
        postgresql://blinko:blinko_password@postgres:5432/blinko

    volumes:

      - ./data:/app/data

    restart:
      unless-stopped
```


---

# 部署过程中遇到的问题

## 问题1：Blinko 镜像名称错误


第一次启动：

```bash
docker compose up -d
```


失败：

```
pull access denied for blinkoai/blinko
repository does not exist
```


原因：

Docker 镜像地址错误。


错误：

```yaml
image: blinkoai/blinko
```


通过：

```bash
docker search blinko
```

确认正确镜像：

```
blinkospace/blinko
```


修改：

```yaml
image:
  blinkospace/blinko:latest
```


---

## 问题2：Docker Hub 拉取 PostgreSQL 失败


错误：

```
EOF
context canceled
```


原因：

Docker Hub 网络连接中断。


解决：

手动拉取：

```bash
docker pull postgres:16-alpine
```


成功：

```
Downloaded newer image
```


重新启动即可。


---

# 启动 Blinko


执行：

```bash
docker compose up -d
```


输出：

```
Container blinko-postgres Started

Container blinko Started
```


查看：

```bash
docker ps
```


结果：

```
blinko              Up

blinko-postgres     Up
```


端口：

```
1111 -> Blinko
```


---

# Web访问测试


浏览器访问：

```
http://NAS_IP:1111
```


例如：

```
http://192.168.100.172:1111
```


成功进入：

```
Blinko 登录页面
```


说明：

- Docker运行正常
- 数据库正常
- 网络正常
- 数据目录挂载正常


---

# 最终部署结构


```
QNAP TS-464c2


/share/CACHEDEV1_DATA/docker

        |
        |
       blinko

        |
        +---- blinko container
        |
        +---- postgres container



数据：

docker/blinko/data

docker/blinko/postgres
```


---

# 后续优化计划


## 域名访问

目标：

```
https://blinko.example.com
```


方案：

```
腾讯云DNS

        |

家庭公网IP

        |

路由器端口映射

        |

Nginx Proxy Manager

        |

Blinko
```


---

## HTTPS

使用：

```
Let's Encrypt
```

实现：

- HTTPS访问
- 自动证书续期
- 手机安全访问


---

## 手机访问

推荐：

HarmonyOS 手机：

```
Pura 80 Pro
```

访问方式：

方案1：

```
局域网访问
http://NAS_IP:1111
```

方案2：

```
个人域名 + HTTPS
```


---

## 数据备份

当前数据：

```
/share/CACHEDEV1_DATA/docker/blinko
```


建议：

使用：

- QNAP HBS3
- 快照
- 定期数据库导出


备份：

```
data/

postgres/

docker-compose.yml
```


---

# 总结


本次完成：

✅ QNAP Docker环境准备

✅ Entware环境整理

✅ PostgreSQL部署

✅ Blinko部署

✅ 数据持久化

✅ Web访问验证


最终实现：

```
私人知识库

        +

NAS存储

        +

Docker部署

        +

未来AI增强
```


后续可以继续扩展：

- 域名访问
- HTTPS
- 手机App化体验
- 自动备份
- 与 Obsidian / AI RAG 集成


---

本文原载于 [巴巴变的博客](http://blog.bugxch.top)，遵循CC BY-NC-SA 4.0协议，复制请保留原文出处。
