### EPICS开发中可能遇到的问题以及搜集到的答案

#### linux以及虚拟机相关问题

###### [linux下source /etc/profile关闭终端失效问题](https://www.cnblogs.com/senlinyang/p/9431855.html)

##### [vim与windows/linux之间的复制粘贴小结](https://www.cnblogs.com/siashan/p/5546354.html)（其实对于这个问题建议用gedit算了）

##### [BASH同时执行多个命令](https://blog.csdn.net/magicpang/article/details/4106666)

##### [安装vmtools之后任然不能在虚拟机和主机之间复制粘贴的问题](https://blog.csdn.net/xc_zhou/article/details/80732396?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.not_use_machine_learn_pai&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.not_use_machine_learn_pai)

##### [启动Tomcat 出现 Can't load AMD 64-bit .dll on a IA 32-bit platform错误的解决办法](https://blog.csdn.net/qinkang1993/article/details/52671368)

windows设置静态ip无效的问题：注意设置的是电脑主机网卡的ip，如果和设备相连，设备ip可能会导致冲突。

#### IOC开发相关问题

1. 在复制拷贝ioc的时候，要注意到envPath是会同样复制的，这会导致一些常量没有改变，结果就是可能会使得协议用得是以前的协议

#### 开放工具使用相关问题

1.xftp无法连接，解决的主要思路是[先打开windows的ftp服务，然后更改协议，端口其实还没有查看先](https://cloud.tencent.com/developer/article/1618556?from=article.detail.1618573)，[xftp的编码问题见这里](https://www.xshellcn.com/zhishi/shezhi-bianma.html)





#### 其他问题

1如果出现了modbus拒绝连接的问题，可能是因为对应的端口没有打开，请谷歌如何打开502端口；然后还要记住，在使用modbus的时候，在命令行中指定了主机地址，所以当主机的地址发生改变的时候，一定要记得修改地址。