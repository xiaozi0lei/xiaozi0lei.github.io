---
layout: default
title:  "Rest-Assured 断言方法总结"
date:   2023-07-31 20:16:50 +0800
categories: testing
---

**断言方法**

```java
// everyItem(): 对断言对象中的 list 或者数组的属性值，进行遍历判断是否都等于 XXX
everyItem(equalTo("XXX"));
// hasItem(): 对断言对象中的 list 或者数组的属性值，包含指定的值
hasItem("XXX");
// equalTo(): 判断属性值与预期结果相等
equalTo("XXX");

```

**初始化配置**

```java
// 有时候需要把一些请求头或者cookie进行统一处理，因为所有的请求都需要带上这些参数，可以使用 requestSpecification 来进行统一设置
RestAssured.requestSpecification = new RequestSpecBuilder().build().header("Authorization", "Bearer " + token);
```



