# Git

### git的常用命令

<https://git-scm.com/download/win> 进行下载git

进入一个文件夹，右击现在git bash here

```bash
# 第一次安装git需要需要设置用户签名 这个什么都可以 不会去验证邮箱
git config --global user.name 用户名 # 设置用户签名
git config --global user.email 邮箱 # 设置用户签名
git init # 初始化本地库
git status # 查看本地库状态
git add 文件名 #添加到暂存区 在这里还可以删除
git commit -m "日志信息（版本号）" 文件名 # 提交到本地库
git reflog #查看历史记录
git log # 查看历史记录 更加详细
git reset --hard 版本号（版本号前7个数就可以） #版本穿梭

```



**git的分支操作**

```bash
git branch 分支名   # 创建分支
git branch -v # 查看分支
git checkout 分支名  # 切换分支
git merge 分支名  #将指定的分支合并到当前分支上
```



**远程库操作(前提在github中创建好了库)**

```bash
git remote -v # 查看当前所有的远程地址的别名
git remote add 别名 远程地址 # 给一个远程地址起一个别名
git push 别名 分支 # 推送本地分支上的内容到远程仓库
git clone 远程地址 #将远程库的内容克隆到本地
# 注意 克隆是不需要登陆的，会自动拉取代码 初始化本地库 创建别名
git pull 远程地址别名 远程分支名 #将远程仓库这个分支的最新的内容拉下来于本地的分支直接进行合并

```



### GitHub_跨团队协作

	1. 将远程仓库的地址复制发给邀请跨团队协作的人，比如东方不败。
 	2. 在东方不败的 GitHub 账号里的地址栏复制收到的链接，然后点击 网页右上方的Fork按钮，将项目叉到自己的本地仓库。
 	3. 东方不败就可以在线编辑叉取过来的文件
 	4. 接下来点击上方的 Pull 请求，并创建一个新的请求。



### ssh免密登陆

到window的用户主目录，输入ssh -keyyen -t rsa -C 邮箱

然后，将生成的公钥添加至Github账号SSH设置



### idea集合git

**配置 Git 忽略文件**

​		创建忽略规则文件 xxxx.ignore（前缀名随便起，建议是 git.ignore），这个文件的存放位置原则上在哪里都可以，为了便于让~/.gitconfig 文件引用，**建议**也放在用户家目录下。

```bash
# Compiled class file
# idea的进行排除一些文件进行推送
*.class

# Log file
*.log

# BlueJ files
*.ctxt

# Mobile Tools for Java (J2ME)
.mtj.tmp/# Package Files #
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar

hs_err_pid*

.classpath
.project
.settings
target
.idea
*.iml
```

![image-20220427175218520](D:\学习笔记\图片\image-20220427175218520.png)



**在.gitconfig 文件中引用忽略配置文件（此文件在 Windows 的家目录中）**

```bash
[user]
	name = zyq
	email = 294056189@qq.com
[core]
	excludesfile = C:/Users/lenovo/git.ignore

```

![image-20220427175335040](D:\学习笔记\image-20220427175335040.png)



**在idea中找到git的设置定位到git.exe即可**



### DEA集成GitHub_分享项目到GitHub

**前提条件**

idea连接上github

![image-20220427183236421](D:\学习笔记\图片\image-20220427183236421.png)

![image-20220427183304643](D:\学习笔记\图片\image-20220427183304643.png)





### GitLab_安装&初始化服务&启动服务

#### 编写安装脚本(linux)

安装 gitlab 步骤比较繁琐，因此我们可以参考[官网编写 gitlab 的安装脚本](https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh)。

```bash
[root@gitlab-server module]# vim gitlab-install.sh
sudo rpm -ivh /opt/module/gitlab-ce-13.10.2-ce.0.el7.x86_64.rpm

sudo yum install -y curl policycoreutils-python openssh-server cronie

sudo lokkit -s http -s ssh

sudo yum install -y postfix

sudo service postfix start

sudo chkconfig postfix on

curl https://packages.gitlab.com/install/repositories/gitlab/gitlabce/script.rpm.sh | sudo bash

sudo EXTERNAL_URL="http://gitlab.example.com" yum -y install gitlabce

```

给脚本增加执行权限

```bash
chmod +x gitlab-install.sh
```

然后执行该脚本，开始安装 gitlab-ce。注意一定要保证服务器可以上网。

```bash
./gitlab-install.sh
```



#### 初始化GitLab服务



```bash
gitlab-ctl reconfigure
```



#### 启动GitLab服务

```bash
gitlab-ctl start
```



### GitLab_登录GitLab并创建远程库

**使用主机名或者 IP 地址即可访问 GitLab 服务。可配一下 windows 的 hosts 文件（C:\Windows\System32\drivers\etc）**
