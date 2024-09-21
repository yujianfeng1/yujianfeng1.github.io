### 1. 基础命令

- **查看 Docker 版本**
  ```bash
  docker --version
  ```

- **查看 Docker 状态**
  ```bash
  docker info
  ```

### 2. 容器操作

- **拉取镜像**
  ```bash
  docker pull <image_name>
  ```
  从 Docker Hub 拉取指定镜像。

- **列出所有镜像**
  ```bash
  docker images
  ```

- **运行容器**
  ```bash
  docker run <options> <image_name>
  ```
  启动一个新容器，常用选项：
  - `-d`: 后台运行
  - `-p <host_port>:<container_port>`: 端口映射
  - `--name <container_name>`: 指定容器名称

- **列出正在运行的容器**
  ```bash
  docker ps
  ```

- **列出所有容器（包括停止的）**
  ```bash
  docker ps -a
  ```

- **停止容器**
  ```bash
  docker stop <container_id_or_name>
  ```

- **启动已停止的容器**
  ```bash
  docker start <container_id_or_name>
  ```

- **重启容器**
  ```bash
  docker restart <container_id_or_name>
  ```

- **删除容器**
  ```bash
  docker rm <container_id_or_name>
  ```

### 3. 镜像操作

- **删除镜像**
  ```bash
  docker rmi <image_id_or_name>
  ```

- **构建镜像**
  ```bash
  docker build -t <image_name>:<tag> <path_to_dockerfile>
  ```

### 4. 运行命令

- **在容器内执行命令**
  ```bash
  docker exec -it <container_id_or_name> <command>
  ```
  例如，进入一个运行中的容器：
  ```bash
  docker exec -it <container_id_or_name> /bin/bash
  ```

### 5. 网络与卷

- **创建网络**
  ```bash
  docker network create <network_name>
  ```

- **列出网络**
  ```bash
  docker network ls
  ```

- **创建卷**
  ```bash
  docker volume create <volume_name>
  ```

- **列出卷**
  ```bash
  docker volume ls
  ```

### 6. 日志和信息

- **查看容器日志**
  ```bash
  docker logs <container_id_or_name>
  ```

- **查看容器信息**
  ```bash
  docker inspect <container_id_or_name>
  ```

<!-- ##{"timestamp":1498060609}## -->