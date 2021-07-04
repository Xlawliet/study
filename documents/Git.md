# Git

### 版本控制

- 实现跨区域多人协同开发
- 追踪和记载一个或者多个文件的历史记录
- 组织和保护你的源代码和文档
- 统计工作量
- 并行开发、提高开发效率
- 跟踪记录整个软件的开发过程
- 减轻开发人员的负担，节省时间，同时降低认为错误



### 配置

> Git配置

查看配置 git config -l

所有的配置文件其实都保存在本地！

```bash
#查看系统config
git config --system --list

#查看当前用户（global）配置
git config --global list
```

Git相关的配置文件：

1. Git\etc\gitconfig :Git安装目录下的gitconfig --system 系统级别
2. C:\Users\xiong\gitconfig   --global 用户级别

> 设置用户名与邮箱（用户标识，必要）

安装git后首先要做的事情就是设置用户名称和e-mail地质。

```bash
git config --global user.name "xzy" #名称
git config --global user.email 434927530@qq.com #邮箱 

git config --global --list #查看当前的设置
```



### Git 命令

> 基本命令

git add . 将需要进行版本管理的文件加入暂存区

git commit 提交到本地git仓库

git push 提交到远程仓库



> 创建项目

1、创建全新的仓库，需要用git管理的项目的根目录执行

```bash
# 在当前目录新建一个Git代码库
git init  初始化项目文件
```

2、执行后目录下会有一个.git目录，是隐藏文件需要打开查看隐藏文件；

> 克隆远程仓库

1、将远程服务器上的仓库完全镜像一份至本地

```bash
# 克隆一个项目和它的整个代码历史（版本信息）
git clone [url]
```



> 查看文件状态

```bash
#查看文件状态
git status

#添加所有文件到暂存区
git add .

#提交暂存区中的内容到本地仓库   -m 提交信息  "xxxxxxxx"
git commit -m "xxxxxxxx"
```



> 忽略文件

有些时候不需要将所有文件都纳入版本控制中，比如数据库文件，临时文件，设计文件等

在主目录下建立".gitignore"文件，规则如下：

1. 忽略文件中的空行或以井号（#）开始的行将会被忽略。
2. 可以使用Linux通配符。例如：星号（*）代表任意多个字符，问号（？）代表一个字符，方括号（[abc]）代表可选字符范围，大括号（{string1,string2,...}）代表可选字符串等。
3. 如果名称的最前面有一个感叹号（！），表示为例外规则，将不被忽略。
4. 如果名称的**最前面**是一个路径分隔符（/），表示要忽略的文件在此目录下，而子目录中的文件不忽略。
5. 如果名称的**最后面**是一个路径分隔符（/），表示要忽略的是此目录下该名称的子目录，而非文件（默认文件或目录都忽略）。

```bash
#为注释
*.txt         #忽略所有 .txt结尾的文件
!lib.txt      #但lib.txt 除外
/temp         #仅忽略项目根目录下的TODO文件，不包括其他目录temp
build/        #忽略build/目录下的所有文件
doc/*.txt     #会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
```



> GIT分支

```bash
#查看分支情况
git branch

#新建一个分支，但依然停留在当前分支
git branch [branch-name]

#新建一个分支，并且切换到该分支
git checkout -b [branch]

#合并指定分支到当前分支
git merge [branch]

#删除分支
git branch -d [branch-name]

#删除远程分支
git push origin --delete [branch-name]
git branch -dr [remove/branch]
```

多个分支如果并行执行，就会使得同时存在多个版本。

如果同一个文件在合并分支时都被修改了则会引起冲突：解决方法就是我们可以修改冲突文件后再提交，选择要保留他的代码还是我的代码。

