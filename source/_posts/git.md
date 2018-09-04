---
title: git
date: 2018-08-29 21:12:13
categories: others
comment: true
---
git基础~

<!--more-->

## 常用

- git init 将目录变成git管理目录

- git add file  保存文件到暂存区

- git commit -m "文件修改说明"  提交文件到分支

- git status  查看仓库当前的状态

- git diff   查看当前版本和上一版本的区别

  ```python
  git diff    #是工作区(work dict)和暂存区(stage)的比较
  git diff --cached    #是暂存区(stage)和分支(master)的比较
  git diff HEAD  #查看工作区和版本库里面最新版本的区别
  git checkout -- file  #可以丢弃工作区的修改：总之，就是让这个文件回到最近一次git commit或git add时的状态。
  git reset HEAD <file> 可以把暂存区的修改撤销掉
  ```

- git log  查看历史记录

- git reset -hard HEAD^  版本回退到上一个版本（也可以将暂存区的修改退回到工作区）

- git reset -hard id   版本回退到commit id对应的版本

- git checkout --file  撤销在工作区的修改

- git rm file  删除版本库中的文件

- ### 本地分支回滚

  - 如果你在本地做了错误提交，那么回退版本的方法很简单,先用下面命令找到要回退的版本的commit id：

    `git reflog `

  - 接着回退版本:

    `git reset --hard Obfafd`

  - <font color='red'>0bfafd</font>  就是你要回退的版本的commit id的前面几位

- ### 自己的远程分支版本回退的方法

  - 首先要回退本地分支：方法同上.	
  - 紧接着强制推送到远程分支：`git push -f`
  - 注意：本地分支回滚后，版本将落后远程分支，必须 <font color='red'>使用强制推送覆盖远程分支</font> ，否则无法推送到远程分支

##  关于远程仓库
-   git remote add origin git@github.com:17307/Learning.git
-   git push origin master   推送到远程仓库
-   git clone -    git@github.com:PrettyMask/learngit.-    git  从远程仓库克隆
-   git remote rm origin  删掉远程仓库的命令  

##  分支

-   git branch dev  创建名为 dev 的分支
-   git checkout dev  切换到dev分支
-   git merge dev 合并dev分支
-   git branch -d dev 删除分支dev
-   git branch 查看分支并确定当前分支

##   git使用

- 创建好私钥后
     ```
     eval `ssh-agent -s`
     ```
    返回 `Agent pid 4784`
- 然后将私钥添加到缓存

  `$ ssh-add .ssh/id_rsa`

- `$ ssh -p 2200 -i ~/.ssh/id_rsa_test user@ssh.test.com`
- config配置文件

  ```con
  $ vim ~/.ssh/config
  Host sshtest
      HostName ssh.test.com
      User user
      Port 2200
      IdentityFile ~/.ssh/id_rsa_test
  
  Host ssttest2
      HostName ssh.test2.com
      User user2
      Port 2345
      IdentityFile ~/.ssh/id_rsa_test2
  
  ~路径为C：/User/.ssh
  ```

增加一段普通的话,作为测试.

test test testjkjkkkkkkkkkkkk