# Git服务器搭建

## 目录

[TOC]

## 下载包

```sh
# 地址 https://git-scm.com/download
cd /data/download
# 下载
wget https://www.kernel.org/pub/software/scm/git/git-2.15.0.tar.gz
# 解压
tar -vzxf git-2.15.0.tar.gz
```

## 依赖库

```shell
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
```

## 创建用户&组

```shell
groupadd git
useradd -g git git
```

## 编译&安装

```shell
make prefix=/usr/local/git all
make prefix=/usr/local/git install
```

## Git仓库

```shell
mkdir /home/git
cd home/git
# 创建一个版本仓库并初始化
mkdir foo.git
cd foo.git
git init --bare
# 修改文件用户组
chown -R git:git /home/git/foo.git
```

## SSH连接(git-shell)

```shell
vim /etc/passwd
```

```shell
# 原配置 将最后一个执行文件路径改为
git:x:502:503::/home/newbmiao:/bin/bash
# 在安装包bin目录下
git:x:502:503::/home/git:/usr/local/git/bin/git-shell 
# 要启用还需源码报的git-shell命令交互
# 这样用户用git账户ssh连接后只能使用git命令了
cp /data/download/git-2.15.0/contrib/git-shell-commands /home/git/
```

## SSH免密码验证连接

- 服务端

```shell
cd /home/git/
#生成rsa密钥，可以直接一路回车，执行默认操作
ssh-keygen -C 'your@email.com' -t rsa
# 生成密钥后，会出现
# .ssh
# ├── id_rsa
# └── id_rsa.pub #公钥 服务端需要里边内容验证连接着身份

#粘贴客户端id_rsa.pub里的公钥，保存退出
vim /home/git/.ssh/authorized_keys

# 打开文件
vim /etc/ssh/sshd_config
# 开启RSA认证功能
RSAAuthentication yes
# 开启公匙认证
PubkeyAuthentication yes
# 据说不改会强制要求登录用户和文件拥有者用户相同
StricModes no

#然后要启动sshd和git-daemon
/etc/init.d/git-daemon restart 
#上边git-daemon在安装目录下/usr/local/git/libexec/git-core/git-daemon
/etc/init.d/sshd start
```

- 客户端

```shell
#生成rsa密钥，可以直接一路回车，执行默认操作
ssh-keygen -C 'your@email.com' -t rsa
# 生成密钥后，会出现
# .ssh
# ├── id_rsa
# └── id_rsa.pub #公钥 服务端需要里边内容验证连接着身份
# 打开 id_rsa.pub 复制里边内容
```

##  代码同步（HOOK）

```shell
# 假定网站目录在/www/web下
cd /home/git/repo.git/hooks
# 创建一个钩子
vim post-receive
# 保存退出
chown git:git post-receive
chmod +x post-receive
```

```shell
# 写入下面内容
GIT_WORK_TREE=/www/web  git checkout -f
```

##  客户端分支

```shell
# 创建一个空的工作目录
git init #初始化git
# 编辑些内容保存
# 添加到git缓存中
git add foo
# 提交修改
git commit -m 'init foo'
# 添加远程git仓库
git remote add origin git@your_host_name:/home/git/repo.git
# 同步到服务器
git push origin master
```

## 克隆&推送

```shell
# 克隆和推送：
git clone git@your_host_name:/home/git/repo.git
cd repo
vim README
git commit -am 'fix for the README file'
# 推送
git push origin master
```