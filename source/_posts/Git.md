---
title: Git
date: 2021-01-07 15:28:10
updated: 2021-01-08 10:21:10
tags: git
categories: git # 目录
---
# Git

## 合并分支、打tag

参考 [https://wiki.datastory.com.cn/pages/viewpage.action?pageId=38618481#id-%E5%B7%A5%E5%9C%BA%E5%90%8E%E7%AB%AF%E5%BC%80%E5%8F%91README-4.3%E5%8F%91%E5%B8%83&%E6%89%93tag](https://wiki.datastory.com.cn/pages/viewpage.action?pageId=38618481#id-工场后端开发README-4.3发布&打tag)

### 合并分支操作：（以合并dev到master为例子）

a. 切换到master分支 git checkout master

b. 合并代码 git merge dev

c. 推到远程 git push


### 发布前


dev提交pr到master --> idea 下载 gitlab project插件，然后配置下gitlab的服务器 

![gitlab](/image/gitlab-project.png)

review 然后合并到master

修改版本号 mvn versions:set -DnewVersion=1.4.2.0-SNAPSHOT -DgenerateBackupPoms=false 并提交

发布系统打tag

### 完成后重新rebase
