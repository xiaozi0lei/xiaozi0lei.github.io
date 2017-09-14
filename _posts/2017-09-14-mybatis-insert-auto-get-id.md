---
layout: default
title:  "MyBatis 进行 MySQL 数据库插入时自动获取主键 ID"
date:   2017-09-14 14:40:59 +0800
categories: mybatis
---

参考：  
http://blog.csdn.net/lzx1104/article/details/37760333  
http://chenzhou123520.iteye.com/blog/1849881

我们在实际的开发过程中，往往需要在插入数据后获得插入数据的主键ID进行后续的操作。如果只是用 JDBC 来进行操作的话，我们一般是插入数据后立即获取 ID 最大值，但是在高并发的场景，往往无法保证获取的 ID 就是自己插入数据的 ID。

使用 MyBatis 做持久层框架，MyBatis 自身已经提供了相应的功能，方便我们在插入数据时获取对应的主键值。

在 mapper.xml 文件中如下配置即可：
```xml
<insert id="insertUser" parameterType="com.xxx.User" useGeneratedKeys="true" keyProperty="userID">  
    insert into user_table(name,password) values(#{name},#{password})  
</insert>
```
或者
```xml
<insert id="insertUser" parameterType="com.xxx.User">  
    <selectKey keyProperty="userID" order="AFTER" resultType="int">  
      SELECT LAST_INSERT_ID() AS userID  
    </selectKey>  
    insert into user_table(name,password) values(#{name},#{password})  
</insert>
```

通过如上配置后，执行完插入操作后，User 对象实例的 userId 值就会自动设置上。  
另外注意，自增主键ID是通过设置传入的user对象的成员变量返回的，而不是通过接口函数的返回值返回，返回值表示插入记录数。