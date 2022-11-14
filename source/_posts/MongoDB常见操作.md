---
title: MongoDB常见操作
date: 2022-03-24 10:56:28
tags: 知识点
---
## 评价查询操作符

### $mod - 取模计算

**示例**

```
>db.c1.find()

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af89afa87dc1bed94331"),"age":8,"length_1":30}{"_id":ObjectId("4fb4af8cafa87dc1bed94332"),"age":6,"length_1":30}
```

查询age取模6等于1的数据，如下面的代码所示：

```
>db.c1.find({age:{$mod:[6,1]}})

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}
```

可以看出，只显示age取模6等于1的数据，其他不符合规则的数据并没有显示出来。



## 比较查询操作符

### $gt  - 大于

**示例**

```
>db.c1.find()

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af89afa87dc1bed94331"),"age":8,"length_1":30}{"_id":ObjectId("4fb4af8cafa87dc1bed94332"),"age":6,"length_1":30}
```

查询age的值不等于8的数据，如下面的代码所示：

```
>db.c1.find({age:{$gt:6}})

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af89afa87dc1bed94331"),"age":8,"length_1":30}
```

可以看出，只显示age大于6的数据，age等于6的数据没有显示出来。



### $lte - 小于等于

**示例**

```
>db.c1.find()

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af89afa87dc1bed94331"),"age":8,"length_1":30}{"_id":ObjectId("4fb4af8cafa87dc1bed94332"),"age":6,"length_1":30}
```

查询age的值不等于8的数据，如下面的代码所示：

```
>db.c1.find({age:{$lte:7}})

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af8cafa87dc1bed94332"),"age":6,"length_1":30}
```

可以看出，只显示age小于等于7的数据，age等于8的数据没有显示出来。



 ### $in - 包含

**示例**

```
>db.c1.find()

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af89afa87dc1bed94331"),"age":8,"length_1":30}{"_id":ObjectId("4fb4af8cafa87dc1bed94332"),"age":6,"length_1":30}
```

查询age的值在7、8范围内的数据，如下面的代码所示：

```
>db.c1.find({age:{$in:[7,8]}})

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af89afa87dc1bed94331"),"age":8,"length_1":30}
```

可以看出只显示age等于7或8的数据，其他不符合规则的数据并没有显示出来。

### $nin - 不包含

### $lt - 小于

### $gte - 大于等于

### $ne - 不等于

### $eq - 等于

**示例**

```
>db.c1.find()

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}{"_id":ObjectId("4fb4af89afa87dc1bed94331"),"age":8,"length_1":30}{"_id":ObjectId("4fb4af8cafa87dc1bed94332"),"age":6,"length_1":30}
```

查询age的值不等于8的数据，如下面的代码所示：

```
>db.c1.find({age:{$eq:7}})

结果：
{"_id":ObjectId("4fb4af85afa87dc1bed94330"),"age":7,"length_1":30}
```



## 数组更新操作符

### $pushAll - 用法同$push一样，只是$pushAll一次可以追加多个值到一个数组字段内。

**示例**

```
>db.t3.find()

结果：
{"_id":ObjectId("4fe67b008414d282f712fae6"),"userid":3,"name":["wangwenlong"]}
```

可以看到当前别名有1个，是"wangwenlong"，接下来将"N1"和"N2"名字加入到name字段数据组里，如下面的代码所示：

```
>db.t3.update({"userid":3},{$pushAll:{"name":["N1","N2"]}})
>db.t3.find()
结果：
{"_id":ObjectId("4fe67b008414d282f712fae6"),"name":["wangwenlong","N1","N2"],"userid":3}
```

可以看到更新后当前别名里又多了2个，分别是"N1"和"N2"。

### $pullAll - 用法同$pull一样，只是$pullAll可以一次删除数组内的多个值。

**注意：**  数组更新操作符只能更新数组，如果键值是乱序的会报错。["xiao ming", "xiao dong", "xiao hong "]正确，[0 => "xiao ming", 2 => "xiao dong", 5 =>  "xiao hong "]报错。



## 数组更新操作符

### $set - 设置某一个字段的值。

**示例**

```
>db.t3.find()

结果：
{"_id":ObjectId("4fe676348414d282f712fae4"),"name":"wangwenlong","age":35}
```

可以看到当前年龄是35岁，接下来将age调整为40，如下面的代码所示：

```
>db.t3.update({name:"wangwenlong"},{$set:{age:40}})
>db.t3.find()
结果：
{"_id":ObjectId("4fe676348414d282f712fae4"),"name":"wangwenlong","age":40}>
```

可以看到，更新后年龄从30变成了40。

更多内容，可以查看[mongoDB](https://www.mongodb.org.cn/manual/230.html)