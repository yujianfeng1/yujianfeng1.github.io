
## 什么是Docker？🐳

想象一下，你有一个精心制作的应用程序，但每次在不同的电脑或服务器上运行时，总会遇到"我的电脑上可以运行"的尴尬情况。Docker就是为了解决这个问题而生的！

Docker可以把你的应用程序连同它所需的所有环境、依赖，打包成一个标准的"集装箱"（容器）。无论是开发、测试还是生产环境，这个"集装箱"都可以稳定、一致地运行。

## Docker的核心概念 🧱

### 1. 镜像（Image）
就像是应用程序的"照片"，包含了运行应用所需的所有文件和配置。

### 2. 容器（Container）
镜像的"活动实例"，类似于从"照片"中创建出来的实物。一个镜像可以创建多个容器。

### 3. Dockerfile
定义镜像的"配方"，描述如何构建一个Docker镜像。

## Docker的优势 🚀

1. **一次编写，到处运行**
   - 不再担心"在我电脑上明明可以运行"的问题
   - 开发、测试、生产环境保持高度一致

2. **资源利用率高**
   - 比传统虚拟机更轻量
   - 秒级启动，占用更少系统资源

3. **快速部署**
   - 应用打包成镜像后，部署就像复制粘贴一样简单
   - 可以快速横向扩展

## 一个实际的Docker示例 👨‍💻

### 部署一个简单的Web应用

```dockerfile
# 使用官方Python基础镜像
FROM python:3.9-slim

# 设置工作目录
WORKDIR /app

# 复制项目文件
COPY . /app

# 安装依赖
RUN pip install flask

# 暴露端口
EXPOSE 5000

# 启动应用
CMD ["python", "app.py"]
```

### 构建并运行

```bash
# 构建镜像
docker build -t mywebapp .

# 运行容器
docker run -p 5000:5000 mywebapp
```

## Docker生态系统 🌐

- **Docker Compose**：多容器编排
- **Docker Swarm**：容器集群管理
- **Kubernetes**：大规模容器编排


## 常用Docker命令 🚢

### 镜像相关命令

1. **拉取镜像**
```bash
docker pull <image_name>:<tag>
# 例如：docker pull ubuntu:20.04
```

2. **查看本地镜像**
```bash
docker images
# 或
docker image ls
```

3. **删除镜像**
```bash
docker rmi <image_id>
# 强制删除：docker rmi -f <image_id>
```

4. **构建镜像**
```bash
docker build -t <image_name>:<tag> .
# 例如：docker build -t myapp:v1 .
```

### 容器相关命令

1. **运行容器**
```bash
# 基本运行
docker run <image_name>

# 后台运行
docker run -d <image_name>

# 指定端口映射
docker run -p 8080:80 <image_name>

# 挂载卷
docker run -v /host/path:/container/path <image_name>
```

2. **查看运行中的容器**
```bash
docker ps
# 查看所有容器（包括已停止）
docker ps -a
```

3. **停止和删除容器**
```bash
# 停止容器
docker stop <container_id>

# 删除容器
docker rm <container_id>

# 强制删除运行中的容器
docker rm -f <container_id>
```

4. **进入容器**
```bash
# 交互式进入
docker exec -it <container_id> /bin/bash

# 执行命令
docker exec <container_id> <command>
```

## Dockerfile编写指南 📝

### Dockerfile基本结构

```dockerfile
# 基础镜像
FROM ubuntu:20.04

# 设置维护者信息
LABEL maintainer="your_email@example.com"

# 设置工作目录
WORKDIR /app

# 更新系统并安装依赖
RUN apt-get update && apt-get install -y \
    python3 \
    pip \
    && rm -rf /var/lib/apt/lists/*

# 复制项目文件
COPY . /app

# 安装Python依赖
RUN pip install -r requirements.txt

# 设置环境变量
ENV APP_ENV=production

# 暴露端口
EXPOSE 8000

# 启动命令
CMD ["python3", "app.py"]
```

### Dockerfile常用指令详解

1. **FROM**
   - 指定基础镜像
   - 通常是官方镜像，如 `ubuntu`, `python`, `node`

2. **WORKDIR**
   - 设置工作目录
   - 后续的 `COPY`、`RUN` 等指令都在此目录下执行

3. **COPY 与 ADD**
   - `COPY`：简单复制文件
   - `ADD`：可以解压缩，支持URL下载

4. **RUN**
   - 执行shell命令
   - 每个 `RUN` 会创建一个新的镜像层

5. **ENV**
   - 设置环境变量
   - 可以在容器运行时使用

6. **EXPOSE**
   - 声明容器运行时监听的端口
   - 仅是声明，不会自动映射

7. **CMD 与 ENTRYPOINT**
   - `CMD`：默认执行的命令
   - `ENTRYPOINT`：容器的主要命令

### 最佳实践 🌟

1. 尽量使用官方基础镜像
2. 减少镜像层数
3. 使用 `.dockerignore` 文件
4. 不要在镜像中存储敏感信息
5. 优化镜像大小

## 常见问题解答 ❓

- **如何减小镜像大小？**
  - 使用多阶段构建
  - 选择轻量级基础镜像
  - 清理不必要的缓存和文件

- **如何处理依赖？**
  - 使用 `requirements.txt`（Python）
  - 使用 `package.json`（Node.js）
  - 在 `RUN` 指令中安装依赖

