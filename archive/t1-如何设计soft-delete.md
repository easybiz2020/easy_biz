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
           表T [主键(ID)，某业务唯一编码（UNIQUE_KEY），删除标记(DEL_S)，其他省略]
           DEL_S: 删除标记,0-未删除
           
问题：update T set DEL_S=删除 where ID=${ID}，多次删除 UNIQUE_KEY 冲突了
如何设计一个合理的软删除/逻辑删除？
```



## 解决方案

来自@[siu91](https://github.com/siu91) ，以下两个方案与@[drj_2020](https://github.com/drj_2020)的想法是一致的

```txt
方案A:UNIQUE_KEY与DEL_S联合唯一 + 使用序列方式更新DEL_S（或是其它类型的DEL_S）
1）应用层实现方式
  a）JPA（Hibernate）
     @SQLDelete(sql = "UPDATE T SET DEL_S = nextval( 'seq' ) WHERE ID = ?")
     @Where(clause = "DEL_S = 0")
     
  b）Mybatis-plus官方未支持这种方式（待确认），官方方式见【附3】
    在Mybatis-plus github 上找到两个issues，有人提出了问题和解决思路
    1、https://github.com/baomidou/mybatis-plus/issues/1750
    2、https://github.com/baomidou/mybatis-plus/issues/1386
    
方案B：UNIQUE INDEX WHERE CAUSE 方式
1）数据库实现
   a）Postgres：
      CREATE UNIQUE index UNIQUE_KEY_unq on T(UNIQUE_KEY) where DEL_S = 0;
      参见：【附4】【附5】
   b）Oracle 12c 支持Partial Indexes（未验证是否满足需求），见【附6】
   b）其他数据库实现方式暂未查找

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

来自@[drj_2020](https://github.com/drj_2020)
```txt
1.创建联合索引，将UNIQUE_KEY与DEL_S创建为联合索引
（1）创建删除序列，队列从1开始，未删除数据删除标记(DEL_S)设为0，删除数据删除标记(DEL_S)设为取序列值。
（2）删除字段采用int8,未删除的还是标记未0，删除的标记(DEL_S)数据取时间戳。

2.对于pg等数据库，可以创建带条件的联合索引
  create unique index idx_ey_unq on t1(UNIQUE_KEY) where DEL_S = 0;


```

来自@[lcs559](https://github.com/lcs559)

```txt
系统数据量比较大的场景下
建议设计：增加备份表（删除记录表）（附录2）
操作方式：
1. 在原表表空间（数据库）下，新增加1张表，表名为原表名+backup。
2. 复制原表的所有字段定义。
3. 新增自增长Id字段作为新表的Id。
4. 新增deleteTime字段。（假设原表中没有deleteTime字段。）
5. 删除复制于原表Id及其他字段的唯一性约束。

使用方式：
当删除数据时，在1个事务里，同时将数据从原表中删除，并插入到新表中。其他操作与物理删除时的设计，保持不变。

使用好处：
1. 除删除操作外，其他操作与物理删除时的设计完全一致，客户端代码简单。（因为对于原表确实是物理删除。）
2. 业务Id可以作为主键，不用建立普通索引，查询更快。
3. 查询时不用过滤被逻辑删除的数据，查询更快。
4. 如果觉得逻辑删除没有意义，想恢复物理删除。只需要修改删除逻辑即可，其他地方没有任何影响。

缺点：需要逻辑删除的数据都要有对应备份表。
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
- [附2-「mwhgmwhg」的原创文章-关于数据库表字段逻辑删除设计的思考](https://blog.csdn.net/mwhgmwhg/article/details/84927037)
- [附3-MyBatis-Plus 逻辑删除](https://mp.baomidou.com/guide/logic-delete.html)
- [附4-Postgres部分索引](https://www.postgresql.org/docs/current/indexes-partial.html)
- [附5-stackoverflow：postgresql-conditionally-unique-constraint](https://stackoverflow.com/questions/16236365/postgresql-conditionally-unique-constraint)
- [附6-oracle 12c partial indexes](https://docs.oracle.com/database/121/VLDBG/GUID-256BA7EE-BF49-42DE-9B38-CD2480A73129.htm)
