---
title: "Go分布式爬虫"
date: 2020-08-31T20:45:42+08:00
categories: ["go"]
tags: ["go", "crawler", "distributed", "docker"]
---

# go-crawler-distributed
---

Github：[https://github.com/golang-collection/go-crawler-distributed](https://github.com/golang-collection/go-crawler-distributed)

Distributed crawler projects, the project supports personalization page parser secondary development, the whole project using micro service architecture, messages are sent asynchronously through message queues, Use the following framework: redigo, gorm, goquery, easyjson, viper, closer, zap, go-micro, and containable deployment is realized through Docker, intermediate crawler nodes support horizontal expansion.

分布式爬虫项目，本项目支持个性化定制页面解析器二次开发，项目整体采用微服务架构，通过消息队列实现消息的异步发送，使用到的框架包括：redigo, gorm, goquery, easyjson, viper, amqp, zap, go-micro，并通过Docker实现容器化部署，中间爬虫节点支持水平拓展。

# 目录结构

- cache redis相关操作
- config 存放配置文件
- crawler 处理爬虫相关逻辑
    - crawerConfig 爬虫对应的消息队列配置
    - douban 豆瓣网页解析
    - fetcher 网页抓取
    - meituan 美团网页解析
    - persistence 定义用于保存数据的结构体
    - worker 具体的工作逻辑，通过它实现代码的解耦
    - crawlOperation 除存储外统一爬虫处理模块
- db mysql操作
- dependencies 项目依赖的环境相关的docker-compose文件
- deploy 脚本
    - buildScript 部署脚本
    - deploy 直接启动项目的脚本
    - dockerBuildScript 构建docker镜像
    - service 用于存放服务的Dockerfile
- elastic elastic的相关操作
- model 结构体定义
- mq 消息队列操作
- runtime 日志文件
- service 微服务
    - cache redis微服务通过grpc操作
      - proto 定义grpc的proto文件
      - server 定义grpc的服务端
    - douban 豆瓣相关服务
      - crawl_tags 用于爬取豆瓣的[tags](https://book.douban.com/tag/)，此页面为爬虫的起始界面
      - crawl_list 用于爬取豆瓣的[list](https://book.douban.com/tag/%E5%B0%8F%E8%AF%B4)
      - crawl_detail 用于爬取豆瓣的图书具体内容[detail](https://book.douban.com/subject/25955474/)
      - storage_detail 用于存储豆瓣的图书具体内容，存储到MySQL中
    - elastic elasticSearch微服务通过grpc操作
      - proto 定义grpc的proto文件
      - server 定义grpc的服务端
    - meituan 美团相关服务
      - crawl_tags 用于爬取美团的[list](https://tech.meituan.com/)，此页面为爬虫的起始界面
      - crawl_list 用于爬取美团的[urllist](https://tech.meituan.com/page/2.html)
      - crawl_detail 用于爬取美团技术文章的页面具体内容[detail](https://tech.meituan.com/2020/04/23/octo-watt.html)
      - storage_detail 用于存储美团技术文章的具体内容，存储到ElasticSearch中
    - watchConfig 配置相关
- tools 小工具
- unifiedLog 统一日志操作

# 配置文件
You need to customize your configuration. Create the config.json file under the config folder in the project root directory.
The sample of config.json.
```json
{
  "mysql": {
    "user": "root",
    "password": "example",
    "host": "mysql:3306",
    "db_name": "short_story"
  },
  "redis": {
    "host": "redis:6379"
  },
  "rabbitmq": {
    "user": "guest",
    "password": "guest",
    "host": "rabbitmq:5672"
  },
  "elastic": {
    "url": "http://elastic_server:9200",
    "index": "article"
  }
}
```

# Parser

## [douban](./crawer/douban)
Parser：[parser](https://github.com/golang-collection/go-crawler-distributed/tree/master/crawler/douban) uses css selectors for page parsing.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200902114015756.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NfODQyNDk5NDY3,size_16,color_FFFFFF,t_70#pic_center)


## [meituan](./crawer/meituan)
Parser: [parser](https://github.com/golang-collection/go-crawler-distributed/tree/master/crawler/meituan) uses css selectors for page parsing.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200902114028371.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NfODQyNDk5NDY3,size_16,color_FFFFFF,t_70#pic_center)


# Framework

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200902114042130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NfODQyNDk5NDY3,size_16,color_FFFFFF,t_70#pic_center)


# Architecture

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200902114052402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NfODQyNDk5NDY3,size_16,color_FFFFFF,t_70#pic_center)


# Installation

Deploying projects locally or in the cloud provides two ways:

- Direct Deploy
- Docker(Recommended)

# Pre-requisite

- Go 1.13.6
- Redis 6.0
- MySQL 5.7
- RabbitMQ management
- ElasticSearch
- Others [go.mod](https://github.com/golang-collection/go-crawler-distributed/blob/master/go.mod)


# Quick Start

Please open the command line prompt and execute the command below. Make sure you have installed `docker-compose` in advance.

```bash
git clone https://github.com/Knowledge-Precipitation-Tribe/go-crawler-distributed
cd go-crawler-distributed/deploy/buildScript
bash linux_build
cd ../
docker network create -d bridge crawler
docker-compose up -d
```

The above command has started the basic services, including Redis, elasticSearch, RabbitMQ, and mysql. RPC services for Redis and elasticSearch are also included.

You can then switch to the douban or meituan folder and launch the service with the following command.


Like:
```bash
cd meituan
docker-compose up -d
```

# Run

## Basic services

Please use `docker-compose` to one-click to start up. Create a file named `docker-compose.yml` and input the code below.

```yaml
version: "3"

services:

  cache:
    build:
      context: cache
      dockerfile: Dockerfile
    networks:
      - crawler

  elastic:
    build:
      context: elastic
      dockerfile: Dockerfile
    depends_on:
      - elastic_server
    networks:
      - crawler

  redis:
    image: redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - crawler

  mysql:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
    networks:
      - crawler

  elastic_server:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.8.0
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
       - elastic-data:/data
    environment:
      - discovery.type=single-node
    networks:
      - crawler

  rabbitmq:
    image: rabbitmq:management
    hostname: myrabbitmq
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq
    networks:
      - crawler

volumes:
  elastic-data:
  rabbitmq-data:
  redis-data:

networks:
  crawler:
    external: true
```

Then execute the command below, and the basic service will start up. 

```bash
docker-compose up -d
```

The above command has started the basic services, including Redis, elasticSearch, RabbitMQ, and mysql. RPC services for Redis and elasticSearch are also included.

## crawler service

Create a docker-composite.yml file under the meituan or douban folder.

```yaml
version: "3"

services:

  crawl_list:
    build:
      context: crawl_list
      dockerfile: Dockerfile
    networks:
      - crawler

  crawl_tags:
    build:
      context: crawl_urllist
      dockerfile: Dockerfile
    networks:
      - crawler

  crawl_detail:
    build:
      context: crawl_detail
      dockerfile: Dockerfile
    networks:
      - crawler

  storage_detail:
    build:
      context: storage_detail
      dockerfile: Dockerfile
    networks:
      - crawler

networks:
  crawler:
    external: true
```

Then execute the command below, and the crawler service will start up. 

```bash
docker-compose up -d
```

If you want to start multiple crawler nodes you can use the following command.

```bash
docker-compose up --scale crawl_list=3 -d
```

## Direct 
Start the service directly, but be aware that you need to rely on redis, RabbitMQ and other services

```bash
cd deploy/deploy/
bash start-meituan-direct.sh
```

# Appendix

- docker安装：[https://docs.docker.com/](https://docs.docker.com/)
- docker-compose安装：[https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

# License

[MIT](https://github.com/Knowledge-Precipitation-Tribe/DigitRecognitionService/blob/master/LICENSE)

Copyright (c) 2020 Knowledge-Precipitation-Tribe