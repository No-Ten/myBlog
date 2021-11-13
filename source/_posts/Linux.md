---
title: Linux
date: 2021-11-12 22:01:17
index_img: /img/articleImages/linuxIndex.jpg
banner_img: /img/articleImages/linuxIndex.jpg
category: 操作系统
comment: 'valine'
tags:
  - Linux
---





# Linux笔记

Linux操作系统是基于UNIX操作系统发展而来的一种克隆系统。

<!--more-->

# 目录结构：

![image-20211102201829355](image-20211102201829355.png)

- /：根目录，所有目录最顶层的目录

- **~**：当前用户的主目录，如果是root用户就是/root/目录，如果是其他用户就是/home/下用户名的用户，如/home/admin

- ==**/bin**：bin是Binary的缩写, 这个目录存放着最经常使用的命令。==
- **/boot：** 这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。
- **/dev ：** dev是Device(设备)的缩写, 存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。
- ==**/etc：** 这个目录用来存放所有的系统管理所需要的配置文件和子目录。==
- **/home**：用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。
- **/lib**：这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。
- **/lost+found**：这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。
- **/media**：linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。
- **/mnt**：系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。
- ==**/opt**：这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。==
- **/proc**：这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。
- **/root**：该目录为系统管理员，也称作超级权限者的用户主目录。
- **/sbin**：s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。
- **/srv**：该目录存放一些服务启动之后需要提取的数据。
- **/sys**：这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。
- **/tmp**：这个目录是用来存放一些临时文件的。
- ==**/usr**：这是一个非常重要的目录，用户的很多应用程序和文件都放在这个目录下，类似于windows下的program files目录。==
- **/usr/bin：** 系统用户使用的应用程序。
- **/usr/sbin：** 超级用户使用的比较高级的管理程序和系统守护程序。
- **/usr/src：** 内核源代码默认的放置目录。
- **/var**：这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。
- **/run**：是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。



# 常用命令：



sudo -i用不了

进入vim /etc/sudoers，按照root账户形式，在下面添加一行

admin ALL =(ALL) ALL



## 关机命令：

- sync：将数据保存到硬盘中

- shutdown -r now：马上重启
- shutdown -h now：马上关机
- shutdown：关机
- init 0：关机
- reboot：重启 
- halt：关闭系统，等同于shutdown - h now 和poweroff



## 目录管理：

- cd：切换目录
- ./：当前目录
- cd.. ：返回上一级目录



### ls：列出目录

- ls -a：all，查看全部的文件，包含隐藏文件
- ls -l： 列出所有的文件，包含文件的属性和权限，没有隐藏文件



### cd命令

cd ../usr：相对路径进入usr文件

cd /home/admin: 绝对路径进去admin文件夹

cd ~：回到当前用户目录下



### pwd命令

- 显示当前用户所在的目录



### mkdir创建目录

- mkdir test：创建目录

- mkdir -p test/test1/test2/test3：递归创建多级目录




### rmdir 删除目录

- rmdir test3：删除一个空的目录

- rmdir -p test/test1/test2: 递归删除多个目录




### cp:复制文件或者目录

- cp 原来的地方 新的地方：cp 1.txt test

- 如果当前还有其他目录，需要用到-r进行递归，如：cp -r 1.txt test




### rm：异常文件或者目录

- rm -f：忽略不存在的文件，不会出现警告，强制删除

- rm -r：递归删除目录，需要确定是否删除

- rm -i :互动，询问是否删除

- rm -rf /*：直接删除系统中所有文件，没有警告，非常危险！！！不建议执行！！！！否则你将会这样


```bash
# 此处省略N行。。。
rm: cannot remove ‘/proc/697/task/713/mem’: Permission denied
rm: cannot remove ‘/proc/697/task/713/cwd’: Permission denied
rm: cannot remove ‘/proc/697/task/713/root’: Permission denied
rm: cannot remove ‘/proc/697/task/713/exe’: Permission denied
rm: cannot remove ‘/proc/697/task/713/mounts’: Permission denied
rm: cannot remove ‘/proc/697/task/713/mountinfo’^C
[root@localhost test]# rm: cannot remove ‘test2’: Is a directory
-bash: /usr/libexec/pk-command-not-found: /lib64/ld-linux-x86-64.so.2: bad ELF interpreter: No such file or directory
[root@localhost test]# rm: cannot remove ‘test2’: Is a directory
-bash: /usr/libexec/pk-command-not-found: /lib64/ld-linux-x86-64.so.2: bad ELF interpreter: No such file or directory
[root@localhost test]# ls
-bash: /bin/ls: No such file or directory
[root@localhost test]# cd ../
cd: error retrieving current directory: getcwd: cannot access parent directories: No such file or directory
[root@localhost ]# pwd
/home/admin/test/../
[root@localhost ]# cd /home
[root@localhost home]# ls
-bash: /bin/ls: No such file or directory
[root@localhost home]# cd ~
[root@localhost ~]# ls
-bash: /bin/ls: No such file or directory
[root@localhost ~]# ls -al
-bash: /bin/ls: No such file or directory
[root@localhost ~]# dir
-bash: /bin/dir: No such file or directory
[root@localhost ~]# 

```

![image-20211102165244390](image-20211102165244390.png)





### mv：移动文件或者目录，重命名

- mv -f：强制移动
- mv -u：只替换已经更新过的文件
- mv test test2：将test改名为test2
- mv：移动文件，如将a.txt文件移动到b文件夹下，mv a.txt b



- zxvf：解压文件

  


## 基本属性

![image-20211102203952548](image-20211102203952548.png)

实例中，boot文件的第一个属性用"d"表示。"d"在Linux中代表该文件是一个目录文件。

在Linux中第一个字符代表这个文件是目录、文件或链接文件等等：

- 当为[ **d** ]则是目录
- 当为[ **-** ]则是文件；
- 若是[ **l** ]则表示为链接文档 ( link file )；
- 若是[ **b** ]则表示为装置文件里面的可供储存的接口设备 ( 可随机存取装置 )；
- 若是[ **c** ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标 ( 一次性读取装置 )。

接下来的字符中，以三个为一组，且均为【rwx】 的三个参数的组合。

- [ r ]代表可读(read)
- [ w ]代表可写(write)
- [ x ]代表可执行(execute)

要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。

![image-20211102204306150](image-20211102204306150.png)

从左至右用0-9这些数字来表示。

第0位确定文件类型，第1-3位确定属主（该文件的所有者）拥有该文件的权限。第4-6位确定属组（所有者的同组用户）拥有该文件的权限，第7-9位确定其他用户拥有该文件的权限。

1、4、7：读

2、5、8：写

3、6、9：操作

![image-20211102204619828](image-20211102204619828.png)



## 更改文件属性

### chgrp：更改文件的属组

- chgrp -R 属组 文件名：如chgrp -R root test，test原本所属组为admin，现在改为root
- -R：递归更改文件属组，就是在更改某个目录文件的属组时，如果加上-R的参数，那么该目录下的所有文件的属组都会更改。

```bash
[root@localhost Ten]# ls -l
total 0
drwxrwxr-x. 2 admin admin 6 Nov  2 20:09 test
drwxrwxr-x. 2 admin admin 6 Nov  2 20:11 test2
[root@localhost Ten]# chgrp -R root test
[root@localhost Ten]# ls -l
total 0
drwxrwxr-x. 2 admin root  6 Nov  2 20:09 test
drwxrwxr-x. 2 admin admin 6 Nov  2 20:11 test2
[root@localhost Ten]# 
```

![image-20211102205944102](image-20211102205944102.png)

### chown：更改文件的属主

- chown -R 属主名 文件名：如chown -R root test，将test的属主改为root
- chown -R 属主名:属组名 文件名：如chown -R admin:admin test，将test的所属主名和属组名都改名admin



### chmod：更改文件9个属性

- chmod -R xyz 文件或者目录：如，chmod -R 777 test，将test文件的三个权限都更改rwx。

Linux文件属性有两种设置方法，一种是数字，一种是符号。

Linux文件的基本权限就有九个，分别是owner/group/others三种身份各有自己的read/write/execute权限。

```
r:4     w:2         x:1
```

每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如当权限为：[-rwxrwx---] 分数则是：

- owner = rwx = 4+2+1 = 7
- group = rwx = 4+2+1 = 7
- others= --- = 0+0+0 = 0



## 文件内容查看

- ifconfig：查看网络配置

- ==cat：由第一行开始显示文件内容==
- tac：从最后一行开始显示文件内容，cat和tac是倒过来写的
- ==nl：显示文章行号==

![image-20211102231332124](image-20211102231332124.png)

- ==more：一页一页的显示文件内容，空格翻页，enter代表向下一行，:f 查看当前行号，按B可以往上翻==
- ==less：与more类似，可以往前翻页，空格翻页，上下键代表翻动页面，q表示退出，查询字符串/要查询的字符串（向下查询），向上查询使用?要查询的字符串，如果还想继续查询可以用n向上查询，N向下==
- head：只看头几行，如head -n 20 csh.login ，查看csh.login文件的前20行
- tail：之后尾几行，如tail -n 20 csh.login，查看csh.login文件的后20行



## 拓展：Linux链接的概念（了解即可）

Linux的链接分为两种：硬链接、软链接

**硬链接：**硬连接指通过索引节点来进行连接。在 Linux 的文件系统中，保存在磁盘分区中的文件不管是什么类型都给它分配一个编号，称为索引节点号(Inode Index)。允许一个文件拥有多个有效路径名，这样用户就可以建立硬连接到重要文件，以防止“误删”的功能，如，B是A的硬链接，当A被删除时，B依旧可以访问，反之亦然。只有当所有的硬链接都删除后，才可以彻底删除文件。

**软链接：**另外一种连接称之为符号连接（Symbolic Link），也叫软连接。软链接文件有类似于 Windows 的快捷方式。它实际上是一个特殊的文件。比如，给A创建一个软链接B，当A被删除时，B就无法访问了。也可以理解为A为源文件，B为快捷键，源文件被删除了，该文件的快捷键当然也是访问不了的。



- touch：创建文件，如touch f1，创建f1文件
- echo：输入字符串，也可以输入到文件中，如echo "Hello" >> f1，往f1中添加”Hello“
- ln：创建一个硬链接，如ln f1 f2，创建一个硬链接，从f1指向f2
- ln -s ：创建一个软链接，如ln -s f1 f3，创建一个软链接，从f1指向f3

```bash
[admin@localhost Ten]$ touch f1				# 创建f1
[admin@localhost Ten]$ ls
f1  test  test2
[admin@localhost Ten]$ ln f1 f2				# 创建从f1指向f2的硬链接
[admin@localhost Ten]$ ls
f1  f2  test  test2
[admin@localhost Ten]$ ln -s f1 f3			# 创建从f1指向f3的软链接（符号链接）
[admin@localhost Ten]$ ls
f1  f2  f3  test  test2
[admin@localhost Ten]$ ls -l
total 0
-rw-rw-r--. 2 admin admin 0 Nov  3 11:37 f1
-rw-rw-r--. 2 admin admin 0 Nov  3 11:37 f2
lrwxrwxrwx. 1 admin admin 2 Nov  3 11:37 f3 -> f1
drwxrwxr-x. 2 admin admin 6 Nov  2 20:09 test
drwxrwxr-x. 2 admin admin 6 Nov  2 20:11 test2
[admin@localhost Ten]$ 
[admin@localhost Ten]$ echo "Hello" >> "f1"	# 往f1中写入内容
[admin@localhost Ten]$ cat f1				# f1正常读
Hello
[admin@localhost Ten]$ cat f2				# f2正常读
Hello
[admin@localhost Ten]$ cat f3				# f3正常读
Hello
[admin@localhost Ten]$ 
```



删除

```bash
[admin@localhost Ten]$ ls
f1  f2  f3  test  test2
[admin@localhost Ten]$ rm -rf f1		# 删除f1
[admin@localhost Ten]$ ls 
f2  f3  test  test2
[admin@localhost Ten]$ cat f2			# f2是硬链接，没有影响，依旧可以访问
I am Ten
[admin@localhost Ten]$ cat f3			# f3是软链接，已经失效
cat: f3: No such file or directory
[admin@localhost Ten]$ ^C
```

- ls -li：显示索引节点号（Inode Index）

```bash
[admin@localhost Ten]$ ls -li
total 8
 5194981 -rw-rw-r--. 2 admin admin 6 Nov  3 11:38 f1
 5194981 -rw-rw-r--. 2 admin admin 6 Nov  3 11:38 f2
 5195004 lrwxrwxrwx. 1 admin admin 2 Nov  3 11:37 f3 -> f1
13209000 drwxrwxr-x. 2 admin admin 6 Nov  2 20:09 test
   77723 drwxrwxr-x. 2 admin admin 6 Nov  2 20:11 test2
[admin@localhost Ten]$ 

```



## Vim编辑器

基本上 vi/vim 共分为三种模式，分别是**命令模式（Command mode）**，**输入模式（Insert mode）**和**底线命令模式（Last line mode）**。这三种模式的作用分别是：

- Vim Study：创建或编辑一个Study的文件，如果存入Study就编辑，如果不存在就会新建

**命令模式：**

- **i** 切换到输入模式，以输入字符。
- **x** 删除当前光标所在处的字符。
- **:** 切换到底线命令模式，以在最底一行输入命令。

**输入模式：**

- **字符按键以及Shift组合**，输入字符
- **ENTER**，回车键，换行
- **BACK SPACE**，退格键，删除光标前一个字符
- **DEL**，删除键，删除光标后一个字符
- **方向键**，在文本中移动光标
- **HOME**/**END**，移动光标到行首/行尾
- **Page Up**/**Page Down**，上/下翻页
- **Insert**，切换光标为输入/替换模式，光标将变成竖线/下划线
- **ESC**，退出输入模式，切换到命令模式

**底线命令模式**

- :q，退出程序
- :w，保存文件

![image-20211103165822413](image-20211103165822413.png)



**第一部分：一般模式可用的光标移动、复制粘贴、搜索替换等**

| 移动光标的方法     |                                                              |
| :----------------- | ------------------------------------------------------------ |
| h 或 向左箭头键(←) | 光标向左移动一个字符                                         |
| j 或 向下箭头键(↓) | 光标向下移动一个字符                                         |
| k 或 向上箭头键(↑) | 光标向上移动一个字符                                         |
| l 或 向右箭头键(→) | 光标向右移动一个字符                                         |
| [Ctrl] + [f]       | 屏幕『向下』移动一页，相当于 [Page Down]按键 (常用)          |
| [Ctrl] + [b]       | 屏幕『向上』移动一页，相当于 [Page Up] 按键 (常用)           |
| [Ctrl] + [d]       | 屏幕『向下』移动半页                                         |
| [Ctrl] + [u]       | 屏幕『向上』移动半页                                         |
| +                  | 光标移动到非空格符的下一行                                   |
| -                  | 光标移动到非空格符的上一行                                   |
| ==n< space>==      | 那个 n 表示『数字』，例如 20 。按下数字后再按空格键，光标会向右移动这一行的 n 个字符。 |
| 0 或功能键[Home]   | 这是数字『 0 』：移动到这一行的最前面字符处 (常用)           |
| $ 或功能键[End]    | 移动到这一行的最后面字符处(常用)                             |
| H                  | 光标移动到这个屏幕的最上方那一行的第一个字符                 |
| M                  | 光标移动到这个屏幕的中央那一行的第一个字符                   |
| L                  | 光标移动到这个屏幕的最下方那一行的第一个字符                 |
| G                  | 移动到这个档案的最后一行(常用)                               |
| nG                 | n 为数字。移动到这个档案的第 n 行。例如 20G 则会移动到这个档案的第 20 行(可配合 :set nu) |
| gg                 | 移动到这个档案的第一行，相当于 1G 啊！(常用)                 |
| n< Enter>          | n 为数字。光标向下移动 n 行(常用)                            |



| 搜索替换  |                                                              |
| :-------- | ------------------------------------------------------------ |
| ==/word== | 向光标之下寻找一个名称为 word 的字符串。例如要在档案内搜寻 vbird 这个字符串，就输入 /vbird 即可！(常用) |
| ?word     | 向光标之上寻找一个字符串名称为 word 的字符串。               |
| ==n==     | 这个 n 是英文按键。代表重复前一个搜寻的动作。举例来说， 如果刚刚我们执行 /vbird 去向下搜寻 vbird 这个字符串，则按下 n 后，会向下继续搜寻下一个名称为 vbird 的字符串。如果是执行 ?vbird 的话，那么按下 n 则会向上继续搜寻名称为 vbird 的字符串！ |
| ==N==     | 这个 N 是英文按键。与 n 刚好相反，为『反向』进行前一个搜寻动作。例如 /vbird 后，按下 N 则表示『向上』搜寻 vbird 。 |

| 删除、复制与粘贴 |                                                              |
| :--------------- | ------------------------------------------------------------ |
| x, X             | 在一行字当中，x 为向后删除一个字符 (相当于 [del] 按键)， X 为向前删除一个字符(相当于 [backspace] 亦即是退格键) (常用) |
| nx               | n 为数字，连续向后删除 n 个字符。举例来说，我要连续删除 10 个字符， 『10x』。 |
| dd               | 删除游标所在的那一整行(常用)                                 |
| ndd              | n 为数字。删除光标所在的向下 n 行，例如 20dd 则是删除 20 行 (常用) |
| d1G              | 删除光标所在到第一行的所有数据                               |
| dG               | 删除光标所在到最后一行的所有数据                             |
| d$               | 删除游标所在处，到该行的最后一个字符                         |
| d0               | 那个是数字的 0 ，删除游标所在处，到该行的最前面一个字符      |
| yy               | 复制游标所在的那一行(常用)                                   |
| nyy              | n 为数字。复制光标所在的向下 n 行，例如 20yy 则是复制 20 行(常用) |
| y1G              | 复制游标所在行到第一行的所有数据                             |
| yG               | 复制游标所在行到最后一行的所有数据                           |
| y0               | 复制光标所在的那个字符到该行行首的所有数据                   |
| y$               | 复制光标所在的那个字符到该行行尾的所有数据                   |
| p, P             | p 为将已复制的数据在光标下一行贴上，P 则为贴在游标上一行！举例来说，我目前光标在第 20 行，且已经复制了 10 行数据。则按下 p 后， 那 10 行数据会贴在原本的 20 行之后，亦即由 21 行开始贴。但如果是按下 P 呢？那么原本的第 20 行会被推到变成 30 行。(常用) |
| J                | 将光标所在行与下一行的数据结合成同一行                       |
| c                | 重复删除多个数据，例如向下删除 10 行，[ 10cj ]               |
| u                | 复原前一个动作。(常用)                                       |
| [Ctrl]+r         | 重做上一个动作。(常用)                                       |



**第二部分：一般模式切换到编辑模式的可用的按钮说明**

| 进入输入或取代的编辑模式 |                                                              |
| :----------------------- | ------------------------------------------------------------ |
| ==i, I==                 | 进入输入模式(Insert mode)：i 为『从目前光标所在处输入』， I 为『在目前所在行的第一个非空格符处开始输入』。(常用) |
| a, A                     | 进入输入模式(Insert mode)：a 为『从目前光标所在的下一个字符处开始输入』， A 为『从光标所在行的最后一个字符处开始输入』。(常用) |
| o, O                     | 进入输入模式(Insert mode)：这是英文字母 o 的大小写。o 为『在目前光标所在的下一行处输入新的一行』；O 为在目前光标所在处的上一行输入新的一行！(常用) |
| r, R                     | 进入取代模式(Replace mode)：r 只会取代光标所在的那一个字符一次；R会一直取代光标所在的文字，直到按下 ESC 为止；(常用) |
| [Esc]                    | 退出编辑模式，回到一般模式中(常用)                           |



**第三部分：一般模式切换到指令行模式的可用的按钮说明**

| 指令行的储存、离开等指令                                     |                                                              |
| :----------------------------------------------------------- | ------------------------------------------------------------ |
| :w                                                           | 将编辑的数据写入硬盘档案中(常用)                             |
| :w!                                                          | 若文件属性为『只读』时，强制写入该档案。不过，到底能不能写入， 还是跟你对该档案的档案权限有关啊！ |
| :q                                                           | 离开 vi (常用)                                               |
| :q!                                                          | 若曾修改过档案，又不想储存，使用 ! 为强制离开不储存档案。    |
| 注意一下啊，那个惊叹号 (!) 在 vi 当中，常常具有『强制』的意思～ |                                                              |
| ==:wq==                                                      | 储存后离开，若为 :wq! 则为强制储存后离开 (常用)              |
| ZZ                                                           | 这是大写的 Z 喔！若档案没有更动，则不储存离开，若档案已经被更动过，则储存后离开！ |
| :w [filename]                                                | 将编辑的数据储存成另一个档案（类似另存新档）                 |
| :r [filename]                                                | 在编辑的数据中，读入另一个档案的数据。亦即将 『filename』 这个档案内容加到游标所在行后面 |
| :n1,n2 w [filename]                                          | 将 n1 到 n2 的内容储存成 filename 这个档案。                 |
| :! command                                                   | 暂时离开 vi 到指令行模式下执行 command 的显示结果！例如 『:! ls /home』即可在 vi 当中看 /home 底下以 ls 输出的档案信息！ |
| ==:set nu==                                                  | 显示行号，设定之后，会在每一行的前缀显示该行的行号           |
| :set nonu                                                    | 与 set nu 相反，为取消行号！                                 |



## 账号管理

**添加用户，useradd命令**

- useradd -m  用户名：自动创建这个用户的目录/home/Test，如useradd -m Test,创建一个名为Test的用户

- cat /etc/passwd，就可以看到刚刚被创建的用户
- useradd -G 用户名：分配组



**删除用户 userdel**

- userdel - r Test，将刚刚创建的Test用户的目录一并删掉



**修改用户** usermod

- usermod 对应修改的内容 修改的用户，如usermod  -d /home/233 Test，将Test用户的主目录修改为233，修改完查看配置文件即可



**切换用户**

- su 用户名，如su Test，或者su - root
- #：超级管理员，$：普通用户

![image-20211103174249440](image-20211103174249440.png)

- hostname 名字:修改主机名，如hostname Test，将主机名改为Test，改完之后重新连接。



**修改用户的密码**

root下：

- passwd 用户名，如passwd Test，修改

用户下：

- 直接passwd就可以输入了



**锁定用户**

- passwd -l 用户名，如passwd -l Test，锁定Test用户，用户就不能登录了
- passwd -d 用户名，没有密码也不能登录
- passwd -u 用户名， 将锁定的用户解锁



## 用户组管理

属主、属组

每个用户都有一个用户组，系统可以对一个用户组中的所有用户进行集中管理。用户组的管理涉及用户组的添加、删除和修改。组的增加、删除和修改实际上就是对/etc/group文件的更新。

**groupadd：创建一个用户组**

- groupadd Test：创建一个Test的用户组
- groupadd -g 111 Test1，创建一个端口号为111的Test1组，如果不设置用户组id，他会自增。
- cat /etc/group：查看用户组的情况

**groupdel：删除用户组**

- groupdel Test：删除Test的用户组

**groupmod：修改**

- groupmod -g 666 -n newTest Test：将Test的用户组的用户组id改为666，并将名字改为newTest

**切换用户组**

```bash
# 登录当前用户 Test
$ newgrp root
```



**拓展：文件的查看**（了解）

Linux系统中的每个用户都在/etc/passwd文件中有一个对应的记录行，它记录了这个用户的一些基本属性。

- /etc/passwd:

```bash
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
...
ntp:x:38:38::/etc/ntp:/sbin/nologin
tcpdump:x:72:72::/:/sbin/nologin
admin:x:1000:1000:admin:/home/admin:/bin/bash
Test:x:1001:666::/home/Test:/bin/bash
```



```
用户名:口令（登录密码，不可见）:用户标识号:组标识号:注释性描述:主目录:登录Shell
```

- /etc/shadow

登录口令：把真正的加密后的用户口令字存放到/etc/shadow文件中，而在/etc/passwd文件的口令字段中只存放一个特殊的字符，例如“x”或者“*”。

- /etc/group

用户组的所有信息都存放在/etc/group文件中。



## 磁盘管理

df（列出文件系统整体的磁盘使用）  du（检查磁盘空间使用量）

- df -h:列出多少M
- du -a:查看所有文件使用情况，可以看到子文件
- du -sm /* :检查根目录下，每个目录所占用的容量



**Mac或者想要Linux挂载我们的一些本地磁盘或者文件**

挂载

- mount /dev/Test /mnt/Test：将外部设备Test的U盘挂载到模mnt下面

卸载：umount -f 挂载位置 ：强制卸载



## 进程管理

==Linux中一切皆文件（文件：读写执行（查看、创建、删除、移动、复制、编辑 ），权限（用户、用户组）。系统：（磁盘、进程））==

**基本概念**

1. 在Linux中，每一个程序都有自己的一个进程，每一个进程都有一个id号
2. 每一个进程，都有一个父进程！ 
3. 进程可以有两种存在方式：前台！后台运行！
4. 一般的话服务都是后天运行的，基本的程序都是前天运行

**命令**

- ps：查看当前系统中正在执行的各种进程的信息
- ps -a:显示当前终端运行的所有的进程信息
- ps -u:以用户的信息显示信息
- ps -x:显示后台运行进程的参数
- ps -aux:查看所有的进程

```bash
ps -aux | grep mysql		# 过滤出与mysql相关的进程
# | 在Linux中这个叫做管道符		A|B，将A的输出结果作为B的输出条件
# grep 查找文件中符合条件的字符串！
```

**ps -ef:可以查看到父进程的信息**

```bash
ps -ef | grep mysql		# 看父进程一般通过目录树结构来查看


pstree -pu
pstree -p				# 显示父id
pstree -u				# 显示当前用户组
```

![image-20211103222647990](image-20211103222647990.png)



结束进程：杀掉进程，等价于window结束任务

- kill -9 进程的id：强制结束该进程



# 环境安装

## JDK安装(rpm)

rpm安装jdk不用配置环境变量

```bash
# 检查当前系统是否存在java环境	java -version
# 如果存在就卸载
# rpm -qa|grep jdk		# 检测JDK版本信息
# rpm -e --nodeps jdk_	# 强制卸载 

# 卸载完后再重新安装
# rpm -ivh rpm包

# 配置环境变量
```

配置环境变量：/etc/profile

```bash
export JAVA_HOME=/usr/local/java/jdk1.8.0_121
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}

```

```bash
export JAVA_HOME=/usr/local/java/java1.8...
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:${CLASSPATH}
export PATH=$PAHT:${JAVA_HOME}/bin:${JRE_HOME}/bin
```



让这个配置文件生效：source /etc/profi



查看端口：

firewall-cmd --list-ports

开启防火墙端口：

firewall-cmd --zone=pulic --add-port=9000/tcp --p ermanent

重启防火墙：

systemctl restart firewalld.service

检查当前的网卡文件：

ip addr



## Tomcat安装（tar.gz）

1. 解压

```bash
unzip 文件名 		# 如果是压缩包是zip格式的，使用
tar -zvxf 包名
```

2. 启动Tomcat测试，./xxx.sh脚本即可运行

```bash
 # 执行 ./startup.sh
 # 停止 ./shutdown.sh
```

3. 查看防火墙，并开启8080端口

```bash
# 查看firewall服务状态
systemctl status firewalld

# 开启、重启、关闭、firewalld.service服务
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop

# 查看防火墙规则
firewall-cmd --list-all    # 查看全部信息
firewall-cmd --list-ports  # 只看端口信息

# 开启端口
开端口命令：firewall-cmd --zone=public --add-port=80/tcp --permanent
重启防火墙：systemctl restart firewalld.service

命令含义：
--zone #作用域
--add-port=80/tcp  #添加端口，格式为：端口/通讯协议
--permanent   #永久生效，没有此参数重启后失效
```

4. 测试。访问192.168.242.3:8080。出现页面，表示安装成功。



## 安装Docker（yum）

1. 查看系统版本信息

   ```bash
   [root@localhost bin]# cat /etc/redhat-release 
   CentOS Linux release 7.8.2003 (Core)
   ```

   

2. 安装我们的准备环境

   ```bash
   yum -y install 包名		# yum install 安装命令 	-y	所有的提示都为y
   yum -y install gcc
   yum -y install gcc-c++
   ```

   

3. 或者用宝塔面板安装 





# VMare使用

### 快照

保留当前系统的信息为快照，随时可以恢复，以防未来系统被损坏，就好比游戏中的归档



# 问题

## 使用xftp上传文件时，状态显示错误

![image-20211104114435498](image-20211104114435498.png)

原因可能是：内存不够、权限不够（切换root）、或者是目录权限不够，如果是目录权限不够可以用以下命令设置权限

```bash
chmod 777 目录名
```



使用tar.gz包

安装到/home/admin/java

```bash
tar -zxvf jdk包名
```

配置环境变量

vim /etc/profile

```bash
export JAVA_HOME=/home/admin/java/jdk1.8.0_121
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

```

刷新配置文件：

```bash
source /etc/profile
```

最后java -version检查是否配置成功



这个笔记是在学习狂神说Linux做的笔记，如需原版，请前往b站。地址：https://www.bilibili.com/video/BV187411y7hF?spm_id_from=333.999.0.0
