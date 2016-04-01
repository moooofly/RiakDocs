

# Riak 安装校验

原文地址：[这里](http://docs.basho.com/riak/2.1.3/installing/verify-install/)

安装后，需要验证下每个节点是否能够正确处理请求。

*In this document, we cover ways of verifying that your Riak nodes are operating correctly. After you've determined that your nodes are functioning and you're ready to put Riak to work, be sure to check out the resources in the Now What? section below.*    
在本文中，将介绍几种确认 Riak 节点是否运行正常的方法。

### 启动一个 Riak 节点


> 源码安装注意事项

> 需要将 Riak 可执行程序所在路径添加到 PATH 中。

后台启动 Riak 节点可以执行：

```shell
riak start
```

*A successful start will return no output. If there is a problem starting the node, an error message is printed to standard error.*    
没有输出就是正确的输出。

以交互式 console 前台启动 Riak 节点：

```shell
riak console
```

*A Riak node is typically started in console mode as part of **debugging** or **troubleshooting** to gather more detailed information from the Riak startup sequence. Note that if you start a Riak node in this manner, it is running as a foreground process that will be exited when the console is closed.*

可以在 erlang console 上通过下面的命令退出：

```erlang
q().
```

节点成功启动后，可以通过下面的命令检查节点是否在运行：

```shell
riak ping
```

若节点运行正常，则会回应 pong ；否则会回应 Node <nodename> not responding to pings 。

实际测试情况如下

```shell
[root@Betty riak-2.1.3]# ./rel/riak/bin/riak start
!!!!
!!!! WARNING: ulimit -n is 1024; 65536 is the recommended minimum.
!!!!
[root@Betty riak-2.1.3]# 
[root@Betty riak-2.1.3]# ./rel/riak/bin/riak ping
pong
[root@Betty riak-2.1.3]# 
[root@Betty riak-2.1.3]# ./rel/riak/bin/riak stop
ok
[root@Betty riak-2.1.3]# 
[root@Betty riak-2.1.3]# ./rel/riak/bin/riak ping
Node 'riak@127.0.0.1' not responding to pings.
[root@Betty riak-2.1.3]# 
```


> 关于文件描述符数目限制的问题

> - 正如上面实验中的输出，在未调整可打开文件数目限制（ulimit -n）前，Riak 会在启动的时候给出警告信息。    
> - 强烈建议在运行 Riak 的机器上，增大系统默认的可打开文件数目限制；
> - 需要进行调整的原因详见[这里](http://docs.basho.com/riak/2.1.3/ops/tuning/open-files-limit/)


### 节点是否能够正常工作？

测试 Riak 节点是否已经主备就绪的方法为，通过 riak-admin 测试其是否能够进行读写操作：

```shell
riak-admin test
```

成功执行的输出如下：

```shell
Attempting to restart script through sudo -H -u riak
Successfully completed 1 read/write cycle to '<nodename>'
```

---

实际测试

```shell
# 失败情况
[root@Betty riak-2.1.3]# ./rel/riak/bin/riak-admin test
Failed to read test value: {error,{insufficient_vnodes,0,need,1}}

# 成功情况
[root@Betty riak-2.1.3]# ./rel/riak/bin/riak-admin test
Successfully completed 1 read/write cycle to 'riak@127.0.0.1'
[root@Betty riak-2.1.3]# 
```

---

你还可以通过 curl 命令行工具测试 Riak 是否正常；

当你已经在一个节点上运行了 Riak 时，可以通过下面的命令获取与 [bucket 类型](http://docs.basho.com/riak/2.1.3/dev/advanced/bucket-types/)相关的属性值：

```shell
curl -v http://127.0.0.1:8098/types/default/props
```

---

实际测试

```shell
[root@Betty riak-2.1.3]# curl -v http://127.0.0.1:8098/types/default/props
* About to connect() to 127.0.0.1 port 8098 (#0)
*   Trying 127.0.0.1... connected
* Connected to 127.0.0.1 (127.0.0.1) port 8098 (#0)
> GET /types/default/props HTTP/1.1
> User-Agent: curl/7.19.7 (x86_64-redhat-linux-gnu) libcurl/7.19.7 NSS/3.14.0.0 zlib/1.2.3 libidn/1.18 libssh2/1.4.2
> Host: 127.0.0.1:8098
> Accept: */*
> 
< HTTP/1.1 200 OK
< Vary: Accept-Encoding
< Server: MochiWeb/1.1 WebMachine/1.10.8 (that head fake, tho)
< Date: Tue, 29 Mar 2016 02:55:51 GMT
< Content-Type: application/json
< Content-Length: 447
< 
* Connection #0 to host 127.0.0.1 left intact
* Closing connection #0
{"props":{"allow_mult":false,"basic_quorum":false,"big_vclock":50,"chash_keyfun":{"mod":"riak_core_util","fun":"chash_std_keyfun"},"dvv_enabled":false,"dw":"quorum","last_write_wins":false,"linkfun":{"mod":"riak_kv_wm_link_walker","fun":"mapreduce_linkfun"},"n_val":3,"notfound_ok":true,"old_vclock":86400,"postcommit":[],"pr":0,"precommit":[],"pw":0,"r":"quorum","rw":"quorum","small_vclock":50,"w":"quorum","write_once":false,"young_vclock":20}}

[root@Betty riak-2.1.3]# 
```
---

将例子中的 127.0.0.1 替换 Riak 节点的实际 IP 地址，或者 FQDN(fully qualified domain name) ，你将能够获得类似下面的应答信息：

```shell
* About to connect() to 127.0.0.1 port 8098 (#0)
*   Trying 127.0.0.1... connected
* Connected to 127.0.0.1 (127.0.0.1) port 8098 (#0)
> GET /riak/test HTTP/1.1
> User-Agent: curl/7.21.6 (x86_64-pc-linux-gnu)
> Host: 127.0.0.1:8098
> Accept: */*
>
< HTTP/1.1 200 OK
< Vary: Accept-Encoding
< Server: MochiWeb/1.1 WebMachine/1.9.0 (someone had painted it blue)
< Date: Wed, 26 Dec 2012 15:50:20 GMT
< Content-Type: application/json
< Content-Length: 422
<
* Connection #0 to host 127.0.0.1 left intact
* Closing connection #0
{"props":{"name":"test","allow_mult":false,"basic_quorum":false,
 "big_vclock":50,"chash_keyfun":{"mod":"riak_core_util",
 "fun":"chash_std_keyfun"},"dw":"quorum","last_write_wins":false,
 "linkfun":{"mod":"riak_kv_wm_link_walker","fun":"mapreduce_linkfun"},
 "n_val":3,"notfound_ok":true,"old_vclock":86400,"postcommit":[],"pr":0,
 "precommit":[],"pw":0,"r":"quorum","rw":"quorum","small_vclock":50,
 "w":"quorum","young_vclock":20}}
```

*The output above shows a successful response (HTTP 200 OK) and additional details from the verbose option. The response also contains the bucket properties for the default bucket type.*    
上面的输出表示获取信息成功。


### Riaknostic

需要知道的是，针对 Riak 的一些基本配置和节点健康情况进行常规检查是很有必要的；
可以使用 Riak 内置的诊断工具 [Riaknostic](http://riaknostic.basho.com/) 。

---

以下内容取自：[这里](http://riaknostic.basho.com/)

#### **Riaknostic diagnostic tools for Riak**

##### 概述

有些时候，我们知道 Riak 出现了问题。但我们怎样才能知道到底出了什么问题？Riaknostic 就是我们所需的诊断工具。

```shell
$ riak-admin diag

15:34:52.736 [warning] Riak crashed at Wed, 07 Dec 2011 21:47:50 GMT, leaving crash dump in /srv/riak/log/erl_crash.dump. Please inspect or remove the file.
15:34:52.736 [notice] Data directory /srv/riak/data/bitcask is not mounted with 'noatime'. Please remount its disk with the 'noatime' flag to improve performance.
```

Riaknostic 可以通过上面的命令触发；    
Riaknostic 由一组诊断检查命令集构成，用于发现目标 Riak 节点的常规运行问题，并提出问题解决建议；    
这些检查方案来自于非常有经验的 Basho Client Services Team ，以及基于邮件列表、IRC 聊天室，和其他在线媒体中产生的公共讨论内容；    

成功启动 Riaknostic 的前提为，确保 Riak 已经在节点上正在运行，之后就可以调用如下命令

```shell
riak-admin diag
```

More extensive documentation for Riaknostic can be found in the [Inspecting a Node](http://docs.basho.com/riak/2.1.3/ops/running/nodes/inspecting/) guide.


---


### 现在到哪了？

你已经拥有了一个能够正常工作的 Riak 节点了！

接下来，你可以研究下面的内容了：[客户端库](http://docs.basho.com/riak/2.1.3/dev/using/libraries/)  。


















