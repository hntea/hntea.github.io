---
layout: post
title: "Github-Pages与Jekyll博客搭建"
modified:
categories: blog
excerpt:
tags: []
image:
  feature:
date: 2017-01-18T15:39:55-04:00
---

**本次搭建依赖系统环境：ubuntu14.04，搭建原理同样适合其它操作系统**

# 安装依赖环境

### 1.[Ruby Install](https://www.ruby-lang.org/en/documentation/installation/#apt)

- 注意版本：Ruby version >= 2.0
- 若使用这种安装方式，会有版本太老的问题：


```
sudo apt-get install ruby-full
```

- 直接下载最新源码包并解压，进入源码目录


```
./configure
make
sudo make install
```

### 2.[RubyGems Install](https://rubygems.org/pages/download)

- 进入官网下载源码，解压进入目录
- 执行


```
cd rubygems-2.6.8/
sudo ruby setup.rb
```

### 3.[NodeJS Install](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions)

- 安装最新版，执行以下命令


```
curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
sudo apt-get install -y nodejs
```


### 4.Python2.7 或以上
---

#  Jekyll 安装与使用

## 1.安装 Jeklly

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

## 2.使用 Jeklly

**1. 创建博客站点**
```
jeklly new blog
```
如提示错误信息：`jekyll 3.3.1 | Error:  bundler`,是由于该工具没有安装导致的，执行
```
gem install bundler
```

**2. 启动jeklly本地服务对博客进行预览**

- 启动服务:`jeklly server`

- 打开浏览器预览 Jekyll服务默认端口是4000,具体看输出地址打开即可，或者使用下面本地网址：
```
http://localhost:4000
```
**注意：** 启动服务时应该进入刚创建的 `blog`目录中

**3. 添加博客文件**

- 进入 `blog/_posts/` 目录中,创建新的 `markdown` 文件

- 当文件博客文件创建好后，回到 `blog`目录使用 `jeklly build` 编译，编译后生成的文件可以在`_site`目录下查看，重启服务，即可看到网页有更新
- 注意

	-	文件命名必须严格遵守:**YYYY-MM-DD-name-of-post.xxx** 格式
	-	在 markdown 文件头部添加格式说明，以便转换成网页

---

# 将本地 Jeklly 博客部署到 GitHup Pages 上


## 1.GitHup Pages 创建

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

## 2.部署

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

# 更换主题

-	挑选：主题大全：[Jekyll Themes](https://jekyllthemes.io/)
-	这里选择 **so-simple-theme** 主题进行示范：
	-	下载 `git clone https://github.com/mmistakes/so-simple-theme.git`

	-	进入本地仓库，复制主题目录下的所有文件到该目录下（若本地仓`_posts`已经有文件，记得保存好）

	-	安装，执行：`bundle install` 并配置 `_config.yml`
	-	格式化，编译执行：`bundle exec jekyll build`

	-	本地服务查看：`bundle exec jekyll serve`;在修改网页文件时可以保持服务在启动状态，每次修改完文件后，可以刷新网页查看对应的修改情况
	-	对于主题中的博客，如果不想要直接删除就好了。


# 购买并绑定域名

...穷
