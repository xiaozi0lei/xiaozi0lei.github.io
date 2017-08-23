---
layout: default
title:  "JMeter InfluxDB Grafana 集成实时监控压测"
date:   2017-08-23 15:26:57 +0800
categories: jmeter
---

**参考**  
http://jmeter.apache.org/usermanual/realtime-results.html
http://www.testautomationguru.com/jmeter-real-time-results-influxdb-grafana/
http://jmeter.apache.org/usermanual/component_reference.html#Backend_Listener

最近这段时间一直在做一个自动化的性能测试工具，用到的技术主要包含了这几个方面：
* SpringMVC
* MyBatis
* MySQL
* RabbitMQ
* JMeter
* InfluxDB
* Grafana

***

* 简单的架构就是前端页面提交表单，写入数据库，成功后写入 MQ 队列，这是生产者
* 后端消费程序一直监听指定的 queue ，获取队列里的 id ，查询数据库取出数据，调用 JMeter API 执行性能测试，测试完成后更新数据库信息，将测试结果信息写入数据库供前端页面展示
* 在此之后，鉴于 Jmeter 可以实时查看压测结果，参考官网和网络文章配置了一个实时展示自动化压测结果的监控页面，方便压测人员在压测过程中实时查看数据

**期间遇到几个问题记录一下：**  
* MySQL 数据库连接8小时断开，连接池中的连接失效
    - 这个问题真的是解决了好久，以至于解决完了我还专门去 debug 了 Spring Autowired 的源码
    - 我的问题出在数据源配置格式不对，在上篇文章已经介绍了问题所在，[yaml 语法规则是有要求的](http://www.ruanyifeng.com/blog/2016/07/yaml.html?f=tt)
    - 我多写了一层 tomcat ，导致 Spring 取不到连接池的参数配置
    - 当跟别的数据源同级时，Spring 认为 tomcat 是一个数据源的别名
* MyBatis 映射 Java 实体类和数据库表
    - 最后发现是之前我使用的时候，在 MyBatis 的配置文件中加了驼峰、下划线自动转换的配置，这次是通过 Java 配置的方式，所以没有加自动转换的配置，造成总是取不到数据
    - 在 mapper xml 文件中做了 resultMap 的手动映射解决这个问题
* JMeter API 调用
    - JMeter 的社区貌似是推荐使用 JMeter GUI 的，所以直接调用 API 进行测试的例子很少，国内这么做的人也很少，只在 Google 上找到几个老外的帖子参考，费了九牛二虎之力最终调通了，其中有个 `loopController.setLoops(-1);` ，这个参数一定要设定为-1，不然 duration 无法生效，这个是通过第六感试出来的...
* JMeter 配置 InfluxDB 无法写入数据
    - 这个完全是犯了低级错误，忘记打开防火墙 iptables，放行 InfluxDB 的 8086 端口
    - 还有个低级的错误，忘记修改 InfluxDB 配置文件，安装好后默认是关闭 http 8086 端口的连接方式

**希望自己以后再仔细一些吧**
