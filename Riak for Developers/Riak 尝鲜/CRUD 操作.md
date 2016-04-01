
# CRUD 操作

原文地址：[这里](http://docs.basho.com/riak/2.1.3/dev/taste-of-riak/)

本节内容的前提条件：

- Installing Riak
- 安装 Erlang
- 创建并启动一个 Riak 节点


### 客户端安装

从 GitHub 上下载最新的 Erlang 客户端代码([zip](https://github.com/basho/riak-erlang-client/archive/master.zip), [GitHub repository](https://github.com/basho/riak-erlang-client/))。

接着，就可以按照下面的方式启动 Erlang 控制台（直接指定客户端库路径）：

```shell
erl -pa CLIENT_LIBRARY_PATH/ebin/ CLIENT_LIBRARY_PATH/deps/*/ebin
```

现在，让我们创建客户端库到 Riak 节点的 link 。如果你是基于单个本地 Riak 节点进行测试，则使用下面的命令创建 link ：

```erlang
{ok, Pid} = riakc_pb_socket:start("127.0.0.1", 8087).
```

如果你建立的是本地 Riak 集群，则使用下面的代码片段进行操作（默认配置情况）：

```erlang
{ok, Pid} = riakc_pb_socket:start_link("127.0.0.1", 10017).
```

到此，我们已经可以和 Riak 进行交互了。

---

实验结果

```shell
[root@Betty riak-erlang-client]# erl -pa ./ebin/ -pa ./deps/*/ebin
Erlang/OTP 17 [erts-6.0] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V6.0  (abort with ^G)
1> 
1> code:which(riakc_pb_socket).
"./ebin/riakc_pb_socket.beam"
2> 
2> {ok, Pid} = riakc_pb_socket:start_link("127.0.0.1", 8087).
** exception exit: {tcp,econnrefused}
3> 
3> {ok, Pid} = riakc_pb_socket:start_link("127.0.0.1", 10017).
{ok,<0.41.0>}
4> 
```
---

### 在 Riak 中创建对象

First, let’s create a few Riak objects. For these examples we'll be using the bucket test.

```erlang
MyBucket = <<"test">>.
Val1 = 1.
Obj1 = riakc_obj:new(MyBucket, <<"one">>, Val1).
riakc_pb_socket:put(Pid, Obj1).
```

In this first example, we have stored the integer 1 with the lookup key of one. Next, let’s store a simple string value of two with a matching key.

```erlang
Val2 = <<"two">>.
Obj2 = riakc_obj:new(MyBucket, <<"two">>, Val2).
riakc_pb_socket:put(Pid, Obj2).
```

That was easy. Finally, let’s store something more complex, a tuple this time. You will probably recognize the pattern by now.

```erlang
Val3 = {value, 3}.
Obj3 = riakc_obj:new(MyBucket, <<"three">>, Val3).
riakc_pb_socket:put(Pid, Obj3).
```


### 从 Riak 中读取对象

Now that we have a few objects stored, let’s retrieve them and make sure they contain the values we expect.

```erlang
{ok, Fetched1} = riakc_pb_socket:get(Pid, MyBucket, <<"one">>).
{ok, Fetched2} = riakc_pb_socket:get(Pid, MyBucket, <<"two">>).
{ok, Fetched3} = riakc_pb_socket:get(Pid, MyBucket, <<"three">>).

Val1 =:= binary_to_term(riakc_obj:get_value(Fetched1)). %% true
Val2 =:= riakc_obj:get_value(Fetched2).                 %% true
Val3 =:= binary_to_term(riakc_obj:get_value(Fetched3)). %% true
```

That was easy. We simply request the objects by bucket and key.

---

实验结果

```erlang
4> 
4> MyBucket = <<"test">>.
<<"test">>
5> Val1 = 1.
1
6> Obj1 = riakc_obj:new(MyBucket, <<"one">>, Val1).
{riakc_obj,<<"test">>,<<"one">>,undefined,[],undefined,1}
7> riakc_pb_socket:put(Pid, Obj1).
ok
8> 
8> 
8> Val2 = <<"two">>.
<<"two">>
9> Obj2 = riakc_obj:new(MyBucket, <<"two">>, Val2).
{riakc_obj,<<"test">>,<<"two">>,undefined,[],undefined,
           <<"two">>}
10> riakc_pb_socket:put(Pid, Obj2).
ok
11> 
11> Val3 = {value, 3}.
{value,3}
12> Obj3 = riakc_obj:new(MyBucket, <<"three">>, Val3).
{riakc_obj,<<"test">>,<<"three">>,undefined,[],undefined,
           {value,3}}
13> riakc_pb_socket:put(Pid, Obj3).
ok
14> 
14> {ok, Fetched1} = riakc_pb_socket:get(Pid, MyBucket, <<"one">>).
{ok,{riakc_obj,<<"test">>,<<"one">>,
               <<107,206,97,96,96,96,204,96,202,5,82,60,71,131,54,114,47,
                 127,112,250,33,68,40,...>>,
               [{{dict,3,16,16,8,80,48,
                       {[],[],[],[],[],[],[],[],[],[],[],[],...},
                       {{[],[],[],[],[],[],[],[],[],[],...}}},
                 <<131,97,1>>}],
               undefined,undefined}}
15> {ok, Fetched2} = riakc_pb_socket:get(Pid, MyBucket, <<"two">>).
{ok,{riakc_obj,<<"test">>,<<"two">>,
               <<107,206,97,96,96,96,204,96,202,5,82,60,115,237,56,78,
                 113,220,249,251,16,34,148,...>>,
               [{{dict,2,16,16,8,80,48,
                       {[],[],[],[],[],[],[],[],[],[],[],[],...},
                       {{[],[],[],[],[],[],[],[],[],[],...}}},
                 <<"two">>}],
               undefined,undefined}}
16> {ok, Fetched3} = riakc_pb_socket:get(Pid, MyBucket, <<"three">>).
{ok,{riakc_obj,<<"test">>,<<"three">>,
               <<107,206,97,96,96,96,204,96,202,5,82,60,91,88,101,61,57,
                 250,255,184,65,132,18,...>>,
               [{{dict,3,16,16,8,80,48,
                       {[],[],[],[],[],[],[],[],[],[],[],[],...},
                       {{[],[],[],[],[],[],[],[],[],[],...}}},
                 <<131,104,2,100,0,5,118,97,108,117,101,97,3>>}],
               undefined,undefined}}
17> 
17> riakc_obj:get_value(Fetched1).
<<131,97,1>>
18> binary_to_term(riakc_obj:get_value(Fetched1)).
1
19> riakc_obj:get_value(Fetched2).                
<<"two">>
20> 
20> binary_to_term(riakc_obj:get_value(Fetched3)).
{value,3}
21> 
```

---


### 更新 Riak 中的对象

While some data may be static, other forms of data may need to be updated. This is also easy to do. Let’s update the value in the third example to 42, update the Riak object, and then save it.

```erlang
NewVal3 = setelement(2, Val3, 42).
UpdatedObj3 = riakc_obj:update_value(Fetched3, NewVal3).
{ok, NewestObj3} = riakc_pb_socket:put(Pid, UpdatedObj3, [return_body]).
```

We can verify that our new value was saved by looking at the value returned.

```erlang
rp(binary_to_term(riakc_obj:get_value(NewestObj3))).
```

---

实验结果

```
21> 
21> NewVal3 = setelement(2, Val3, 42).
{value,42}
22> UpdatedObj3 = riakc_obj:update_value(Fetched3, NewVal3).
{riakc_obj,<<"test">>,<<"three">>,
           <<107,206,97,96,96,96,204,96,202,5,82,60,91,88,101,61,57,
             250,255,184,65,132,18,25,243,...>>,
           [{{dict,3,16,16,8,80,48,
                   {[],[],[],[],[],[],[],[],[],[],[],[],[],[],...},
                   {{[],[],[],[],[],[],[],[],[],[],[[...]|...],[],...}}},
             <<131,104,2,100,0,5,118,97,108,117,101,97,3>>}],
           undefined,
           {value,42}}
23> {ok, NewestObj3} = riakc_pb_socket:put(Pid, UpdatedObj3, [return_body]).
{ok,{riakc_obj,<<"test">>,<<"three">>,
               <<107,206,97,96,96,96,202,96,202,5,82,60,91,88,101,61,57,
                 250,255,184,1,217,140,...>>,
               [{{dict,3,16,16,8,80,48,
                       {[],[],[],[],[],[],[],[],[],[],[],[],...},
                       {{[],[],[],[],[],[],[],[],[],[],...}}},
                 <<131,104,2,100,0,5,118,97,108,117,101,97,42>>}],
               undefined,undefined}}
24> 
24> 
24> rp(binary_to_term(riakc_obj:get_value(NewestObj3))).
{value,42}
ok
25> 
```

---


### 从 Riak 中删除对象

Nothing is complete without a delete, as they say. Fortunately, that's easy too.

```erlang
riakc_pb_socket:delete(Pid, MyBucket, <<"one">>).
riakc_pb_socket:delete(Pid, MyBucket, <<"two">>).
riakc_pb_socket:delete(Pid, MyBucket, <<"three">>).
```

Now we can verify that the objects have been removed from Riak.

```erlang
{error,notfound} =:= riakc_pb_socket:get(Pid, MyBucket, <<"one">>).
{error,notfound} =:= riakc_pb_socket:get(Pid, MyBucket, <<"two">>).
{error,notfound} =:= riakc_pb_socket:get(Pid, MyBucket, <<"three">>).
```

---

实验结果

```
25> 
25> riakc_pb_socket:delete(Pid, MyBucket, <<"one">>).
ok
26> riakc_pb_socket:delete(Pid, MyBucket, <<"two">>).
ok
27> riakc_pb_socket:delete(Pid, MyBucket, <<"three">>).
ok
28> 
28> riakc_pb_socket:get(Pid, MyBucket, <<"one">>).
{error,notfound}
29> 
29> riakc_pb_socket:get(Pid, MyBucket, <<"two">>).
{error,notfound}
30> 
30> riakc_pb_socket:get(Pid, MyBucket, <<"three">>).
{error,notfound}
31> 
```

---


### 复杂对象操作

*Since the world is a little more complicated than simple integers and bits of strings, let’s see how we can work with more complex objects. Take, for example, this record that encapsulates some information about a book.*    
创建一个关于 book 的 record 

```erlang
rd(book, {title, author, body, isbn, copies_owned}).
MobyDickBook = #book{title="Moby Dick",
                     isbn="1111979723",
                     author="Herman Melville",
                     body="Call me Ishmael. Some years ago...",
                     copies_owned=3}.
```

*So we have some information about our Moby Dick collection that we want to save. Storing this to Riak should look familiar by now:*    
这里使用了 book 的 isbn 号作为 key

```erlang
MobyObj = riakc_obj:new(<<"books">>,
                        list_to_binary(MobyDickBook#book.isbn),
                        MobyDickBook).
riakc_pb_socket:put(Pid, MobyObj).
```

*Some of you may be thinking: “How does the Erlang Riak client encode/decode my object?” If we fetch our book back and print the value, we shall know:*    
开始思考 Riak 如何针对存储对象进行编解码问题！！

```erlang
{ok, FetchedBook} = riakc_pb_socket:get(Pid,
                                        <<"books">>,
                                        <<"1111979723">>).
rp(riakc_obj:get_value(FetchedBook)).
```

The response:

```erlang
<<131,104,6,100,0,4,98,111,111,107,107,0,9,77,111,98,121,
  32,68,105,99,107,107,0,15,72,101,114,109,97,110,32,77,
  101,108,118,105,108,108,101,107,0,34,67,97,108,108,32,
  109,101,32,73,115,104,109,97,101,108,46,32,83,111,109,
  101,32,121,101,97,114,115,32,97,103,111,46,46,46,107,0,
  10,49,49,49,49,57,55,57,55,50,51,97,3>>
```

这里可以看到，Riak Erlang 客户端库会把所有内容都以二进制数据形式进行编码。    
若我们想要得到 book 对象的友好显示，需要使用 binary_to_term/1 进行数据处理。    

```erlang
rp(binary_to_term(riakc_obj:get_value(FetchedBook))).
```

好，让我们将一切尘归尘土归土。

```erlang
riakc_pb_socket:delete(Pid, <<"books">>, <<"1111979723">>).
riakc_pb_socket:stop(Pid).
```

---

实验结果

```erlang
31> 
31> rd(book, {title, author, body, isbn, copies_owned}).
book
32> MobyDickBook = #book{title="Moby Dick",
32>                      isbn="1111979723",
32>                      author="Herman Melville",
32>                      body="Call me Ishmael. Some years ago...",
32>                      copies_owned=3}.
#book{title = "Moby Dick",author = "Herman Melville",
      body = "Call me Ishmael. Some years ago...",
      isbn = "1111979723",copies_owned = 3}
33> 
33> 
33> 
33> MobyObj = riakc_obj:new(<<"books">>,
33>                         list_to_binary(MobyDickBook#book.isbn),
33>                         MobyDickBook).
{riakc_obj,<<"books">>,<<"1111979723">>,undefined,[],
           undefined,
           #book{title = "Moby Dick",author = "Herman Melville",
                 body = "Call me Ishmael. Some years ago...",
                 isbn = "1111979723",copies_owned = 3}}
34> riakc_pb_socket:put(Pid, MobyObj).
ok
35> 
35> 
35> 
35> 
35> {ok, FetchedBook} = riakc_pb_socket:get(Pid,
35>                                         <<"books">>,
35>                                         <<"1111979723">>).
{ok,{riakc_obj,<<"books">>,<<"1111979723">>,
               <<107,206,97,96,96,96,204,96,202,5,82,60,71,131,54,114,47,
                 127,112,154,9,34,148,...>>,
               [{{dict,3,16,16,8,80,48,
                       {[],[],[],[],[],[],[],[],[],[],[],[],...},
                       {{[],[],[],[],[],[],[],[],[],[],...}}},
                 <<131,104,6,100,0,4,98,111,111,107,107,0,9,77,111,98,
                   121,32,68,...>>}],
               undefined,undefined}}
36> rp(riakc_obj:get_value(FetchedBook)).
<<131,104,6,100,0,4,98,111,111,107,107,0,9,77,111,98,121,
  32,68,105,99,107,107,0,15,72,101,114,109,97,110,32,77,
  101,108,118,105,108,108,101,107,0,34,67,97,108,108,32,
  109,101,32,73,115,104,109,97,101,108,46,32,83,111,109,
  101,32,121,101,97,114,115,32,97,103,111,46,46,46,107,0,
  10,49,49,49,49,57,55,57,55,50,51,97,3>>
ok
37> 
37> 
37> rp(binary_to_term(riakc_obj:get_value(FetchedBook))).
#book{title = "Moby Dick",author = "Herman Melville",
      body = "Call me Ishmael. Some years ago...",
      isbn = "1111979723",copies_owned = 3}
ok
38> 
38> 
38> riakc_pb_socket:delete(Pid, <<"books">>, <<"1111979723">>).
ok
39> riakc_pb_socket:stop(Pid).
ok
40> 
```

---

### 下一步

More complex use cases can be composed from these initial create, read, update, and delete (CRUD) operations. In the next chapter we will look at how to store and query more complicated and interconnected data, such as documents.