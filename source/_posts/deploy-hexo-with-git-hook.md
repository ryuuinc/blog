---
title: 使用 Git Hook 部署 Hexo 到 VPS
date: 2018-04-14 16:17:24
categories:
  - Unix
tags:
  - Hexo
index_img: /img/deploy-hexo-with-git-hook/banner.png
---

装好 Hexo 以后，总是忍不住写点什么，刚好想到可以把自动部署的过程拿出来说一下。网上很多教程都是教你如何将 Hexo 部署到 `Github Pages` 上，但是今天要说的是把 Hexo 部署到自己的 VPS 上，参考了一些教程，写一个我觉得不错的办法。

<!-- more -->

### 配置 VPS 的 Git 环境

由于我的 VPS 上是 `Ubuntu 18.04`，所以说下我的步骤，其他系统类似。

### 第一步

安装 Git:

```bash
$ apt-get install git
```

### 第二步

设置 SSH 通过密钥登录，具体步骤可以参考[这里](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)

### 第三步

初始化 Git 仓库，我将其放在 `/root/hexobot/blog.git` 目录下：

```bash
$ mkdir /root/hexobot && cd /root/hexobot
$ git init --bare blog.git
```

使用 --bare 参数，Git 就会创建一个裸仓库，裸仓库没有工作区。

### 第四步

配置 `Git Hooks`，详情可参考[这里](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)。这里使用的是 `post-receive`，这个 Hook 会在 git 操作完后被运行。

```bash
$ cd /root/hexobot/blog.git/hooks
$ vim post-receive
```

在 `post-receive` 中写入如下内容：

```bash
#!/bin/sh
git --work-tree=/home/wwwroot/blog --git-dir=/root/hexobot/blog.git checkout -f
```

注意，`/home/wwwroot/blog` 要换成你自己的部署目录，上面的命令会在每次 push 以后把部署目录更新到最新的版本，当然，不要忘记给这个脚本提升权限。

```bash
$ chmod +x post-receive
```

### 本地配置

修改 hexo 目录下的 `_config.yml` 文件，找到 deploy 条目，并修改为：

```bash
deploy:
  type: git
  repo: git@xxx.xxx.xxx.xxx:/root/hexobot/blog.git
  branch: master
```

要注意换成你自己的服务器地址，以及服务器端 git 仓库的目录。至此，我们的 hexo 自动部署已经全部配置好了。

> 有些情况下，hexo-deployer-git 没有被安装，请自行安装。

### 使用

```bash
$ hexo new "post"

# write something ....

$ hexo clean && hexo g -d
```

当然了，如果你是 zsh 用户，你还可以将这个 alias 加入 `~/.zshrc` 中：

```bash
alias Post='hexo clean && hexo g -d'
```

Finally，just enjoy~
