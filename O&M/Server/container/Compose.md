##   1. Docker Compose 简介

Docker Compose 是 Docker 官方提供的工具，可以通过一个 YAML 文件定义并运行多个容器应用。它可以轻松管理服务间的依赖关系、网络配置以及数据卷映射，从而实现多容器应用的快速部署和调试。

------

## 2. Docker Compose 文件结构

Docker Compose 使用 YAML 格式文件（通常命名为 docker-compose.yml），主要包含以下几个部分：

- **version**：指定 Compose 文件的版本，不同版本支持的功能有所区别，常用的版本有 "2"、"2.1"、"3"、"3.8" 等。
- **services**：定义各个服务（容器），每个服务可以配置镜像、构建、环境变量、端口映射、数据卷挂载等。
- **networks**：定义容器之间使用的网络配置，支持自定义网络名称及驱动类型。
- **volumes**：定义数据卷，用于持久化数据和共享文件。

------

## 3. 示例 docker-compose.yml 文件解析

下面是一个简单的示例文件，展示如何定义一个 Web 应用和数据库服务：

```yaml
version: "3.8"
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./web:/usr/share/nginx/html
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

- **version**：这里使用的是 "3.8" 版本。
- **web 服务**：使用 nginx 镜像，将主机的 80 端口映射到容器的 80 端口，并将本地目录挂载到容器中。
- **db 服务**：使用 mysql 镜像，通过 environment 配置数据库的根密码，并将数据卷挂载到容器的数据目录。
- **volumes**：定义了名为 db_data 的数据卷，用于持久化 MySQL 数据。

------

## 4. 常用配置项说明

### 4.1 服务配置

- **image**：指定容器启动所用的镜像名称与标签。
- **build**：如果需要从 Dockerfile 构建镜像，可以指定 build 上下文（路径）以及 Dockerfile 文件（可选）。
- **ports**：定义主机与容器之间的端口映射，例如 "主机端口:容器端口"。
- **volumes**：用于挂载主机目录或数据卷到容器内，格式可以为 "主机路径:容器路径" 或 "数据卷名称:容器路径"。
- **environment**：传递环境变量，支持在容器中定义配置参数。
- **depends_on**：定义服务依赖关系，确保依赖的服务在当前服务启动前先启动。

### 4.2 网络配置

- **networks**：可以在服务中指定使用的网络，或者在文件底部统一定义网络配置，支持 driver（如 bridge、overlay）等参数。

### 4.3 数据卷配置

- **volumes**：用于定义匿名卷或命名卷，实现数据持久化。命名卷可以在多个服务之间共享数据。

------

## 5. 常用命令

- **启动服务**：运行 `docker-compose up` 可以启动所有服务，添加 `-d` 参数可以在后台运行。
- **停止服务**：运行 `docker-compose down` 可停止并删除容器，同时可以加上 `-v` 参数删除数据卷。
- **查看日志**：使用 `docker-compose logs` 查看各个服务的日志输出，有助于排查问题。
- **扩展服务**：例如 `docker-compose up --scale web=3` 可以水平扩展 web 服务，启动 3 个实例。

------

## 6. 编写技巧与最佳实践

- **分离配置**：将不同环境（开发、测试、生产）的配置分离到不同的 Compose 文件中，或使用环境变量实现动态配置。
- **版本控制**：将 docker-compose.yml 文件纳入版本控制系统，方便团队协作和历史追踪。
- **日志与监控**：配置容器日志输出，并结合 Docker 日志驱动或第三方监控工具，便于及时发现问题。
- **安全性**：避免在 Compose 文件中硬编码敏感信息（如密码），可以通过 .env 文件传递环境变量。

```
```

