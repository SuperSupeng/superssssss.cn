---
title: "Go创建RestfulAPI服务"
date: 2020-08-31T20:48:07+08:00
categories: ["go"]
tags: ["go", "restfulAPI"]
---

# Go-RestfulAPI
---

Github：[https://github.com/golang-collection/Go-RestfulAPI](https://github.com/golang-collection/Go-RestfulAPI)

本项目使用Go语言构建Restful API服务，项目采用gin构建，包括jwt，swagger，viper, zap, gorm, make，bash等模块，通过docker一键部署并且可以动态扩缩容实现负载均衡。

This project uses Go language to build Restful API services, and gin to build the project, including JWT, Swagger, Viper, Zap, Gorm, Make, bash and other modules. It can be deployed by docker with one key and dynamically expand capacity to achieve load balance.

# 目录结构

- conf 用于存放配置文件
- config 用于读取配置文件
- docs swagger生成文档
- handler 构建具体的api服务
    - sd 健康检查
    - user user操作
    - runtime 获取运行时ip
- model 结构体定义
- pkg 常用功能模块
    - auth 用户加密
    - constvar 分页查询
    - db 数据库连接池
    - errno 统一错误码配置
    - file 文件操作
    - logging 统一log
    - token 生成token
    - version 获取项目版本
- router 配置路由规则
    - middleware 中间件
- runtime 存储日志
- service 用于存放服务层逻辑
- util 常用工具
- admin.sh 启动，停止服务脚本
- Makefile 编译构建项目
- main.go 项目统一入口
- Dockerfile 通过此文件构建docker镜像
- docker-compose.yml 通过此文件启动服务

# Config
You need to customize your config. Create the config.json file under conf folder.

The configuration sample is shown below
```json
{
  "mysql": {
    "user": "",
    "password": "",
    "host": "",
    "db_name": ""
  },
  "redis": {
    "host": ""
  },
  "rabbitmq": {
    "user": "",
    "password": "",
    "host": ""
  }
}
```


# Installation

Deployment projects offer the following two approaches:

- Direct Deploy
- Docker(Recommended)

## Pre-requisite

- Go 1.13.6
- MySQL 5.7
- Others：[go.mod](./go.mod)

## Quick Start

Please open the command line prompt and execute the command below. Make sure you have installed `docker-compose` in advance.

```
git clone git@github.com:golang-collection/Go-RestfulAPI.git
cd Go-RestfulAPI
make
docker-compose up -d
```

Next, you can look into the `docker-compose.yml` (with detailed config params).

## Run

### make
The make command generates a binary executable of the service from the Makefile.

### Docker

Please use `docker-compose` to one-click to start up. By doing so, you don't even have to configure RabbitMQ , Reds, MySQ,ElasticSearch. Create a file named `docker-compose.yml` and input the code below.

```yaml
version: "3"

services:

  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports: ["8080"]

  lb:
    image: dockercloud/haproxy
    links:
      - web
    ports:
      - 80:80
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
  
```

Then execute the command below, and the project will start up. Open the browser and enter `http://localhost:80/swagger/index.html` to see the swagger UI interface.

```bash
docker-compose up
```

You can expand the service by following this command, and then the load balancer will automatically assign each request to each service.

```bash
docker-compose up --scale web=3 -d
```

### Direct

```bash
cd Go-RestfulAPI
make
bash admin.sh start
```
You can check the health of the service by following the command

```bash
bash admin.sh status
```

# Appendix

- docker安装：[https://docs.docker.com/](https://docs.docker.com/)
- docker-compose安装：[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

# License

[MIT](https://github.com/golang-collection/Go-RestfulAPI/blob/master/LICENSE)

Copyright (c) 2020 Knowledge-Precipitation-Tribe