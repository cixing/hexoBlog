---
title: 添加SSH密钥对
date: 2021-09-13 16:36:55
tags: ssh
---
有时候我们需要给不同的git仓库添加ssh密钥对，方便我们的项目能够直接连接git仓库.

## 配置多个SSH密钥对

### 生成密钥对
1. 进入 key 的存储路径,SSH目录： ~/.ssh

``` bash
$ cd ~/.ssh
```
2. 生成命令如下，邮箱用自已的就行.

``` bash
$ ssh-keygen -t rsa -C "XXXXX@XXX.com"
```
会有提示，让输入存放 Key 的文件名，我输入了 test.rsa。（文件存在的话，可选择覆盖。不存在会自动创建。）

再后来会让设置一个密码和确认密码，可不设置，连续点再次回车即行。（若设置了，以后每次 git 提交都要此密码确认。）

### 配置对应仓库使用的密钥对
1.配置Config
此时github还无法识别使用哪一个私钥，若有config文件夹，修改
``` bash
Host github.com
IdentityFile ~/.ssh/github_id_rsa
```
此时就可以使用git clone直接拉取git仓库代码了，若使用https方式，每次都需要输入github的密码。
