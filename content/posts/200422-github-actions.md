---
title: "Github Actions自动部署到阿里云服务器"
date: 2020-04-22T21:01:41+08:00
categories: ["hugo"]
tags: ["hu go", "自动部署" ,"CI/CD", "Github Actions"]
---

---
**我的站点：[Subranium的小站](http://superssssss.cn)**

之前我们已经介绍过了，我们的hugo博客是部署在阿里云服务器上，并通过nginx服务器进行对外代理，现在使用Github Action实现将代码push到Github后自动构建并推送到云服务器，实现自动部署的功能

# Github Actions
基础内容可以参考阮一峰老师的：[GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

在这里主要推荐一下Github Actions的[Marketplace](https://github.com/marketplace?type=actions)，在这里你可以快速的找到你需要的操作，例如你想要查找关于ssh，Github Action是怎么操作的，可以直接搜索
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200422205622815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NfODQyNDk5NDY3,size_16,color_FFFFFF,t_70)
然后选择一个stars较高的工具，按照其配置进行操作即可。

----
我们这里主要介绍一下当前博客部署的工作

# 自动部署部分
在介绍之前先介绍一下我的服务器结构，访问站点后nginx代理到/root/public目录，所以我们要实现的功能如下：

1. 代码push到github触发Github Actions流程
2. 通过Github Actions先删除云服务器上已经存在的public文件夹
3. 将最新版本的hugo博客生成的public文件夹推送到云服务器

## push代码触发流程
首先我们点击Actions，首次接触可以选择simple workflow。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200422204125712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NfODQyNDk5NDY3,size_16,color_FFFFFF,t_70)
点击之后会在你的主目录的.github/workflow下创建一个yml文件，在这个文件中编写我们的集成与部署流程。

首先我们要做一些准备工作，例如构建环境，hugo生成public文件。
```yml
# 获得源码
- name: Checkout
  uses: actions/checkout@master
  with:
    submodules: true
    fetch-depth: 1

# 构建hugo站点
- name: Build site
  uses: jakejarvis/hugo-build-action@master
    with:
    args: --minify

# 生成public文件夹-可选？
- name: artifact
    uses: actions/upload-artifact@master
    with:
      name: public
      path: './public'
```

## Github Actions删除云服务器文件夹
我们这里使用ssh登陆到远端服务器，然后执行一个我们编辑好的shell脚本
shell脚本del.sh如下
```bash
if [ ! -d "/root/public/" ];then
echo "文件夹不存在"
else
rm -rf /root/public
fi
```
然后通过ssh-action登陆到运城服务器执行script里的命令
```yml
- name: executing remote ssh commands using password
  uses: appleboy/ssh-action@master
  with:
    host: ${{ secrets.ALIYUN_HOST }}
    username: ${{ secrets.ALIYUN_USER }}
    password: ${{ secrets.ALIYUN_PASSWORD }}
    script: bash /root/del.sh
```
**注意：为了保护隐私需要将你的敏感数据保存到sercet里，然后通过${{}}形式读取**
配置方式如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200422205145538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NfODQyNDk5NDY3,size_16,color_FFFFFF,t_70)
## Github Action推送public文件夹到云服务器
```yml
- name: copy file via ssh password
  uses: appleboy/scp-action@master
  with:
    host: ${{ secrets.ALIYUN_HOST }}
    username: ${{ secrets.ALIYUN_USER }}
    password: ${{ secrets.ALIYUN_PASSWORD }}
    source: "./public"
    target: "/root"
```
在这里我们使用scp命令将public文件夹推送到云服务器上。


## 完整yml文件
```yml
name: CI

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build-and-deploy:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    - name: Checkout
      uses: actions/checkout@master
      with:
        submodules: true
        fetch-depth: 1
    
    - name: Build site
      uses: jakejarvis/hugo-build-action@master
      with:
        args: --minify
    
    - name: artifact
      uses: actions/upload-artifact@master
      with:
        name: public
        path: './public'
    
    - name: executing remote ssh commands using password
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.ALIYUN_HOST }}
        username: ${{ secrets.ALIYUN_USER }}
        password: ${{ secrets.ALIYUN_PASSWORD }}
        script: bash /root/del.sh
    
    - name: copy file via ssh password
      uses: appleboy/scp-action@master
      with:
        host: ${{ secrets.ALIYUN_HOST }}
        username: ${{ secrets.ALIYUN_USER }}
        password: ${{ secrets.ALIYUN_PASSWORD }}
        source: "./public"
        target: "/root"
```