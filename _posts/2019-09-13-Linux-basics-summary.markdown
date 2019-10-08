---
layout:      post
title:       "Linux基础知识概要"
subtitle:    "Basic knowledge of Linux"
author:      "Ashior"
header-img:  "img/post-bg-ubuntu.jpg"
catalog:     true
tags:
  - 工作
  - Linux
  - 读书笔记
---

> 目前知识一些基础知识点，并没有涉及深层次的内容(这些是看视频时总结的部分内容)  
> 本人目前也处于试水状态，比较杂乱，之后会陆续完善，望见谅~  

## CentOS系统部署
系统分区：
> / 余下的空间(**至少有一个分区**)  
> /swap 8096M(内存的1~2倍)  
> /boot 1024M  
> /data  

注意：KDUMP相当于黑匣子，记录系统的运行过程。（目前推荐关闭）网络需要手动开启，同时将网卡设置成为自动启动，可以对网络进行详细的设置。主机名的设置需要便于管理与配置。
NTFS文件系统支持超过4G的文件

#### GNU BASH
实现对Linux系统的大部分管理：文件管理、用户管理、权限管理、磁盘管理、软件管理、网络管理。
在BASH Shell中，蓝色为目录，其余的为文件。语法为：
-  命令 选项(影响命令行为) 参数(命令作用对象)

#### BASH提示符
通过echo $PS1可以查看显示规则。
快捷键：
- ^C：终止前台命令
- ^D：关闭终端
- ^L：清屏
- ^A：光标移动到命令最前面
- ^E：光标移动到命令最后面
- ^K：删除光标位置后面的全部命令
- ^U：删除光标位置前面的全部命令
- ^R：搜索历史命令，利用关键字
- Alt+.：引用上一个命令的最后一个参数，等价于!$
- ESC .：
- history：查看历史命令，!num执行第num条历史命令。
- alisa：查看、创建命令别名。'\命令' 跳过别名
- cd -：返回上一次的目录

一个字节等于八位，即1B=8b，1KB=1024B

---  

## Linux目录结构
目录结构
- dev：设备文件：
- /dev/null 控设备，类似回收站
- /dev/random 产生随机数
- /dev/zero 零设备
- proc：虚拟的文件系统，反映出来的是内核，进程信息或实施状态
- usr：相当于C盘的Windows文件夹
- /usr/local 软件安装的目录，相当于C:\Program
- /usr/bin 普通用户使用的应用程序
- /usr/sbin 管理员使用的应用程序
- /usr/lib 库文件Glibc(32bit)
- /usr/lib64 库文件Glibc(64bit)
- boot：存放系统启动相关文件，例如kernel，grub(引导装载程序)
- **etc**：配置文件(重要)
系统相关如网络/etc/sysconfig/network，主机名/etc/hostname，应用相关配置文件如/etc/ssh/sshd_config...

---

## Linux文件管理
熟记基础命令与用法。
文件的四种时间：
- 访问时间：atime，查看内容；
- 修改时间：mtime，修改内容；
- 改变时间：ctime，文件属性，比如权限；
- 删除时间：dtime，文件被删除时间；

type：查看命令类型，该命令是alias，还是内置命令还是某个文件
file：查看文件类型，例如是文本文件还是二进制文件，管道文件等
stat：查看文件的属性，例如文件的名称，大小，权限，时间等

#### 权限

#### 用户/组基本概念
/etc/passwd 用户信息  
`root:x:0:0:root:/root:/bin/bash`  
`用户名 : x : uid : gid : 描述 : HOME : shell`  
x为密码占位符  
/etc/shadow 密码信息  
`root : $1$MYSDJABS6$a1jsdhnabAPNKCsd0sO0 : 15636 : 0 : 99999 : 7: : :`
$id$salt$encrypted$...(主要为三段组成，后面为密码策略，诸如密码过期时间)  
加密算法$id：  
$1:MD5  
$5:SHA-256  
$6:SHA-512  

系统约定：  
uid: 0     特权用户  
uid: 1-499 系统用户  
uid: 500+  普通用户  

**/etc/group 组信息**  
`useradd/groupadd name`创建  
用户创建时有有且只有一个主组(有指定，则指定组作为该用户的主组[-g group_name]，未指定，系统会创建一个和用户同名的组作为用户的主组)  
`userdel -r user_name`  可以连带删除其家目录与邮件目录。也可以使用rm -rf 删除其家目录与邮件目录，在用户不存在时  
ps：是进程访问文件，但是一个进程是需要一个特定用户去运行  
/sbin/nologin：作为进程的启动者存在，访问ftp的用户(安全的用户)  
/bin/bash：登录系统，实现管理。  
shell是登录用户后运行的第一个程序。如果使用/usr/sbin/poweroff（给root）则root用户一登录，就立马关机。(前提是关闭高级保护机制：SELinux，通常关闭，因为过于严格。setenforce 0，可以使测试成功，通常也都是关闭的)  
用户创建时，通常不让其登录：  
`useradd user_name -s /sbin/nologin`  
查询可以登录的用户：  
`grep 'bash$' /etc/passwd`  

可以从passwd，用户信息的文件中查看哪些用户有权登录CentOS进行管理。  

以下两个文件的决定使用useradd命令创建用户时所执行默认操作。  
/etc/login.defs  
/etc/default/useradd  
新建的用户会从/etc/skel目录下，拷贝文件(四个)  
`gpassword -a yellow wheel`将用户yellow加入wheel组中(相当于变成管理员)  

#### Linux文件权限
权限对象：属主u，属组g，其他人o  
权限类型：读4r，写2w，执行1x  
所有者可以修改权限  
`chown -R username.groupname filename`  
改变属主与属组，-R递归  
`> chmod (u+x,g+x,o+x)/a+x filename`  

给所有者，所属组与其他成员增加文件执行权限
PS：权限是4位，并非三位，还有**特殊权限**  

对目录有W权限，可以在目录中创建新文件，可以删除文件夹中的文件（此权限(删除文件)与文件权限无关）。实际上，删除文件是对目录进行操作  
文件：X权限，目录：W权限，均需小心给予  

#### 基本权限(F)ACL
**在查看文件权限的时候，最后面一位有'+'，需要使用getfacl命令查看详细的命令，因为权限显示的为mask**  
查看文件的acl：  
`getfacl /home/file_name`  
修改用户Alice的权限：  
setfacl -m u:alice:rw /home/file_name  
**mask**：用于临时(如果有任何权限的变更，如给其他人赋予权限，则mask自动变更)降低用户或组（除属主和其他人）的权限
mask决定了他们的最高权限  
建议为方便管理文件权限，将其他人的权限设置为空：  
`setfacl -m o::-/home/file`  
`setfacl -m m::--- /home/file`  
**继承**：  
要求Alice能够对/home以及以后在/home下新建的文件有读写执行权限。  
`setfacl -m u:alice:rwx /home`  
赋予Alice对/home读写执行权限  
`setfacl -m d:u:alice:rwx /home`  

赋予Alice对以后在/home下新建的文件有读写执行权限(使Alice的权限继承)  
两种给普通用户提权的手段：  
sudo：了解，有针对性，例如针对某个用户以能够以root身份执行某些命令  
suid：基本针对所有用户，任何用户在执行有suid权限的程序时(例如/usr/bin/rm)，都是以root身份在执行  

高级权限suid, sgid, sticky  
suid：普通用户通过suid提权 <针对文件>  
在进程文件(二进制，可执行)上增加suid权限。以所有者身份执行  
sgid：新建文件继承目录数组 <针对目录>  
在文件夹上增加sgid，新创建的文件属组会得到继承  
文件谁可以删除：root，文件的所有者，目录的所有者  
使用案例：/tmp 目录  
sticky：用户只能删除自己的文件 <针对目录>  

新建的文件/目录权限受umask的影响，umask表示要减去的权限(每一个进程都拥有自己的umask)  

#### 文件属性
`lsattr filename`  
查看文件属性  
`chattr +a filename`   
日志文件很合适，只能在文件处增加内容  

#### 进程

程序文件：  
/usr/bin/passwd  
disk 28K  
memory / cpu / network 均不分配  

**进程的生命周期**  
父进程复制自己的地址空间(fock)创建一个新的(子)进程结构。每个新进程分配一个唯一的进程ID(PID)，满足跟踪安全性。PID和父进程ID(PPID)是子进程环境的元素，任何进程都可以创建子进程，所有进程都是第一个系统进程的后代：
centos7：systemd(以前是init，可以通过命令pstree进行查看)  

子进程继承父进程的安全性身份、过去和当前的文件描述符、端口和资源特权、环境变量，以及程序代码。随后，子进程可能exec自己的程序代码。通常，父进程在子进程运行期间处于睡眠(sleeping)状态。当子进程完成时发出(exit)信号请求，在退出时，子进程已经关闭或释放其资源环境，剩余的部分称之为僵停(僵尸zombie：除了ID号还保留，其余的所有资源都被回收)。父进程在子进程退出时收到信号而被唤醒，清理剩余的解构，然后继续执行其自己的程序代码  
example：ls/cd/rm...等命令的父进程为bash shell  

静态查看进程ps/top(查看手册)  
`ps aux |less`  
USER：运行进程的用户；PID：进程ID；%CPU：CPU占用率；%MEM：内存占用率；VSZ：占用虚拟内存；RSS：占用实际内存(驻留内存)；TTY：进程运行的终端；STAT：进程状态(man ps查看)；START：进程的启动时间；TIME：进程占用CPU的总时间；COMMAND：进程文件，进程名  
`cat /run/sshd.pid`  
查看某个进程的pid  
`ps aux |grep sshd`  
过滤查看某京城的PID  

#### 给进程发送信号
kill -l：查看所有支持的命令  

重启restart(stop, start)  
重新加载(reload, reconfigure)  
例：某个配置文件更改  
`vsftpd -- /etc/vsftpd/vsftpd.conf`  
`httpd -- /etc/httpd/conf/httpd.conf`  

`pkill -u alice` 杀掉Alice用户，包括其运行的终端  
`pkill -t pts/2` 终止pts/2上所有进程  
通过w命令查看各个终端信息，在通过此命令杀掉某个远程终端的运行程序，再加上-9就连进程带终端一起杀掉  

#### 进程优先级nice
nice值越高，表示优先级越低，该进程容易将CPU使用量让给其他人  
nice值越高，表示优先级越高，该进程更不倾向于让出CPU
使用ps命令查看nice进程  
`ps axo pid, command, nice --sort=-nice`  
TS表示该进程使用的调度策略为SCHED_OTHER  
启动进程时，通常会继承父进程的nice级别，默认为0  

#### 作业控制 jobs
作业控制作为一个命令行功能，允许一个shell实例来运行和管理多个命令  
如果没有作业控制，父进程fork()一个子进程后，将sleeping，直到子进程退出。使用作业控制，可以选择性暂停，回复，以及异步运行命令，让shell可以在子进程运行期间返回接收其他命令  
^Z可以将前台(forground)进程挂起(暂停)到后台  

jobs查看后台作业  
bg %2 让作业(%)2在后台运行  
fg %1 让作业(%)1在前台运行  
kill 2  杀死进程ID为2的进程  
kill %2 杀死作业号为2的作业  

& 放在命令的最后才是后台符(让程序在后台运行)  

**screen**
可以管理会话，断网断电后可以恢复jobs等  

---

## 管道及I/O重定向
输出重定向：  
`>` 覆盖  
`>>` 追加  
`<` 输入重定向  
- 0: stdin  
- 1: stdout  
- 2: stderr  
- &: 混合输出  

如果/dev/null设备被删除(rm -f /dev/null)  
mknod -m 666 /dev/null c 1 3  
创建块设备(有缓存)或者字符文件(无缓存)，权限666，c表示为字符设备，主设备号是1，从设备号是3  
主设备号与从设备号是设备文件独有的属性  
主设备号：表示为同一种设备类型，也可以认为kernel使用的是相同的块设备(驱动)  
从设备号：在同一类型设备中的一个序号  

`cat >file.txt <<-EOF`  
创建一个文件，将从键盘或指定的内容输入在文件中，遇到EOF结束，**-**表示忽略空格与tab键  

**subshell**  
通过()可以将命令在子shell中执行  
主要执行一些会影响当前shell造成影响的命令  
不希望某些命令对当前shell造成影响  

#### 进程管道Piping
**use redirection characters to control optput to file.**  
**use piping to control output to other programs.**  

**将/etc/passwd中的用户按UID大小排序**  
`sort -t":" -k3 -nr /etc/passwd |head`  
以:为分隔符，第三列按照数值逆序(从大到小)排列，最后将结果交给head，打印出前十个  

**统计出最占用CPU的5个进程**  
`ps aux --sort=-%cpu |head -6 |grep -v '%CPU'`  
`ps aux`命令的sort参数，-表示逆序排序，此时第一行为不需要的内容，所有有6行，我们可以再将结果进行过滤，-v表示不匹配带有%CPU的字段  

**统计出当前/etc/passwd中用户使用的shell类型**  
取出第七列(shell)|排序(把相同归类)|去重  
`awk -F: '{print $7}' /etc/passwd`  
将passwd文件中第七列取出  
`|sort |uniq -c`  
排序   去重 显示每一种的数量  

#### tee管道
将中间结果重定向到指定的文件  
在需要显示某一个阶段后面再次使用"管道" `|tee file.txt`，表示将中间结果存放至file.txt文件中  

## 存储管理

硬件设备命名  
物理硬盘：  
/dev/sd[a-z]  
KVM虚拟化：  
/dev/vd[a-z] (半虚拟化，增加硬盘可以online)  
/dev/sd[a-z] (全虚拟化，增加硬盘必须offline)  

#### 分区方式
MBR(DOS disklabel) < 2TB fdisk 14个分区(4个主分区，扩展分区，逻辑分区) 例如：3主+1扩展(可以再分n逻辑分区)  
GPT gdisk(parted) 128个主分区  
注意：从MBR转到GPT，或从GPT转换到MBR会导致数据全部丢失！  

MBR占用512字节(一个扇区大小)，前面446字节存放引导代码(boot code)，后面有16x4字节，存放四个主分区表。扩展分区建立扩展分区表，来记录逻辑分区的位置。  
实用场景：双系统，先装Windows，创建了MBR分区表，再安装Linux，抹去原有的分区表，但是在C盘有Windows的引导代码(每一个分区前面都有引导代码)  

#### 基本分区(首先创建RAID)

**基本分区管理**  
步骤：基本分区(MBR|GPT) --> Filesystem --> mount  


块大小是文件存储的最小单元：  
ext4最小为1024字节(1KB)  
xfs最小为4096字节(4KB)  
修改/etc/fstab文件，将分区自动挂载在某个文件夹下  

挂载就是将一个分区与一个文件夹关联起来  

**交换分区管理 Swap**
"提升"内存的容量，防止OOM(out of memory)  
查看：free -m  /  swapon -s  

#### EXT2/3/4文件系统
索引式文件系统/日志式文件系统  

在Linux中无论是磁盘空间还是inode号被占用完毕，都无法再创建新的文件  

新建一个文件的过程：  
1. 先确定使用者对于想创建文件目录是否具有W与X的权限
2. 根据inode bitmap找到没有使用的inode号，并将文件的权限和属性写入
3. 根据block bitmap找到没有使用的block号，将文件的实际数据写入block中，且更新inode的block指向信息
4. 将刚刚写入的inode与block信息同步更新inode bitmap与block bitmap，并更新superblock的内容。注意：如果突然断电，kernel发生错误等，则写入的信息仅有inode table以及data block而已，最后一个同步更新的步骤并没有做完，此时就会发生metadata的内容与实际信息产生不一致(inconsistent)的情况。

日志式文件系统(journaling filesystem)  
1. 准备：当系统要写入一个文件时，会现在日志区记录某个文件准备写入的信息
2. 实际写入：写入文件的权限与数据，更新metadata的信息
3. 结束：完成数据与medata的更新后，在日志记录区块中完成文件的记录

#### FAT文件系统
FAT文件系统没有inode，所以FAT没有将文件所有的block在一开始就读取出来。每个文件写入的block过于分散，读取速度慢。可以通过碎片整理的方式将分散的block尽量整理到一起。  

#### XFS文件系统
EXT家族支持度最广  
但创建文件系统(格式化)慢  
但修复慢，文件系统存储容量有限  

XFS同样是一种日志式文件系统：  
高容量，支持大存储  
高性能，创建/修复文件系统快  
inode与block都是系统需要时才用到时才动态配置生产  

#### 文件链接
符号链接(软链接)  
软链接就是记录路径，源文件路径越长，软链接就越大  
视频位置(79)  

硬链接  
通过命令`ll -i /etc/filename`就可以查看链接次数  
创建硬链接  
`ln /etc/filename /usr/filename`  
有多个文件，指向同一个inode  
**注意：**  
1. 硬链接不能跨文件系统(分区)
2. 不支持目录做硬链接

#### 磁盘阵列 RAID
RAID：廉价磁盘冗余阵列(redundant array of independent disks)  
作用：容错，提升读写速率  

#### lsof 恢复文件
lsof(list open files)  
FD文件句柄  

1. 查看打开文件message的进程/var/log/messages
2. 备份后模拟误删除文件
`cp -rf /var/log/messges /var/log/messages.bak`
`rm -rf /var/log/messages`
3. losf再次查看message文件状态
4. 查看相应进程的文件描述符FD，然后再拷贝回去
**注意：**  配置文件不能恢复，因为就是在开启服务的时候读，lsof只能够恢复持续运行的文件

---

## 文件管理

#### 文件查找
grep：文件内容过滤  
find：文件查找，针对文件名  
**命令文件**：which command  
为什么which找命令的位置这么快？  
PATH：环境变量，影响我们所执行的命令  
**常规文件**：locate、find(会遍历所指定的目录)  

#### 文件打包、压缩
`tar -czf/-cjf/-cJff filename /etc`  
-z调用gzip，-j调用bzip2，-J调用xz进行压缩，-f表示压缩后的文件名称  
tar -xf filename -C /tmp 对文件进行解压至tmp目录  
最与zip文件，直接使用unzip就可以直接解压  

案例：hostA /etc (海量小文件) 拷贝至 hostA /tmp  
`tar -czf - /etc |tar -xzf - -C /tmp`  
其中-表示直接进入内存，不进入硬盘，然后通过管道定向到tmp目录解压  
案例：hostA /etc (海量小文件) 拷贝至 hostB /tmp  
案例：  
`firewall -cmd --permanent --add-port=8888/tcp`  
对8888取消防火墙  
`firewall -cmd --reload`  
重新加载防火墙配置  
`nc -l 8888 |tar -zxf - -C /tmp`  
监听8888端口，然后将接收的到信息以管道的方式传入内存中，然后移到tmp目录。在hostA主机上面开始将数据打包，从内存直接发完hostB的8888端口  
`tar -czf - /etc |nc IPAddress 8888`  

---

## 软件包管理
安装/查询/卸载  
#### 软件类型
1. 源码包 需要编译  
2. 二进制包 以编译  

常见二进制包  
系统平台    包类型    工具    在线安装(自动解决依赖)
centos       RPM      rpm,rpmbuild    yum
ubuntu       DPKG     dpkg            apt
不管是源码包，还是二进制包，安装时都可能会有依赖关系！
卸载的时候使用yum history undo ID，即可将相关的依赖包一同卸载  
yum源可以更改成阿里云或者网易163的源镜像  

#### yum管理RPM源
`yum [list, search] [software_name]`查询已经安装/未安装的软件包，通常在后面增加包的名称或者通过|grep 进行过滤，其中search不仅关注软件包的名称，同时关注软件包的描述  
`yum -y remove httpd`  
卸载的时候不会把相关的依赖包一同卸载  

`yum -y update [software_name]`  
这个命令是将整个系统进行更新，更新某个软件则加相应的名称  

`uname -[r,m,a] r`  
是查看内科版本，m是查看系统位数，a是查看全部信息  

`yum provides vim`  
可以查看vim软件包的提供者，能够提供路径就尽量提供路径(*/vim)  

---

## 计划任务

#### 一次性调度执行at
作用：计划任务主要是做一些周期性的任务，目前最主要的用途是定期备份数据  
`yum -y install at`  
`systemctl start atd`  
`systemctl enable atd`  
atq查询任务队列  

#### 循环调度执行cron 用户级
crond进程没分钟会处理一次计划任务  
`* * * * * command`  
min  hour  day   mou   week  
00 02 * 6 5 /mysql_sh  
每年6月的周五2点整执行脚本mysql_sh  

---

## 日志管理基础

rsyslog 日志管理  
logrotate 日志轮转  
采集 --- 分析  

rsyslogd：绝大部分日志记录，和系统操作有关，安全，认证sshd, su计划任务at, cron等  
httpd/nginx/mysql：可以以自己的方式记录日志  

案例：统计登录失败的top5  
`grep  'Fail' /var/log/secure |awk '{print $11}' |sort |uniq -c |sort -k1 -n -r |head -5`  

---

## 网络管理

nmcli device 查看设备信息  
nmcli connection  查看设备连接关系  

CentOS使用NetWorkManager提供的工具有：nmcli, nmtui, nm-connect-editor  

/etc/hoostname 存放主机名  
/etc/resolv.conf 存放DNS解析服务的地址  
/etc/hosts 可以在这里加上指定网址的解析  
eg：119.75.218.70 www.baidu.com  
通常完成对自己的名字解析  

`ip -s link show eth0 `查看eth0网卡的数据包收发情况，包括传输与接收过程中丢弃的，接收的，错误的，碰撞的等  

traceroute www.sina.com 对目标进行路由跟踪  

临时配置网络信息  
ip/netmask  
ip addr add dev eth1 3.3.3.3/24  
ip addr del dev eth1 3.3.3.3/24  

geteway  
ip route del default  
ip route add default via 192.168.122.3  
ip route add 10.10.10.0/24 via 192.168.122.5  

hostname  
hostname yellow.com  

---

## vsftpd/NFS/CIFS

#### FTP Server

FTP：文件传输协议  
软件包：vsftpd  
FTP端口：控制端口 command 21/tcp  
         数据端口 data 20/tcp(主动模式)  
配置文件：/etc/vsftpd/vsftpd.conf  

#### FTP Client

`yum -y lftp wget`  

#### NFS
项目名称：为集群中的web server配置后端存储  
NFS：network file system网络文件系统，unit系统之间共享文件的一种协议  
NFS的客户端主要为Linux，支持多节点同时挂载以及并发写入  

#### CIFS
CIFS   

---

## DNS 域名系统
hosts文件  
作用：实现名字解析，主要为本地主机名，集群节点提供快速解析  
数据库：平面式结构，集中式数据库  
域名服务DNS  
作用：实现名字解析(例如将主机名称解析为IP)  
命名空间name sapce：用于给互联网上的主机命名的一种机制  
DNS数据库Database：层次化的，分布式的数据库  
权威名称服务器：存储并提供某个区域的实际数据，例如126.com的DNS服务器，记录了126.com域中所有的主机  
