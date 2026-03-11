---
layout: post
title: "Linux 根目录结构总览"
subtitle: "深入理解 Linux 文件系统架构"
date: 2026-03-11
author: "b0t"
header-img: "img/post-bg-linux.jpg"
catalog: true
tags:
  - Linux
  - 操作系统
  - 文件系统
---


从 `/` 开始，一个一个讲清楚。


/  
├── bin  
├── boot  
├── dev  
├── etc  
├── home  
├── lib  
├── lib64  
├── media  
├── mnt  
├── opt  
├── proc  
├── root  
├── run  
├── sbin  
├── snap  
├── srv  
├── sys  
├── tmp  
├── usr  
├── var  
├── swap.img  
└── lost+found

下面逐个完整解释。

---

# 🧠 一、Linux核心概念

Linux 一切从 `/` 开始  
所有文件 = 树结构

/  
└── 所有系统文件都在这里

不像Windows有C盘D盘  
Linux只有一个根

---

# 📁 1️⃣ bin —— 基础命令目录 ⭐⭐⭐⭐⭐

binary 可执行文件

存放系统最基础命令：

ls  
cp  
mv  
cat  
bash  
sh  
echo

你输入：

ls

实际执行：

/bin/ls

👉 没有 `/bin`  
系统无法操作  
连命令都没

---

# 📁 2️⃣ boot —— 启动目录 ⭐⭐⭐⭐

系统启动文件

包含：

- Linux内核
    
- grub引导
    
- initrd
    

开机流程：

BIOS  
 ↓  
boot目录  
 ↓  
加载Linux内核  
 ↓  
进入系统

黑客常改这里做：

- 内核后门
    
- 提权
    
- 启动劫持
    

---

# 📁 3️⃣ dev —— 设备文件目录 ⭐⭐⭐⭐⭐

Linux核心思想：

> 一切皆文件

硬件也是文件：

/dev/sda      硬盘  
/dev/null     黑洞  
/dev/random   随机数  
/dev/tty      终端  
/dev/zero     无限0

例：

echo hello > /dev/null

= 丢进黑洞

CTF必考：

/dev/mem  
/dev/kmem

---

# 📁 4️⃣ etc —— 系统配置中心 ⭐⭐⭐⭐⭐

超级重要

所有配置：

用户  
密码  
网络  
服务  
ssh

重点文件：

/etc/passwd  
/etc/shadow  
/etc/hosts  
/etc/ssh/

CTF提权第一步：

cat /etc/passwd

---

# 📁 5️⃣ home —— 普通用户家目录 ⭐⭐⭐⭐⭐

每个用户一个：

/home/ubuntu  
/home/b0t  
/home/user

存：

- 代码
    
- 下载
    
- ssh密钥
    
- 历史命令
    

黑客常翻：

.bash_history  
.ssh  
flag

---

# 📁 6️⃣ root —— root用户的家 ⭐⭐⭐⭐⭐

注意：

/root ≠ /

只是root用户的home：

/root

里面常有：

flag  
root脚本  
ssh密钥

---

# 📁 7️⃣ lib —— 系统库 ⭐⭐⭐⭐

类似Windows：

dll

Linux：

.so

存放：

libc.so  
libm.so

程序运行必须依赖它

CTF常用：

LD_PRELOAD劫持  
glibc利用

---

# 📁 8️⃣ lib64 —— 64位库

64位系统使用

64位动态库

---

# 📁 9️⃣ media —— 外接设备

U盘插入后：

/media/用户名/U盘

自动挂载

---

# 📁 🔟 mnt —— 手动挂载点

手动挂载：

mount /dev/sdb1 /mnt

就会出现在：

/mnt

---

# 📁 11️⃣ opt —— 第三方软件 ⭐⭐⭐⭐

optional

手动安装软件：

/opt/google  
/opt/burpsuite  
/opt/node

CTF常藏：

flag  
web源码  
数据库密码

---

# 📁 12️⃣ proc —— 内存映射目录 ⭐⭐⭐⭐⭐

最重要之一🔥

不是硬盘  
是内存

/proc/cpuinfo  
/proc/meminfo  
/proc/self/maps

查看程序内存：

cat /proc/self/maps

CTF神器

---

# 📁 13️⃣ run —— 运行时数据

存：

pid文件  
socket  
服务状态

系统运行时产生  
重启清空

---

# 📁 14️⃣ sbin —— 管理员命令 ⭐⭐⭐⭐

system binary

只有root用：

reboot  
fdisk  
ifconfig  
shutdown

---

# 📁 15️⃣ snap —— Ubuntu软件包

Ubuntu的应用商店

snap安装的软件在这里

---

# 📁 16️⃣ srv —— 服务数据

server data

存：

web服务  
ftp服务

但实际很少用

---

# 📁 17️⃣ sys —— 内核接口 ⭐⭐⭐⭐

类似 `/proc`

但更底层：

硬件信息  
驱动  
内核状态

黑客可调：

内核参数

---

# 📁 18️⃣ tmp —— 临时目录 ⭐⭐⭐⭐⭐

任何人可写

/tmp

重启清空

黑客最爱：

木马  
提权脚本  
临时flag

CTF必翻：

ls /tmp

---

# 📁 19️⃣ usr —— 最大目录 ⭐⭐⭐⭐⭐

90%软件在这里

/usr/bin  
/usr/lib  
/usr/share

安装的软件：

python  
gcc  
vim

---

# 📁 20️⃣ var —— 经常变化 ⭐⭐⭐⭐⭐

variable

存：

日志  
缓存  
数据库  
web文件

重点：

/var/log  
/var/www

看日志：

tail -f /var/log/syslog

---

# 📁 21️⃣ swap.img —— 虚拟内存

内存不够：

→ 用硬盘当内存

---

# 📁 22️⃣ lost+found —— 磁盘修复

ext文件系统用

系统崩溃后：

找回碎片文件

平时不用管

---

# 🧠 黑客视角必翻目录排行

拿到shell第一件事：

ls /

然后：

/home  
/root  
/opt  
/tmp  
/var/www  
/etc  
/proc

90% flag 在这