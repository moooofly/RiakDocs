


# Querying 尝鲜

原文地址：[这里](http://docs.basho.com/riak/2.1.3/dev/taste-of-riak/querying/)

既然已经尝试过了 Riak  的 CRUD 接口，接下来我们可以针对数据的设计和查询进一步探索了，相关内容：[二级索引](http://docs.basho.com/riak/2.1.3/dev/using/2i/) and [key/value 操作](http://docs.basho.com/riak/2.1.3/dev/using/basics/) 。


### 配置变更

在进行下面的实验之前，首先需要对 Riak 实例的配置进行一下微调。

（针对 CentOS/RHEL 系列）配置文件所在路径：**/etc/riak/riak.conf**

Open the riak.conf file in your favorite text editor.

#### 测试二级索引需要使用 LevelDB 作为后端

将配置文件 riak.conf 中的 storage_backend 设置由 bitcask 变为 leveldb ；    
因为只有 [leveldb](http://docs.basho.com/riak/2.1.3/ops/advanced/backends/leveldb/) 支持二级索引；

保存配置信息后，重启 riak 节点，运行 ping 命令确认节点已经正常运行。

**注意**：如果是基于集群测试该功能，则需要为集群中的每个节点变更相应的配置信息后重启相应的节点。


### 关于查询（Querying）和模式（Schemas）的简单说明

即使在 key/value 存储的世界中，模式也并非一个“脏词”；    
逻辑数据库模式用于描述数据之间的关联；    
模式的使用就如同“跨多个 bucket 使用相同的 key 存储不同类型的 value ，并以此描述现实中的数据结构”一样简单（这些数据基于相同的 key 建立了关联）；    
查询方法的设计将会把你引入到“如何在 Riak 进行数据存储设计”的思考中；    

关于 key/value 模型更加全面的讨论可以参考：[这里](http://docs.basho.com/riak/2.1.3/dev/data-modeling/key-value/) 。


### 反范式（Denormalization）

*If you're coming from a relational database, the easiest way to get your application's feet wet with NoSQL is to denormalize your data into related chunks. For example, with a customer database, you might have separate tables for customers, addresses, preferences, etc. In Riak, you can denormalize all that associated data into a single object and store it into a Customer bucket. You can keep pulling in associated data until you hit one of the big denormalization walls:*    
如果你来自于关系型数据库阵营，令你的应用与 NoSQL “有染”的最简单方式是，将数据以反范式的方式保存成关联块；例如，针对 customer 数据库，你可以将表拆分成 customers, addresses, preferences 等；而在 Riak 中，你可以将关联数据反范式成为一个单独的对象，并将其保存到名为 Customer 的 bucket 中。    

你可以基于这种方式获取关联数据，直到遇到某座反范式的“高墙”：

- 大小限制问题（对象大小超过了 1MB）
- 共享/引用数据（即对象持有不属于自己的数据）
- 存在差别的访问模式（一次读/写模式 vs. 反复读写模式）

只要遇到上述任何一个问题，我们都将不得不进行模型拆分。

### 相同 key ，不同 Bucket

*The simplest way to split up data would be to use the same identity key across different buckets. A good example of this would be a Customer object, an Order object, and an OrderSummaries object that keeps rolled up info about orders such as total, etc.*    
最简单的数据拆分方式就是在不同的 bucket 中使用相同的 key 。

首先需要通过 Erlang 控制台构造一些数据。

```erlang
rd(customer, {customer_id, name, address, city, state, zip, phone, created_date}).
rd(item, {item_id, title, price}).
rd(order, {order_id, customer_id, salesperson_id, items, total, order_date}).
rd(order_summary_entry, {order_id, total, order_date}).
rd(order_summary, {customer_id, summaries}).

Customer = #customer{ customer_id= 1,
                      name= "John Smith",
                      address= "123 Main Street",
                      city= "Columbus",
                      state= "Ohio",
                      zip= "43210",
                      phone= "+1-614-555-5555",
                      created_date= {{2013,10,1},{14,30,26}}}.

Orders =  [ #order{
              order_id= 1,
              customer_id= 1,
              salesperson_id= 9000,
              items= [
                #item{
                  item_id= "TCV37GIT4NJ",
                  title= "USB 3.0 Coffee Warmer",
                  price= 15.99 },
                #item{
                  item_id= "PEG10BBF2PP",
                  title= "eTablet Pro, 24GB, Grey",
                  price= 399.99 }],
              total= 415.98,
              order_date= {{2013,10,1},{14,42,26}}},

            #order{
              order_id= 2,
              customer_id= 1,
              salesperson_id= 9001,
              items= [
                #item{
                  item_id= "OAX19XWN0QP",
                  title= "GoSlo Digital Camera",
                  price= 359.99 }],
              total= 359.99,
              order_date= {{2013,10,15},{16,43,16}}},

            #order {
              order_id= 3,
              customer_id= 1,
              salesperson_id= 9000,
              items= [
                #item{
                  item_id= "WYK12EPU5EZ",
                  title= "Call of Battle= Goats - Gamesphere 4",
                  price= 69.99 },
                #item{
                  item_id= "TJB84HAA8OA",
                  title= "Bricko Building Blocks",
                  price= 4.99 }],
              total= 74.98,
              order_date= {{2013,11,3},{17,45,28}}}
          ].

OrderSummary =  #order_summary{
                  customer_id= 1,
                  summaries= [
                    #order_summary_entry{
                      order_id= 1,
                      total= 415.98,
                      order_date= {{2013,10,1},{14,42,26}}
                    },
                    #order_summary_entry{
                      order_id= 2,
                      total= 359.99,
                      order_date= {{2013,10,15},{16,43,16}}
                    },
                    #order_summary_entry{
                      order_id= 3,
                      total= 74.98,
                      order_date= {{2013,11,3},{17,45,28}}}]}.

## Remember to replace the ip and port parameters with those that match your cluster.
{ok, Pid} = riakc_pb_socket:start_link("127.0.0.1", 10017).

CustomerBucket = <<"Customers">>.
OrderBucket = <<"Orders">>.
OrderSummariesBucket = <<"OrderSummaries">>.

CustObj = riakc_obj:new(CustomerBucket,
                        list_to_binary(
                          integer_to_list(
                            Customer#customer.customer_id)),
                        Customer).

riakc_pb_socket:put(Pid, CustObj).

StoreOrder = fun(Order) ->
  OrderObj = riakc_obj:new(OrderBucket,
                           list_to_binary(
                             integer_to_list(
                               Order#order.order_id)),
                           Order),
  riakc_pb_socket:put(Pid, OrderObj)
end.

lists:foreach(StoreOrder, Orders).


OrderSummaryObj = riakc_obj:new(OrderSummariesBucket,
                                list_to_binary(
                                  integer_to_list(
                                    OrderSummary#order_summary.customer_id)),
                                OrderSummary).

riakc_pb_socket:put(Pid, OrderSummaryObj).
```

---

实际结果

```shell
[root@Betty riak-erlang-client]# erl -pa ./ebin/ -pa ./deps/*/ebin
Erlang/OTP 17 [erts-6.0] [source] [64-bit] [smp:4:4] [async-threads:10] [hipe] [kernel-poll:false]

Eshell V6.0  (abort with ^G)
1> 
1> 
1> 
1> rd(customer, {customer_id, name, address, city, state, zip, phone, created_date}).
customer
2> rd(item, {item_id, title, price}).
item
3> rd(order, {order_id, customer_id, salesperson_id, items, total, order_date}).
order
4> rd(order_summary_entry, {order_id, total, order_date}).
order_summary_entry
5> rd(order_summary, {customer_id, summaries}).
order_summary
6> 
6> 
6> Customer = #customer{ customer_id= 1,
6>                       name= "John Smith",
6>                       address= "123 Main Street",
6>                       city= "Columbus",
6>                       state= "Ohio",
6>                       zip= "43210",
6>                       phone= "+1-614-555-5555",
6>                       created_date= {{2013,10,1},{14,30,26}}}.
#customer{customer_id = 1,name = "John Smith",
          address = "123 Main Street",city = "Columbus",
          state = "Ohio",zip = "43210",phone = "+1-614-555-5555",
          created_date = {{2013,10,1},{14,30,26}}}
7> 
7> 
7> Orders =  [ #order{
7>               order_id= 1,
7>               customer_id= 1,
7>               salesperson_id= 9000,
7>               items= [
7>                 #item{
7>                   item_id= "TCV37GIT4NJ",
7>                   title= "USB 3.0 Coffee Warmer",
7>                   price= 15.99 },
7>                 #item{
7>                   item_id= "PEG10BBF2PP",
7>                   title= "eTablet Pro, 24GB, Grey",
7>                   price= 399.99 }],
7>               total= 415.98,
7>               order_date= {{2013,10,1},{14,42,26}}},
7> 
7>             #order{
7>               order_id= 2,
7>               customer_id= 1,
7>               salesperson_id= 9001,
7>               items= [
7>                 #item{
7>                   item_id= "OAX19XWN0QP",
7>                   title= "GoSlo Digital Camera",
7>                   price= 359.99 }],
7>               total= 359.99,
7>               order_date= {{2013,10,15},{16,43,16}}},
7> 
7>             #order {
7>               order_id= 3,
7>               customer_id= 1,
7>               salesperson_id= 9000,
7>               items= [
7>                 #item{
7>                   item_id= "WYK12EPU5EZ",
7>                   title= "Call of Battle= Goats - Gamesphere 4",
7>                   price= 69.99 },
7>                 #item{
7>                   item_id= "TJB84HAA8OA",
7>                   title= "Bricko Building Blocks",
7>                   price= 4.99 }],
7>               total= 74.98,
7>               order_date= {{2013,11,3},{17,45,28}}}
7>           ].
[#order{order_id = 1,customer_id = 1,salesperson_id = 9000,
        items = [#item{item_id = "TCV37GIT4NJ",
                       title = "USB 3.0 Coffee Warmer",price = 15.99},
                 #item{item_id = "PEG10BBF2PP",
                       title = "eTablet Pro, 24GB, Grey",price = 399.99}],
        total = 415.98,
        order_date = {{2013,10,1},{14,42,26}}},
 #order{order_id = 2,customer_id = 1,salesperson_id = 9001,
        items = [#item{item_id = "OAX19XWN0QP",
                       title = "GoSlo Digital Camera",price = 359.99}],
        total = 359.99,
        order_date = {{2013,10,15},{16,43,16}}},
 #order{order_id = 3,customer_id = 1,salesperson_id = 9000,
        items = [#item{item_id = "WYK12EPU5EZ",
                       title = "Call of Battle= Goats - Gamesphere 4",
                       price = 69.99},
                 #item{item_id = "TJB84HAA8OA",
                       title = "Bricko Building Blocks",price = 4.99}],
        total = 74.98,
        order_date = {{2013,11,3},{17,45,28}}}]
8> 
8> 
8> 
8> OrderSummary =  #order_summary{
8>                   customer_id= 1,
8>                   summaries= [
8>                     #order_summary_entry{
8>                       order_id= 1,
8>                       total= 415.98,
8>                       order_date= {{2013,10,1},{14,42,26}}
8>                     },
8>                     #order_summary_entry{
8>                       order_id= 2,
8>                       total= 359.99,
8>                       order_date= {{2013,10,15},{16,43,16}}
8>                     },
8>                     #order_summary_entry{
8>                       order_id= 3,
8>                       total= 74.98,
8>                       order_date= {{2013,11,3},{17,45,28}}}]}.
#order_summary{customer_id = 1,
               summaries = [#order_summary_entry{order_id = 1,
                                                 total = 415.98,
                                                 order_date = {{2013,10,1},{14,42,26}}},
                            #order_summary_entry{order_id = 2,total = 359.99,
                                                 order_date = {{2013,10,15},{16,43,16}}},
                            #order_summary_entry{order_id = 3,total = 74.98,
                                                 order_date = {{2013,11,3},{17,45,28}}}]}
9> 
9> 
9> 
9> 
9> {ok, Pid} = riakc_pb_socket:start_link("127.0.0.1", 10017).
{ok,<0.43.0>}
10> 
10> 
10> CustomerBucket = <<"Customers">>.
<<"Customers">>
11> OrderBucket = <<"Orders">>.
<<"Orders">>
12> OrderSummariesBucket = <<"OrderSummaries">>.
<<"OrderSummaries">>
13> 
13> 
13> CustObj = riakc_obj:new(CustomerBucket,
13>                         list_to_binary(
13>                           integer_to_list(
13>                             Customer#customer.customer_id)),
13>                         Customer).
{riakc_obj,<<"Customers">>,<<"1">>,undefined,[],undefined,
           #customer{customer_id = 1,name = "John Smith",
                     address = "123 Main Street",city = "Columbus",
                     state = "Ohio",zip = "43210",phone = "+1-614-555-5555",
                     created_date = {{2013,10,1},{14,30,26}}}}
14> 
14> 
14> 
14> 
14> riakc_pb_socket:put(Pid, CustObj).
ok
15> 
15> 
15> StoreOrder = fun(Order) ->
15>   OrderObj = riakc_obj:new(OrderBucket,
15>                            list_to_binary(
15>                              integer_to_list(
15>                                Order#order.order_id)),
15>                            Order),
15>   riakc_pb_socket:put(Pid, OrderObj)
15> end.
#Fun<erl_eval.6.106461118>
16> 
16> 
16> 
16> lists:foreach(StoreOrder, Orders).
ok
17> 
17> 
17> 
17> OrderSummaryObj = riakc_obj:new(OrderSummariesBucket,
17>                                 list_to_binary(
17>                                   integer_to_list(
17>                                     OrderSummary#order_summary.customer_id)),
17>                                 OrderSummary).
{riakc_obj,<<"OrderSummaries">>,<<"1">>,undefined,[],
           undefined,
           #order_summary{customer_id = 1,
                          summaries = [#order_summary_entry{order_id = 1,
                                                            total = 415.98,
                                                            order_date = {{2013,10,1},{14,42,26}}},
                                       #order_summary_entry{order_id = 2,total = 359.99,
                                                            order_date = {{2013,10,15},{16,43,16}}},
                                       #order_summary_entry{order_id = 3,total = 74.98,
                                                            order_date = {{2013,11,3},{17,45,28}}}]}}
18> 
18> 
18> riakc_pb_socket:put(Pid, OrderSummaryObj).
ok
19> 
19> 
```

---

*While individual Customer and Order objects don't change much (or shouldn't change), the OrderSummaries object will likely change often. It will do double duty by acting as an index for all a customer's orders, and also holding some relevant data such as the order total, etc. If we showed this information in our application often, it's only one extra request to get all the info.*     
尽管单独的 Customer 和 Order 对象内容没有太多变化（或说不应该变化），但 OrderSummaries 对象却经常发生变化。    
OrderSummaries 对象做了双份工作：既充当了 customer 的 order 的索引；又维护了一些相关数据，例如总的 order 等。    
如果需要在我们的应用中经常展示这些信息，只需要一次额外的请求就可以获得。    

```erlang
{ok, FetchedCustomer} = riakc_pb_socket:get(Pid,
                                            CustomerBucket,
                                            <<"1">>).
{ok, FetchedSummary} = riakc_pb_socket:get(Pid,
                                           OrderSummariesBucket,
                                           <<"1">>).
rp({binary_to_term(riakc_obj:get_value(FetchedCustomer)),
    binary_to_term(riakc_obj:get_value(FetchedSummary))}).
```

执行后，可以得到联合对象：

```erlang
{#customer{customer_id = 1,name = "John Smith",
           address = "123 Main Street",city = "Columbus",
           state = "Ohio",zip = "43210",phone = "+1-614-555-5555",
           created_date = {{2013,10,1},{14,30,26}}},
 #order_summary{customer_id = 1,
                summaries = [#order_summary_entry{order_id = 1,
                                                  total = 415.98,
                                                  order_date = {{2013,10,1},{14,42,26}}},
                             #order_summary_entry{order_id = 2,total = 359.99,
                                                  order_date = {{2013,10,15},{16,43,16}}},
                             #order_summary_entry{order_id = 3,total = 74.98,
                                                  order_date = {{2013,11,3},{17,45,28}}}]}}
```

---

实际操作

```erlang
19> 
19> {ok, FetchedCustomer} = riakc_pb_socket:get(Pid,
19>                                             CustomerBucket,
19>                                             <<"1">>).
{ok,{riakc_obj,<<"Customers">>,<<"1">>,
               <<107,206,97,96,96,96,204,96,202,5,82,60,71,131,54,114,47,
                 127,112,250,33,144,205,...>>,
               [{{dict,3,16,16,8,80,48,
                       {[],[],[],[],[],[],[],[],[],[],[],[],...},
                       {{[],[],[],[],[],[],[],[],[],[],...}}},
                 <<131,104,9,100,0,8,99,117,115,116,111,109,101,114,97,1,
                   107,0,10,...>>}],
               undefined,undefined}}
20> {ok, FetchedSummary} = riakc_pb_socket:get(Pid,
20>                                            OrderSummariesBucket,
20>                                            <<"1">>).
{ok,{riakc_obj,<<"OrderSummaries">>,<<"1">>,
               <<107,206,97,96,96,96,204,96,202,5,82,60,91,88,101,61,57,
                 183,44,79,135,8,37,...>>,
               [{{dict,3,16,16,8,80,48,
                       {[],[],[],[],[],[],[],[],[],[],[],[],...},
                       {{[],[],[],[],[],[],[],[],[],[],...}}},
                 <<131,104,3,100,0,13,111,114,100,101,114,95,115,117,109,
                   109,97,114,121,...>>}],
               undefined,undefined}}
21> rp({binary_to_term(riakc_obj:get_value(FetchedCustomer)),
21>     binary_to_term(riakc_obj:get_value(FetchedSummary))}).
{#customer{customer_id = 1,name = "John Smith",
           address = "123 Main Street",city = "Columbus",
           state = "Ohio",zip = "43210",phone = "+1-614-555-5555",
           created_date = {{2013,10,1},{14,30,26}}},
 #order_summary{customer_id = 1,
                summaries = [#order_summary_entry{order_id = 1,
                                                  total = 415.98,
                                                  order_date = {{2013,10,1},{14,42,26}}},
                             #order_summary_entry{order_id = 2,total = 359.99,
                                                  order_date = {{2013,10,15},{16,43,16}}},
                             #order_summary_entry{order_id = 3,total = 74.98,
                                                  order_date = {{2013,11,3},{17,45,28}}}]}}
ok
22> 
```

---


尽管这种操作方式相对于查询复杂性而言非常简单，并且速度非常快，但却需要应用自身了解这些数据的内在联系。


### 二级索引

如果你来自 SQL 的世界，就会发现二级索引(2i)和 SQL 索引非常相似；    
基于二级索引可以非常快速的查询出目标对象，而不需要扫描整个数据集；    
基于二级索引可以非常容易的查出包含特定值的、按组划分的关联数据，甚至可以针对一定范围的值进行查询。    

下面的例子用于展示二级索引的使用。

```erlang
FormatDate = fun(DateTime) ->
  {{Year, Month, Day}, {Hour, Min, Sec}} = DateTime,
  lists:concat([Year,Month,Day,Hour,Min,Sec])
end.

AddIndicesToOrder = fun(OrderKey) ->
  {ok, Order} = riakc_pb_socket:get(Pid, OrderBucket,
                                    list_to_binary(integer_to_list(OrderKey))),

  OrderData = binary_to_term(riakc_obj:get_value(Order)),
  OrderMetadata = riakc_obj:get_update_metadata(Order),

  MD1 = riakc_obj:set_secondary_index(OrderMetadata,
                                      [{{binary_index, "order_date"},
                                        [FormatDate(OrderData#order.order_date)]}]),

  MD2 = riakc_obj:set_secondary_index(MD1,
                                      [{{integer_index, "salesperson_id"},
                                        [OrderData#order.salesperson_id]}]),

  Order2 = riakc_obj:update_metadata(Order,MD2),
  riakc_pb_socket:put(Pid,Order2)
end.

lists:foreach(AddIndicesToOrder, [1,2,3]).
```

*As you may have noticed, ordinary Key/Value data is opaque to 2i, so we have to add entries to the indices at the application level. Now let's find all of Jane Appleseed's processed orders, we'll lookup the orders by searching the saleperson_id_int index for Jane's id of 9000.*
你可能会发现，普通的 Key/Value 数据对于 2i 来说是不透明的，所以我们不得不在应用级别上为 index 添加 entry 。现在，让我们查询...

```erlang
riakc_pb_socket:get_index_eq(Pid, OrderBucket, {integer_index, "salesperson_id"}, 9000).
```

Which returns:

```erlang
{ok,{index_results_v1,[<<"1">>,<<"3">>],
                      undefined,undefined}}
```

---

实验结果

在未变更 backend 的情况下（默认配置为 bitcask），会输出下面的错误

```erlang
25> 
25> riakc_pb_socket:get_index_eq(Pid, OrderBucket, {integer_index, "salesperson_id"}, 9000).
{error,<<"{error,{indexes_not_supported,riak_kv_bitcask_backend}}">>}
26> 
```

在变更 backend 的情况下（变为 leveldb），会输出下面结果

```erlang
25> 
25> riakc_pb_socket:get_index_eq(Pid, OrderBucket, {integer_index, "salesperson_id"}, 9000).
{ok,{index_results_v1,[<<"3">>,<<"1">>],
                      undefined,undefined}}
26> 
```

---


*Jane processed orders 1 and 3. We used an “integer” index to reference Jane's id, next let's use a “binary” index. Now, let's say that the VP of Sales wants to know how many orders came in during October 2013. In this case, we can exploit 2i's range queries. Let's search the order_date_bin index for entries between 20131001 and 20131031.*    
上面的操作使用了“整数”索引来引用 Jane 的 id ；    
下面我们将使用“二进制”索引；    

假设销售副经理想要知道在 2013 年 10 月有多少订单，则可以基于 2i 的范围查询得到；

```erlang
riakc_pb_socket:get_index_range(Pid, OrderBucket,
                                {binary_index, "order_date"},
                                <<"20131001">>, <<"20131031">>).
```

Which returns:

```erlang
{ok,{index_results_v1,[<<"1">>,<<"2">>],
                      undefined,undefined}}
```

小菜一碟，我们使用 2i 的范围索引特性搜索了一定范围内的值，并且验证了二进制索引功能。


下面回顾一下重点：

- 你可以使用二级索引快速查询基于 secondary id 构建的对象，而不用基于对象的 key
- 索引既可以是整数，也可以是二进制（字符串）
- 你可以针对特定值进行搜索，也可以针对范围值进行搜索
- Riak 将返回匹配 index 查询的 key 列表