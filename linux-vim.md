---
title: vim编辑器的配置
date: 2016-05-31 11:41:01
categories: Linux
tags: Linux
---
一、前言
  vim是一款功能强大的文本编辑器，它在vi的基础上增加了很多的特性以及做了很多改进，而学习Linux不免要和vim打交道，本篇博客主要说的是关于vim的配置问题。

二、关于vim的基本配置
  当你看到这样的编辑器时你还想继续使用它吗？
  ![1](http://o6lb63nu0.bkt.clouddn.com/vim1.png)
  本人看到这样的编辑器是没有一点继续使用的兴趣了，那么能否将这样一个编译器配置得高大上呢？
  首先来看看vim编译器的一些基本配置：
  用vim打开~/.vimrc文件，没有的话也没关系，可以自己touch一个，在该文件中写入以下内容：
	  
      1 "显示语法高亮
	  2 syntax enable
	  3 syntax on
	  4 "取消默认注释
	  5 set paste
	  6 "关于字体颜色的配置
	  7 "colorscheme koehler
	  8 "colorscheme slate 
	  9 colorscheme torte
	 10 "显示行号
	 11 set number
	 12 "历史记录数
	 13 set history=1000
	 14 "ai 自动缩进 
	 15 set autoindent
	 16 "智能缩进
	 17 set smartindent
	 18 "括号匹配
	 19 set showmatch
	 20 "开启代码折叠 
	 21 set foldenable
	 22 "手动折叠
	 23 set fdm=manual
     24 "自动语法折叠
	 25 "set foldmethod=syntax
	 26 "默认制表符为4
	 27 set tabstop=4
	 28 set shiftwidth=4
	 29 "用浅色高亮当前行
	 30 autocmd InsertLeave * se nocul
	 31 "用浅色高亮当前行
	 32 autocmd InsertEnter * se cul
	 33 "显示标尺
	 34 set ruler
	 35 "输入的命令显示出来，看的清楚些
	 36 set showcmd
	 37 "设置在vim中可以使用鼠标  -->这个功能较鸡肋，请华丽的忽视它吧
	 38 "set mouse=a
     39 "编码设置
     40 set enc=utf-8
     41 set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936
     42 "语言设置
     43 set langmenu=zh_CN.UTF-8
	 44 set helplang=cn

  配置好之后效果图如下：
  ![2](http://o6lb63nu0.bkt.clouddn.com/vim2.png)
  是不是有一种眼前一亮的感觉。

三、vim的一些其他配置  
  前面讲的那些配置只需要在~/.vimrc文件中写入内容即可，接下来要说的配置不仅要在~/.vimrc文件中写入相关内容还需要下载安装插件。
  1.taglist安装
  下载地址：http://www.vim.org/scripts/script.php?script_id=273
  解压：unzip -d taglist taglist_xx.zip
  将解压后将doc/taglist.txt和plugin/taglist.vim中的分别放在~/.vim/doc和~/.vim/plugin两个文件夹中。接下来在~/.vimrc文件中写入以下内容：
	
     45 "*****************************************************
	 46 "                   taglist配置                      *
	 47 "*****************************************************
	 48 
	 49 "启动vim后自动打开taglist窗口
	 50 "let Tlist_Auto_Open = 1
	 51 "不同时显示多个文件的tag，仅显示一个
	 52 let Tlist_Show_One_File = 1
	 53 "，退出vim
	 54 let Tlist_Exit_OnlyWindow = 1
	 55 "taglist窗口显示在右侧
	 56 let Tlist_Use_Right_Window = 1
	 57 "设置taglist窗口大小
	 58 "let Tlist_WinHeight = 100
	 59 let Tlist_WinWidth = 25
	 60 "单击跳转
	 61 let Tlist_Use_SingClick= 1
	 62 "设置taglist打开关闭的快捷键F3
	 63 nnoremap <silent> <F3> :TlistToggle<CR>
  配置好之后在命令模式下输入：Tlist，(或者像我一样设置快捷键<F3>，直接按快捷键)结果如图所示：
  ![3](http://o6lb63nu0.bkt.clouddn.com/vim3.png)
  2.NERDTree树状结构配置
  下载地址：http://www.vim.org/scripts/script.php?script_id=1658
  解压：unzip -d NERD_tree NERD_tree.zip
  解压缩之后，把 plugin/NERD_tree.vim 和doc/NERD_tree.txt分别放到~/.vim/plugin 和 ~/.vim/doc 目录，其他文件直接放在~/.vim目录下。接下来在~/.vimrc中写入以下内容：

	 64 "********************************************************
	 65 "                      NERD_Tree 配置                   *
	 66 "********************************************************
	 67 
	 68 "显示增强
	 69 let NERDChristmasTree= 1
	 70 "自动调整焦点
	 71 let NERDTreeAutoCenter= 1
	 72 "鼠标模式:目录单击,文件双击
	 73 let NERDTreeMouseMode= 2
	 74 "打开文件后自动关闭
	 75 let NERDTreeQuitOnOpen= 1
	 76 "显示文件
	 77 let NERDTreeShowFiles= 1
	 78 "显示隐藏文件
	 79 let NERDTreeHightCursorline= 1
	 80 "显示行号
	 81 let NERDTreeShowLineNumbers= 1
	 82 "窗口位置
	 83 let NERDTreeWinPos= 'left'
	 84 "窗口宽度
	 85 let NERDTreeWinSize= 25
	 86 "不显示'Bookmarks' label 'Press ? for help'
	 87 let NERDTreeMinimalUI= 1
	 88 "快捷键
	 89 nnoremap <silent> <F4> :NERDTreeToggle<CR>
   配置好之后在命令模式下输入：NERDTree，(或者像我一样设置快捷键<F4>，直接按快捷键)结果如图所示：
   ![4](http://o6lb63nu0.bkt.clouddn.com/vim4.png)
   3.ctags的使用
   在读代码时遇到调用的函数就需要到函数定义的地方看看函数实现的功能，而当代码很多时就不能很快跳转到函数定义的位置，而ctags插件就帮我们解决了这一问题。
   下载安装：
   方法一、直接输入命令：yum install ctags
   方法二、下载压缩包解压安装：
         下载地址：http://www.vim.org/scripts/script.php?script_id=610
         解压：tar -xzvf ctags-xx.tar.gz
         输入命令：cd ctags-xx
                  make
                  make install   // 需要root权限
   在程序所在的目录下输入命令：ctags -R
   在该目录下会生成一个tags文件
   此时用vim打开文件通过按"ctrl+]"和"ctrl+T"进行跳转。
   如图：
   ![5](http://o6lb63nu0.bkt.clouddn.com/vim5.png)
   按"ctrl+]"跳转到函数定义的位置
   ![6](http://o6lb63nu0.bkt.clouddn.com/vim6.png)
   按"ctrl+T"回到函数调用的位置
   ![7](http://o6lb63nu0.bkt.clouddn.com/vim7.png)
   这样配置之后是不是感觉这个文本编辑器相要美观很多。
