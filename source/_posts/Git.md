---
title: Git
date: 2021-01-07 15:28:10
tags: git
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

![/image/gitlab-project.png](gitlab)

review 然后合并到master

修改版本号 mvn versions:set -DnewVersion=1.4.2.0-SNAPSHOT -DgenerateBackupPoms=false 并提交

发布系统打tag

### 打tag

1. 代码准备：合并代码等，确定打tag的分支（一般是master分支，cic需要打tag用cic分支）
2. pom文件的版本号修改/确认，如需要发布 1.4.2.0 版本，可以执行： mvn versions:set -DnewVersion=1.4.2.0-SNAPSHOT -DgenerateBackupPoms=false
3. 到 https://deploy.datastory.com.cn/#/deploy/Create，选择项目 ds-trident-backend 、分支，选择操作（“打tag”）及平台（rocket）然后确认，注意确认编译的参数，然后会自动跳到“发布进度”，需要再次点击确认 

### 完成后重新rebase



