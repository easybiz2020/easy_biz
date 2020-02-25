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
如何设计一个合理的软删除/删除？
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



## 总结

```txt
xxxx 方案在xxx场景下是一个好的解决方法
xxxx
xxxx
xxxx
xxxx
```



## 附录

- [附1-xxxx博客-关于如何设计xxxx的实践总结与思考](https://github.com/)
- [附2-xxxx博客-关于如何设计xxxx的实践总结与思考](https://github.com/)
- [附3-xxxx博客-关于如何设计xxxx的实践总结与思考](https://github.com/)
