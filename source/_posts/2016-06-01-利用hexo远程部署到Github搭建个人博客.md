---
title: 利用hexo远程部署到Github搭建个人博客
date: 2016-05-31 21:15:31
tags: 搭建个人博客
---

## 搭建准备工作
* 搭建git环境，推荐[廖雪峰的git教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/00137396287703354d8c6c01c904c7d9ff056ae23da865a000)
* 安装Node.js，推荐[菜鸟教程关于Node.js部分](http://www.runoob.com/nodejs/nodejs-install-setup.html)
* 拥有一个[Github账号](https://github.com/)

<!-- more -->

## 安装hexo并初始化
在想要建站的位置右键打开菜单选择 'Git Bash Here'，并输入
```
$ npm install -g hexo-cli #安装指令```

在执行完安装指令之后继续执行以下初始化命令

```
hexo init .  #. 表示在当前文件夹执行初始化
npm install```


## 相关配置
hexo中最常用的是主题设置，其他还有网站相关属性、网站的设置、新文章的默认配置、分类、标签设置等
### 主题设置
修改``_config.yml`` 中的``theme:``属性可修改主题样式。
推荐主题可阅读[来自知乎的hexo主题推荐](https://www.zhihu.com/question/24422335)
PS：当前blog使用的是 [next](http://theme-next.iissnan.com/)
### 生成标签页
新建一个名字为tags的页面：
```
$ hexo new page "tags"```

在生成的页面的[Front-matter](https://hexo.io/zh-cn/docs/front-matter.html)加上
```
type: "tags"
#如果有加上多说或者其他评论系统，请加上以下代码以在本页面关闭评论系统
comments: false```

在添加完之后在主题配置文件的menu节点加上tags后重新部署即可完成
```
menu:
  home: /
  archives: /archives
  tags: /tags```
  
### 其他配置
可参考[Hexo的说明文档](https://hexo.io/zh-cn/docs/configuration.html)进行配置

## 部署
hexo可以部署到本地调试，也可以部署到远程服务器
### 部署到本地
在执行完hexo的初始化之后可执行下面的命令运行

```
hexo g 
hexo s```

在执行完之后在浏览器输入地址``http://localhost:4000/``。

### 部署到Github
#### 前置条件

本机的ssh已加到Github上，[添加ssh教程](http://blog.csdn.net/binyao02123202/article/details/20130891)。

#### 开始部署
在Github上新建一个项目（repository）并配置``_config.yml``文件的``deploy:``属性

```bash
## Deployment
### Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: ##这里为项目的Git地址
  branch: gh-pages ##要部署到的项目的分支(branch)
```

在配置完成之后执行以下命令运行

```
hexo g
hexo d```

完成后在浏览器输入``_config.yml``中配置的``url:``即可打开个人博客

## F&Q
* F：为什么在本地上部署成功，而远程部署到Github时会出现没有格式的情况？
  Q：必须在``_config.yml``中设置对应的``url:``和``root: ``属性，其中``url:``必须为完整的网站地址，``root: ``网站除去站点地址后面的那一部分，比如：我的blog设置为 ``url: http://naughtboy.github.io/blog/ root: /blog/``

* F：怎么不让在首页的时候整篇文章都显示出来？
  Q：文章截断，在需要截断的地方插入 ``<!--more-->``
  