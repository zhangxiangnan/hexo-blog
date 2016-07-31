---
title: 使用hexo与next主题搭建个人博客
date: 2016-05-10 17:56:39
tags:
- blog
- hexo
- next theme
categories: blog
---

借助于github的github pages，通过hexo创建个人博客，本文描述了如何搭建，以及常见的问题及经验。

<!-- more -->

### 本人环境配置
    hexo: 3.2.0
    nodejs:v4.2.1
    npm: 2.14.5
    git: 1.9.5.msysgit.1
    windows:win7 x64


### 环境配置
* [安装git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)
* [安装nodejs](https://nodejs.org/en/download/)
* [安装node的npm包](https://github.com/nodejs-tw/nodejs-wiki-book/blob/master/zh-tw/node_npm.rst)

### 开发配置
#### 快速搭建博客骨架  
**博客目录d:/blog，以下命令均在blog目录下执行：**

    //全局安装hexo
    $ npm install -g hexo

    // 创建个目录，命令行进入该目录，执行命令，初始化博客目录
    $ hexo init

    // 新版本hexo自动安装依赖，旧版本估计需要执行如下：
    $ npm install

    // 生成静态页面，简写hexo g
    $ hexo generate

    // 本地预览，访问localhost:4000
    hexo server

[点击查看本地预览](http://localhost:4000)



// hexo高版本用来安装git部署器，发布博客到github用
$ npm install hexo-deployer-git --save


### 博客部署到github上
#### 安装git的部署器
**博客目录d:/blog，以下命令均在blog目录下执行：**

    // hexo高版本用来安装git部署器，发布博客到github用
    $ npm install hexo-deployer-git --save


#### github上创建项目
  ##### 项目名称注意
  github上必须创建username.github.io的repository名，才能当作博客，username为你的github用户名。

  ##### 创建时勾选 Initialize this repository with a README
  勾选该复选框，可以立即克隆仓库到本地。

  ##### github项目设置deploy keys
  选择刚创建的项目，settings->左边deploy keys->添加本机的ssh公钥即可

  #### 站点配置文件_config.xml配置git信息
      //在deploy节点下配置type类型(git、老版本貌似是github)、仓库地址、分支
      deploy:
      type: git
      repository: https://github.com/zhangxiangnan/zhangxiangnan.github.io.git
      branch: master

  ##### 克隆项目到本地
  克隆刚创建的项目到本地，把.git文件夹拷贝到博客目录下，或者相反都行，然后即可提交到github上。

    $ git clone 仓库地址

  #### 提交项目到github上
      $ hexo clean
      $ hexo generate
      $ hexo deploy  #之后输入用户名、密码

  deploy操作会把blog目录下的public文件夹下的内容发布到github上刚创建的项目。

  ### 主题及其设定、常见第三方服务见:
  http://theme-next.iissnan.com/getting-started.html

### 常用命令

 #### 清理项目


  清理项目
  hexo clean

#### 构建生成项目
hexo generate 或者hexo g

#### 本地预览项目
hexo server

#### 新建文章
hexo new title "xx"
