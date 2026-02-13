---
layout: post
title: "Linux 基础命令学习"
subtitle: "掌握 Linux 系统中最常用的基础命令"
date: 2026-02-13
author: "b0t"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - Linux
---

今天学习一些基础的指令
# ls

显示目录内容列表

## 补充说明

**ls命令** 就是list的缩写，用来显示目标列表，在Linux中是使用率较高的命令。ls命令的输出信息可以进行彩色加亮显示，以分区不同类型的文件。

### 语法

```shell
ls [选项] [文件名...]
   [-1abcdfgiklmnopqrstuxABCDFGLNQRSUX] [-w cols] [-T cols] [-I pattern] [--full-time] 
   [--format={long,verbose,commas,across,vertical,single-col‐umn}] 
   [--sort={none,time,size,extension}] [--time={atime,access,use,ctime,status}] 
   [--color[={none,auto,always}]] [--help] [--version] [--]
```

### 选项

```shell
-C     # 多列输出，纵向排序。
-F     # 每个目录名加 "/" 后缀，每个 FIFO 名加 "|" 后缀， 每个可运行名加“ * ”后缀。
-R     # 递归列出遇到的子目录。
-a     # 列出所有文件，包括以 "." 开头的隐含文件。
-c     # 使用“状态改变时间”代替“文件修改时间”为依据来排序（使用“-t”选项时）或列出（使用“-l”选项时）。
-d     # 将目录名像其它文件一样列出，而不是列出它们的内容。
-i     # 输出文件前先输出文件系列号（即 i 节点号: i-node number）。 -l  列出（以单列格式）文件模式
       # （file mode），文件的链接数，所有者名，组名，文件大小（以字节为单位），时间信息，及文件名。
       # 缺省时，时间信息显示最近修改时间；可以以选项“-c”和“-u”选择显示其它两种时间信息。对于设备文件，
       # 原先显示文件大小的区域通常显示的是主要和次要的信号（majorand minor device numbers）。
-q     # 将文件名中的非打印字符输出为问号。（对于到终端的输出这是缺省的。）
-r     # 逆序排列。
-t     # 按时间信息排序。
-u     # 使用最近访问时间代替最近修改时间为依据来排序（使用“-t”选项时）或列出（使用“-l”选项时）。
-1     # 单列输出。
-1, --format=single-column  # 一行输出一个文件（单列输出）。如标准输出不是到终端，此选项就是缺省选项。
-a, --all # 列出目录中所有文件，包括以“.”开头的文件。
-b, --escape # 把文件名中不可输出的字符用反斜杠加字符编号(就像在 C 语言里一样)的形式列出。
-c, --time=ctime, --time=status
      # 按文件状态改变时间（i节点中的ctime）排序并输出目录内
      # 容。如采用长格式输出（选项“-l”），使用文件的状态改
      # 变时间取代文件修改时间。【译注：所谓文件状态改变（i节
      # 点中以ctime标志），既包括文件被修改，又包括文件属性（ 如所有者、组、链接数等等）的变化】
-d, --directory
      # 将目录名像其它文件一样列出，而不是列出它们的内容。
-f    # 不排序目录内容；按它们在磁盘上存储的顺序列出。同时启 动“ -a ”选项，如果在“ -f ”之前存在“ -l”、
      # “ - -color ”或“ -s ”，则禁止它们。
-g    # 忽略，为兼容UNIX用。
-i, --inode
      # 在每个文件左边打印  i  节点号（也叫文件序列号和索引号:  file  serial  number and index num‐
      # ber）。i节点号在每个特定的文件系统中是唯一的。
-k, --kilobytes
      # 如列出文件大小，则以千字节KB为单位。
-l, --format=long, --format=verbose
      # 输出的信息从左到右依次包括文件名、文件类型、权限、硬链接数、所有者名、组名、大小（byte）
      # 、及时间信息（如未指明是其它时间即指修改时间）。对于6个月以上的文件或超出未来
      # 1小时的文件，时间信息中的时分将被年代取代。
      # 每个目录列出前，有一行“总块数”显示目录下全部文件所占的磁盘空间。块默认是1024字节；
      # 如果设置了 POSIXLY_CORRECT 的环境变量，除非用“-k”选项，则默认块大小是 512 字节。
      # 每一个硬链接都计入总块数（因此可能重复计数），这无 疑是个缺点。

# 列出的权限类似于以符号表示（文件）模式的规范。但是 ls
      # 在每套权限的第三个字符中结合了多位（ multiple bits ） 的信息，如下： s 如果设置了  setuid
      # 位或 setgid   位，而且也设置了相应的可执行位。 S 如果设置了 setuid 位或 setgid
      # 位，但是没有设置相应的可执行位。 t 如果设置了  sticky  位，而且也设置了相应的可执行位。  T
      # 如果设置了 sticky 位，但是没有设置相应的可执行位。              x
      # 如果仅仅设置了可执行位而非以上四种情况。 - 其它情况（即可执行位未设置）。
-m, --format=commas
      # 水平列出文件，每行尽可能多，相互用逗号和一个空格分隔。
-n, --numeric-uid-gid
      # 列出数字化的 UID 和 GID 而不是用户名和组名。
-o    #  以长格式列出目录内容，但是不显示组信息。等于使用“         --format=long          --no-group
      # ”选项。提供此选项是为了与其它版本的 ls 兼容。
-p    #  在每个文件名后附上一个字符以说明该文件的类型。类似“ -F ”选项但是不 标示可执行文件。
-q, --hide-control-chars
      # 用问号代替文件名中非打印的字符。这是缺省选项。
-r, --reverse
      # 逆序排列目录内容。
-s, --size
      # 在每个文件名左侧输出该文件的大小，以    1024   字节的块为单位。如果设置了   POSIXLY_CORRECT
      # 的环境变量，除非用“ -k ”选项，块大小是 512 字节。
-t, --sort=time
      # 按文件最近修改时间（ i 节点中的 mtime ）而不是按文件名字典序排序，新文件 靠前。
-u, --time=atime, --time=access, --time=use
      # 类似选项“    -t    ”，但是用文件最近访问时间（    i     节点中的     atime     ）取代文件修
      # 改时间。如果使用长格式列出，打印的时间是最近访问时间。
-w, --width cols
       # 假定屏幕宽度是      cols      （      cols     以实际数字取代）列。如未用此选项，缺省值是这
       # 样获得的：如可能先尝试取自终端驱动，否则尝试取自环境变量          COLUMNS          （如果设
       # 置了的话），都不行则取 80 。

-x, --format=across, --format=horizontal
       # 多列输出，横向排序。

-A, --almost-all
       # 显示除 "." 和 ".." 外的所有文件。

-B, --ignore-backups
       # 不输出以“ ~ ”结尾的备份文件，除非已经在命令行中给出。

-C, --format=vertical
       # 多列输出，纵向排序。当标准输出是终端时这是缺省项。使用命令名 dir 和 d 时， 则总是缺省的。

-D, --dired
       # 当采用长格式（“-l”选项）输出时，在主要输出后，额外打印一行：  //DIRED//  BEG1 END1 BEG2
       # END2 ...

# BEGn 和 ENDn 是无符号整数，记录每个文件名的起始、结束位置在输出中的位置（
#        字节偏移量）。这使得          Emacs          易于找到文件名，即使文件名包含空格或换行等非正
#        常字符也无需特异的搜索。
# 
# 如果目录是递归列出的（“ -R ”选项），每个子目录后列出类似一行：
       # //SUBDIRED//  BEG1 END1 ...  【译注：我测试了 TurboLinux4.0 和 RedHat6.1 ，发现它们都是在 “
       # //DIRED//     BEG1...     ”之后列出“     //SUBDIRED//     BEG1     ...      ”，也即只有一个
       # 而不是在每个子目录后都有。而且“ //SUBDIRED// BEG1 ... ”列出的是各个子目 录名的偏移。】

-F, --classify, --file-type
       # 在每个文件名后附上一个字符以说明该文件的类型。“  * ”表示普通的可执行文件； “ / ”表示目录；“
       # @ ”表示符号链接；“ | ”表示FIFOs；“ = ”表示套接字 (sockets) ；什么也没有则表示普通文件。

-G, --no-group
       # 以长格式列目录时不显示组信息。

-I, --ignorepattern
       # 除非在命令行中给定，不要列出匹配shell文件名匹配式（pattern ，不是指一般
       # 表达式）的文件。在shell中，文件名以"."起始的不与在文件名匹配式(pattern)
       # 开头的通配符匹配。

-L, --dereference
       # 列出符号链接指向的文件的信息，而不是符号链接本身。

-N, --literal
       # 不要用引号引起文件名。

-Q, --quote-name
       # 用双引号引起文件名，非打印字符以 C 语言的方法表示。

-R, --recursive
       # 递归列出全部目录的内容。

-S, --sort=size
       # 按文件大小而不是字典序排序目录内容，大文件靠前。

-T, --tabsize cols
       # 假定每个制表符宽度是 cols 。缺省为 8。为求效率， ls 可能在输出中使用制表符。  若 cols 为
       0，则不使用制表符。

-U, --sort=none
       # 不排序目录内容；按它们在磁盘上存储的顺序列出。（选项“-U”和“-f”的不
       # 同是前者不启动或禁止相关的选项。）这在列很大的目录时特别有用，因为不加排序
       # 能显著地加快速度。

-X, --sort=extension
       # 按文件扩展名（由最后的 "." 之后的字符组成）的字典序排序。没有扩展名的先列 出。

--color[=when]
       # 指定是否使用颜色区别文件类别。环境变量  LS_COLORS  指定使用的颜色。如何设置 这个变量见 dir‐
       # colors(1) 。 when 可以被省略，或是以下几项之一：
none # 不使用颜色，这是缺省项。
       # auto 仅当标准输出是终端时使用。 always 总是使用颜色。指定 --color 而且省略 when  时就等同于
       # --color=always 。

--full-time
       # 列出完整的时间，而不是使用标准的缩写。格式如同          date(1)          的缺省格式；此格式
       # 是不能改变的，但是你可以用 cut(1) 取出其中的日期字串并将结果送至命令 “ date -d ”。

# 输出的时间包括秒是非常有用的。（ Unix 文件系统储存文件的时间信息精确到秒，
       # 因此这个选项已经给出了系统所知的全部信息。）例如，当你有一个         Makefile          文件
       # 不能恰当地生成文件时，这个选项会提供帮助。
```

### 参数

目录：指定要显示列表的目录，也可以是具体的文件。

### 实例

```shell
$ ls       # 仅列出当前目录可见文件
$ ls -l    # 列出当前目录可见文件详细信息
$ ls -hl   # 列出详细信息并以可读大小显示文件大小
$ ls -al   # 列出所有文件（包括隐藏）的详细信息
$ ls --human-readable --size -1 -S --classify # 按文件大小排序
$ du -sh * | sort -h # 按文件大小排序(同上)
```

显示当前目录下包括影藏文件在内的所有文件列表

```shell
[root@localhost ~]# ls -a
.   anaconda-ks.cfg  .bash_logout   .bashrc  install.log         .mysql_history  satools  .tcshrc   .vimrc
..  .bash_history    .bash_profile  .cshrc   install.log.syslog  .rnd            .ssh     .viminfo
```

输出长格式列表

```shell
[root@localhost ~]# ls -1

anaconda-ks.cfg
install.log
install.log.syslog
satools
```

显示文件的inode信息

索引节点（index inode简称为“inode”）是Linux中一个特殊的概念，具有相同的索引节点号的两个文本本质上是同一个文件（除文件名不同外）。

```shell
[root@localhost ~]# ls -i -l anaconda-ks.cfg install.log
2345481 -rw------- 1 root root   859 Jun 11 22:49 anaconda-ks.cfg
2345474 -rw-r--r-- 1 root root 13837 Jun 11 22:49 install.log
```

水平输出文件列表

```shell
[root@localhost /]# ls -m

bin, boot, data, dev, etc, home, lib, lost+found, media, misc, mnt, opt, proc, root, sbin, selinux, srv, sys, tmp, usr, var
```

修改最后一次编辑的文件

最近修改的文件显示在最上面。

```shell
[root@localhost /]# ls -t

tmp  root  etc  dev  lib  boot  sys  proc  data  home  bin  sbin  usr  var  lost+found  media  mnt  opt  selinux  srv  misc
```

显示递归文件

```shell
[root@localhost ~]# ls -R
.:
anaconda-ks.cfg  install.log  install.log.syslog  satools

./satools:
black.txt  freemem.sh  iptables.sh  lnmp.sh  mysql  php502_check.sh  ssh_safe.sh
```

打印文件的UID和GID

```shell
[root@localhost /]# ls -n

total 254
drwxr-xr-x   2 0 0  4096 Jun 12 04:03 bin
drwxr-xr-x   4 0 0  1024 Jun 15 14:45 boot
drwxr-xr-x   6 0 0  4096 Jun 12 10:26 data
drwxr-xr-x  10 0 0  3520 Sep 26 15:38 dev
drwxr-xr-x  75 0 0  4096 Oct 16 04:02 etc
drwxr-xr-x   4 0 0  4096 Jun 12 10:26 home
drwxr-xr-x  14 0 0 12288 Jun 16 04:02 lib
drwx------   2 0 0 16384 Jun 11 22:46 lost+found
drwxr-xr-x   2 0 0  4096 May 11  2011 media
drwxr-xr-x   2 0 0  4096 Nov  8  2010 misc
drwxr-xr-x   2 0 0  4096 May 11  2011 mnt
drwxr-xr-x   2 0 0  4096 May 11  2011 opt
dr-xr-xr-x 232 0 0     0 Jun 15 11:04 proc
drwxr-x---   4 0 0  4096 Oct 15 14:43 root
drwxr-xr-x   2 0 0 12288 Jun 12 04:03 sbin
drwxr-xr-x   2 0 0  4096 May 11  2011 selinux
drwxr-xr-x   2 0 0  4096 May 11  2011 srv
drwxr-xr-x  11 0 0     0 Jun 15 11:04 sys
drwxrwxrwt   3 0 0 98304 Oct 16 08:45 tmp
drwxr-xr-x  13 0 0  4096 Jun 11 23:38 usr
drwxr-xr-x  19 0 0  4096 Jun 11 23:38 var
```

列出文件和文件夹的详细信息

```shell
[root@localhost /]# ls -l

total 254
drwxr-xr-x   2 root root  4096 Jun 12 04:03 bin
drwxr-xr-x   4 root root  1024 Jun 15 14:45 boot
drwxr-xr-x   6 root root  4096 Jun 12 10:26 data
drwxr-xr-x  10 root root  3520 Sep 26 15:38 dev
drwxr-xr-x  75 root root  4096 Oct 16 04:02 etc
drwxr-xr-x   4 root root  4096 Jun 12 10:26 home
drwxr-xr-x  14 root root 12288 Jun 16 04:02 lib
drwx------   2 root root 16384 Jun 11 22:46 lost+found
drwxr-xr-x   2 root root  4096 May 11  2011 media
drwxr-xr-x   2 root root  4096 Nov  8  2010 misc
drwxr-xr-x   2 root root  4096 May 11  2011 mnt
drwxr-xr-x   2 root root  4096 May 11  2011 opt
dr-xr-xr-x 232 root root     0 Jun 15 11:04 proc
drwxr-x---   4 root root  4096 Oct 15 14:43 root
drwxr-xr-x   2 root root 12288 Jun 12 04:03 sbin
drwxr-xr-x   2 root root  4096 May 11  2011 selinux
drwxr-xr-x   2 root root  4096 May 11  2011 srv
drwxr-xr-x  11 root root     0 Jun 15 11:04 sys
drwxrwxrwt   3 root root 98304 Oct 16 08:48 tmp
drwxr-xr-x  13 root root  4096 Jun 11 23:38 usr
drwxr-xr-x  19 root root  4096 Jun 11 23:38 var
```

列出可读文件和文件夹详细信息

```shell
[root@localhost /]# ls -lh

total 254K
drwxr-xr-x   2 root root 4.0K Jun 12 04:03 bin
drwxr-xr-x   4 root root 1.0K Jun 15 14:45 boot
drwxr-xr-x   6 root root 4.0K Jun 12 10:26 data
drwxr-xr-x  10 root root 3.5K Sep 26 15:38 dev
drwxr-xr-x  75 root root 4.0K Oct 16 04:02 etc
drwxr-xr-x   4 root root 4.0K Jun 12 10:26 home
drwxr-xr-x  14 root root  12K Jun 16 04:02 lib
drwx------   2 root root  16K Jun 11 22:46 lost+found
drwxr-xr-x   2 root root 4.0K May 11  2011 media
drwxr-xr-x   2 root root 4.0K Nov  8  2010 misc
drwxr-xr-x   2 root root 4.0K May 11  2011 mnt
drwxr-xr-x   2 root root 4.0K May 11  2011 opt
dr-xr-xr-x 235 root root    0 Jun 15 11:04 proc
drwxr-x---   4 root root 4.0K Oct 15 14:43 root
drwxr-xr-x   2 root root  12K Jun 12 04:03 sbin
drwxr-xr-x   2 root root 4.0K May 11  2011 selinux
drwxr-xr-x   2 root root 4.0K May 11  2011 srv
drwxr-xr-x  11 root root    0 Jun 15 11:04 sys
drwxrwxrwt   3 root root  96K Oct 16 08:49 tmp
drwxr-xr-x  13 root root 4.0K Jun 11 23:38 usr
drwxr-xr-x  19 root root 4.0K Jun 11 23:38 var
```

显示文件夹信息

```shell
[root@localhost /]# ls -ld /etc/

drwxr-xr-x 75 root root 4096 Oct 16 04:02 /etc/
```

按时间列出文件和文件夹详细信息

```shell
[root@localhost /]# ls -lt

total 254
drwxrwxrwt   3 root root 98304 Oct 16 08:53 tmp
drwxr-xr-x  75 root root  4096 Oct 16 04:02 etc
drwxr-x---   4 root root  4096 Oct 15 14:43 root
drwxr-xr-x  10 root root  3520 Sep 26 15:38 dev
drwxr-xr-x  14 root root 12288 Jun 16 04:02 lib
drwxr-xr-x   4 root root  1024 Jun 15 14:45 boot
drwxr-xr-x  11 root root     0 Jun 15 11:04 sys
dr-xr-xr-x 232 root root     0 Jun 15 11:04 proc
drwxr-xr-x   6 root root  4096 Jun 12 10:26 data
drwxr-xr-x   4 root root  4096 Jun 12 10:26 home
drwxr-xr-x   2 root root  4096 Jun 12 04:03 bin
drwxr-xr-x   2 root root 12288 Jun 12 04:03 sbin
drwxr-xr-x  13 root root  4096 Jun 11 23:38 usr
drwxr-xr-x  19 root root  4096 Jun 11 23:38 var
drwx------   2 root root 16384 Jun 11 22:46 lost+found
drwxr-xr-x   2 root root  4096 May 11  2011 media
drwxr-xr-x   2 root root  4096 May 11  2011 mnt
drwxr-xr-x   2 root root  4096 May 11  2011 opt
drwxr-xr-x   2 root root  4096 May 11  2011 selinux
drwxr-xr-x   2 root root  4096 May 11  2011 srv
drwxr-xr-x   2 root root  4096 Nov  8  2010 misc
```

按修改时间列出文件和文件夹详细信息

```shell
[root@localhost /]# ls -ltr

total 254
drwxr-xr-x   2 root root  4096 Nov  8  2010 misc
drwxr-xr-x   2 root root  4096 May 11  2011 srv
drwxr-xr-x   2 root root  4096 May 11  2011 selinux
drwxr-xr-x   2 root root  4096 May 11  2011 opt
drwxr-xr-x   2 root root  4096 May 11  2011 mnt
drwxr-xr-x   2 root root  4096 May 11  2011 media
drwx------   2 root root 16384 Jun 11 22:46 lost+found
drwxr-xr-x  19 root root  4096 Jun 11 23:38 var
drwxr-xr-x  13 root root  4096 Jun 11 23:38 usr
drwxr-xr-x   2 root root 12288 Jun 12 04:03 sbin
drwxr-xr-x   2 root root  4096 Jun 12 04:03 bin
drwxr-xr-x   4 root root  4096 Jun 12 10:26 home
drwxr-xr-x   6 root root  4096 Jun 12 10:26 data
dr-xr-xr-x 232 root root     0 Jun 15 11:04 proc
drwxr-xr-x  11 root root     0 Jun 15 11:04 sys
drwxr-xr-x   4 root root  1024 Jun 15 14:45 boot
drwxr-xr-x  14 root root 12288 Jun 16 04:02 lib
drwxr-xr-x  10 root root  3520 Sep 26 15:38 dev
drwxr-x---   4 root root  4096 Oct 15 14:43 root
drwxr-xr-x  75 root root  4096 Oct 16 04:02 etc
drwxrwxrwt   3 root root 98304 Oct 16 08:54 tmp
```

按照特殊字符对文件进行分类

```shell
[root@localhost nginx-1.2.1]# ls -F

auto/  CHANGES  CHANGES.ru  conf/  configure*  contrib/  html/  LICENSE  Makefile  man/  objs/  README  src/
```

列出文件并标记颜色分类

```shell
[root@localhost nginx-1.2.1]# ls --color=auto

auto  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
```

## 扩展知识

### 不同颜色代表的文件类型

- 蓝色：目录
    
- 绿色：可执行文件
    
- 白色：一般性文件，如文本文件，配置文件等
    
- 红色：压缩文件或归档文件
    
- 浅蓝色：链接文件
    
- 红色闪烁：链接文件存在问题
    
- 黄色：设备文件
    
- 青黄色：管道文件
# systemctl

系统服务管理器指令

## 补充说明

**systemctl命令** 是系统服务管理器指令，它实际上将 service 和 chkconfig 这两个命令组合到一起。

| 任务         | 旧指令                           | 新指令                                                                                      |
| ---------- | ----------------------------- | ---------------------------------------------------------------------------------------- |
| 使某服务自动启动   | chkconfig --level 3 httpd on  | systemctl enable httpd.service                                                           |
| 使某服务不自动启动  | chkconfig --level 3 httpd off | systemctl disable httpd.service                                                          |
| 检查服务状态     | service httpd status          | systemctl status httpd.service （服务详细信息） systemctl is-active httpd.service （仅显示是否 Active) |
| 显示所有已启动的服务 | chkconfig --list              | systemctl list-units --type=service                                                      |
| 启动服务       | service httpd start           | systemctl start httpd.service                                                            |
| 停止服务       | service httpd stop            | systemctl stop httpd.service                                                             |
| 重启服务       | service httpd restart         | systemctl restart httpd.service                                                          |
| 重载服务       | service httpd reload          | systemctl reload httpd.service                                                           |

### 实例

```shell
systemctl start nfs-server.service . # 启动nfs服务
systemctl enable nfs-server.service # 设置开机自启动
systemctl disable nfs-server.service # 停止开机自启动
systemctl status nfs-server.service # 查看服务当前状态
systemctl restart nfs-server.service # 重新启动某服务
systemctl list-units --type=service # 查看所有已启动的服务
```

开启防火墙22端口

```shell
iptables -I INPUT -p tcp --dport 22 -j accept
```

如果仍然有问题，就可能是SELinux导致的

关闭SElinux：

修改`/etc/selinux/config`文件中的`SELINUX=""`为disabled，然后重启。

彻底关闭防火墙：

```shell
sudo systemctl status firewalld.service
sudo systemctl stop firewalld.service          
sudo systemctl disable firewalld.service
```

# mkdir

用来创建目录

## 补充说明

**mkdir命令** 用来创建目录。该命令创建由dirname命名的目录。如果在目录名的前面没有加任何路径名，则在当前目录下创建由dirname指定的目录；如果给出了一个已经存在的路径，将会在该目录下创建一个指定的目录。在创建目录时，应保证新建的目录与它所在目录下的文件没有重名。 

注意：在创建文件时，不要把所有的文件都存放在主目录中，可以创建子目录，通过它们来更有效地组织文件。最好采用前后一致的命名方式来区分文件和目录。例如，目录名可以以大写字母开头，这样，在目录列表中目录名就出现在前面。

在一个子目录中应包含类型相似或用途相近的文件。例如，应建立一个子目录，它包含所有的数据库文件，另有一个子目录应包含电子表格文件，还有一个子目录应包含文字处理文档，等等。目录也是文件，它们和普通文件一样遵循相同的命名规则，并且利用全路径可以唯一地指定一个目录。

### 语法

```shell
mkdir (选项)(参数)
```

### 选项

```shell
-Z：设置安全上下文，当使用SELinux时有效；
-m<目标属性>或--mode<目标属性>建立目录的同时设置目录的权限；
-p或--parents 若所要建立目录的上层目录目前尚未建立，则会一并建立上层目录；
--version 显示版本信息。
```

### 参数

目录：指定要创建的目录列表，多个目录之间用空格隔开。

### 实例

在目录`/usr/meng`下建立子目录test，并且只有文件主有读、写和执行权限，其他人无权访问

```shell
mkdir -m 700 /usr/meng/test
```

在当前目录中建立bin和bin下的os_1目录，权限设置为文件主可读、写、执行，同组用户可读和执行，其他用户无权访问

```shell
mkdir -p-m 750 bin/os_1
```

# touch

创建新的空文件

## 补充说明

**touch命令** 有两个功能：一是用于把已存在文件的时间标签更新为系统当前的时间（默认方式），它们的数据将原封不动地保留下来；二是用来创建新的空文件。

### 语法

```shell
touch(选项)(参数)
```

### 选项

```shell
-a：或--time=atime或--time=access或--time=use  只更改存取时间；
-c：或--no-create  不建立任何文件；
-d：<时间日期> 使用指定的日期时间，而非现在的时间；
-f：此参数将忽略不予处理，仅负责解决BSD版本touch指令的兼容性问题；
-m：或--time=mtime或--time=modify  只更该变动时间；
-r：<参考文件或目录>  把指定文件或目录的日期时间，统统设成和参考文件或目录的日期时间相同；
-t：<日期时间>  使用指定的日期时间，而非现在的时间；
--help：在线帮助；
--version：显示版本信息。
```

### 参数

文件：指定要设置时间属性的文件列表。

### 实例

```shell
touch ex2
```

在当前目录下建立一个空文件ex2，然后，利用`ls -l`命令可以发现文件ex2的大小为0，表示它是空文件。
# cp

将源文件或目录复制到目标文件或目录中

## 补充说明

**cp命令** 用来将一个或多个源文件或者目录复制到指定的目的文件或目录。它可以将单个源文件复制成一个指定文件名的具体的文件或一个已经存在的目录下。cp命令还支持同时复制多个文件，当一次复制多个文件时，目标文件参数必须是一个已经存在的目录，否则将出现错误。

### 语法

```shell
cp(选项)(参数)
```

### 选项

```shell
-a：此参数的效果和同时指定"-dpR"参数相同；
-d：当复制符号连接时，把目标文件或目录也建立为符号连接，并指向与源文件或目录连接的原始文件或目录；
-f：强行复制文件或目录，不论目标文件或目录是否已存在；
-i：覆盖既有文件之前先询问用户；
-l：对源文件建立硬连接，而非复制文件；
-p：保留源文件或目录的属性；
-R/r：递归处理，将指定目录下的所有文件与子目录一并处理；
-s：对源文件建立符号连接，而非复制文件；
-u：使用这项参数后只会在源文件的更改时间较目标文件更新时或是名称相互对应的目标文件并不存在时，才复制文件；
-S：在备份文件时，用指定的后缀“SUFFIX”代替文件的默认后缀；
-b：覆盖已存在的文件目标前将目标文件备份；
-v：详细显示命令执行的操作。
```

### 参数

- 源文件：制定源文件列表。默认情况下，cp命令不能复制目录，如果要复制目录，则必须使用`-R`选项；
- 目标文件：指定目标文件。当“源文件”为多个文件时，要求“目标文件”为指定的目录。

### 实例

下面的第一行中是 cp 命令和具体的参数（-r 是“递归”， -u 是“更新”，-v 是“详细”）。接下来的三行显示被复制文件的信息，最后一行显示命令行提示符。这样，只拷贝新的文件到我的存储设备上，我就使用 cp 的“更新”和“详细”选项。

通常来说，参数 `-r` 也可用更详细的风格 `--recursive`。但是以简短的方式，也可以这么连用 `-ruv`。

```shell
cp -r -u -v /usr/men/tmp ~/men/tmp
```

版本备份 `--backup=numbered` 参数意思为“我要做个备份，而且是带编号的连续备份”。所以一个备份就是 1 号，第二个就是 2 号，等等。

```shell
$ cp --force --backup=numbered test1.py test1.py
$ ls
test1.py test1.py.~1~ test1.py.~2~
```

如果把一个文件复制到一个目标文件中，而目标文件已经存在，那么，该目标文件的内容将被破坏。此命令中所有参数既可以是绝对路径名，也可以是相对路径名。通常会用到点`.`或点点`..`的形式。例如，下面的命令将指定文件复制到当前目录下：

```shell
cp ../mary/homework/assign .
```

所有目标文件指定的目录必须是己经存在的，cp命令不能创建目录。如果没有文件复制的权限，则系统会显示出错信息。

将文件file复制到目录`/usr/men/tmp`下，并改名为file1

```shell
cp file /usr/men/tmp/file1
```

将目录`/usr/men`下的所有文件及其子目录复制到目录`/usr/zh`中

```shell
cp -r /usr/men /usr/zh
```

交互式地将目录`/usr/men`中的以m打头的所有.c文件复制到目录`/usr/zh`中

```shell
cp -i /usr/men m*.c /usr/zh
```

我们在Linux下使用cp命令复制文件时候，有时候会需要覆盖一些同名文件，覆盖文件的时候都会有提示：需要不停的按Y来确定执行覆盖。文件数量不多还好，但是要是几百个估计按Y都要吐血了，于是折腾来半天总结了一个方法：

```shell
cp aaa/* /bbb
# 复制目录aaa下所有到/bbb目录下，这时如果/bbb目录下有和aaa同名的文件，需要按Y来确认并且会略过aaa目录下的子目录。

cp -r aaa/* /bbb
# 这次依然需要按Y来确认操作，但是没有忽略子目录。

cp -r -a aaa/* /bbb
# 依然需要按Y来确认操作，并且把aaa目录以及子目录和文件属性也传递到了/bbb。

\cp -r -a aaa/* /bbb
# 成功，没有提示按Y、传递了目录属性、没有略过目录。
```

递归强制复制目录到指定目录中覆盖已存在文件

```shell
cp -rfb ./* ../backup
# 将当前目录下所有文件，复制到当前目录的兄弟目录 backup 文件夹中
```

拷贝目录下的隐藏文件如 `.babelrc`

```shell
cp -r aaa/.* ./bbb
# 将 aaa 目录下的，所有`.`开头的文件，复制到 bbb 目录中。

cp -a aaa ./bbb/ 
# 记住后面目录最好的'/' 带上 `-a` 参数
```

复制到当前目录

```sh
cp aaa.conf ./
# 将 aaa.conf 复制到当前目录
```

# rm

用于删除给定的文件和目录

## 补充说明

**rm** **命令** 可以删除一个目录中的一个或多个文件或目录，也可以将某个目录及其下属的所有文件及其子目录均删除掉。对于链接文件，只是删除整个链接文件，而原有文件保持不变。

注意：使用rm命令要格外小心。因为一旦删除了一个文件，就无法再恢复它。所以，在删除文件之前，最好再看一下文件的内容，确定是否真要删除。rm命令可以用-i选项，这个选项在使用文件扩展名字符删除多个文件时特别有用。使用这个选项，系统会要求你逐一确定是否要删除。这时，必须输入y并按Enter键，才能删除文件。如果仅按Enter键或其他字符，文件不会被删除。

### 语法

```shell
rm (选项)(参数)
```

### 选项

```shell
-d：直接把欲删除的目录的硬连接数据删除成0，删除该目录；
-f：强制删除文件或目录；
-i：删除已有文件或目录之前先询问用户；
-r或-R：递归处理，将指定目录下的所有文件与子目录一并处理；
--preserve-root：不对根目录进行递归操作；
-v：显示指令的详细执行过程。
```

### 参数

文件：指定被删除的文件列表，如果参数中含有目录，则必须加上`-r`或者`-R`选项。

### 实例

交互式删除当前目录下的文件test和example

```shell
rm -i test example
Remove test ?n（不删除文件test)
Remove example ?y（删除文件example)
```

删除当前目录下除隐含文件外的所有文件和子目录

```shell
# rm -r *
```

应注意，这样做是非常危险的!

**rm 命令删除当前目录下的 node_modules 目录**

```shell
find . -name 'node_modules' -type d -prune -exec rm -rf '{}' +
```

**rm 命令删除文件**

```shell
# rm 文件1 文件2 ...
rm testfile.txt
```

**rm 命令删除目录**

> rm -r [目录名称] -r 表示递归地删除目录下的所有文件和目录。 -f 表示强制删除

```shell
rm -rf testdir
rm -r testdir
```

**删除操作前有确认提示**

> rm -i [文件/目录]

```shell
rm -r -i testdir
```

**rm 忽略不存在的文件或目录**

> -f 选项（LCTT 译注：即 “force”）让此次操作强制执行，忽略错误提示

```shell
rm -f [文件...]
```

**仅在某些场景下确认删除**

> 选项 -I，可保证在删除超过 3 个文件时或递归删除时（LCTT 译注： 如删除目录）仅提示一次确认。

```shell
rm -I file1 file2 file3
```

**删除根目录**

> 当然，删除根目录（/）是 Linux 用户最不想要的操作，这也就是为什么默认 rm 命令不支持在根目录上执行递归删除操作。 然而，如果你非得完成这个操作，你需要使用 --no-preserve-root 选项。当提供此选项，rm 就不会特殊处理根目录（/）了。

```shell
不给实例了，操作系统都被你删除了，你太坏了😆
```

**rm 显示当前删除操作的详情**

```shell
rm -v [文件/目录]
```
rm -rv递归删除并显示操作过程

# cat

连接多个文件并打印到标准输出。

## 概要

```shell
cat [OPTION]... [FILE]...
```

## 主要用途

- 显示文件内容，如果没有文件或文件为`-`则读取标准输入。
- 将多个文件的内容进行连接并打印到标准输出。
- 显示文件内容中的不可见字符（控制字符、换行符、制表符等）。

## 参数

FILE（可选）：要处理的文件，可以为一或多个。

## 选项

```shell
长选项与短选项等价

-A, --show-all           等价于"-vET"组合选项。
-b, --number-nonblank    只对非空行编号，从1开始编号，覆盖"-n"选项。
-e                       等价于"-vE"组合选项。
-E, --show-ends          在每行的结尾显示'$'字符。
-n, --number             对所有行编号，从1开始编号。
-s, --squeeze-blank      压缩连续的空行到一行。
-t                       等价于"-vT"组合选项。
-T, --show-tabs          使用"^I"表示TAB（制表符）。
-u                       POSIX兼容性选项，无意义。
-v, --show-nonprinting   使用"^"和"M-"符号显示控制字符，除了LFD（line feed，即换行符'\n'）和TAB（制表符）。

--help                   显示帮助信息并退出。
--version                显示版本信息并退出。
```

## 返回值

返回状态为成功除非给出了非法选项或非法参数。

## 例子

```shell
# 合并显示多个文件
cat ./1.log ./2.log ./3.log
# 显示文件中的非打印字符、tab、换行符
cat -A test.log
# 压缩文件的空行
cat -s test.log
# 显示文件并在所有行开头附加行号
cat -n test.log
# 显示文件并在所有非空行开头附加行号
cat -b test.log
# 将标准输入的内容和文件内容一并显示
echo '######' |cat - test.log
```

### 注意

1. 该命令是`GNU coreutils`包中的命令，相关的帮助信息请查看`man -s 1 cat`或`info coreutils 'cat invocation'`。
2. 当使用`cat`命令查看**体积较大的文件**时，文本在屏幕上迅速闪过（滚屏），用户往往看不清所显示的内容，为了控制滚屏，可以按`Ctrl+s`键停止滚屏；按`Ctrl+q`键恢复滚屏；按`Ctrl+c`（中断）键可以终止该命令的执行，返回Shell提示符状态。
3. 建议您查看**体积较大的文件**时使用`less`、`more`命令或`emacs`、`vi`等文本编辑器。

### 参考链接

1. [Question about LFD key](https://superuser.com/questions/328054/is-there-an-lfd-key-on-my-keyboard)

# more

显示文件内容，每次显示一屏

## 补充说明

**more命令** 是一个基于vi编辑器文本过滤器，它以全屏幕的方式按页显示文本文件的内容，支持vi中的关键字定位操作。more名单中内置了若干快捷键，常用的有H（获得帮助信息），Enter（向下翻滚一行），空格（向下滚动一屏），Q（退出命令）。

该命令一次显示一屏文本，满屏后停下来，并且在屏幕的底部出现一个提示信息，给出至今己显示的该文件的百分比：--More--（XX%）可以用下列不同的方法对提示做出回答：

- 按 `Space` 键：显示文本的下一屏内容。
- 按 `Enter` 键：只显示文本的下一行内容。
- 按斜线符`|`：接着输入一个模式，可以在文本中寻找下一个相匹配的模式。
- 按H键：显示帮助屏，该屏上有相关的帮助信息。
- 按B键：显示上一屏内容。
- 按Q键：退出more命令。

### 语法

```shell
more(语法)(参数)
```

### 选项

```shell
-<数字>：指定每屏显示的行数；
-d：显示“[press space to continue,'q' to quit.]”和“[Press 'h' for instructions]”；
-c：不进行滚屏操作。每次刷新这个屏幕；
-s：将多个空行压缩成一行显示；
-u：禁止下划线；
+<数字>：从指定数字的行开始显示。
```

### 参数

文件：指定分页显示内容的文件。

### 实例

显示文件file的内容，但在显示之前先清屏，并且在屏幕的最下方显示完成的百分比。

```shell
more -dc file
```

显示文件file的内容，每10行显示一次，而且在显示之前先清屏。

```shell
more -c -10 file
```

# less

分屏上下翻页浏览文件内容

## 补充说明

**less命令** 的作用与more十分相似，都可以用来浏览文字档案的内容，不同的是less命令允许用户向前或向后浏览文件，而more命令只能向前浏览。用less命令显示文件时，用PageUp键向上翻页，用PageDown键向下翻页。要退出less程序，应按Q键。

### 语法

```shell
less(选项)(参数)
```

### 选项

```shell
-e：文件内容显示完毕后，自动退出；
-f：强制显示文件；
-g：不加亮显示搜索到的所有关键词，仅显示当前显示的关键字，以提高显示速度；
-l：搜索时忽略大小写的差异；
-N：每一行行首显示行号；
-s：将连续多个空行压缩成一行显示；
-S：在单行显示较长的内容，而不换行显示；
-x<数字>：将TAB字符显示为指定个数的空格字符。
```

### 参数

文件：指定要分屏显示内容的文件。

## 实例

```shell
sudo less /var/log/shadowsocks.log
```