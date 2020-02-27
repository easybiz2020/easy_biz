# 如何设计软删除/逻辑删除

发布时间：25/02/2020 13:07

主题来自：[siu91](https://github.com/siu91)  

截稿时间：29/02/2020 21:00



## 背景

```txt
业务背景描述：
项目中遇到某些业务数据是不能物理删除的，必须用软删除的方式标记删除的数据。
PS：互联网企业收集的个人信息、电商类平台用户的订单、金融类平台的交易记录，肯定是不能删除

具体表设计举例：
           表T [主键(ID)，某业务唯一编码（UNIQUE_EY），删除标记(DEL_S)，其他省略]
           DEL_S: 删除标记,0-未删除
           
问题：update T set DEL_S=删除 where ID=${ID}，多次删除 UNIQUE_EY 冲突了
如何设计一个合理的软删除/逻辑删除？
```



## 解决方案

来自@[siu91](https://github.com/siu91) 

[样例代码](../demo/demo1.md)

```txt
xxxxx  
xxxx
xxxx
xxxx
xxxx
```

来自@[水哥](https://github.com/siu91) 

```txt
xxxxx
xxx
xxxx
xxx
```

来自@[Sev7nzy](https://github.com/Sev7nzy) 

```txt
1. 创建历史删除表
每个表新建一个历史删除表，存储已经删除的历史数据，缺点是大量的历史表。
改进方法：所有表共享一张历史删除表，历史表中存储数据库scheme和tableName信息，数据的信息通过json的形式存储在历史表中。

2. 添加delete_token字段
当某一条记录需要删除时，将该字段设置为一个UUID，将name、delete_token设置为唯一键，这样当is_delete=0时，delete_token保持一个默认值，能够有效地限制name唯一，当记录被删除时，由于delete_token是一个唯一的UUID，便能保证删除的记录不会被唯一约束束缚。（附1）

3. 使用多delete值
未删除数据删除标记(DEL_S)设为0，删除数据删除标记(DEL_S)设为非0的任何值。
改进方法:删除数据删除标记(DEL_S)使用删除时间戳来替代，同一秒时间，相同唯一字段可能性基本为零，使用初始值0或者Null来作为未删除标志符，会占用一定的存储空间，但可以显示删除时间。

4 删除设置为Null
将删除标记设置默认值(例如0)，将唯一字段与删除标记添加唯一键约束。当某一记录需要删除时，将删除标记置为NULL。（附1）
```

## 总结

```txt
xxxx 方案在xxx场景下是一个好的解决方法
xxxx
xxxx
xxxx
xxxx
```



## 附录

- [附1-梦想超人博客-数据库逻辑删除的解决方案探讨](https://blog.csdn.net/weixin_43379172/article/details/86743532)
- [附2-xxxx博客-关于如何设计xxxx的实践总结与思考](https://github.com/)
- [附3-xxxx博客-关于如何设计xxxx的实践总结与思考](https://github.com/)
