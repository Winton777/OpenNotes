
# 前言
首先感谢本文原作者，安装教程参考自[lxyoucan/CentOS7Oracle11gInstallHelper: CentOS7中Oracle11g安装助手 (github.com)](https://github.com/lxyoucan/CentOS7Oracle11gInstallHelper)，我补充了一些我在Linux服务器上安装Oracle数据库的过程中遇见的问题和解决方案，断断续续安装了四五天才调试成功，希望能对大家有所帮助。(σﾟ∀ﾟ)σ..:*☆哎哟不错哦！

相关文件：https://pan.baidu.com/s/1PR7vUBR_NuB1-wEtIT8Fpw?pwd=ofkr 

# 环境信息
不同的环境可能会有小小的差异，都是大同小异的。下面给出我用的版本信息：
项目     | 版本
-------- | :---:
操作系统  | CentOS Linux release 7.9.2009 (Core) 
oracle  | linux.x64_11gR2_11.2.0.1.0           
PL/SQL | Version 13.0.6.1911 64bit            

**Oracle11g版本选择：**
强烈建议下载 11.2.0.4版本的，oracle版本是官网下载的11.2.0.1有点小坑在里面，实在找不到也不影响使用。

# 准备工作
兵马未动，粮草先行。个人觉得linux下安装oracle也就是准备工作比较麻烦一些。

在安装之前，我们要做一些简单的准备工作，大致如下：

1. 创建oracle用户和组
2. 搭建图形化的操作环境：VNC远程。
3. 防火墙放行VNC端口5901和Oracle默认端口1521。
4. 安装oracle安装程序依赖程序包。
5. 安装中文字体解决中文乱码问题。
6. 单独安装pdksh-5.2.14

为了提升安装oracle体验，原作者把以上操作整理成脚本供大家使用，感兴趣的可以看一下代码。

我这里也是用了原作者的脚本，再次感谢大佬，ヾ(o′▽`o)ノ°° .°谢谢ｰ° .°

下载脚本上传到Linux后解压，用root用户执行以下命令，自动完成依赖软件的安装与配置。如果没有unzip工具，用`yum install unzip`命令安装unzip用于文件解压。

root执行以下命令

```bash
cd CentOS7Oracle11gInstallHelper
chmod +x helper.sh
./helper.sh
```

## 创建用户&开启VNC服务
为了安全起见，不建议使用root做为vnc用户。单独创建一个用户比较安全。既然安装oracle，这里用户名我使用 oracle。记得用root给oracle用户设密码:`passwd oracle`

root执行以下命令，直接整体复制粘贴到终端就行（不用一行一行复制）。
```bash
#创建database用户组
groupadd database
#创建oracle用户并放入database组中
useradd oracle -g database
#切换到oracle用户
su oracle
#首次运行，生成~/.vnc/xstartup等配置文件
vncserver :1 -geometry 1024x768
#PS:是否创建 view-only password 选择时输入 n 即可
```
oracle用户执行以下命令，直接整体复制粘贴到终端就行（不用一行一行复制）。

```bash
#配置VNC默认启动openbox
echo "openbox-session &" > ~/.vnc/xstartup
# 停止服务
vncserver -kill :1
#重新开启vnc服务
vncserver :1 -geometry 1024x768
```
## 客户端连接VNC实现远程控制
使用你的VNC客户端连接就行了，会的就略过吧。
我用的是：[VNC Viewer点击下载](https://www.realvnc.com/en/connect/download/viewer/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210129111608185.png)
输入Linux地址，默认端口是5901，然后输入上面设置的连接密码就可以了。

如果你连接的时候发现，没有界面，是黑屏的只有一个鼠标，那么可以**重启一下VNC服务**试试。

```bash
#云服务器还需在控制台打开5901端口
iptables -L #查看5901端口是否开启
iptables -I INPUT 1 -p TCP --dport 5901:5910 -j ACCEPT #开启5901端口
vncserver -list #查看vnc已开启服务
#切换到oracle用户
su oracle
vncserver -kill :1
vncserver :1 -geometry 1024x768
```

## 上传并解压安装包
上传安装包到 CentOS7服务器。我上传到 `/home/oracle/`目录了。
上传以后，文件路径和名称如下：

```bash
[oracle@localhost ~]$ pwd
/home/oracle
[oracle@localhost ~]$ ls
p13390677_112040_Linux-x86-64_1of7.zip  p13390677_112040_Linux-x86-64_2of7.zip
```
只需要`*1of7.zip`,`*2of7.zip` 两个压缩包即可。

oracle用户登录vnc，执行下面命令，解压安装包
```bash
#解压第1个zip
unzip p13390677_112040_Linux-x86-64_1of7.zip
#解压第2个zip
unzip p13390677_112040_Linux-x86-64_2of7.zip
```
前期准备工作已经完毕！

<hr>

# 安装oracle实战
准备工作已经结束了，接下来的安装工作就跟在windows下安装oracle差不多了。先总结一下，基本就是根据界面提示就可以一路“`下一步（N）`”就可以完成了。
需要稍微注意的就是：
1. 桌面类与服务器类的选择
2. 超级管理员密码的设置
3. 先决条件检查

其它的根据自己的需要，或者一路“`下一步（N）`”就可以完成了。为了一些朋友更直观的观看，我把每一步都截图了，可以跳着看。

## oracle用户登录vnc远程桌面。
进入`~/database/`目录。
```bash
#进入安装目录
cd ~/database/
#运行安装程序
./runInstaller
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202144033104.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202144115400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)

## 配置安全更新
根据需要设置，我这里就不设置了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202144454451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 下载软件更新
根据个人需要选择，我这里选择 `跳过软件更新（S）`。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202144643740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 网络安装选项
选择“`创建和配置数据库（C）`”
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202144833620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 桌面类 or 服务器类
描述中已经说的很清楚了，根据自己需要选择。这里我选择的是`服务器类（S）`。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202145134580.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 安装类型
我选默认的，`单实例数据库安装（S）`根据实际需要选择。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202145313770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 典型安装
默认`典型安装（T）`即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021020214545713.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 典型安装配置
主要设置一下密码，其他默认即可。这里密码要在大写字母+小写字母+数字组合。比如：我设置的是`Database123`。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202145942205.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 创建产品清单
默认即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202150120775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 执行先决条件检查
这一步要稍花一些时间处理。每个人的显示可能略有不同。比如：物理内存的检测，我这个1G内存就会提示小于预期。
处理方法：
- 根据提示信息做处理即可，比如：内存小了，加大内存啊。
- 执行`修补并再次检查（F）` 可以自动修复
- 以上都解决不了，百度一下你就知道。基本都是有解决办法的。

### CentOS7安装11.2.0.1遇到的坑：

1. 操作系统内核参数semmni 明明设置正确，先决条件检查以然会提示参数不正确
2. 一些程序包，已经安装过了，先决条件检查以然会提示没有安装
3. 安装到 68%会报错：makefile '/home/oracle/app/oracle/product/11.2.0/dbhome_1/ctx/lib/ins_ctx.mk'的目标'install'时出错

若只剩坑 1，2的话可以直接点右上角忽略，3网上也有解决办法。但是总让人有点不舒服。好在 11.2.0.4版本中已经没有这些坑了，舒心多了。

没解决之前我的显示如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202151138502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
执行`修补并再次检查（F）` 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202151223407.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
方法上面描述的很清楚。
root权限执行：

```bash
/tmp/CVU_11.2.0.4.0_oracle/runfixup.sh
```
执行完后，点击上面对话框中的`确定（O）` 这里发现大部分都修复了。如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202151720157.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
剩下的警告尽量解决，如果自己知道影响不大直接点右上角 `☐全部忽略`即可。
比如：我的虚拟机内存，比预期值差30MB左右，影响不大直接忽略也可以。
解决方法也很简单，加大内存即可。既然写教程了，就追求完美吧，我把虚拟机内存加大一些。

### 解决 包：pdksh-5.2.14 警告
这个警告，我猜测直接忽略就行了。因为本机已经安装了ksh-20120801-142.el7.x86_64。
`yum search pdksh`中搜索没的搜索到它。只能手动安装了。

```bash
#下载安装包
wget  http://vault.centos.org/5.11/os/x86_64/CentOS/pdksh-5.2.14-37.el5_8.1.x86_64.rpm
```
如果没有wget就安装一下 `yum install wget`
执行安装操作。
```bash
rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm
```
执行结果如下，与已经安装的冲突了，安装失败了。
```bash
rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm 
警告：pdksh-5.2.14-37.el5_8.1.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID e8562897: NOKEY
错误：依赖检测失败：
	pdksh 与 (已安裝) ksh-20120801-142.el7.x86_64 冲突
```
**卸载冲突**

```bash
rpm -e ksh-20120801-142.el7.x86_64
```
**再次安装**

```bash
rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm
```
全过程如下：

```bash
[root@localhost ~]# rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm 
警告：pdksh-5.2.14-37.el5_8.1.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID e8562897: NOKEY
错误：依赖检测失败：
	pdksh 与 (已安裝) ksh-20120801-142.el7.x86_64 冲突
[root@localhost ~]# rpm -e ksh-20120801-142.el7.x86_64
[root@localhost ~]# rpm -ivh pdksh-5.2.14-37.el5_8.1.x86_64.rpm
警告：pdksh-5.2.14-37.el5_8.1.x86_64.rpm: 头V3 DSA/SHA1 Signature, 密钥 ID e8562897: NOKEY
准备中...                          ################################# [100%]
正在升级/安装...
   1:pdksh-5.2.14-37.el5_8.1          ################################# [100%]
```
**重新检测，发现警告消失了！！！**

### Swap分区设置
**若检查中无此项，可忽略！** 
这个问题之前有遇到过，写这篇文章又没有了。下面解决办法供参考。

 如果Swap空间不符合要求，oracle 安装文件检查发现swap 空间不足。
 大小一般设置为一般为内存的1.5倍。

（root权限）查询当时Swap分区设置情况。

```bash
swapon -s
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201220640383.png)
或者使用`free`工具来查看内存和Swap情况。

```bash
free -m
```
结果如下单位（MB）：

```bash
[root@localhost ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1475         439         171          13         865         877
Swap:          2047           0        2047
```
**创建Swap文件**
~~接下来我们将在文件系统上创建swap文件。我们要在`根目录/`下创建一个名叫`swapfile`的文件，当然你也可以选择你喜欢的文件名。该文件分配的空间将等于我们需要的swap空间。~~
一般 内存的 1.5倍以上就好了。也可以根据安装程序的提示来。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202155013609.png)

root执行以下命令，创建swap分区，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202000241726.png)

```bash
#创建swap文件 bs=2300的设置的值一般为内存的1.5倍以上 
dd if=/dev/zero of=/var/swap bs=2500 count=1000000
#需要更改swap文件的权限，确保只有root才可读
chmod 600 /var/swap
#告知系统将该文件用于swap
mkswap /var/swap
#开始使用该swap
swapon /var/swap
#使Swap文件永久生效,/etc/fstab加入配置
echo "/var/swap   swap    swap    sw  0   0" >> /etc/fstab
```

> 如果上面创建后发现，大小创建错误了。如何重置呢？
> 
> `swapoff -a`
>  `rm /var/swap`
> 上面命令就可以删除了，然后重新创建合适的swap文件就行了。

### 所有警告消失了
经过我们的不断努力，所有警告都消失了。有一些警告虽然没影响什么，有了总让人不舒服。没有⚠️真是太舒服了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202155357486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 概要
这里显示了安装配置的概要部分，检查一下是否正确。没问题就开始安装吧！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202155757483.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 安装产品
上面折腾了这么久终于迎来了真正的安装操作了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202155906324.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)

## 进度68% ins_ctx.mk错误弹框

有可能是glibc的版本过高所致（高于2.14），解决办法：

下载地址http://mirror.centos.org/centos/7/os/x86_64/Packages/glibc-static-2.17-317.el7.x86_64.rpm

```bash
rpm -ivh glibc-static-2.17-317.el7.x86_64.rpm #安装
```

##### 若出现：安装rpm包时提示错误：依赖检测失败

```bash
#解决办法，最后面加--nodeps --force，忽略依赖安装
rpm -ivh glibc-static-2.17-317.el7.x86_64.rpm --nodeps --force
```

回归正题，继续修改

```bash
#修改文件/home/oracle/app/oracle/product/11.2.0/dbhome_1/ctx/lib/ins_ctx.mk
    ctxhx: $(CTXHXOBJ)
        $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK)
#改为
    ctxhx: $(CTXHXOBJ)
        -static $(LINK_CTXHX) $(CTXHXOBJ) $(INSO_LINK) /usr/lib64/stdc.a

#修改/home/oracle/app/oracle/product/11.2.0/dbhome_1/sysman/lib/ins_emagent.mk
	$(MK_EMAGENT_NMECTL)
#改为：
	$(MK_EMAGENT_NMECTL) -lnnz11
```

修改完成后，点击重试

## 进度70% ins_emagent.mk错误弹框

它来了，它来了，我等待好久了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202160136258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
编辑：
/home/oracle/app/oracle/product/11.2.0/dbhome_1/sysman/lib/ins_emagent.mk
约176行，可以搜索`$(MK_EMAGENT_NMECTL)` 关键字快速找到。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202161046615.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
修改后如下：

```bash
#===========================
#  emdctl
#===========================

$(SYSMANBIN)emdctl:
	$(MK_EMAGENT_NMECTL) -lnnz11

#===========================
#  nmocat
#===========================
```
修改完成后，点击`重试（R）`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202161549679.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 复制数据库文件
上面的问题解决后，安装一会儿就会出现如下的界面。这个时间可能有些长，耐心等待即可。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202161826301.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)

## 数据库创建完成
经过一段时间的等待，终于弹出如下界面。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202162553269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
## 执行配置脚本
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202162714164.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
根据上图提示,root 执行上面两个脚本就可以了。

执行完成这两个脚本，点击`确定`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202163044864.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)

## Oracle Database 的安装已成功
经过我们的努力，终于走到了这一步。
Oracle Database 的安装已成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202163511782.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x4eW91Y2Fu,size_16,color_FFFFFF,t_70)
点击`关闭`即可。

# 防火墙放行1521
默认端口是1521（最前面脚本里也执行过了）

```bash
# 防火墙放行1521oracle端口
firewall-cmd --add-port=1521/tcp
firewall-cmd --add-port=1521/tcp --permanent
```

# 修改oracle默认端口（可选）

```bash
vim $ORACLE_HOME/network/admin/listener.ora #修改文件中PORT = 端口号
vim $ORACLE_HOME/network/admin/tnsnames.ora #修改文件中PORT = 端口号
#/home/oracle/app/oracle/product/11.2.0/dbhome_1/network/admin
#重启数据库服务 重启监听

#重启后可能会出现 "ORA-12514 TNS 监听程序当前无法识别连接描述符中请求服务"，要在listener.ora加以下信息
#添加后可能会出现 "ORA-01034和ORA-27101"，注意修改以下两点
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = ORCL)
      (ORACLE_HOME = /home/oracle/app/oracle/product/11.2.0/dbhome_1/)#此处需要与环境变量一致
      (SID_NAME = orcl)#此处区分大小写
    )
  )
#修改完成均需要重启监听
```

# 配置环境变量

```bash
su oracle
```
切换到oracle用户操作。
编辑配置文件

```bash
vi ~/.bash_profile
```
文件末尾加入以下内容，ORACLE_HOME中换成你实际安装的路径

```bash
export ORACLE_HOME=/home/oracle/app/oracle/product/11.2.0/dbhome_1/
export ORACLE_SID=orcl
export PATH=$PATH:$ORACLE_HOME/bin
```
使用配置文件立即生效。

```bash
source ~/.bash_profile
```
# 日常运维
```bash
su oracle
sqlplus /nolog #用普通用户登录
SQL> connect /as sysdba	#连接
SQL> startup	#开启
SQL> shutdown	#关闭

lsnrctl start	#启动监听器
lsnrctl status	#监听器状态
lsnrctl stop	#停止监听器
```
## sys用户登录

```bash
[oracle@localhost ~]$ sqlplus /nolog
SQL*Plus: Release 11.2.0.4.0 Production on Tue Feb 2 02:59:38 2021
Copyright (c) 1982, 2013, Oracle.  All rights reserved.

SQL> connect /as sysdba
Enter user-name: sys
Enter password: 
Connected.
SQL> select 1 from dual;

	 1
----------
	 1

SQL> 

#sql命令输入错误可采用Ctrl+Backspace进行删除
```
没有问题，说明oracle本地连接oracle成功。
## 启动监听

```bash
lsnrctl start
```
# PLSQL连接测试
我在PLSQL连接时也耽误了很多时间，这里也整理一下流程和一些问题的处理。

### 1.下载安装PLSQL

https://www.allroundautomations.com/files/plsqldev1306x64.msi

### 2.下载安装instantclient

去Oracle官网[Instant Client for Microsoft Windows (x64) 64-bit (oracle.com)](https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html)下载对应数据库版本的instantclient包，我下载的是[instantclient-basic-windows.x64-11.2.0.4.0.zip](https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html#license-lightbox)

下载完成后解压到PLSQL安装目录（自行修改，其实哪都可以）

在D:\Programes\PLSQL Developer\instantclient_11_2\(按你实际文件夹来)下创建tnsnames.ora文件，把HOST一项改成[虚拟机](https://so.csdn.net/so/search?q=虚拟机&spm=1001.2101.3001.7020)中Linux系统的IP地址。

```bash
#内容如下，修改192.168.189.128为你的Linux地址即可
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.189.128)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

### 3.配置环境变量

> TNS_ADMIN = D:\Programes\PLSQL Developer\instantclient_11_2（指tnsnames.ora文件所在的目录名）
> NLS_LANG = SIMPLIFIED CHINESE_CHINA.ZHS16GBK
> ORACLE_HOME = D:\Programes\PLSQL Developer\instantclient_11_2
> Path后面加一个 D:\Programes\PLSQL Developer\instantclient_11_2

### 4.PLSQL设置、激活

第一次使用PLSQL登录时直接点Cancel，以无登录的状态打开PLSQL，点击上面图标preferences首选项设置

在connection项里面把Oracle Home设置成D:\Programes\PLSQL Developer\instantclient_11_2

OCI Library设置成D:\Programes\PLSQL Developer\instantclient_11_2\oci.dll

> PLSQL13 注册码：
> 打开plsql 找到帮助——register——填写以下信息：
> 产品号/product code： kfsvzt6zh2exaxzxgjk44rv5kp2yp68vgk 
> 序列号/serial Number：186220 
> 密码/password：xs374ca


重新启动PLSQ

### 5.Linux配置

修改`/home/oracle/app/oracle/product/11.2.0/dbhome_1/network/admin`目录下、

`listener.ora`和	`tnsnames.ora`文件中HOST = 0.0.0.0（即接受所有访问）

在服务器监听开启、数据库连接、数据库开启的状态下

### 6.启动PLSQL

启动PLSQL，输入`sys`、`密码`、`192.168.189.128:1521/orcl(地址按你的写)`、`SYSDBA`

登录测试

# 总结
CentOS7安装Oracle 11g不难，遇到问题都能百度解决。就是对比windows下安装有些麻烦。安装中遇到的小问题大多因为oracle 11g年岁己高导致的。我猜测在新版的系统中安装新版的Oracle 可能会更简单。甚至可能像windows中那样简单吧！或者使用 Oracle 自己的linux系统安装起来会不会更容易呢？等我以后有空了，可以测试一下。




# 附：

##### 我的操作系统内核参数设置	/etc/sysctl.conf

```
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.rp_filter = 1
fs.file-max = 6815744 #设置最大打开文件数
fs.aio-max-nr = 1048576
kernel.shmall = 2097152 #共享内存的总量，8G内存设置：2097152*4k/1024/1024
kernel.shmmax = 2147483648 #最大共享内存的段大小
kernel.shmmni = 4096 #整个系统共享内存端的最大数
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500 #可使用的IPv4端口范围
net.core.rmem_default = 262144
net.core.rmem_max= 4194304
net.core.wmem_default= 262144
net.core.wmem_max= 1048576
```

输入：sysctl -p 命令使其生效

若kernel.sem = 250 32000 100 128，先决条件检查时还出现`操作系统内核参数：semmni 失败`则直接忽略即可

##### CentOS7下载地址：http://isoredirect.centos.org/centos/7/isos/x86_64/



# 本次安装出现的其它问题

记录一下ヾ(｡｀Д´｡)ﾉ彡

### ORA-27154: post/wait create failed

ORA-27300: OS system dependent operation:semget failed with status: 28
ORA-27301: OS failure message: No space left on device
ORA-27302: failure occurred at: sskgpcreates

本次设置参数 kernel.sem = 250 32000 100 128 就好了

### ORA-00845: MEMORY_TARGET not supported on this system 数据库启动时报错解决

在oracle 11g中新增的内存自动管理的参数MEMORY_TARGET,它能自动调整SGA和PGA，这个特性需要用到/dev/shm共享文件系统，而且要求/dev/shm必须大于MEMORY_TARGET，如果/dev/shm比MEMORY_TARGET小就会报错

**解决方案**

1.初始化参数MEMORY_TARGET或MEMORY_MAX_TARGET不能大于共享内存(/dev/shm),为了解决这个问题，可以增大/dev/shm

2.如果/dev/shm没有挂载也会报上面的错，所认需要确保已经挂载

```bash
mount -o remount,size=16G /dev/shm
vim /etc/fstab
#添加这行
/dev/shm        tmpfs   tmpfs  defaults,size=16g      0 0
```

### ORA-00210、ORA-00202、ORA-27086的问题

日志提示oracle控制文件已经被占用了，数据库却没有启动成功。具体的要想办法关闭oracle所有进程，重启数据库服务器是最佳的选择。

重启解决一切٩(๑❛ᴗ❛๑)۶！！！

注意：重启数据库服务器要自己手动启动oracle数据库才能解决问题，如果oracle数据库设置开机自启动请先关闭自启，然后再重启数据库服务器！！！

### ORA-12541: TNS: 无监听程序、ORA-12514: TNS: 监听程序当前无法识别连接描述符中请求服务

大概率Oracle 监听配置文件(listener.ora)和TNS配置文件(tnsnames.ora) IP配置问题，也可能环境变量Path、TNS_ADMIN 错误

### ORA-12546: TNS:permission denied

未使用oracle用户使用sqlplus

### bash: sqlplus: 未找到命令

错误原因：使用`su oracle`切换用户，环境变量出问题，导致无法找到命令；

su oracle 和 su - oracle 的区别
su - oracle：相当于重新登陆,此时用户的家目录和PATH等信息会发生改变
su oracle：切换到oracle身份后用户的家目录和PATH仍然是原先用户的家目录和PATH

加了"-"，是以login shell登陆的，所以会设置环境变量。如果不加，使用的还是切换前用户的环境变量，所以会出错。

