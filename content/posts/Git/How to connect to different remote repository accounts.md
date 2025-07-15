+++
date = '2025-07-15T10:03:00+08:00'
draft = false
title = '如何连接到不同的远程仓库账号'
tags = ["Git"]
categories = ["Git"]
+++

# 如何连接到不同的远程仓库账号

## 📌 需求
以GitHub举例，我现在有两个GitHub账号，在不同的本地仓库里我需要连接到不同的GitHub账号，比如一个账号是`personal`,一个是`work`。

## 🔍 Github是如何区分上传的数据是属于哪个用户的

### 根据你上传到GitHub的密钥

以ssh协议举例，当你使用ssh协议连接到Github的，首先一定需要在GitHub账号上将本地的公钥上传给GitHub。假设我在本地生成了两对密钥，分别发给`personal`和`work`账号。

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github_personal -C "your_personal_email@example.com"
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_github_work -C "your_work_email@example.com"
```

Github会根据你使用的是哪个密钥连接GitHub来判断你当前使用的是`personal`还是`work`。使用以下命令即可知道本机系统现在会使用哪个密钥来与Github连接。

```bash
ssh -T -v git@github.com
```

当你看到这两行输出表示最终使用的密钥是哪个
```bash
debug1: Offering public key: /Users/你的用户名/.ssh/id_ed25519_github_A
debug1: Server accepts key: /Users/你的用户名/.ssh/id_ed25519_github_A
```

在最后看到这个输出表示GitHub告诉你你现在是谁，你也可以不使用-v这样只会输出这句话。

```bash
Hi Ripple-zjw! You've successfully authenticated, but GitHub does not provide shell access.
```

### 当前仓库所使用的用户名和邮箱

这不会影响连接到GitHub账号，但会影响当前仓库上传到GitHub后commit的作者。使用一下命令查看用户名和邮箱等信息是否正确。

```bash
git config --list
```

## ✅ 让不同的本地仓库连接到不同的GitHub账号

根据前面的解释，其实只需要让当前的本地仓库使用正确的密钥连接GitHub即可。

首先在`~/.ssh/config`文件中配置两个自定义的ssh连接，它们都访问`git@github.com`但使用不同的密钥。

```ssh
Host github-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_personal
    IdentitiesOnly yes

Host github-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_work
    IdentitiesOnly yes
```

然后进入你的本地仓库，将remote url修改为你写的ssh Host。

```bash
git remote set-url origin git@github-work:your_work_username/repo.git
# 或
git remote set-url origin git@github-personal:your_personal_username/repo.git
```

使用以下命令查看远程地址是否被修改。

```
git remote -v
```

这样当你在仓库push操作时，仓库根据`git remote`的远程地址会自动使用ssh config文件里的配置连接GitHub。



