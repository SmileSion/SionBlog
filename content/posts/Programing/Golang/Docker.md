+++
date = '2026-04-20T10:23:07+08:00'
draft = false
title = 'Docker的使用'
tags = ['Golang','Docker']
showtoc = true
+++

> Docker 是一种容器化技术，它可以把应用程序、运行环境、依赖库和配置文件一起打包成镜像，再通过容器运行起来。相比直接部署在服务器上，Docker 更容易做到环境一致、快速发布、快速回滚和服务隔离。

## Docker 是什么

在没有 Docker 之前，我们部署一个服务通常需要在服务器上安装语言环境、依赖包、配置数据库连接、开放端口等。不同服务器的系统版本、依赖版本、环境变量只要有一点不同，就可能出现“本地能跑，线上不能跑”的问题。

Docker 解决的核心问题就是：**把应用和运行环境一起打包，让应用在不同机器上尽量保持一致的运行结果。**

Docker 中有几个重要概念：

- **镜像（Image）**：应用的只读模板，里面包含代码、依赖、运行环境等。
- **容器（Container）**：镜像运行起来之后的实例，可以理解成一个轻量级的独立运行环境。
- **Dockerfile**：用于描述如何构建镜像的脚本文件。
- **仓库（Registry）**：存放镜像的地方，比如 Docker Hub、Harbor、阿里云镜像仓库。
- **数据卷（Volume）**：用于持久化容器数据，避免容器删除后数据丢失。
- **网络（Network）**：用于容器之间或容器与宿主机之间通信。

---

## Docker 的基本命令

### 查看版本和运行状态

```bash
docker version
docker info
```

### 拉取镜像

```bash
docker pull nginx
docker pull mysql:8.0
docker pull redis:7
```

镜像名后面的 `:8.0`、`:7` 是标签（tag），通常用来区分版本。如果不写 tag，默认使用 `latest`，但生产环境不建议依赖 `latest`，最好固定明确版本。

### 查看本地镜像

```bash
docker images
```

### 运行容器

```bash
docker run -d --name my-nginx -p 8080:80 nginx
```

参数说明：

- `-d`：后台运行。
- `--name my-nginx`：给容器起名。
- `-p 8080:80`：把宿主机的 `8080` 端口映射到容器内的 `80` 端口。
- `nginx`：使用的镜像名。

访问 `http://localhost:8080` 就能看到 nginx 页面。

### 查看容器

```bash
docker ps
docker ps -a
```

- `docker ps`：查看正在运行的容器。
- `docker ps -a`：查看所有容器，包括已经停止的容器。

### 停止、启动、重启容器

```bash
docker stop my-nginx
docker start my-nginx
docker restart my-nginx
```

### 查看日志

```bash
docker logs my-nginx
docker logs -f my-nginx
docker logs --tail=100 my-nginx
```

### 进入容器

```bash
docker exec -it my-nginx sh
```

如果容器里有 bash，也可以使用：

```bash
docker exec -it my-nginx bash
```

### 删除容器和镜像

```bash
docker rm my-nginx
docker rmi nginx
```

如果容器还在运行，需要先停止容器：

```bash
docker stop my-nginx
docker rm my-nginx
```

---

## Dockerfile 编写

Dockerfile 是 Docker 构建镜像的核心文件，它描述了镜像从哪个基础镜像开始、复制哪些文件、安装哪些依赖、暴露哪个端口、启动时执行什么命令。

### 常用指令

```dockerfile
FROM        # 指定基础镜像
WORKDIR     # 指定工作目录
COPY        # 复制文件到镜像中
RUN         # 构建镜像时执行命令
ENV         # 设置环境变量
EXPOSE      # 声明容器监听端口
CMD         # 容器启动时默认执行命令
ENTRYPOINT  # 容器启动入口
```

### Go 项目的 Dockerfile 示例

假设有一个简单的 Go Web 服务，入口文件是 `main.go`，监听 `8080` 端口，可以使用多阶段构建：

```dockerfile
# 第一阶段：编译 Go 程序
FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o server .

# 第二阶段：只保留运行需要的二进制文件
FROM alpine:3.20

WORKDIR /app

COPY --from=builder /app/server ./server

EXPOSE 8080

CMD ["./server"]
```

这里使用了 **多阶段构建**：

- 第一阶段使用 `golang` 镜像编译代码。
- 第二阶段使用更小的 `alpine` 镜像运行程序。
- 最终镜像里不会包含 Go 编译器和源码，体积更小，也更适合部署。

如果你的 Go 程序不依赖系统动态库，也可以使用更小的运行镜像：

```dockerfile
FROM scratch

COPY --from=builder /app/server /server

EXPOSE 8080

CMD ["/server"]
```

不过 `scratch` 是空镜像，没有 shell、证书、时区文件，排查问题不如 `alpine` 方便。实际项目中可以根据需求选择。

### Node 项目的 Dockerfile 示例

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "run", "start"]
```

### .dockerignore 文件

`.dockerignore` 用来排除不需要复制进镜像的文件，作用类似 `.gitignore`。

```gitignore
.git
.idea
.vscode
node_modules
dist
tmp
*.log
Dockerfile
docker-compose.yml
```

这样可以减少构建上下文大小，加快构建速度，也能避免把无关文件打进镜像。

---

## 镜像打包与运行

### 构建镜像

```bash
docker build -t my-go-app:1.0 .
```

参数说明：

- `-t my-go-app:1.0`：指定镜像名和版本。
- `.`：表示构建上下文是当前目录。

### 查看镜像

```bash
docker images
```

### 运行镜像

```bash
docker run -d \
  --name my-go-app \
  -p 8080:8080 \
  my-go-app:1.0
```

### 传递环境变量

```bash
docker run -d \
  --name my-go-app \
  -p 8080:8080 \
  -e APP_ENV=prod \
  -e DB_HOST=127.0.0.1 \
  my-go-app:1.0
```

也可以通过环境变量文件传递：

```bash
docker run -d \
  --name my-go-app \
  -p 8080:8080 \
  --env-file .env \
  my-go-app:1.0
```

`.env` 示例：

```env
APP_ENV=prod
DB_HOST=mysql
DB_PORT=3306
```

### 挂载目录

```bash
docker run -d \
  --name my-nginx \
  -p 8080:80 \
  -v ./html:/usr/share/nginx/html \
  nginx
```

`-v ./html:/usr/share/nginx/html` 表示把宿主机当前目录下的 `html` 目录挂载到容器内的 `/usr/share/nginx/html`。

### 使用数据卷

```bash
docker volume create mysql-data

docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0
```

这样 MySQL 的数据会保存在 Docker volume 中，即使删除容器，数据卷仍然存在。

---

## Docker Compose

当项目只有一个容器时，`docker run` 还比较简单。但如果一个项目包含后端服务、MySQL、Redis、Nginx 等多个容器，单独写多个 `docker run` 命令就比较麻烦。

Docker Compose 用来编排多个容器，可以把服务、端口、环境变量、数据卷、网络等配置写到一个 `docker-compose.yml` 文件中。

### docker-compose.yml 示例

下面是一个 Go 服务 + MySQL + Redis 的示例：

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: go-app
    ports:
      - "8080:8080"
    environment:
      APP_ENV: prod
      DB_HOST: mysql
      DB_PORT: 3306
      REDIS_HOST: redis
    depends_on:
      - mysql
      - redis
    networks:
      - app-network

  mysql:
    image: mysql:8.0
    container_name: go-mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: app
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    container_name: go-redis
    ports:
      - "6379:6379"
    networks:
      - app-network

volumes:
  mysql-data:

networks:
  app-network:
    driver: bridge
```

### Compose 常用命令

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs -f
docker compose restart
docker compose build
```

说明：

- `docker compose up -d`：后台启动所有服务。
- `docker compose down`：停止并删除 Compose 创建的容器和网络。
- `docker compose logs -f`：实时查看日志。
- `docker compose build`：重新构建镜像。

如果要连数据卷也一起删除：

```bash
docker compose down -v
```

这个命令会删除 volume，数据库数据也会丢失，使用时要谨慎。

### depends_on 的注意点

`depends_on` 只能保证容器启动顺序，不能保证 MySQL 已经初始化完成。实际项目中，应用服务连接数据库时应该有重试机制，或者给数据库配置 `healthcheck`。

```yaml
services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 3s
      retries: 10
```

---

## 镜像导入导出

有些场景不能直接访问镜像仓库，或者需要把镜像离线传到另一台服务器，就可以使用导入导出。

### 导出镜像

```bash
docker save -o my-go-app.tar my-go-app:1.0
```

或者压缩一下：

```bash
docker save my-go-app:1.0 | gzip > my-go-app.tar.gz
```

### 导入镜像

```bash
docker load -i my-go-app.tar
```

如果是 gzip 文件：

```bash
gunzip -c my-go-app.tar.gz | docker load
```

### 导出容器文件系统

```bash
docker export -o container.tar my-go-app
```

### 导入为镜像

```bash
docker import container.tar my-go-app:imported
```

### save/load 和 export/import 的区别

- `docker save/load` 操作的是镜像，会保留镜像层、tag、构建历史等信息。
- `docker export/import` 操作的是容器文件系统，不会保留镜像层历史，也不会保留原来的 CMD、ENTRYPOINT 等元数据。

日常迁移镜像一般优先使用 `docker save` 和 `docker load`。

---

## 配置文件与环境配置

### 通过环境变量配置

Docker 中最常见的配置方式是环境变量。应用程序启动时读取环境变量，根据不同环境连接不同服务。

```bash
docker run -d \
  --name app \
  -e APP_ENV=prod \
  -e LOG_LEVEL=info \
  -e DB_DSN="root:123456@tcp(mysql:3306)/app" \
  app:1.0
```

Go 程序中可以这样读取：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	env := os.Getenv("APP_ENV")
	fmt.Println("env:", env)
}
```

### 通过配置文件挂载

如果配置比较多，也可以把配置文件放在宿主机上，再挂载进容器。

```bash
docker run -d \
  --name app \
  -v ./config.yaml:/app/config.yaml \
  app:1.0
```

Compose 中可以这样写：

```yaml
services:
  app:
    image: app:1.0
    volumes:
      - ./config.yaml:/app/config.yaml
```

### 时区配置

很多镜像默认使用 UTC 时间。如果服务需要使用本地时区，可以设置环境变量：

```yaml
services:
  app:
    image: app:1.0
    environment:
      TZ: Asia/Shanghai
```

也可以挂载宿主机时区文件：

```yaml
services:
  app:
    image: app:1.0
    volumes:
      - /etc/localtime:/etc/localtime:ro
```

### 资源限制

为了避免某个容器占用过多资源，可以限制 CPU 和内存：

```bash
docker run -d \
  --name app \
  --memory=512m \
  --cpus=1.0 \
  app:1.0
```

Compose 中可以使用：

```yaml
services:
  app:
    image: app:1.0
    mem_limit: 512m
    cpus: 1.0
```

---

## 常用排查命令

### 查看容器日志

```bash
docker logs -f app
```

### 查看容器详细信息

```bash
docker inspect app
```

### 查看容器资源占用

```bash
docker stats
```

### 查看容器内进程

```bash
docker top app
```

### 进入容器排查

```bash
docker exec -it app sh
```

### 查看网络

```bash
docker network ls
docker network inspect bridge
```

### 清理无用资源

```bash
docker system df
docker system prune
```

如果要清理未使用的镜像、容器、网络和构建缓存：

```bash
docker system prune -a
```

这个命令会删除未使用的镜像，执行前最好先确认是否还有需要保留的镜像。

---

## Docker 使用建议

- **镜像版本要固定**：生产环境尽量使用 `mysql:8.0`、`redis:7-alpine` 这类明确版本，不要依赖 `latest`。
- **镜像尽量小**：Go 项目可以使用多阶段构建，减少最终镜像体积。
- **不要把敏感信息写进镜像**：密码、密钥、数据库连接等配置应通过环境变量、配置文件或 Secret 管理。
- **数据要挂载到 volume**：数据库、上传文件等需要持久化的数据不要只放在容器内部。
- **应用要支持优雅退出**：容器停止时会收到信号，服务应正确处理退出逻辑。
- **日志输出到 stdout/stderr**：容器日志最好直接输出到标准输出，方便 `docker logs` 和日志系统收集。
- **配置健康检查**：生产环境中可以通过 `healthcheck` 判断服务是否可用。

---

## 一句话总结

> Docker 的核心价值是把应用和运行环境一起标准化封装。开发阶段可以用它快速搭建依赖服务，部署阶段可以用 Dockerfile 构建镜像，用 Docker Compose 管理多容器服务，再通过环境变量、数据卷、网络和导入导出等能力完成完整的应用交付。
