

# 基于源码安装 Riak

原文地址：[这里](http://docs.basho.com/riak/latest/installing/source/)

需要基于源码安装 Riak 的情况：

- 官方未提供针对特定目标平台的版本；
-  希望对 Riak 的源码进行研究

### 安装依赖

#### **Erlang**

> Riak 成功安装的前提是正确的预先安装好 Erlang 版本。强烈建议基于 Basho 公司 patched 过的 Erlang 版本进行 Riak 2.0 系列版本的安装。    
> 该 patched 版本中的变更内容将会被添加到后续的 Erlang/OTP 官方版本中。

See [Installing Erlang](http://docs.basho.com/riak/2.1.3/installing/source/erlang/) for instructions.


----------

Erlang 安装关键点（摘录）：

预打包好的 Riak 版本中包含 Erlang 安装。    
如果你想基于源码构建 Riak ，你将需要安装 Basho 提供的 patched 版本的 Erlang 。    
如果你未使用 Basho 提供的版本，你将无法使用 Riak 的[安全特性](http://docs.basho.com/riak/2.1.3/ops/running/authz/) 。

> 关于官方支持的说明

> Please note that only packaged Riak installs are officially support. Visit Installing Riak for installing a supported Riak package.

----------

#### **Git**

Riak 的项目构建依赖多个 Git 仓库中的源码，需要通过 git 命令进行获取；

#### **GCC**

Riak 不能基于 Clang 进行编译，请确保你的默认 C/C++ 编译器为 GCC


### 安装

下面的指令用于生成一个完备的、自包含 Riak 包；    
生成路径为 $RIAK/rel/riak ，其中 $RIAK 为源码接下路径或者 git clone 路径；    


#### 源码安装

可以从 [Download Center](http://basho.com/resources/downloads/) 下载 Riak 源码进行构建：

```shell
curl -O http://s3.amazonaws.com/downloads.basho.com/riak/2.1/2.1.3/riak-2.1.3.tar.gz
tar zxvf riak-2.1.3.tar.gz
cd riak-2.1.3
make locked-deps
make rel
```

#### 从 GitHub 上获取

[Riak Github 仓库](https://github.com/basho/riak) 中包含更多关于编译和安装的信息。可以通过下面的命令进行获取和编译：

```shell
git clone git://github.com/basho/riak.git
cd riak
make locked-deps
make rel
```

### 平台相关操作指令

针对特定平台的操作执行，详见：

Debian & Ubuntu
FreeBSD
Mac OS X
[RHEL & CentOS](http://docs.basho.com/riak/2.1.3/installing/rhel-centos/#Installing-From-Source)

If you are running Riak on a platform not in the list above and need some help getting it up and running, join The Riak Mailing List and inquire about it there. We are happy to help you get up and running with Riak.

===

以下内容取自 [RHEL & CentOS](http://docs.basho.com/riak/2.1.3/installing/rhel-centos/#Installing-From-Source) （摘录）

源码安装

Riak requires an Erlang installation. Instructions can be found in Installing Erlang.
Building from source will require the following packages:

- gcc
- gcc-c++
- glibc-devel
- make
- pam-devel

You can install these with yum:

```shell
sudo yum install gcc gcc-c++ glibc-devel make git pam-devel
```

Now we can download and install Riak:

```shell
wget http://s3.amazonaws.com/downloads.basho.com/riak/2.1/2.1.3/riak-2.1.3.tar.gz
tar zxvf riak-2.1.3.tar.gz
cd riak-2.1.3
make rel
```

You will now have a fresh build of Riak in the rel/riak directory.



