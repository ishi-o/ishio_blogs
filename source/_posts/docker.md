---
title: 'Docker-Compose: 容器部署'
categories:
  - Tools & Utilities
  - 3P tools
tags:
  - docker
mathjax: false
date: 2025-06-17T16:36:12.000Z
---
<!-- placeholder -->
<!-- more -->

# `Docker-Compose`

## `docker-compose.yml`

- `docker-compose.yml`是唯一的必需文件
- `version`：`docker-compose`版本声明
- `services`：服务定义
- `networks`：网络定义（可选）
- `volumes`：卷定义（可选）
- `configs`：配置定义（可选）
- `secrets`：密钥定义（可选）

### `services`配置

- 子字段均是自定义的服务名称
- 自定义的服务名称下是主要需要配置的
- `image`：镜像名+标签（默认为`latest`），也可使用私有仓库的镜像

  `alpine`标签通常是及其轻便的，适用于容器化部署

  默认则是`latest`标签

- `build`：除了`image`还可以使用`build`，即从`Dockerfile`构建：
  - `context`：`Dockerfile`所在目录
  - `dockerfile`：`Dockerfile`文件名
  - `args`：构建参数
- 通用配置：
  - `container_name`：容器名
  - `hostname`：主机名
  - `ports`：端口映射，是一个数组（`宿主机port:容器port`）
  - `working_dir`：工作目录
  - `depends_on`：依赖其它服务，是一个数组
  - `networks`：自定义网络引用，是一个数组
  - `volumes`：自定义卷引用，是一个数组，每个元素是`卷引用:数据地址`
  - `command`：覆盖默认的启动命令，是一个数组
  - `environment`：注入环境变量
  - `env_file`：从文件注入环境变量
  - `restart`：重启策略，包括`"no"`、`always`、`on-failure`、`unless-stopped`
  - `user`：指定运行的用户和用户组
  - `group_add`：附加额外的用户组，是一个数组
  - `logging`：日志分割配置时，需要主动覆盖，设置`logging.driver`为`json-file`并设置`logging.options.max-size`（每个文件最大大小）和`max-file`（最多保留文件个数）

### `networks`配置

- 子字段均是自定义的网络名称
- 只写网络名称是最常用的，默认`driver`为`bridge`

### `volumes`配置

- 子字段均是自定义的卷名称
- 只写卷名称是最常用的，默认`driver`为`local`
- 可以在`hub.docker.com/_/mysql`查询数据存放的目录
- `docker inspect container_name:latest --format='{{json .Config.Volumes}}' | python -m json.tool`也适用于大多数容器的数据卷映射建议

## 常用`docker-compose`命令

- `docker-compose up -d [services]`：启动，`-d`为`--daemon`，可指定`services`只启动部分服务
- `docker-compose ps [-a] [services]`：查看运行状态，`-a`为查看详细信息，可指定`services`只查看部分服务
- `docker-compose logs [services]`：查看日志
- `docker-compose down`：关闭并删除所有容器和服务，可附加`-v`把卷同时删除
- `docker-compose stop [services]`：关闭服务，保留容器
- `docker-compose restart [services]`：配合`stop`使用，重启服务
- `docker-compose exec service_name sh_name`：指定`sh`和服务名称进入该容器
- `docker compose`：这是`Docker Compose v2`的命令，上述均为`v1`的

