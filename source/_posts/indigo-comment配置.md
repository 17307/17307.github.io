---
title: indigo-comment配置
date: 2018-09-08 18:27:35
categories: others
---

# indigo-comment 评论设置 #

## gitment 集成 ##

幸运的是,indigo的主题,已经默认集成了`gitment`,所以只需要配置一下这些内容就好:

![](1.png)

其中的`owner`为你的用户名的哪个标识,`repo`为仓库名称,<font color='red'>不是地址</font>,clien_id与clien_secret为注册后得到的内容.

注册信息的填充方法

![](2.png)

其中最主要的是最后一个内容:`Authorization callback URL`.

## 问题 ##

### 出现 404 ###

说明是`owner` 或者 `repo` 等信息填充有误.

### 评论初始化失败 ###

**原因**: 文章标题过长

**解决**: 通过在`themes\indigo\layout\_partial\plugins\gitment.ejs`的模板中新增`id`字段.

![](3.png)