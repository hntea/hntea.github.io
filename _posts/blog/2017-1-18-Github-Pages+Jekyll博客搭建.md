---
layout: post
title: "Github-Pages+Jekyll博客搭建"
modified:
categories: [网页搭建]
excerpt: 该博客简述如何一步步使用 Jekyll 搭建自己的博客
tags: [Jekyll]
image:
  feature:
date: 2017-01-18T15:39:55-04:00
---

**本次搭建依赖系统环境：ubuntu14.04，搭建原理同样适合其它操作系统**

## 安装依赖环境

### 1.[<u>Ruby Install</u>](https://www.ruby-lang.org/en/documentation/installation/#apt)

- 注意版本：Ruby version >= 2.0
- 若使用这种安装方式，会有版本太老的问题：


```
sudo apt-get install ruby-full
```

- 点击上方标题进入官网主页，直接下载最新源码包并解压，进入源码目录


```
./configure
make
sudo make install
```

### 2.[<u>RubyGems Install</u>](https://rubygems.org/pages/download)

- 进入官网下载源码，解压进入目录
- 执行


```
cd rubygems-2.6.8/
sudo ruby setup.rb
```

### 3.[<u>NodeJS Install</u>](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)

- 安装最新版，执行以下命令


```
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```


### 4.Python2.7 或以上
---

##  Jekyll 安装与使用

### 1.安装 Jeklly

在终端执行：


```
sudo gem install jekyll
```

若要查看已经安装的版本号：


```
jekyll --version or
gem list jekyll
```

在某些情况下，如果版本过低，可以使用以下命令进行更新


```
gem update jekyll
```

### 2.使用 Jeklly

**1. 创建博客站点**


```
jeklly new blog
```

如提示错误信息：`jekyll 3.3.1 | Error:  bundler`,是由于该工具没有安装导致的，执行


```
gem install bundler
```

**2. 启动jeklly本地服务对博客进行预览**



- 启动服务:

```
jeklly server
```

在版本不协调时，可以使用解决`bundle exec jekyll serve`


- 打开浏览器预览 Jekyll服务默认端口是4000,具体看输出地址打开即可，或者使用下面本地网址：


```
http://localhost:4000
```


**注意：** 启动服务时应该进入刚创建的 `blog`目录中

**3. 添加博客文件**

- 进入 `blog/_posts/` 目录中,创建新的 `markdown` 文件

- 当文件博客文件创建好后，回到 `blog`目录使用:

```
jeklly build
```

编译，编译后生成的文件可以在`_site`目录下查看，重启服务，即可看到网页有更新，同样，当出现版本不协调是使**`bundle exec jekyll build`** 代替

- 注意

	-	文件命名必须严格遵守:**YYYY-MM-DD-name-of-post.xxx** 格式
	-	在 markdown 文件头部添加格式说明，以便转换成网页

---

## 将本地 Jeklly 博客部署到 GitHup Pages 上


### 1.GitHup Pages 创建

这一步比较简单，登录 githup 帐号

1.创建仓库，仓库遵守命名规则：`username.github.io`username：你在githup上的用户名，如我自己的:

```
hntea/hntea.github.io
```

2.设置仓库，随便选择主题即可


3.访问查看效果，在浏览器中输入并访问:
```
https://hntea.github.io/
```

### 2.部署

**1.克隆仓库到本地**

```
git clone https://github.com/hntea/hntea.github.io
```

**2.将上面创建的 `blog` 文件夹下的所有目录复制到仓库下**
```
cp blog/* ./hntea.githup.io/
```

**3.提交到本地仓库**

```
git add --all
git commit -m "信息记录"
```
**4.上传到 githup 仓库的主分支**

```
git push -u origin master
```
**5.在终端执行 `jeklly server` 启动服务，可查看网页是否生效**

---

## 更换主题

### 1.挑选中意的主题

首先需要感谢各路开源大师提供的各种高逼格的模板，敢肯定的是一定有一种能俘虏你的模板，耐心看看找找。

- 挑选：请戳主题大全链接：[<u>Jekyll Themes</u>](https://jekyllthemes.io/)

### 2.安装配置
  这里选择 **so-simple-theme** 主题进行示范：

1. 下载 :
`git clone https://github.com/mmistakes/so-simple-theme.git`

2. 进入本地仓库，复制主题目录下的所有文件到该目录下（若本地仓`_posts`已经有文件，记得保存好）

3. 安装执行：`bundle install` 


4. 格式化，编译执行：`bundle exec jekyll build`

5. 启动服务:`bundle exec jekyll serve`

6. 配置：直接修改配置文件 `_config.yml`在修改网页文件时可以保持服务在启动状态，每次修改完文件后，可以刷新网页查看对应的修改情况
    
7. 对于主题中的博客，如果不想要直接删除就好了。


---

## 添加博客评论

由于静态博客没有评论功能，不过想要评论功能也可以通过第三方提供的相关插件来处理。若不考虑国墙，可以使用　**Disqus**，若不想被墙，那么综合考虑下来就还是使用[多说](http://dev.duoshuo.com/docs)比较好点，当然没有**Disqus**功能好。至于如何添加，具体操作如下：

1. 进入[多说](http://duoshuo.com/)，先注册登陆
　
2. 点击我要安装，创建站点，名字取自己喜欢的。

3. 进入后台操作界面如下所示，并将代码复制过来，修改多说的中文修改部分，具体操作如下：

- 找到模板文件，如我的主题是在 _layout/post.html 文件中，找到如下代码片段（博客内容）


在尾部添加多说的代码即可，具体修改如下:

>data-thread-key 字段改成" "{\{ page.id }\}"
>
>data-title 字段改成  "{\{ page.title }\}"
>
>data-url 字段改成 "web site/\{\{ page.url \}\}"

比如：将 data-url 修改成：“http://localhost:4000/\{\{ page.url \}\}”


- 同时在 _config.yml 文件中添加如下代码

```
comments :
  provider : duoshuo
  duoshuo :
    short_name : hntea //这里是你在多说注册的站点名字
```


# 购买并绑定域名

...穷
