---
title: OD断点设置技巧
categories: 安全
date: 2017-11-10 16:45:11
tags:
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**断点是一个信号，它通知调试器，在某个特定点上暂时将程序执行挂起。当执行在某个断点处挂起时，我们称程序处于中断模式。进入中断模式并不会终止或结束程序的执行。执行可以在任何时候继续。断点的本质还是一种中断。**<!-- more -->
# 主要的四种断点设置方式：
1.字符串断点
2.API断点
3.条件断点
4.run跟踪断点
* 断点设置：
    * API断点：
        * 1.可以在菜单栏—API断点处下断
        * 2.在命令行输入 bp+API函数名字，然后执行到返回（alt+F9）
    * 条件断点：
        * 1.运行
        * 2.在window窗口找到Edit（需要刷新）记录句柄信息
        * 3.在反汇编窗口，ctrl+G输入函数名字，定位在函数里面
        * 4.shift+F2设置条件断点，条件为【esp+4】==Edit地址
        * **注意：esp+4是句柄信息（也是GetWindowTextW的第一个参数)；条件是断在call的第一个参数处！！**
    * run追踪：
        * 1.Ctrl+E 找到程序模块
        ![](https://i.imgur.com/8sq9suE.png)
        * 2.Ctrl+T 暂停run跟踪条件，输入如下信息（选中EIP位于范围内，范围是基址和大小）
        ![](https://i.imgur.com/TRcA3SG.png)
        * 3.运行。
        * 补充内容：
            * 在反汇编窗口的快捷菜单中选择“Run跟踪［Run trace］|添加到所有函数入口处［Add entries of all procedures］”，<font color=#DC143C>这样能够检查每个可识别的函数被调用的次数。</font>
            * 另一个命令“Run跟踪［Run trace］|添加到函数中所有的分支［Add branches in procedure］”<font color=#DC143C>会强行跟踪此函数中所有识别的跳转目的地址的内容。</font>在这种情况下，统计功能能够找到最频繁执行的分支，您可以优化这部分的代码，以提高速度。
            * 在反汇编窗口中的某条命令上使用快捷菜单中选择“搜索［Search for］|Run跟踪的最新记录［Last record in run trace］”用于查找该命令是否被执行过，如果执行过，最后一次执行在哪里。  
* 其他断点设置：
    * 1.万能断点:ebp hmemcpy
	   * ebp 在调用函数处下断（bp是指在调用函数内部下断）
	   * hmemcpy---16位函数，截获大部分的字符串输入
    * 2.条件断点参考:[http://bbs.pediy.com/thread-16494.htm](http://bbs.pediy.com/thread-16494.htm)
    * 3.合适于xp的万能断点：
        * 1）首先查看user32模块
        * 2）Ctrl+B，利用字符串·搜索F3 A5 8B C8 83 E1 03 F3 A4 E8
        * 3）或者：8B C1 C1 E9 02 F3 A5 8B C8 83 E1 03 F3 A4 E8
        * 4）在停的地方下断，此时代码处在系统领空
        * 5）alt+F9，回到程序领空
    * **4.利用消息断点寻找关键跳转：**
        * 1）打开“windows”窗口（程序运行）
        * 2）在一些关键的地方设置消息断点（选择那些容易触发事件的地方）
        * 3）根据条件设置断点类型（如202鼠标右键消息类型）
        * 4）此时会断在系统领空，我们按Alt+F9回到用户·代码
        ![](https://i.imgur.com/KuaDdHq.png)
        * 5)补充：
            * 对于这片文章[http://bbs.pediy.com/showthread.php?t=46520](http://bbs.pediy.com/showthread.php?t=46520)出现下消息断点输入不完整的情况可以，下完断点，禁止断点，然后运行输入完毕后（不要点击确定）激活断点，运行就可！
