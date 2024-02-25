---
title: 将jar包打成exe可在任何window运行
date: 2024-2-25 21:16:00
comment: 'valine'
category: jar包制作exe
tags:
  - jar包制作exe
---



# 将jar包打成exe可在任何window运行

## 制作jar包

略。

## 制作exe

注意：这个制作不包含jre，只是将jar包制作成exe。

下载exe4j。可在网上搜索破解版，根据教程安装好。https://www.jb51.net/softs/541579.html

1.开始制作，选择Project type -> "JAR in EXE" mode

![image-20240225203939498](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225203939498.png)

2.填写软件名称和输出位置，next

![image-20240225204118259](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204118259.png)

3.根据jar包情况，选择对应的executable type，填写软件名，并选择34或者64位

![image-20240225204436041](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204436041.png)

![image-20240225204504308](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204504308.png)



4.选择java invocation 点击右边的绿色加好，选择archive，选择要制作的jar包，ok

![image-20240225204608238](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204608238.png)

5.选择启动类

![image-20240225204714552](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204714552.png)

6.选择jre版本如1.8，点击advanced Options，选择search sequence

![image-20240225204759636](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204759636.png)

7.全选，清空

![image-20240225204901348](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204901348.png)



8.点击绿色加号，添加jre路径。建议将jar和jre文件放在同一目录下。

![image-20240225204954967](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225204954967.png)

如：

![image-20240225205035420](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225205035420.png)

之后一直next，直到完成。

之后会在指定目录生成一个exe文件，如

![image-20240225205138190](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225205138190.png)

双击运行，检查是否正常。如果不正常，建议重新以上步骤。

**注意：此时只是将jar包制作成exe，里面并不包含运行环境jre。**

下面需要开始下载另外一个软件 inno setup

## inno setup

https://jrsoftware.org/isdl.php

![image-20240225205622534](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225205622534.png)

安装完成后，开始打包。

1.File -> new -> next

![image-20240225205709574](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225205709574.png)

2.填写软件名称，版本等信息，next。

![image-20240225205757326](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225205757326.png)

3.保持默认，直接next。

![image-20240225205826008](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225205826008.png)

4.选择exe所在位置，和添加根目录，next。



![image-20240225210143087](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210143087.png)

此时我的目录结构是

​	![image-20240225210209792](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210209792.png)



5.默认，next

![image-20240225210230062](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210230062.png)



6.默认，next

![image-20240225210251600](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210251600.png)



7.next

![image-20240225210304200](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210304200.png)

8.默认，next

![image-20240225210319620](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210319620.png)

9.默认，next

![image-20240225210334976](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210334976.png)

10.添加输出路径，next

![image-20240225210414634](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210414634.png)

11.默认，next

![image-20240225210440267](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210440267.png)

12.finish

![image-20240225210455402](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210455402.png)

12 是

![image-20240225210511177](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210511177.png)



13.是

![image-20240225210523072](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210523072.png)



14.等待编译完成

![image-20240225210556713](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210556713.png)

即可得到一个exe文件

![image-20240225210644715](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210644715.png)

对他进行安装，完成之后验证运用是否正常。下图可知，安装包已经包含了exe以及jre，说明可以打包成功。后序只需要将上一步的mysetup.exe文件发给别人安装即可运行你的程序了。

![image-20240225210805406](%E5%B0%86jar%E5%8C%85%E6%89%93%E6%88%90exe%E5%8F%AF%E5%9C%A8%E4%BB%BB%E4%BD%95window%E8%BF%90%E8%A1%8C/image-20240225210805406.png)

