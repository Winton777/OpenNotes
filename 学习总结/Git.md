# GIT

## git命令

```bash
git config --global user.name 用户名 #设置用户签名 
git config --global user.email 邮箱 #设置用户签名 
git init #初始化本地库 
git status #查看本地库状态 
git add 文件名 #添加到暂存区 
git restore --staged 文件名 #把文件从暂存区移动到工作区，且不会撤销修改的内容；
git restore 文件名 #文件仍在暂存区且会撤销文件修改的内容；
git commit -m "日志信息" 文件名 #提交到本地库 
git reflog #查看版本历史记录 
git log #(--oneline 简要展示) #查看版本详细信息
git reset --hard 版本号 #版本穿梭
git revert 版本号 #版本还原,并将还原版本提交最最新版本
git tag 标签 版本号 #给某版本号记录标签,其他操作可以用标签替代版本号
git tag #列出当前标签
git commit --amend #修改commit信息
```

## 分支的操作

 ```bash
 git branch 分支名 #创建分支 
 git branch -d 分支名 #删除分支
 git branch -v #查看分支 
 git checkout -b 分支名 #创建并切换到这个分支
 git checkout 分支名 #切换分支 
 git merge 分支名 #把指定的分支合并到当前分支上
 ```

## 解决冲突

```bash
1.人工修改冲突文件，保存最终版本
<<<<<<< HEAD 
当前分支的代码 
======= 
合并过来的代码 
>>>>>>> branch2
2.添加到暂存区（git add 文件名）
3.提交（git commit -m "日志信息" ~~文件名~~）不能加文件名
```

##  远程仓库操作

```bash
git remote -v #查看当前所有远程地址别名 
git remote add 别名 远程地址 #起别名 (也有remane/remove可以操作)
git push 别名 分支 #推送本地分支上的内容到远程仓库 
git clone 远程地址 #将远程仓库的内容克隆到本地 
#clone 会做如下操作。1、拉取代码。2、初始化本地仓库。3、创建别名
git pull 远程库地址别名 远程分支名 #将远程仓库对于分支最新内容拉下来后与 当前本地分支直接合并

git fetch 远程库地址别名 远程分支名 #获取远程仓库对于分支最新内容
git fetch 远程库地址别名 #将远程仓库对于分支最新内容
git diff #比较本地代码与刚刚从远程下载下来的代码的区别
#与git pull相比git fetch相当于是从远程获取最新版本到本地，但不会自动merge。如果需要有选择的合并git fetch是更好的选择。效果相同时git pull将更为快捷;
```

## ssh免密登录

```bash
#1.检查用户名和邮箱
git config --global --list #查看用户名和邮箱配置是否是GitHub账号上的
#2.生成SSH密钥
ssh-keygen -t rsa -C GitHub邮箱
#或
ssh-keygen -m PEM -t rsa -b 4096 -C GitHub邮箱
#第二种可以在springcloud config分布式配置中心同步使用
#
#3.仓库中配置SSH地址
git remote set-url origin git@github.com:USERNAME/REPOSITORY.git
#并在github账户设置ssh公钥
ssh -T git@github.com #测试
```

## IDEA配置 Git 和 ignore文件

```bash
1.新建git.ignore忽略文件
# Compiled class file
*.class
# Log file
*.log
# BlueJ files
*.ctxt
# Mobile Tools for Java (J2ME)
.mtj.tmp/
# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar
# virtual machine crash logs, see 
http://www.java.com/en/download/help/error_hotspot.xml
hs_err_pid*
.classpath
.project
.settings
target
.idea
*.iml

2.在.gitconfig 文件中引用忽略配置文件
[core]
	excludesfile = C:/Users/Winton/git.ignore

3.设置-Version Control-Git 设置安装目录/bin/git.exe 点test
```

# 连接不上Github

先要查看下 `github.com` 和 `github.global.ssl.fastly.net` 对应的 ip 地址，通过这个网站进行查询：[IP地址查询](https://www.ipaddress.com/)

然后修改hosts文件，配置这两个域名对应的ip，此时可以 `ping github.com` 和 `ping github.global.ssl.fastly.net` 试一下是否通畅，可能速度不是很快，但实测还是能通的

```bash
# 例如
140.82.112.3 gitHub.com
151.101.1.194 github.global.ssl.fastly.net
```

# 问题整理

1. fatal: unable to access 'https://github.com/Winton777/gitDemo.git/': SSL certificate problem: unable to get local issuer certificate

```bash
	这是由于当你通过HTTPS访问Git远程仓库的时候，如果服务器上的SSL证书未经过第三方机构认证，git就会报错。原因是因为未知的没有签署过的证书意味着可能存在很大的风险。解决办法就是通过下面的命令将git中的sslverify关掉
git config --global http.sslverify false
　　上面这行命令的影响范围是系统当前用户，如果要设置为全局所有用户，可以改成这样：
git config --system http.sslverify false
```

2. 执行 git status或其他命令，不显示中文

```bash
在命令端输入命令设置
   git config --global core.quotepath false
同时需要执行以下命令：
   git config --global i18n.commitencoding utf-8 # 设置提交命令的时候使用utf-8编码集提交
   git config --global i18n.logoutputencoding utf-8 # 日志输出时使用utf-8编码集显示
   export LESSCHARSET=utf-8 # 设置LESS字符集为utf-8
```

3. fatal: helper error (-1): User cancelled dialog.
   remote: Support for password authentication was removed on August 13, 2021.

```
原因：官方不再支持通过账号密码验证了
解决方案：
1.网页点击头像->Settings->DeveloperSettings->Personal access tokens (classic) 创建一个Token
2.git bash终端输入 git config --list 可以查看 credential.helper 的状态
输入 git config --global credential.helper store
再次git clone输入密码时，输入创建的Token即可
```

