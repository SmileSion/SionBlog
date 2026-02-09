+++
date = '2026-02-09T16:44:59+08:00'
draft = false
title = '泰拉瑞亚服务器开服教程｜Docker镜像打包部署实战'
tags = ["Linux","Macos","教程","Docker"] 
showtoc = true
+++

# Terraria Server Docker 教程

教你学会如何把 Terraria Linux 服务端打成 Docker 镜像，并用 Docker Compose 在 x86_64 Linux 服务器运行。本教程从 `Dockerfile`、启动脚本到 `docker-compose.yml`，一步步解释原理与用法，并包含多实例部署方式。

首先在[泰拉瑞亚服务端下载](https://terraria.org/)页面最下方点击PC Dedicated Server下载服务端

---

## 目录结构

- `Dockerfile`：镜像构建文件（原生运行）
- `entrypoint.sh`：启动脚本，自动生成 `serverconfig.txt`
- `docker-compose.yml`：单实例编排
- `docker-compose.multi.yml`：多实例编排（普通/专家/大师）
- `data/` / `data-normal` / `data-expert` / `data-master`：持久化目录

---

## 一、Dockerfile：把服务端打进镜像

Dockerfile 的核心目标是：
1. 选择基础Linux镜像（这里是 Debian）
2. 安装必要运行库
3. 把服务端文件复制进去
4. 设置启动脚本

### Dockerfile 示例
```dockerfile
# 基础镜像：Debian 稳定版（包含 glibc，兼容性好）
FROM debian:bookworm-slim

# 入口脚本需要的默认环境变量
ENV TERRARIA_HOME=/opt/terraria \
    TERRARIA_DATA=/data

# 安装运行原生服务端所需的运行库
# --no-install-recommends：减少镜像体积
# 清理 apt 缓存以减少镜像大小
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        libstdc++6 \
        libgcc-s1 \
    && rm -rf /var/lib/apt/lists/*

# 设置容器内工作目录
WORKDIR ${TERRARIA_HOME}

# 复制服务端文件到镜像内
COPY . ${TERRARIA_HOME}

# 确保二进制可执行
RUN chmod +x /opt/terraria/TerrariaServer.bin.x86_64

# 拷贝启动脚本
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# 元信息：声明服务端监听端口
EXPOSE 7777/tcp

# 声明数据卷：世界/配置放在 /data
VOLUME ["/data"]

# 容器启动时执行入口脚本
ENTRYPOINT ["/entrypoint.sh"]

```

### 关键点
- `libstdc++6` / `libgcc-s1`：Terraria 原生二进制依赖的运行库
- `VOLUME ["/data"]`：让世界文件与配置落在数据卷（容器可删除，数据仍在）
- `ENTRYPOINT`：容器启动时执行脚本

---

## 二、entrypoint.sh：启动脚本与自动配置

目标：
1. 确保 `/data` 目录存在
2. 如果没有配置文件，就用环境变量生成 `serverconfig.txt`
3. 启动 Terraria 原生服务端

### 脚本逻辑（已在本目录）
```sh
#!/usr/bin/env sh
# -e: 任一命令失败即退出；-u: 使用未定义变量即退出
set -eu

# 容器内目录（允许用环境变量覆盖）
TERRARIA_HOME="${TERRARIA_HOME:-/opt/terraria}"
TERRARIA_DATA="${TERRARIA_DATA:-/data}"
# 允许通过 TERRARIA_CONFIG 指定配置文件路径
CONFIG_PATH="${TERRARIA_CONFIG:-${TERRARIA_DATA}/serverconfig.txt}"

# 确保数据目录存在
mkdir -p "${TERRARIA_DATA}"

# 如果没有配置文件，就按环境变量生成一个
if [ ! -f "${CONFIG_PATH}" ]; then
  # 世界与服务器参数（可通过环境变量覆盖）
  WORLD_NAME="${WORLD_NAME:-world}"
  WORLD_PATH="${WORLD_PATH:-${TERRARIA_DATA}/${WORLD_NAME}.wld}"
  WORLD_SIZE="${WORLD_SIZE:-2}"
  DIFFICULTY="${DIFFICULTY:-0}"
  MAX_PLAYERS="${MAX_PLAYERS:-16}"
  SERVER_PORT="${SERVER_PORT:-7777}"
  MOTD="${MOTD:-}"
  PASSWORD="${PASSWORD:-}"
  AUTOCREATE="${AUTOCREATE:-1}"
  SECURE="${SECURE:-1}"

  # 写入基础配置
  cat > "${CONFIG_PATH}" <<CFG
world=${WORLD_PATH}
worldname=${WORLD_NAME}
autocreate=${AUTOCREATE}
worldsize=${WORLD_SIZE}
difficulty=${DIFFICULTY}
maxplayers=${MAX_PLAYERS}
port=${SERVER_PORT}
secure=${SECURE}
CFG

  # 可选字段：MOTD 和密码
  if [ -n "${MOTD}" ]; then
    printf 'motd=%s\n' "${MOTD}" >> "${CONFIG_PATH}"
  fi
  if [ -n "${PASSWORD}" ]; then
    printf 'password=%s\n' "${PASSWORD}" >> "${CONFIG_PATH}"
  fi
fi

# 如果用户传了自定义命令，就执行该命令
if [ "$#" -gt 0 ]; then
  exec "$@"
fi

# 默认启动：运行原生服务端
exec "${TERRARIA_HOME}/TerrariaServer.bin.x86_64" -config "${CONFIG_PATH}" ${TERRARIA_ARGS:-}

```

### 自动生成配置
- 用户只需要传环境变量就能快速启动
- 不需要提前准备 `serverconfig.txt`

---

## 三、Docker Compose：一键运行

Compose 用来描述“服务如何启动、端口映射、数据卷挂载、环境变量”等。

### 1) 单实例（`docker-compose.yml`）
```yaml
services:
  terraria:
    # 使用当前目录 Dockerfile 构建镜像
    build: .
    # 镜像名称
    image: terraria-server:1.0
    # 容器名称，便于查看/管理
    container_name: terraria-server
    # 强制使用 x86_64 平台（在非 x86 主机上很重要）
    platform: linux/amd64
    # 开启交互输入（用于进入控制台）
    stdin_open: true
    # 分配伪终端（用于 attach）
    tty: true
    # 端口映射：宿主 7777 -> 容器 7777
    ports:
      - "7777:7777"
    # 持久化数据：世界/配置文件保存在宿主机
    volumes:
      - ./data:/data
    # 供 entrypoint.sh 生成配置用的环境变量
    environment:
      WORLD_NAME: world
      WORLD_SIZE: 2
      DIFFICULTY: 0
      MAX_PLAYERS: 16
      SERVER_PORT: 7777
      AUTOCREATE: 1
      SECURE: 1
      # PASSWORD: "your_password"
      # MOTD: "Welcome to Terraria"
    # 异常退出自动重启，除非手动停止
    restart: unless-stopped

```

### 2) 多实例（`docker-compose.multi.yml`）
普通/专家/大师三服共存：

```yaml
services:
  terraria-normal:
    image: terraria-server:1.0
    container_name: terraria-normal
    platform: linux/amd64
    stdin_open: true
    tty: true
    ports:
      - "7777:7777"
    volumes:
      - ./data-normal:/data
    environment:
      WORLD_NAME: normal
      WORLD_SIZE: 2
      DIFFICULTY: 0
      MAX_PLAYERS: 16
      SERVER_PORT: 7777
      AUTOCREATE: 1
      SECURE: 1
    restart: unless-stopped

  terraria-expert:
    image: terraria-server:1.0
    container_name: terraria-expert
    platform: linux/amd64
    stdin_open: true
    tty: true
    ports:
      - "7778:7777"
    volumes:
      - ./data-expert:/data
    environment:
      WORLD_NAME: expert
      WORLD_SIZE: 2
      DIFFICULTY: 1
      MAX_PLAYERS: 16
      SERVER_PORT: 7777
      AUTOCREATE: 1
      SECURE: 1
    restart: unless-stopped

  terraria-master:
    image: terraria-server:1.0
    container_name: terraria-master
    platform: linux/amd64
    stdin_open: true
    tty: true
    ports:
      - "7779:7777"
    volumes:
      - ./data-master:/data
    environment:
      WORLD_NAME: master
      WORLD_SIZE: 2
      DIFFICULTY: 2
      MAX_PLAYERS: 16
      SERVER_PORT: 7777
      AUTOCREATE: 1
      SECURE: 1
    restart: unless-stopped

```

### 关键点解释
- `platform: linux/amd64`：保证用 x86_64 镜像
- `stdin_open: true` / `tty: true`：开启交互终端，方便进入服务器控制台
- `ports`：把容器端口映射到宿主端口
- `volumes`：每个实例用独立目录，避免世界文件冲突
- `restart: unless-stopped`：机器重启自动拉起

---

## 四、构建镜像

如果你能访问 Docker Hub：
```bash
cd /Users/smilesion/code/Docker/TerrariaServer/1454/Linux

# 可选：使用代理确保dockerhub可连接
# export http_proxy=http://127.0.0.1:7897
# export https_proxy=http://127.0.0.1:7897

# 需要 buildx
# 第一次使用可创建 builder
# docker buildx create --use --name terraria-builder

docker buildx build --platform linux/amd64 -t terraria-server:1.0 --load .
```

---

## 五、打包镜像并传到服务器（离线部署）

```bash
docker save -o terraria-server-1.0.tar terraria-server:1.0

# 复制到服务器
# scp terraria-server-1.0.tar docker-compose.yml docker-compose.multi.yml user@server:/opt/terraria/
# 如需带已有世界，也可复制 data/ 目录
```

---

## 六、服务器部署（x86_64 Linux）

### 单实例
```bash
cd /opt/terraria

# 载入镜像
docker load -i terraria-server-1.0.tar

# 准备持久化目录
mkdir -p data

# 启动
docker compose up -d

# 查看日志
docker compose logs -f
```

### 多实例
```bash
cd /opt/terraria

docker load -i terraria-server-1.0.tar

mkdir -p data-normal data-expert data-master

# 启动多实例
# 注意：使用 -f 指定 compose 文件

docker compose -f docker-compose.multi.yml up -d

# 查看日志
# docker compose -f docker-compose.multi.yml logs -f
```

---

## 七、配置与地图

- 如果 `./data/serverconfig.txt` 不存在，容器会根据环境变量自动生成
- 世界文件默认路径：`./data/world.wld`

你可以在 `docker-compose.yml` 中修改环境变量：
- `WORLD_NAME`
- `WORLD_SIZE` (1/2/3)
- `DIFFICULTY` (0/1/2/3)
- `MAX_PLAYERS`
- `SERVER_PORT`
- `AUTOCREATE`
- `SECURE`
- `PASSWORD`
- `MOTD`

---

## 八、常用命令

```bash
# 查看容器状态
docker compose ps

# 查看日志
docker compose logs -f

# 停止
docker compose down
```

---

## 九、进入控制台（输入服务器指令）

必须开启 `stdin_open: true` 和 `tty: true`。

### 单实例
```bash
docker compose attach terraria
```

### 多实例
```bash
docker compose -f docker-compose.multi.yml attach terraria-normal
```

### 退出但不关闭容器
按 `Ctrl+P` 再 `Ctrl+Q`。

