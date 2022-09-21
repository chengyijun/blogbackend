---
title: git培训讲义
date: 2022-07-08 09:46:53
tags: git
---
### GIT培训讲义

> 作者：程义军
>
> 时间：2022-07-06

#### 1 为什么是GIT而不是SVN

- SVN由于是集中式的架构，一旦中央仓库服务器宕机，版本控制工作就彻底失效；GIT是分布式的，每个工作节点都有相对完整的仓库备份，远程仓库离线也不影响正常的版本控制工作。
- SVN绝大部分操作都要请求网络，版本回退或者切换分支比较慢；GIT大部分操作是基于本地的，版本回退或者切换分支非常快（只是移动了Head指针）。
- 便于和`Jenkins`之类的`CI/DI`测试工作流软件进行集成，进行高效的自动化测试与部署。
- GIT对于文件的存储进行了优化，大幅降低了存储空间占用。

#### 2 git的工作流图

![git工作流程图](git培训讲义/git工作流程图.png)

#### 3 git的基本操作

1. clone

   ```bash
   # 克隆已经存在的版本库
   git clone http://192.168.10.141:9980/cyj/tp03.git
   ```

2. checkout

   ```bash
   # 作用1 创建分支
   git checkout -b branchName [commitID / tagName]
   # 作用2 切换分支
   git checkout branchName
   # 作用3 丢弃工作区改动
   git checkout .
   git checkout -- fileName
   ```

3. add

   ```bash
   # 向暂存区添加改动
   git add .
   git add fileName
   ```

4. commit

   ```bash
   # 向暂存区提交已经添加进来的改动
   git commit -m "变更说明"
   ```

5. pull

   ```bash
   # 从远端分支获取改动 并与当前分支合并
   git pull origin main:main
   ```

6. fetch （fetch+merge=pull）

   ```bash
   # 从远端分支获取改动 不主动与当前分支合并
   git fetch origin main:tmp
   # 常见的后续操作
   git diff tmp
   git merge tmp
   ```

7. merge

   ```bash
   # 见上一条
   ```

8. push

   ```bash
   # 向远端提交本地分支的改动
   git push origin main:main
   
   # **重要** 只要是改动远端分支都离不开push
   # 将指定的标签上传到远程仓库
   git push origin tagName 
   # 将所有不在远程仓库中的标签上传到远程仓库
   git push origin --tags 
   # 删除远端标签
   git push origin :V1.0.2
   # 删除远程分支
   git push origin :branchName 
   ```

9. status

   ```bash
   # 查看当前分支的状态
   git status
   ```

10. log

    ```bash
    # 查看提交历史
    git log --oneline --graph
    
    # 查看reset之前的历史
    git reflog
    ```

11. diff

    ```bash
    # 作用见第六条
    ```

12. init

    ```bash
    # 将某个文件夹变为一个仓库 （本地先创建版本库 再往远端推送会用到）
    git init .
    ```

    

#### 4 git进阶操作

1. 丢弃工作区的改动

   ~~~bash
   # 丢弃所有改动
   git checkout .
   # 丢弃单个文件的改动
   git checkout -- fileName
   ~~~

2. 丢弃暂存区的记录

   ```bash
   # 丢弃所有记录
   git restore --staged .
   # 丢弃单个记录
   git restore --staged fileName
   ```

3. 从本地仓库撤销提交 （commit 逆向操作）

   ```bash
   git revert commitID
   
   # 那么问题来了 他与上面讲的版本回退有什么区别呢
   git reset --hard commitID
   # 见下图 revert反向commit会有一个新的commit生成 可以回溯你撤销的历史 而reset则不会生成新的commit
   ```

   ![image-20220718152553953](git培训讲义/image-20220718152553953.png)

4. 时光机（版本回退）

   ```bash
   git reset --hard commitID # 回退版本 并将改动彻底丢弃 （推荐使用）
   
   git reset --soft commitID # 回退版本 并将改动撤回到暂存中 (回归的状态：已add未commit)
   git reset --mixed commitID # 回退版本 并将改动撤回到工作区中 mixed为缺省参数 （回归的状态：未add未commit）
   
   
   # 一些特殊的简写 (相对于当前Head指针回退一个版本)
   git reset --hard Head^
   git reset --hard Head~1
   ```

5. 合并别人的代码（解决冲突）

   > 所谓**冲突**是指两个不同commit对同一个文件的同一行进行了不同操作

   ```bash
   # 合并之前查看分支差异
   git diff branchName
   # 根据差异解决冲突 然后再合并
   git merge branchName
   ```

   本地和远端同一个文件的同一行存在差异 则会产生冲突，以下是冲突的直观展示：

   `git fetch`

   `git diff origin/main`

   ![image-20220706103112187](git培训讲义/image-20220706103112187.png)

   或者IDE里面可以观察到：

   ![image-20220706103157989](git培训讲义/image-20220706103157989.png)

6. Tag打标发布版本（版本冻结 打包归档）

   > 所谓**Tag**其实就是一种特殊的Branch，主要作用为冻结版本，进行正式版本发布

   ![image-20220718143001430](git培训讲义/image-20220718143001430.png)
   
   ```bash
   # 创建tag
   git tag -a V1.0.1 -m "message"
   
   # 查看所有标签
   git tag -l
   
   # 查看标签详细信息
   git show tagName
   
   # 将指定的标签上传到远程仓库
   git push origin tagName 
   # 将所有不在远程仓库中的标签上传到远程仓库
   git push origin --tags 
   
   # 删除本地标签
   git tag -d V1.0.2
   # 删除远端标签
   git push origin :V1.0.2
   
   # 根据标签打包
   git archive V1.0.1 --format=zip --output=V1.0.1.zip
   
   # 支持的打包格式
   git archive --list
   tar
   tgz
   tar.gz
   zip
   
   # 根据标签创建对应的发版分支
   git checkout -b newBranchName tagName
   ```

   6. 分支操作
   
   ```bash
   # 创建分支
   git branch branchName
   git checkout -b branchName
   git push origin branchName:branchName
   
   # 查看分支
   git branch # 查看本地分支
   git branch -a # 查看本地分支+远程分支
   
   # 删除分支
   git branch -d branchName # 删除本地分支
   git branch -D branchName # 删除本地分支（强制）
   git push origin :branchName # 删除远程分支
   git push origin --delete branchName # 删除远程分支
   
   # 通过fetch获取更新创建分支
   git fetch origin main:tmp
   # 通常的后续操作
   git diff tmp
   git merge tmp
   git branch -d tmp
   ```

#### 5 git的高阶操作

1. 当你正在开发的功能开发了一半，此时来了紧急任务修复重大BUG，你该怎么做？（git stash）

   > 由于功能开发了一半不好提交，但是由于有未提交的文件改动存在，无论是拉取远端，还是切换分支等操作均是做不到的。

   ```bash
   # 临时存储改动
   git stash save "这里是开发了一半的影像中心功能"
   
   # 正常 拉取远端代码  修改bug
   git pull
   # 修改bug
   # bug修改完毕了
   
   # 恢复临时改动
   git stash pop stash@{0}
   # 继续开发影像中心功能
   ...
   
   # 删除临时改动
   git stash drop stash@{0}
   
   # 查看所有的临时存储
   git stash list
   ```

2. 当你的个人的个人开发分支A与同事的开发分支B分离比较久了，同事已经修复了一个重大BUG，你也想在你的feature发布前获取这个修复，你该怎么做？

   ```bash
   # 精确获取其他分支上的指定commit到本分支 并形成一个新的提交
   git cherry-pick commitID
   ```

   ![image-20220719153916438](git培训讲义/image-20220719153916438.png)

   > 那么问题来了，git merge 与 git cherry-pick 有什么区别呢？
   >
   > 可以简单理解为：get merge 是我要别人所有的提交（all in了）；git cherry-pick是我需要别人某一个特定的提交

3. 我把别人远端代码推挂了 我该怎么办？

   ```bash
   # 远端代码回退
   
   # 首先回退本地对应分支代码到指定commit
   git reset --hard commitID
   
   # 强推使远端分支与本地分支对齐
   # 此处必须强推 因为当前分支由于回退过 所以是落后对应的远端分支n个提交的 具体落后几个可以使用git status 查看到
   git push origin dev:dev -f 
   ```

4. 放弃追踪一些文件或者目录

      ```bash
      # 有的时间想要查看一下git跟踪了哪些文件，此命令就可以列出所有git正在跟踪的文件
      git ls-files
      
      # git不再跟踪名为FileName的文件，但是文件保留在工作区和暂存区中，也就是你的目录中
      git rm --cached FileName
      
      # 删除名为FileName的文件，同时git不再跟踪它
      git rm FileName
      ```

5. 强制拉取远端代码

      > 注意这种操作会覆盖掉本地代码

      ```bash
      # 获取远端的更新
      git fetch --all
      # 撤销本地、暂存区、版本库(用远程服务器的origin/master替换本地、暂存区、版本库) 强制进度指针对齐
      git reset --hard origin/master
      # 从远程仓库"同步"代码
      git pull origin master
      
      # 或单条执行：
      git fetch --all && git reset --hard origin/master && git pull
      ```

      

#### 6 常见的难以理解的点

1. fast-forward的作用

   ![image-20220706102201798](git培训讲义/image-20220706102201798.png)

2. rebase变基

  > 注意变基与合并的区别

  首先通过简单的提交节点图解感受一下rebase在干什么
  两个分支master和feature，其中feature是在提交点B处从master上拉出的分支
  master上有一个新提交M，feature上有两个新提交C和D

![变基1](git培训讲义/变基1.png)

```bash
git switch feature
git rebase master

# 以上两句等价于 (可以简单理解为：将master里的commit 放到 feature上)
git rebase master feature

# 如果提示有冲突就解决冲突
...

# 解决冲突后继续变基
git rebase --continue
```

![变基2](git培训讲义/变基2.png)

原理说明：

![merge](git培训讲义/merge.jpg)

![rebase](git培训讲义/rebase.jpg)

#### 7 一些规范

1. git默认的主分支：master分支名已经弃用 现在都叫main分支

   由于黑人开发者的增多，鉴于黑人被种族歧视的历史（Master-Slave），黑人对git中的默认master分支提出抗议，2020年6月8日 Scott 发文呼吁将你的Git默认分支从`master`修改为`main`，目前国外各大主流软件商均已更改，Github也已弃用master改用main分支名，以示尊重。

2. 大公司一般通用的简单分支模型

   ![image-20220706110531715](git培训讲义/image-20220706110531715.png)

 > main分支：只用来打标TAG 用户版本代码冻结 进行发版
 > dev分支：用来正常开发 迭代commit
 > feature分支：用来开发新功能，新功能开发完毕之后向dev合并
 > bug分支：类似feature分支 进行bug修复 修复后向dev合并

3. `.gitignore`文件必须写

   > `.gitignore`文件必须写，指定无需提交的文件及文件夹
   >
   > 如：ide的配置 编译产生的中间产物 第三方依赖库 都需要忽略

#### 8 git与gitlab的配套使用

1. 先在gitlab创建空白仓库再克隆

![image-20220707114032800](git培训讲义/image-20220707114032800.png)

![image-20220707114101464](git培训讲义/image-20220707114101464.png)

> *注意：*将仓库地址进行一个替换 如下：
>
> <http://6c4c5ae0240e/cyj/tp03.git>
> <http://192.168.10.141:9980/cyj/tp03.git>

`git clone http://192.168.10.141:9980/cyj/tp03.git`

2. 针对本地 已经存在的项目 推送到gitlab
   1. 进入项目目录 执行`git init .` 初始化仓库
   2. 在gitlab上创建仓库

![image-20220707114558420](git培训讲义/image-20220707114558420.png)

如果勾选了，会报出如下错误：

![image-20220707114707492](git培训讲义/image-20220707114707492.png)

```bash
cd existing_repo
git remote add origin http://192.168.10.141:9980/cyj/tp03.git # 指定远程仓库的位置
git branch -M main # 将当前分支重命名为main （gitlab默认就会创建main作为主分支 作为对应远程分支）
git push -uf origin main # -u表示--set-upstream和远程仓库建立映射关系  其实等价于 git push origin main:main
```

3. 关于项目成员推送提交

   > 默认是需要项目仓库所有者自己审核PR

   如果想取消可以进行如下操作：

   ![image-20220707115747409](git培训讲义/image-20220707115747409.png)

4. 如果向远端推送，提示换行符警告，可以执行自动转换命令

   ![image-20220708100349895](git培训讲义/image-20220708100349895.png)

```bash

# 补充知识
win： CRLF => "\r\n"
Linux: LF => "\n"

# 补充知识
Git命令行修改AutoCRLF
提交时转换为LF，检出时转换为CRLF（推荐windows）
git config --global core.autocrlf true
提交时转换为LF，检出时不转换（推荐*unix/mac）
git config --global core.autocrlf input
提交检出均不转换
git config --global core.autocrlf false

Git命令行修改SafeCRLF
拒绝提交包含混合换行符的文件
git config --global core.safecrlf true
允许提交包含混合换行符的文件
git config --global core.safecrlf false
提交包含混合换行符的文件时给出警告
git config --global core.safecrlf warn

# 推荐配置
提交检出均不转换
git config --global core.autocrlf false
允许提交包含混合换行符的文件
git config --global core.safecrlf false
```



5. 频繁要求输入账号密码很烦，怎么解决？

   - 终端执行该命令

   ```bash
   git config --global credential.helper store
   ```

   - 进行任意一种输需要身份验证的操作 如push pull等 输入一次正确的用户名和密码
   - 检查你的`用户目录`下 会出现 `.git-credentials` 文件  打开后已经写入了用户名和密码 如下图所示
   - 以后再不会要求输入用户名密码了

![image-20220803180606437](git培训讲义/image-20220803180606437.png)

![image-20220803180754460](git培训讲义/image-20220803180754460.png)

