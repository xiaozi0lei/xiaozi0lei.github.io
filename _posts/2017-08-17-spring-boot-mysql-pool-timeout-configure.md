---
layout: default
title:  "Spring Boot MySQL 解决连接池超时配置"
date:   2017-08-17 17:38:57 +0800
categories: mysql
---

最近和同事在做一个测试平台，使用的 Spring Boot 框架，用的 MySQL 数据库。功能完成后，运行过程总是存在第二天用户无法登录的现象，查看了日志文件知道是数据库连接池超时未做自动检测，重连机制。于是按照网上的说法，配置了 tomcat 连接池 dbcp 的参数，结果发现第二天无法登录的现象依旧。这期间尝试更改很多配置参数都没能解决问题。

昨天上线完性能自动化工具后，有了些许时间，重新研究这个问题。思路如下：

* 先更改了 MySQL 数据库的 wait_timeout 时间为 3 分钟，便于测试

* 连接到 MySQL 数据库，执行 `show processlist;` 查看当前的连接数

* 尝试按照网上多篇文章的连接池设置参数，均无效

* 我的环境是多数据源配置，可能要把连接池配置单独挪到 MySQL 数据源下，也无效

* 最终解决问题是受网上一位朋友的配置启发，我去掉了 IDE 自动提示功能连接池设置的前缀 tomcat，直接把连接池配置参数挂在 MySQL 数据源下，这样之后才解决了问题。



说明网上提供的连接池配置是对的，只是我根据 IDE 的自动提示功能多加了一层，造成 Spring 无法自动配置连接池的参数，参数未生效。

application.yml 配置如下：
```yml
spring:
  datasource:
    sqlServer:
      url: jdbc:sqlserver://x.x.x.x\SERVER14:51103
      username: xx
      password: xxxxxx
      driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
    mysql:
      # 此处千万别加 tomcat
      url: jdbc:mysql://x.x.x.x:3306/test?characterEncoding=UTF-8&useSSL=false
      username: xx
      password: xxxxxx
      driver-class-name: com.mysql.jdbc.Driver
      max-active: 30
      # 从池中取连接的最大等待时间，单位ms
      max-wait: 10000
      # 最大空闲连接
      max-idle: 30
      # 最小空闲连接
      min-idle: 10
      # 初始化连接数量
      initial-size: 10
      # 验证使用的SQL语句
      validation-query: "select 1"
      # 指明连接是否被空闲连接回收器(如果有)进行检验.如果检测失败,则连接将被从池中去除
      test-while-idle: true
      # 借出连接时不要测试，否则很影响性能
      test-on-borrow: false
      # 每30秒运行一次空闲连接回收器
      time-between-eviction-runs-millis: 30000
      # 池中的连接空闲30分钟后被回收,默认值就是30分钟
      min-evictable-idle-time-millis: 1800000
      # 在每次空闲连接回收器线程(如果有)运行时检查的连接数量
      num-tests-per-eviction-run: 10
      # 连接泄漏回收参数，当可用连接数少于3个时才执行
      remove-abandoned: true
      # 连接泄漏回收参数，180秒，泄露的连接可以被删除的超时值
      remove-abandoned-timeout: 180
```
