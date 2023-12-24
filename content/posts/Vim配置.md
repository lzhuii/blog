---
title: 'Vim 配置文件'
date: '2023-12-15'
tags: ['Linux','Vim']
---


```VimScript
set number "显示行号
set relativenumber "显示相对行号
set ruler "显示标尺
set showcmd "显示命令
set nocompatible "取消vi一致性
set wildmenu "增强模式中的命令行自动完成操作
set showmatch "高亮匹配括号
set ignorecase "搜索忽略大小写
set nohlsearch "不要高亮被搜索的句子
set incsearch "在搜索时，输入的词句的逐字符高亮
set autoindent "继承前一行的缩进方式
set smartindent "为C程序提供自动缩进
set cindent "使用C样式的缩进
set tabstop=4 "制表符
set softtabstop=4 "统一缩进
set shiftwidth=4 "统一缩进
set nowrap "不要换行
set smarttab "在行和段开始处使用制表符
 
syntax on "语法高亮
filetype on "侦测文件类型
filetype plugin on "侦测插件文件类型
```