Rest-Assured 断言方法总结

# 断言方法

```java
// everyItem(): 对断言对象中的 list 或者数组的属性值，进行遍历判断是否都等于 XXX
everyItem(equalTo("XXX"));
// hasItem(): 对断言对象中的 list 或者数组的属性值，包含指定的值
hasItem("XXX");
// equalTo(): 判断属性值与预期结果相等
equalTo("XXX");

```

