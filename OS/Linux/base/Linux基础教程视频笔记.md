# Linux基础教程视频笔记
## 硬盘分区
硬盘分区有基本分区和扩展分区两种，基本分区和扩展分区之和不大于四（这是因为硬盘分区表只有四项），扩展分区又可以分若干个逻辑分区。

在下图中，windows环境下，蓝色区域代表主分区，绿色区域代表扩展分区，扩展分区里面蓝色的代表逻辑分区。

![](https://github.com/18862601653/Learning/blob/master/images/2018-04-15_220900.png)

CIH病毒破坏的就是硬盘分区表。

## Linux下的硬盘分区
在Linux看来，所有的东西都是文件。文件有两种，一种是字符设备，另一种是块设备（二进制设备）。

硬盘、光驱、U盘都是块设备，键盘、打印机是字符设备。

/dev/xxyN
- /dev/
  这个子串是所有设备文件所在的目录名。因为分区在硬盘上，而硬盘是设备，所以这些文件代表了在/dev/上所有可能的分区
- xx
  分区名的前两个字母标明分区所在设备的类型。通常是hd（IDE磁盘）或sd（SCSI磁盘）。
- y
  这个字母标明分区所在的设备。例如，/dev/hda(第一个IDE磁盘）或/dev/sdb（第二个SCSI磁盘）。
- N
  最后的数字代表分区，前四个分区（主分区或者扩展分区）是用数字从1排列到4，逻辑分区从5开始，例如/dev/hda3是第一个IDE硬上的第三个主分区或扩展分区，/dev/sdb6代表第二个SCSI硬盘上的第二个逻辑分区。

linux从命令界面切换到图形界面的命令：startx
pwd命令：查阅当前目录
whoami命令：查阅当前用户

磁盘分区与mount point（挂载点）

在linux中，每一个分区都是构成支持一组文件和目录所必需的贮存区的一部分，通过挂载（mounting）实现，挂载是将分区关联到某一个目录的过程。挂载分区使起始于这个指定目录（挂载点）的贮存区能够被使用。

将/dev/cdrom挂载到/mnt/cdr的命令： mount /dev/cdrom /mnt/cdr

取消挂载: umount /dev/cdrom

硬盘物理结构
- 硬盘有数个盘片，每盘片两个面，每个面一个磁头
- 盘片被划分为多个扇形区域即扇区
- 同一盘片不同半径的同心圆为磁道
- 不同盘片相同半径构成的圆柱面即柱面
- 公式：存储容量=磁头数 * 磁道（柱面）数 * 每道扇区数 * 每扇区字节数
- 信息记录可表示为：XX磁道（柱面），XX磁头，XX扇区

MBR（master boot recod）
- 位于硬盘第一个物理扇区（绝对扇区）柱面0，磁头0，扇区1处
- MBR中包含硬盘的主引导程序和硬盘分区表

分区

|挂载点名字|描述|
|-|-|
|/|根分区|
|/usr|应用软件存放位置|
|/home|用户宿主目录的父目录|
|/var|存放临时文件|
|/boot|存放启动文件128M就够了|
|SWAP|交换分区（虚拟内存）|

SWAP分区的几个注意点：
- 一般是实际物理内存的两倍
- 可以不建，但不建议，因为有的程序默认放在SWAP分区上
- Windows中有PAGEFILE.SYS的文件相当于SWAP分区

磁盘分区方案
- 至少有两个分区（/，SWAP）
- 个人桌面分区（/，/boot，/usr，SWAP分区）
- 光盘刻录再加一个/tmp分区

## Linux目录结构
Linux的目录结构
- /Linux 文件系统的入口，也是处于最高一级的目录
- /bin 基础系统所需要的那些命令位于此目录，也是最小系统所需要的命令；比如ls、cp、mkdir等命令；功能和/usr/bin类似，这个目录中的文件都是可执行的，普通用户都可以使用的命令。作为基础系统所需要的最基础的命令就是放在这里
- /boot Linux的内核及引导系统程序所需要的文件，比如vmlinuz initrd.img文件都位于这个目录中。在一般情况下，GRUB或LILO系统引导管理器也位于这个目录
- /dev 设备文件存储目录，比如声卡、磁盘等
- /etc 系统配置文件的所在地，一些服务器的配置文件也在这里，比如用户账号及密码配置文件
- /home 普通用户家目录默认存放目录
- /lib 库文件存放目录
- /usr 这个是系统存放程序的目录，比如命令、帮助文件等。这个目录下有很多的文件和目录。当我们安装一个Linux发行版官方提供的软件包时，大多安装在这里。如果有设计服务器配置文件的，会把配置文件安装在/etc目录中。/usr目录下包括涉及字体目录/usr/share/fonts,帮助目录/usr/share/man或/usr/share/doc，普通用户可执行文件目录/usr/bin或/usr/local/bin或/usr/X11R6/bin，超级权限用户root的可执行命令存放目录，比如/usr/sbin或/usr/X11R6/sbin或/usr/local/sbin等，还有程序的头文件存放目录/usr/include
- /var 这个目录的内容是经常变动的，/var下有/var/log，这是用来存放系统日志的目录。/var/www目录是定义Apache服务器站点存放目录。/var/lib用来存放一些库文件，比如MySQL的以及MySQL数据库的存放地。
- /sbin 大多是涉及系统管理的命令的存放，是超级权限用户root的可执行命令存放地，普通用户无权限执行这个目录下的命令，这个目录和/usr/sbin，/usr/X11R6/sbin或/usr/local/sbin目录是相似的，凡是目录sbin中包含的都是root权限才能执行的
- /tmp 临时文件目录，有时用户运行程序的时候，会产生临时文件。/tmp就是用来存放临时文件的。/var/tmp目录和这个目录类似。

## Linux启动过程
Linux的启动过程：
- 加载bios（硬件信息）
- 读取MBR的配置，找到os
- 加载os内核
- 启动初始化进程
- 执行/etc/rc.d/sysinit
- 启动其他模块（etc/modules.conf)
- 执行运行级脚本
- 执行/etc/rc.d/rc.local（rc.local可配置自动启动）
- 执行/bin/login
- 启动shell

Init（run level -/etc/inittab)
- Init n

|n|解释|
|-|-|
|0|系统停机状态（关机）|
|1|单用户工作状态（只有root用户）|
|2|多用户状态（没有NFS，网络文件系统）|
|3|多用户状态（有NFS）|
|4|系统未使用，留给用户|
|5|图形界面|
|6|系统正常关闭并重新启动（reboot）|

## Linux基本命令
- ls
```
ls -l
将文件或者目录竖着排列，以d开头的是目录，以-开头的是文件，以l开头是链接
如：drwxr-xr-x  3   root    root  4096  Mar 13 16:00  apache
   目录 权限   所占空间 创建人 所有人       创建时间        文件或目录名

ls -m
适应宽度，横着排

ls -R
按树状结构列

```
- rmdir
```
rm -r
递归删除，会在删除每个文件或目录的时候询问

rm -rf
直接删除，不询问
```
- touch
```
touch filename
创建文件
```

- vi
vi有两种模式，一种是命令格式，另一种是编辑模式。
```
dd:删除一行
dw：删除一个单词
o：插入一行
O：上面插入一行
```

- more
```
more filename
查看文件内容
```

- cat/tac
```
cat filename:正序
tac filename：倒序
```

- head/tail
```
head -n filename
头n行
tail -n filename
尾n行
```

- find
```
find /etc -name *local
```

- whereis
```
whereis ls
whereis 命令
```

- echo 
```
echo $PATH
```

- ln
```
ln filename linkname
硬链接，同步

ln -s filename linkname
软链接，同步
软链接相当于windows下的快捷方式，硬链接相当于原文件复制一份
原文件删了之后硬链接还可以查到文件，但是软链接不可以
```

- 用户与用户组 
```
useradd username
添加用户,在home下会增加一个主目录
passwd username
为添加的用户设置密码
/etc/passwd文件里面是所有的用户信息
如 testuser:x:501:502::/home/testuser:/bin/bash
   用户名     用户组 ID   主目录         用到的shell
shell有csh、bsh、ksh、bash、sh 
/etc/group文件里面记录用户组，如果添加一个用户没有指定组，那么就默认添加一个和用户名一样的组
groupadd groupname
添加一个groupname的组
useradd username -g groupname
将用户username添加到groupname这个组
usermod -g groupname username
将用户username的组改成groupname
userdel username
删除用户，但是home下面的文件依然存在
删除目录：rm -rf username
su(switch user) username：切换用户，当一个新的用户登录时，默认的当前目录是主目录
```

- 文件权限
```
ls -l
-rw-r--r-- 1 root root 56 Jun 23 12:44 haha
除了第一位表示文件（-）、目录（d）、链接（l）外还有九位，每三位一组
r代表只读，w代表写，x代表执行，---代表什么权限都没有
第一组代表文件的所有者的权限
第二组代表文件的所有者同组的其他人的权限
第三组代表剩下的人对文件的权限
chmod +x haha ：-rwxr-xr-x
chmod u+x haha : -rwxr--r--
chmod g+x haha : -rw-r-xr--
chmod o+x haha : -rw-r--r-x
chmod 755 haha : -rwxr-xr-x
chown haha username : 修改haha的所有者
```

- wc
统计指定文本文件的行数、字数、字符数  
```
wc -cmlLw filename
-c：bytes，字节数
-m：chars，字符数
-l：lines，行数
-L：max-line-length，最长行的长度
-w：words，字数
```

- grep
```
grep chars filename：查文本文件的哪一行语句包含要找的字符
```

|命令|功能|
|-|-|
|date|显示和设置日期时间|
|stat|显示指定文件的相关信息|
|who、w|显示在线登录用户|
|whoami|显示用户自己的身份|
|id|显示当前用户的id信息|
|hostname|显示主机名称|
|uname|显示操作系统信息|
|dmesg|显示系统启动信息|
|du|显示指定的文件（目录）已使用的磁盘空间的总量|
|df|显示文件系统磁盘空间的使用情况|
|free|显示当前内存和交换空间的使用情况|
|fdisk -l|显示磁盘信息|
|locale|显示当前语言环境|

- 管道
```
ls -Rl /etc | more
把上一个命令执行的结果交给下一个命令
cat /etc/passwd | wc
统计/etc/passwd的行数、字数和字节数
cat /etc/passwd | grep lrj
查lrj用户名的信息
dmesg | grep eth0
查第一块网卡的启动信息
man bash | col -b > bash.txt
把说明文件的内容输出成纯文本文件时，控制字符会变成乱码，col指令能有效过滤这些控制字符
col -b:过滤掉文件里面的控制字符
ls -l | grep "^d"
只列出目录，"^d"是正则表达式，^表示一行开头
ls -l * | grep "^-" | wc -l
当前目录下有多少文件
```

- 命令替换
```
wall:warning all，通知所有人，打开多个terminal，每个terminal都会收到广播信息（警告信息）
wall `date`:把``中命令执行的结果通知所有人
```

- 重定向
```
ls > filename
将命令执行的结果输入到filename文件里
ls >> filename
将命令执行的结果追加到filename文件里
lssssss 2> filename
2>：错误重定向
wall < filename
重定向输入，将filename的内容通知所有人
```

- 如何修改系统的默认启动级别

## 文件共享
https://blog.csdn.net/Han_Wen2015/article/details/76794842

https://blog.csdn.net/dddxxxx/article/details/53771513
















## 参考
[1] https://www.onmpw.com/tm/xwzj/opersys_203.html
