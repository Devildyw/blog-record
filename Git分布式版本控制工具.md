# [Git分布式版本控制工具](file:///D:/BaiduNetdiskDownload/Git讲义.pdf)

[![image-20220127230403996](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220127230403996.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220127230403996.png)

**命令如下：**

1. **clone（克隆）**: 从远程仓库中克隆代码到本地仓库
2. **checkout （检出）**:从本地仓库中检出一个仓库分支然后进行修订
3. **add（添加）**: 在提交前先将代码提交到暂存区
4. **commit（提交）**: 提交到本地仓库。本地仓库中保存修改的各个历史版本
5. **fetch (抓取)** ： 从远程库，抓取到本地仓库，不进行任何的合并动作，一般操作比较少。
6. **pull (拉取)** ： 从远程库拉到本地库，自动进行合并(merge)，然后放到到工作区，相当于fetch+merge
7. **push（推送）** : 修改完成后，需要和团队成员共享代码时，将代码推送到远程仓库



------

**备注：**

**Git GUI**：Git提供的图形界面工具

**Git Bash**：Git提供的命令行工具

当安装Git后首先要做的事情是**设置用户名称**和**email**地址。这是非常重要的，因为每次Git提交都会使用该用户信息

```SH
git config --global user.name "itcast"
git config --global user.email "hello@qq.com"
```

查看配置信息

```SH
git confifig --global user.name
git confifig --global user.email
```

------

## **本地仓库**

要使用Git对我们的代码进行版本控制，首先需要获得本地仓库

1. 在电脑的任意位置创建一个空目录（例如test）作为我们的本地Git仓库
2. 进入这个目录中，点击右键打开Git bash窗口
3. 执行命令**git init** (初始化本文件为本地仓库)
4. 如果创建成功后可在文件夹下看到隐藏的.git目录。

------

## **命令**

- **git touch** 文件名 可以创建文件

------

Git工作目录下对于文件的**修改**(增加、删除、更新)会存在几个状态，这些**修改**的状态会随着我们执行Git的命令而发生变化。

[![image-20220127231451204](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220127231451204.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220127231451204.png)

- **git add** (工作区 –> 暂存区) (**git add . 将所有修改加入暂存区**)
- **git commit** (暂存区 –> 本地仓库) (**git commit -m ‘注释内容’** 将暂存区中的内容提交到本地仓库)
- **git status**:**查看修改的状态**

------

**查看提交日志**

- **git log** [option]
- options
  - **all** 显示所有分支
  - **pretty=oneline** 将提交信息显示为一行
  - **abbrev-commit** 使得输出的commitId更简短
  - **graph** 以图的形式显示

**对log的格式进行自定义 并且对该操作使用alias起了一个别名**

```SH
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

------

**版本回退**

- 作用: 版本切换

- 命令:

   

  git reset –hard commitID

  - **commitID** 可以使用 **git lg** 或 **git log** 指令查看

**如何查看已经删除的记录？**

- **git reflflog**
- 这个指令可以看到已经删除的提交记录

------

**添加文件至忽略列表**

一般我们总会有些文件无需纳入Git 的管理，也不希望它们总出现在未跟踪文件列表。 通常都是些自动生成的文件，比如日志文件，或者编译过程中创建的临时文件等。 在这种情况下，我们可以在工作目录中创建一个名为 **.gitignore 的文件（文件名称固定）**，列出要忽略的文件模式。

```TEX
HELP.md
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**/target/
!**/src/test/**/target/

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/
!**/src/main/**/build/
!**/src/test/**/build/

### VS Code ###
.vscode/
```

## **分支**

**查看本地分支**

- **git branch**

------

**创建本地分支**

- **git branch 分支名**

------

**切换分支**

- **git checkout 分支名**
- **git checkout -b 分支名 (创建并切换)**

------

**合并分支**

- **git merge 分支名称**

------

**删除分支**

- **git branch -d 分支名 (删除分支时，需要做各种检查)**
- **git branch -D 分支名 (不做任何检查，强制删除)**

------

**解决冲突**

当两个分支上对文件的修改可能会存在冲突，例如同时修改了同一个文件的同一行，这时就需要手动解决冲突，解决冲突步骤如下：

1. 处理文件中冲突的地方
2. 将解决完冲突的文件加入暂存区(add)
3. 提交到仓库(commit)

[![image-20220127233455735](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220127233455735.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220127233455735.png)

## **远程仓库**

- **注册gitee(码云)**
- **创建仓库**

**配置SSH公钥**

- **生成SSH公钥**

  - **ssh-keygen -t rsa**

- **Gitee设置账户共公钥**

  - 上面我们以及生成了SSH公钥 这里我们要获取
  - **cat ~/.ssh/id_rsa.pub**
  - 复制这个SSH公钥
  - [![image-20220128220100844](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220128220100844.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220128220100844.png)
  - 验证是否配置成功 **ssh -T [git@gitee.com](mailto:git@gitee.com)**

  #### 操作远程仓库

  - **添加远程仓库** (此操作是先初始化本地仓库,然后与一创建的远程仓库进行对接)

    - **命令: git remote add <远端名称(别名)> <仓库路径(url)>**

      ```SH
      git remote add origin git@gitee.com:Devildyw/spring-mvc.git
      ```

    - **命令: git remote**(查看已添加的远程仓库)

      [![image-20220128221118298](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220128221118298.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220128221118298.png)

------

- **推送到远程仓库**
  - **命令: git push [-f] [–set-upstream] [远端名称[本地分支名 [:远端分支名]]**
    - `git push origin master PLAINTEXT* **-f** 表示强制覆盖* **--set-upstream**: 推送到远端的同时并且建立起和远端分支的关联关系* ```sh  git push --set-upstream origin master `
    - 如果当前分支已经和远端分支建立关联,则可以省略分支名和远端名 **(git push)**
  - **查看本地分支与远程分支的关联关系**
    - **命令: git brach -vv**

------

- 从远程仓库克隆(一般只有一开始会做一次 后续都会使用pull拉取)
  - 命令: git clone <仓库路径> [本地目录]
    - 本地目录可以省略,会自动生成一个目录
- 从远程仓库中抓取和拉取
  - 远程分支和本地的分支一样，我们可以进行merge操作，只是需要先把远端仓库里的更新都下载到本地，再进行操作。
    - 抓取 命令: git fetch [remote name] [branch name]
      - **抓取指令就是将仓库里的更新都抓取到本地，不会进行合并**
      - 如果不指定远端名称和分支名，则抓取所有分支。
    - 拉取 命令: git pull [remote name] [branch name]
      - **拉取指令就是将远端仓库的修改拉到本地并自动进行合并，等同于 ** **fetch+merge**
      - 如果不指定远端名称和分支名，则抓取所有并更新当前分支。

------

- **解决合并冲突**
  - 在一段时间，A、B用户修改了同一个文件，且修改了同一行位置的代码，此时会发生合并冲突。A用户在本地修改代码后优先推送到远程仓库，此时B用户在本地修订代码，提交到本地仓库后，也需要推送到远程仓库，此时B用户晚于A用户，**故需要先拉取远程仓库的提交，经过合并后才能推送到远端分支**
  - 在B用户拉取代码时，因为A、B用户同一段时间修改了同一个文件的相同位置代码，故会发生合并冲突。**远程分支也是分支，所以合并时冲突的解决方式也和解决本地分支冲突相同相同**
  - [![image-20220128223521930](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220128223521930.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220128223521930.png)