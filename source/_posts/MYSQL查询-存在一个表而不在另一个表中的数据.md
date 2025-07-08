---
title: MYSQL查询~ 存在一个表而不在另一个表中的数据
date: 2021-05-18 19:39:28
tags:
- MySQL
- 数据库
- SQL
categories: 技术
---

>业务场景：

account表保存的用户信息 包含 真正用户 和 测试用户(内部测试账号),test表保存测试用户信息，对应account表id字段test.account_id。

查询时需要剔除account表测试用户，找出account表id字段中，不与test表account字段相等的值。

>方法一: 使用 not in ,容易理解,效率低  执行时间为：8.6ms(其他数据相同方法)
```
注：这边因为account_id字段具有唯一性，所以才不用distinct去重。

select account.id from account where account.id not in (select account_id from test)
```

>方法二：使用 left join...on... , "test.account_id is null" 表示左连接之后在test.account_id 字段为 null的记录  执行时间：9ms(其他数据相同方法)

```
select account.id from account left join test on account.id = test.account_id where test.account_id is null
``` 

>方法三: 逻辑相对复杂,但是速度最快  执行时间: 10.2ms(其他数据相同方法)

```
select * from account where (select count(1) as num from test where account.id = test.account_id) = 0

当account表获取的id数据作为test表查询条件(account.id = test.account_id)时，数据为0的account表数据。
```

**注：时间仅作为参考，还是要用实际数据模拟，语句可以用explain优化**

>参考链接
```
https://www.cnblogs.com/softidea/p/9482120.html
```