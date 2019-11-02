---
layout: post
title: "GitHub 使用 gpg 签名"
tagline: ""
description: ""
category: 经验总结
tags: [github, git, gpg, linux, ]
last_updated:
---

Ubuntu 下，GnuPG 2.0 的支持都在 `gnupg2` 这个 [packages](http://packages.ubuntu.com/search?keywords=gnupg2) 下，通过下面命令安装：

    sudo apt-get install gnupg2

GitHub 要求使用 GnuPG 2.1 及以后的版本。


## 生成 GPG 签名
使用如下命令生成签名：

    gpg2 --full-gen-key

1. 选择默认 RSA and RSA
2. 推荐 4096
3. 过期时间
4. 相关信息包括 Real name, Email, Comment(comment 可以写一些标示）
5. 密码

如果遇到尝试卡在生成签名的步骤，提示

    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.

可能需要安装 `rng-tools`

    sudo apt install rng-tools

之后在执行上面的命令。

使用以下命令查看本地密钥

    gpg2 --list-keys --keyid-format LONG

结果

    /home/einverne/.gnupg/pubring.gpg
    ---------------------------
    pub   rsa4096/F80B65AAAAAAAAAA 2018-01-31 [SC]
    uid                 [ultimate] Ein Verne (co) <email@address>
    sub   rsa4096/B63A4CAAAAAAAAAA 2018-01-31 [E]

将其中的第三行 sec 中 `rsa4096` 后面的 ID 记住，拷贝出来。

然后使用

    gpg2 --armor --export ID | xclip -sel c

来获取 GPG KEY，拷贝 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 和 `-----END PGP PUBLIC KEY BLOCK-----` 之前，包括这两行的内容到 GitHub。

使用 `| xclip -sel c` 可以直接将命令输出结果拷贝到系统粘贴板。

## 配置 GPG
产生 GPG，并且已经添加到 [GitHub 后台](https://github.com/settings/gpg/new)，那么需要本地配置，告诉 git 本地签名。查看本地 gpg

    gpg2 --list-keys --keyid-format LONG

添加配置，这里记得使用公钥

    git config --global user.signingkey F80B65AAAAAAAAAA
    git config --global gpg.program gpg2

将 GPG 添加到 bashrc

    echo 'export GPG_TTY=$(tty)' >> ~/.bashrc

## 签名 commit

在提交时使用 `-S` 选项

    git commit -S -m your commit message

来本地签名提交

提交标签同理

    git tag -s mytag

可以使用 `-v` 来验证

    git tag -v mytag

## 其他 gpg 命令
列出公钥

    gpg2 --list-keys

查看秘钥

    gpg2 --list-secret-keys

删除 uid 私钥

    gpg2 --delete-secret-keys [uid]

删除 uid 公钥

    gpg2 --delete-keys [uid]

## 问题
使用过程中遇到的一些问题

### 提交 commit 时 failed to sign the data
在 `git commit -S` 时如果遇到：

    gpg: signing failed: Operation cancelled
    gpg: signing failed: Operation cancelled
    error: gpg failed to sign the data
    fatal: failed to write commit object

尝试

    export GPG_TTY=$(tty)

### 强制每次 git 提交使用 gpg 加密

    git config --global commit.gpgsign true

对应如果选择关闭就直接使用 false 即可。

### 导出私钥用于不同电脑之间同步
在一台电脑中生成的 gpg secret key 可以用于不同的电脑

- 首先运行 `gpg2 --list-secret-keys` 来确认本机的私钥，需要记住私钥的 ID （在第二列）
- 导出私钥 `gpg2 --export-secret-keys $ID > my-private-key.asc`
- 拷贝私钥到目标机器 （scp）
- 导入私钥 `gpg2 --import my-private-key.asc`

如果在第二台机器中已经有了公钥，私钥，那么需要分别删除 `gpg2 --delete-keys` 和 `gpg2 --delete-secret-keys`

如果熟悉 scp 可以

    scp -rp ~/.gnupg name@server_ip:~/

## reference

- <https://help.github.com/articles/signing-commits-with-gpg/>
