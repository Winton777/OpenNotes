
# 前言
首先感谢本文原作者，安装教程参考自 [CentOS / Linux 安装MySQL（超简单详细）](https://zhuanlan.zhihu.com/p/623778183)，实测安装成功，摘录总结下来学习使用！

# 尝试安装

首先，尝试一下直接使用 yum 安装 MySQL

```bash
yum install mysql-community-server
```

如果成功，表示不需要配置MySQL rpm 源信息，直接就安装完成了

但是，如果出现以下错误：

```text
Loading mirror speeds from cached hostfile
没有可用软件包 mysql-community-server。
错误：无须任何处理
```

表示我们没有添加安装包的源信息，需要安装 MySQL rpm 源信息

# 安装 MySQL rpm 源信息

打开 [MySQL :: Download MySQL Yum Repository](https://dev.mysql.com/downloads/repo/yum/)

根据你的系统版本，选择对应的安装包，例如我的是CentOS 7.9，这个系统的Linux内核是 Linux 7，所以我选择了这个安装包，大家依次类推。

![image-20240104134914953](https://typora-wyt.oss-cn-beijing.aliyuncs.com/202401041349027.png)

拼接下载地址头：`http://dev.mysql.com/get/` 得到以下地址

```text
http://dev.mysql.com/get/mysql80-community-release-el7-11.noarch.rpm
```

使用 wget + 刚才拼接的地址，下载安装包源信息

```bash
wget  http://dev.mysql.com/get/mysql80-community-release-el7-11.noarch.rpm
```

rpm 安装源信息

```bash
rpm -ivh mysql80-community-release-el7-11.noarch.rpm
```

# 安装

再尝试使用 yum 安装MySQL

```bash
yum install mysql-community-server
```

安装过程中，会提示让我们确认，一律输入 `y` 按回车即可

安装完成后，yum会自动覆盖自带的mariaDB，所以不需要我们手动卸载它

## 检查安装是否成功

检查一下刚才的安装是否成功

```bash
rpm -qa | grep mysql
```

输出类似以下内容，表示安装完成

```text
mysql-community-libs-compat-8.0.33-1.el7.x86_64
mysql-community-icu-data-files-8.0.33-1.el7.x86_64
mysql80-community-release-el7-7.noarch
mysql-community-common-8.0.33-1.el7.x86_64
mysql-community-libs-8.0.33-1.el7.x86_64
mysql-community-server-8.0.33-1.el7.x86_64
mysql-community-client-8.0.33-1.el7.x86_64
mysql-community-client-plugins-8.0.33-1.el7.x86_64
```

检查mariaDB是否被覆盖

```bash
rpm -qa | grep mariadb
```

输出为空，表示 mariaDB 已经被成功覆盖。

## 登录和修改密码

```bash
#启动
systemctl start mysqld
```

我们安装的时候，并没有设置初始密码

所以 mysql 在第一次启动的时候，会自动初始化一个密码

通过以下这行代码，我们可以查看 mysql 自动初始化的密码：

```bash
# 第一次启动后，可以查看mysql初始化密码
grep 'temporary password' /var/log/mysqld.log

# 输出（root@localhost: 后面的是密码）：
2024-01-03T09:54:16.762127Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: c&Rl)jyM+5ir
```

**登录**

```bash
# 登录mysql，一定要注意：-p和'密码'之间是没有空格的
mysql -u root -p'c&Rl)jyM+5ir'
```

**修改 root 密码**

注意了，默认的密码策略，需要：大写英文 + 特殊字符 + 数字

```bash
ALTER USER 'root'@'localhost' IDENTIFIED BY 'Mysql@123';
```

源文档提供了通过密码验证插件修改密码校验策略的方法，想修改的话可以参考。

## 开放 root 账户远程登录

```bash
# 登录
mysql -u root -p'密码'

# 如果你的数据库是 mysql 8 及以上
# 1、进入数据库
use mysql
# 2、修改user表
update user set host='%' where user='root';
# mysql 5.7 及之前，执行这行代码即可
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;

# 重载授权表
FLUSH PRIVILEGES;
# 退出
exit
# 重启
systemctl restart mysqld
```

## 端口开放

经过实测，使用阿里云、腾讯云、华为云，在服务器上，无需配置 `iptables` 端口信息

但必须在相应的控制台 - 安全组，开放 3306 端口（默认端口）。

**修改端口**

```bash
#编辑Mysql配置文件my.cnf(一般为/etc/my.cnf)
sudo find / -name my.cnf
#添加或修改想要配置的端口号
port = 端口号
#保存并重启
systemctl restart mysqld
```

# 重新安装

出现需要重新安装的情况时，执行以下命令

```bash
rpm -qa | grep mysql
rpm -qa | grep mysql | xargs yum remove
rpm -qa | grep mysql | xargs yum -y remove
```

后从安装Mysql源这一步重新开始即可

# MySQL 常用命令

```bash
# 启动
systemctl start mysqld
# 第一次启动后，可以查看mysql初始化密码
grep 'temporary password' /var/log/mysqld.log
# 重启
systemctl restart mysqld
# 停止
systemctl stop mysqld
#查看状态
systemctl status mysqld
#开机启动
systemctl enable mysqld
systemctl daemon-reload
# 查看进程、版本信息
ps -ef | grep mysql
或 netstat -atp
# 登录
mysql -u root -p'密码内容'
# 查看所有数据库
SHOW DATABASES;
# 新建数据库
CREATE DATABASE 数据库名;
# 进入数据库
USE 数据库名
# 查看所有表
show tables
# 查看某张表信息
desc 表名
```

# 问题总结

1. **Mysql 表名区分大小写**

**解决方法：**输入 `show Variables like '%table_names'` 查看lower_case_table_names 的值，0-区分,1-不区分

在Mysql 配置文件（ `/etc/my.cnf` ）中添加 `lower_case_table_names=1`

保存并重启 `systemctl restart mysqld`

该方法只能在低版本Mysql中使用，官方说明：在8.0版本，你的mysql已经初始化过就不支持修改lower_case_table_names参数了，只能选择重装或者重新初始化。

可以删除数据目录，即删除 `/var/lib/mysql` 目录后，完成上述修改配置文件，保存并重启。

如果要保留旧的数据目录，请 先进行备份！

2. **MyBatis报错 Mysql数据库-密码相关**

>  org.apache.ibatis.exceptions.PersistenceException: 
>
> Error updating database.  Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException: Client does not support authentication protocol requested by server; consider upgrading MySQL client

中文意思就是：客户端不支持服务器请求的身份验证协议，考虑升级mysql客户端。原因是你安装了8.0版本以上的MySQL，密码加密方式发生了变化,所以简单的方法就是升级客户端，或者是去手动修改密码规则。

**解决方法：**

```bash
# 重新修改密码,加上 WITH mysql_native_password 
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';
# 刷新权限
FLUSH PRIVILEGES;
# 详细信息可以在 USER 表中查看 plugin 字段；
```

3. **MyBatis报错  Mysql数据库-字符集相关**

>  org.apache.ibatis.exceptions.PersistenceException: 
>
> Error updating database.  Cause: java.sql.SQLException: Unknown initial character set index '255' received from server. Initial client character set can be forced via the 'characterEncoding' property.

从服务器接收到未知的初始字符集索引’255’。初始客户端字符

这条错误信息说明你连接的数据库使用了未知的字符集编码，导致无法正确解析数据库返回的数据。

**解决方法：**

将 MyBatis 配置文件中 url 添加 `?characterEncoding=utf8` 即可

如 `<property name="url" value="jdbc:mysql://原Url?characterEncoding=utf8"/>`
