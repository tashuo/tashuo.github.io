---
title: vim常用命令
date: 2016-12-19 19:05:28
categories:
- 笔记
- 编程
tags: vim
---

vim常用命令

*   列编辑  
鼠标移动到对应位置，ctrl+v进入列编辑选中要修改的块，shift+i进入编辑模式，输入内容，第一行变化，连续按两次 esc 键，刚才列编辑选中的所有行都发生变化

*   折叠与展开
    -   zi 打开关闭折叠
    -   zv 查看此行
    -   zm 关闭折叠
    -   zM 关闭所有
    -   zr 打开
    -   zR 打开所有
    -   zc 折叠当前行
    -   zo 打开当前折叠
    -   zd 删除折叠
    -   zD 删除所有折叠

*   代码缩进与格式化    
根据语言特征使用自动缩进排版：在命令状态下对当前行用== （连按=两次）, 或对多行用n==（n是自然数）表示自动缩进从当前行起的下面n行

*   翻页与移动   
    - page up: ctrl+b
    - page down: ctrl+f
    - 行首：0
    - 行尾：$
    - 定位到第几行：数字＋G
    - 文件头部：gg
    - 文件尾部：G
<!-- more -->
*   待续...
