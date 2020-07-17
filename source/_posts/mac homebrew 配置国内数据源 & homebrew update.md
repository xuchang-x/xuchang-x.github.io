---
title: mac homebrew 配置国内数据源 & homebrew update
date: 2020-07-17 19:59:00
tags:
    - mac
    - homebrew
description: 最近项目需要依赖python3，需要安装才发现homebrew需要升级，一直用国外数据源又总拉不回来数据升级失败。这里就解决一下homebrew升级和切换数据源的问题。
---
# homebrew升级
## 执行脚本
```shell script
cd "$(brew --repo)"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
brew update
```
执行第三步时候可能会没有相关路径，创建即可

## 报错解决
1. 报错
```text
fatal: unable to access 'https://github.com/Homebrew/homebrew-core/': LibreSSL SSL_read: SSL_ERROR_SYSCALL, errno 54
```

解决：

执行如下下面这句命令，更换为中科院的镜像：
```shell script
git clone git://mirrors.ustc.edu.cn/homebrew-core.git/ /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core --depth=1
```

把homebrew-core的镜像地址也设为中科院的国内镜像
```shell script
brew update
```
即可解决。后边安装python3过程中还是有一些问题，切换成阿里巴巴的地址

## 切换ali数据源
1. brew.git
替换成阿里巴巴的 brew.git 仓库地址:
```shell script
cd "$(brew --repo)" 
git remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git 
```
还原为官方提供的 brew.git 仓库地址
```shell script
cd "$(brew --repo)" 
git remote set-url origin https://github.com/Homebrew/brew.git
```

2. homebrew-core.git 
替换成阿里巴巴的 homebrew-core.git 仓库地址:
```shell script
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core" 
git remote set-url origin https://mirrors.aliyun.com/homebrew/homebrew-core.git 
```
还原为官方提供的 homebrew-core.git 仓库地址
```shell script
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core" 
git remote set-url origin https://github.com/Homebrew/homebrew-core.git
```

TODO:

1. 这里解决更换中科院镜像的时候是下载了一个镜像仓库，为什么这样就可以了。
2. 先下载了中科院的git仓库又执行了切换阿里homebrew-core的命令，对这个仓库中的文件做了什么，阿里有没有仓库以供直接下载。