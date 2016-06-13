---
title: Linux find命令和atime/ctime/mtime
date: 2016-05-19 15:40:28
categories: Linux
tags: Linux
---
一、Linux中的find命令
  1.find命令的一般形式为：
   find pathname -options [-print -exec -ok ...]

  2.find命令的参数
   pathname:find命令所查找的目录路径。例如用.来表示当前目录，用/来表示根目录。
   -print:find命令将匹配的文件输出到标准输出。
   -exec:find命令对匹配的文件执行该参数所给出的shell命令。相应的命令形式为'command'{} \;，注意{}和\;之间的空格。
   -ok:和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

    #print将查找到的文件输出到标准输出
    #-exec command {} \; ——将查到的文件执行command操作
    #-ok和-exec相同，只不过在操作前要询问用户

  3.find命令选项
    -name           
     按照文件名查文件
     ![1](http://o6lb63nu0.bkt.clouddn.com/find1.png)
    -perm         
     按照文件权限来查找文件
     ![2](http://o6lb63nu0.bkt.clouddn.com/find2.png)
    -prune        
     使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用-depth选项，那么-prune将被find命令忽略
     ![3](http://o6lb63nu0.bkt.clouddn.com/find3.png)
    -user         
     按照文件属主来查找文件
     ![4](http://o6lb63nu0.bkt.clouddn.com/find4.png)
    -nouser       
     查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在
    -group        
     按照文件所属组来查找文件
     ![5](http://o6lb63nu0.bkt.clouddn.com/find5.png)
    -nogroup      
     查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在
    -mtime -n +n  
     按照文件的更改时间来查找文件，-n表示文件更改时间在距现在n天以内，+n表示文件更改时间距现在n天以前。find命令还有-atime和-ctime选项
     ![6](http://o6lb63nu0.bkt.clouddn.com/find6.png)
    -newer file1 ! -newer file2
     查找更改时间比文件file1新但比文件file2旧的文件
     ![7](http://o6lb63nu0.bkt.clouddn.com/find7.png)
    -type
     查找某一文件类型，如:
     b - 块设备文件
     d - 目录
     c - 字符设备文件
     l - 符号链接文件
     f - 普通文件
     ![8](http://o6lb63nu0.bkt.clouddn.com/find8.png)
    -size n:[c]
     查找文件长度为n块的文件，带有c时表示文件长度以字节计
     ![9](http://o6lb63nu0.bkt.clouddn.com/find9.png)
    -depth
     在查找文件时，首先查找当期目录中的文件，然后再在其子目录中查找
     ![10](http://o6lb63nu0.bkt.clouddn.com/find10.png)
    -fstype
     查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件/etc/ fstab中找到，该配置文件中包含了本系统中有关文件系统的信息
    -mount
     在查找文件时不跨越文件系统mount点
     ![11](http://o6lb63nu0.bkt.clouddn.com/find11.png)
    -follow
     如果finder命令遇到符号链接文件，就跟踪至链接所指向的文件
    -cpio
     对配对文件使用cpio命令，将这些文件备份到磁带设备中
    -atime/-ctime/-mtime与-amin/-cmin/-mmin的区别
     -atime n   ——查找系统中最后n*24小时访问的文件
     -amin n    ——查找系统中最后n分钟访问的文件
     -ctime n   ——查找系统中最后n*24小时被改变文件状态的文件
     -cmin n    ——查找系统中最后n分钟被改变文件状态的文件
     -mtime n   ——查找系统中最后n*24小时被改变文件数据的文件
     -mmin n    ——查找系统中最后n分钟被改变文件数据的文件

  4.使用exec或ok来执行shell命令
    使用find命令时，只要把想要的操作写在一个文件里，就可以用exec来配合find命令查找，很方便。
    在有些操作系统中只允许-exec选项执行诸如ls或ls -l这样的命令。大多数用户使用这一选项是为了查找旧文件并删除它们。建议在真正执行rm命令删除文件之前，最好先用ls命令看一下，确认它们是所要删除的文件。
    exec选项后面跟随着所要执行的命令或脚本，然后是一对儿{ }，一个空格和一个\，最后是一个分号。为了使用exec选项，必须要同时使用print选项。如果验证一下find命令，会发现该命令只输出从当前路径起的相对路径及文件名。

二、Linux中atime/ctime/mtime
  使用stat 命令来查询文件的inode信息，其中包括ctime atime mtime
  mtime：文件内容改变，ctime文件的状态改变，如chmod权限等。
  调整mtime，ctime会变。调整ctime，mtime不一定变化。
 
  1.文件的atime/ctime/mtime

    文件的 Access time，atime 是在读取文件或者执行文件时更改的任何对inode的访问都会使此处改变。
    文件的Modified time，mtime 是在写入文件时随文件内容的更改而更改的。
    文件的 Change time，ctime 是在写入文件、更改所有者、权限或链接设置时随 Inode 的内容更改而更改的。只要stat出来的内容发生改变就会发生改变。mtime的改变必然导致ctime的改变。
 
    mtime （modification time ）：在写入文件时随文件内容的更改而更改的时间。我们用ls -l看到的时间，就是mtime
    ctime （status time）：是在写入文件、更改所有者、权限或链接设置时随Inode的内容更改而更改的时间。相当于ls -l –time=ctime所看到的时间
    atime （access time）：读取文件或者执行文件时更改的时间。也就是用ls -l –time=atime看到的时间
 
    1）modification time (mtime,修改时间)：这个时间指的是文件内容修改的时间，而不是文件属性的修改，当数据内容修改时，这个时间就会改变，用命令ls -l默认显示的就是这个时间：
    2）status time （ctime,状态时间）：当一个文件的状态改变时，这个时间就会改变，例如更改了文件的权限与属性等，它就会改变。
    3）access time （atime,访问时间）：当读取文件内容时，就会更改这个时间，例如使用cat去读取/etc/man.config,那么该文件的atime就会改变。

  2.文件夹的atime/ctime/mtime

    文件夹的 Access time，atime 是在读取文件或者执行文件时更改的（我们只cd进入一个目录然后cd ..不会引起atime的改变，但ls一下就不同了）。
    文件夹的 Modified time，mtime 是在文件夹中有文件的新建、删除才会改变（如果只是改变文件内容不会引起mtime的改变，换句话说如果ls -f <directory>的结果发生改变mtime就会被刷新。这里可能有人要争论了：我进入dd这个文件夹vi了一个文件然后退出，前后ls -f <directory>的结果没有改变但是文件夹的mtime发生改变了……这点请主意vi命令在编辑文件时会在本文件夹下产生一 个".file.swp"临时文件，该文件随着vi的退出而被删除……这就导致了mtime的改变）。
    文件夹的 Change time，ctime 基本同文件的ctime，其体现的是inode的change time。

    使用find命令时，常为其中的atime/ctime/mtime感到困惑，一直没彻底弄个明白，今天仔细看了以上两篇文章，总算有所领悟，总结如下：
    (1)、含义：
       文件的 Access time，atime 是在读取文件或者执行文件时更改的；
       文件的 Modified time，mtime 是在写入文件时随文件内容的更改而更改的；
       文件的 Create time，ctime 是在写入文件、更改所有者、权限或链接设置时随 Inode 的内容更改而更改的。
    (2)、文件各种事件标记的显示方法
       ls -lc filename         列出文件的 ctime 
       ls -lu filename         列出文件的 atime 
       ls -l filename          列出文件的 mtime 

  参数说明
  -a ：修改atime
  -m ：修改mtime
  -c ：仅修改文件的时间（三个时间一起修改），若该文件不存在则不建立新的文件
  -d ：后面可以接想修改的日期而不用目前的日期，也可以使用 –date=”日期或时间”
  -t ：后面可以接想修改是时间而不用目前的时间，格式为[YYMMDDhhmm]     