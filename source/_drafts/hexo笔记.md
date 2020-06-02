---
title: hexo笔记
tags:
    - 博客
description: 仅用来暂时记录一些小问题，后边会整理成文
---
1. 写deploy时候不能换行，不然会报错，没法只显示description
`YAMLException: can not read a block mapping entry; a multiline key may not be an implicit key at line 8, column 1:`
![deploy换行报错](hexo笔记/deploy换行报错.jpg)
2. 写概述https://blog.csdn.net/yueyue200830/article/details/104470646/
3. FATAL Something's wrong. Maybe you can find the solution here: https://hexo.io/docs/troubleshooting.html
   Nunjucks Error:  [Line 40, Column 95] unexpected token: .
   
   报错：
   ![](hexo笔记/特殊字符报错.jpg)
   报错代码：
   ![](hexo笔记/特殊字符报错source.jpg)
   解决方式:https://blog.csdn.net/weixin_34209406/article/details/88986783
   http://www.w3school.com.cn/html/html_entities.asp