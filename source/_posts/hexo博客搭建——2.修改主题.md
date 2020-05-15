---
title: hexo博客搭建——2.修改主题
date: 2020-05-15 20:57:00
tags:
    - 博客
---
# 主题选择
[hexo官方主题地址](https://hexo.io/themes/)
[知乎主题推荐](https://www.zhihu.com/question/24422335)

# 主题下载
无论是官方主题地址还是知乎推荐的，都需要找到相关的github地址。
这里以知乎推荐的第一个hexo-theme-next主题为例，现在的维护地址在
> https://github.com/theme-next/hexo-theme-next

进入相关github仓库复制仓库地址
![image.png](https://i.loli.net/2020/05/15/41HNUVi2ZJEetYy.png)

进入本地创建博客系统的blog文件夹下的themes文件夹，命令行输入
    ```shell script
    git clone git@github.com:theme-next/hexo-theme-next.git
    ```
这里的`git@github.com:theme-next/hexo-theme-next.git`是以当前主题为例，
实际情况下需替换为所选主题的仓库地址

# 主题生效