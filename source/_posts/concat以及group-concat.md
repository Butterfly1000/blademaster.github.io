---
title: concat以及group_concat
date: 2021-03-22 19:28:22
tags: mysql
---

>concat()函数

1.功能：将多个字符串连成一个字符串。

2.语法：concat(str1,str2,...)
返回结果为连接参数产生的字符串，如果有任何一个参数为NULL，则返回值为NULL。这边测试null，空白都没有效果，只有NULL有效。

3.举例:

| id | 姓 | 名 |
|-------- |------ |------ |
|1 |林 |晓明 |

```sql
select concat(id,first_name,last_name) as info from business_account_info where id=1
```
结果：1林晓明

添加分隔符
```sql
select concat(id,"."，first_name," ",last_name) as info from business_account_info where id=1
```
结果：1.林 晓明

<!--more-->

>concat_ws()函数

concat函数的分隔符虽然多样，但需要一个一个填。如果是分隔符众多且都一致的情况下就显得很麻烦了。函数concat_ws完美解决。
```sql
select concat_ws(",",id,first_name,last_name) as info from business_account_info where id=1
```
结果: 1,林，晓明

当然，如果把分隔符指定为NULL，结果全部变成了NULL,对concat函数、concat_ws都适用。

>group_concat()函数

这个就要复杂那么点。

```sql
select name,min(score) from t1 group by name;
```
这样我们可以看到小明、小东和茉茉的最低成绩:

|name | min(score) |
|-------- |------ |
|小明 | 17|
|小东 | 11|
|茉茉 | 15|


```sql
select name,score from t1 order by name;
```
这样我们可以看到，排序name，而score默认从大打小asc排序。这样也可以看到分数及最小分数。

|name | score |
|-------- |------ |
|茉茉 | 15|
|茉茉 | 18|
|茉茉 | 20|
|小东 | 11|
|小东 | 14|
|小东 | 16|
|小明 | 17|
|小明 | 20|
|小明 | 21|

但不够直观，这时候group_concat()函数就登场了。

语法：group_concat( \[distinct\] 要连接的字段 \[order by 排序字段 asc/desc \] \[separator '分隔符'\] )

说明：通过使用distinct可以排除重复值；如果希望对结果中的值进行排序，可以使用order by子句；separator是一个字符串值，缺省为一个逗号。

```sql
select name,group_concat(score) from t1 group by name;
```

[参考](https://baijiahao.baidu.com/s?id=1595349117525189591&wfr=spider&for=pc)
