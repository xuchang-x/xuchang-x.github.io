---
title: hexo博客搭建——2.修改主题
date: 2020-05-15 20:57:00
tags:
    - 博客
description: 自己在github上搭建博客而没有选择首先在博客平台上的其中一个原因就是，相对而言在github上搭博客自己能控制的相对多一些，比如...找一个好看的皮肤～
---
# 主题选择
[hexo官方主题地址](https://hexo.io/themes/)
[知乎主题推荐](https://www.zhihu.com/question/24422335)

# 主题下载
无论是官方主题地址还是知乎推荐的，都需要找到相关的github地址。
这里以知乎推荐的第一个hexo-theme-next主题为例，现在的维护地址在
> https://github.com/theme-next/hexo-theme-next

进入相关github仓库复制仓库地址
![github clone位置](github%20clone位置.jpg)

进入本地创建博客系统的blog文件夹下的themes文件夹，命令行输入
    ```shell script
    git clone git@github.com:theme-next/hexo-theme-next.git
    ```
这里的`git@github.com:theme-next/hexo-theme-next.git`是以当前主题为例，
实际情况下需替换为所选主题的仓库地址

# 主题生效
打开blog文件夹下_config.yml文件，修改theme配置
![theme配置](主题配置文件位置.jpg)
这里主题为下载下来主题所在的文件夹名
![文件夹名](文件夹名.jpg)
配置好之后重新执行
    ```shell script
    hexo g
    hexo s
    ```
即可本地浏览新主题效果
    ```shell script
    hexo d
    ```
即可部署到github
![新主题](Muse主题.jpg)
## next主题
这里使用的next主题，这个主题下有四个子主题，可以通过修改themes/hexo-theme-next
下的_config.yml实现子主题变更，next主题本身相关的一些配置也都在这个配置文件中。

这里只介绍修改子主题

进入配置文件找到schemes配置项，启用其中一个即可
![schemes](子主题shceme.jpg)
配置后重新生成主题变为如下效果
![Muse子主题](Mist主题.jpg)
