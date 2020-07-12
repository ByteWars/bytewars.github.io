---
title: "How to Encrypt Git Files"
date: 2020-07-12T11:00:42+08:00
draft: false
author: "谷中仁"
authorLink: "https://guzhongren.github.io"
description: "使用git-crypt加密git敏感文件"
license: "Creative Commons Attribution 4.0 International license"

tags: ["git-crypt", "crypt", "RSA", "secret", "gpg", "gnuPG", "gpg", "加密🔐", "transcrypt"]
categories: ["git"]
featuredImage: "https://s1.ax1x.com/2020/07/12/U32INn.png"
images: [""]
---

## 为什么需要crypt

在开发过程中经常会遇到一个问题：

> 怎么将敏感数据让特定的人获取到？

常用的解决方法如下：

* 使用如 AWS 的 KMS, ParameterStore等服务，给不同的用户以访问该数据的角色；
* 有专人托管，在需要的时候联系他，由特定方式转发给你，如加密邮件等

### 分析
- 对于第二种方式，相对简单粗暴，但是不利于管理。
- 第一种方案，你需要依托第三方服务，你得绝对信任他，相信他是不会出问题的；再者，你的数据管理也需要有特定的规范，不然你都不知道你存储了什么数据，更不知道有哪些数据。

针对第一种情况，当我们的数据越来多的时候，我们需要将其状态可控起来，其实就是 `date as code`, 每次对数据的增删改查，我们都可以有追踪和数据保存。

## 有哪些方案呢?

* [git-crypt](https://github.com/AGWA/git-crypt)
* [transcrypt](https://github.com/elasticdog/transcrypt)

在这里，我推荐使用 `git-crypt`, 其特点如下：

* 加密后上传git，在git上保存的是`二进制`文件；
* 分发`密钥`给可信开发人员，进行解密，维护配置文件;
* 解密后为明文内容，如需上传，不用再进行加密，工具`自动`(配合`git hook`)会生成最新的二进制文件后上传;
* 由`c++`编写；
* Mac 可以通过`brew等`方式下载;
* 利用了加密工具`gpg`进行加密处理。

### GPG

![GPG](https://ruanyifeng.com/blogimg/asset/201307/bg2013071202.png)

要了解什么是GPG，就要先了解PGP。

1991年，程序员Phil Zimmermann为了避开政府监视，开发了加密软件PGP。这个软件非常好用，迅速流传开来，成了许多程序员的必备工具。但是，它是商业软件，不能自由使用。所以，自由软件基金会决定，开发一个PGP的替代品，取名为GnuPG。这就是GPG的由来。

gpg 可以对密钥的`增删改查`进行操作,也可以将公钥发送到pgp等服务器，让别人搜索到你，具体操作可以参考阮一峰的这片博文 [《GPG入门教程》](https://ruanyifeng.com/blog/2013/07/gpg.html)

## git-crypt 操作

### 安装gpg 和 git-crypt

```shell
brew install git-crypt
brew install gpg
```

### gpg version
```shell
gpg --version
gpg (GnuPG) 2.2.19
libgcrypt 1.8.5
Copyright (C) 2019 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /Users/c4/.gnupg
支持的算法：
公钥： RSA, ELG, DSA, ECDH, ECDSA, EDDSA
密文： IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
    CAMELLIA128, CAMELLIA192, CAMELLIA256
散列： SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
压缩：  不压缩, ZIP, ZLIB, BZIP2
```
### 生成gpg userId

```shell
gpg --full-generate-key
gpg (GnuPG) 2.2.19; Copyright (C) 2019 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

请选择您要使用的密钥类型：
   (1) RSA 和 RSA （默认）
   (2) DSA 和 Elgamal
   (3) DSA（仅用于签名）
   (4) RSA（仅用于签名）
  (14) Existing key from card
您的选择是？
RSA 密钥的长度应在 1024 位与 4096 位之间。
您想要使用的密钥长度？(2048) 4096
请求的密钥长度是 4096 位
请设定这个密钥的有效期限。
         0 = 密钥永不过期
      <n>  = 密钥在 n 天后过期
      <n>w = 密钥在 n 周后过期
      <n>m = 密钥在 n 月后过期
      <n>y = 密钥在 n 年后过期
密钥的有效期限是？(0)
密钥永远不会过期
这些内容正确吗？ (y/N) y

GnuPG 需要构建用户标识以辨认您的密钥。

真实姓名： guzhongren
电子邮件地址： guzhongren@live.cn
注释： guzhongren
您选定了此用户标识：
    “guzhongren (guzhongren) <guzhongren@live.cn>”

更改姓名（N）、注释（C）、电子邮件地址（E）或确定（O）/退出（Q）？ o
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
gpg: 密钥 25DD25A47AEF036A 被标记为绝对信任
gpg: 吊销证书已被存储为‘/Users/c4/.gnupg/openpgp-revocs.d/4FA612D2DB4244E7D64A1C1025DD25A47AEF036A.rev’
公钥和私钥已经生成并被签名。

pub   rsa4096 2020-07-11 [SC]
      4FA612D2DB4244E7D64A1C1025DD25A47AEF036A
uid                      guzhongren (guzhongren) <guzhongren@live.cn>
sub   rsa4096 2020-07-11 [E]
```

### 获取UserID

```shell
gpg --list-secret-keys --keyid-format LONG
gpg: 正在检查信任度数据库
gpg: 绝对信任密钥 8823E362E95562B8 的公钥未找到
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: 深度：0  有效性：  4  已签名：  0  信任度：0-，0q，0n，0m，0f，4u
gpg: 下次信任度数据库检查将于 2022-01-26 进行
/Users/c4/.gnupg/pubring.kbx
----------------------------
sec   rsa4096/25DD25A47AEF036A 2020-07-11 [SC]
      4FA612D2DB4244E7D64A1C1025DD25A47AEF036A
uid                 [ 绝对 ] guzhongren (guzhongren) <guzhongren@live.cn>
ssb   rsa4096/DCC72940818AB355 2020-07-11 [E]

```

`sec   rsa4096/25DD25A47AEF036A 2020-07-11 [SC]` 中 `25DD25A47AEF036A`就是gpg userId

## git-crypt 操作

### 初始化

进入需要做加密的git repo 做 git-crypt 初始化操作,并将上面获取到的gpg userId 添加进去

```shell
git-crypt init
git-crypt add-gpg-user 25DD25A47AEF036A
```

### 加密文件过滤器`.gitattributes`

格式为： * filter=git-crypt diff=git-crypt ；如下，我需要加密`secretfile`, `*.key`和`secretdir/**`, 那么内容如下：

```
secretfile filter=git-crypt diff=git-crypt
*.key filter=git-crypt diff=git-crypt
secretdir/** filter=git-crypt diff=git-crypt
```

### 清理 config 的 git 缓存

```shell
git rm -r –cached config
```

### 添加需要加密的文件和数据

在`secretdir`文件夹下添加 `secret.yaml`,并添加如下内容：

```yaml
username:password
```
### 上传到 git 仓库

此步不用担心你的`secret.yaml`文件以明文的形式上传，git-crypt会在 commit 之前将过滤后的数据加密成二进制，所以不用担心在仓库中存储敏感信息。

```shell
git add .
git commit -m ‘git-crypt’
git push
```

如果你去git仓库中浏览secret.yaml文件，会是一个二进制文件，如图：

![git-crypt.png](https://i.loli.net/2020/07/12/lXVnNw5EbKCcvhe.png)

## 协作

与他人协作，别人需要知道对应解密的key

### 导出密钥

```shell
git-crypt export-key ~/Desktop/git-crypt.key
```
得到git-crypt.key后，将该密钥分发给可信的团队成员，团队成员将仓库 clone 下来后，使用如下命令解密即可

```shell
git-crypt unlock /path/to/git-crypt.key
```

此时，在本地就可以看到加密后的文件内容了。

### 更新密钥

因为git-crypt没有提供删除或者更新密钥的命令，所以参考了一个issue: https://github.com/AGWA/git-crypt/issues/47#issuecomment-492939759;步骤如下：

* Make a backup (with decrypted files): cp -r . /path/to/backup
* Save a list of files that are encrypted: git crypt status | grep -v 'not encrypted' > ../encrypted-files.txt
* Make git-crypt forget about itself: rm .git-crypt
* Delete the encrypted files: awk '{print $2}' ../encrypted-files.txt | xargs rm
* Commit (at this point you get a repo without git-crypt stuff)
* Add git-crypt from scratch (init and add-gpg-user)
* Copy the decrypted files from the backup: awk '{print $2}' ../encrypted-files.txt | while read l; do cp /path/to/backup/$l $l; done
* Commit (at this point you are done, but be sure to verify things are properly encrypted before publishing)

实践结果：在更新完成之后，协作者需要重新clone 仓库，然后用新的密钥来解密，对密钥管理者来说操作比较麻烦
### 总结

在敏感数据越来越多的时候，作为开发者，我们更应该将所有的数据都`as code`,为以后维护提供方便。`git-crypt`确实是一个比较好的选择。

但是`git-crypt`有一个最大的缺点：

* 只能添加不能删除 gpg userId, 导致更新密钥会比较麻烦，如果要更新密钥，那么就需要做重置


## Reference

* [Demo: git-crypt-test](https://github.com/beef-noodles/git-crypt-test)
* [博客:https://guzhongren.github.io/](https://guzhongren.github.io/)
* [图床:https://sm.ms/](https://sm.ms/)
* [https://gnupg.org/](https://gnupg.org/)
* [AGWA/git-crypt](https://github.com/AGWA/git-crypt)
* [git-crypt 使用](http://einverne.github.io/post/2019/11/git-crypt-usage.html)
* [git上的配置文件如何加密？](https://www.jianshu.com/p/a40fc90df943)
* [Easy Git Crypt User Identification](https://www.devopsgroup.com/blog/easy-git-crypt-user-identification/)
* [Update git crypt key](https://github.com/AGWA/git-crypt/issues/47#issuecomment-492939759)
* [List gpg user](https://github.com/AGWA/git-crypt/issues/189#issuecomment-549787656)
* [Gnu 隐私卫士 (GnuPG) 袖珍 HOWTO (中文版)](https://www.gnupg.org/howtos/zh/index.html)
* [transcrypt使用案例](https://www.lnmpy.com/blog/transcrypt-intro/)

## Hereby declared（特此申明）

本文仅代表个人观点，与[ThoughtWorks](https://www.thoughtworks.com/) 公司无任何关系。

----
![微信公众号](https://s1.ax1x.com/2020/05/23/Yx1I5q.png)

