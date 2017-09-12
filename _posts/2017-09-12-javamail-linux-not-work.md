---
layout: default
title:  "JavaMail Linux 下无法发送邮件"
date:   2017-09-12 09:34:57 +0800
categories: java
---

在做设备管理的一个小系统时用到 JavaMail 来发送邮件通知，在 windows 下调试没有问题，但是上了生产环境 Linux 系统就无法发送邮件，总是提示 wait client timeout。

调试的过程很费神，下面给大家一个思路，希望遇到类似问题能帮助大家找到问题。

* 第一步想到的是打开日志输出，我用的 SpringBoot 框架，相应的在配置文件中如下配置：
    ```yml
    logging:
      level:
        # 调试 javamail
        com.sun.mail: debug
        javax.mail: debug
    ```
    打开了 javamail 的日志后，只看到简单的邮件发送过程，没有详细的有用信息
* 第二步进行 Google 和 百度，答案大都差不多，看到一篇文章不错，了解邮件发送的整个过程 - [邮件原理与JavaMail开发(一)——邮件的发送与接收原理](http://blog.csdn.net/yerenyuan_pku/article/details/52608770)，按照文章直接在 linux 下通过 telnet 发送邮件是可以成功的，所以问题应该在 javamail 或者项目配置上
* 查询官方网站的 FAQ，有 debug 的方法，打开 javamail session 的 debug 开关，可以打印详细的邮件协议相关的日志信息
* 由于 SpringBoot 封装了一层，所以使用如下代码获取 javamail 的 session，设置为 true 进行调试
    ```java
    JavaMailSenderImpl sender = new JavaMailSenderImpl();
    // 调试 mail linux 服务器
    sender.getSession().setDebug(true);
    sender.setHost("mail.test.com");
    sender.setDefaultEncoding("UTF-8");
    sender.setPort(25);
    sender.setProtocol("smtp");
    ```
    打开该调试开关后，控制台将打印出邮件发送过程的所有信息，同直接用 telnet 发送邮件的过程是一样的。到此我发现问题是 javamail 在发送邮件的 body 时，就 hang 住了，邮件服务端在 timeout 时间后没收到客户端的命令，所以返回了 wait client timeout 的错误信息。
* 邮件协议的规定是在 body 书写完毕后，需要发送"\<CRLF\>.\<CRLF\>"字符给服务器，确认邮件发送完成。

* 个人推测，我的问题可能是
    1. windows 和 linux 下的换行符不一致，导致邮件模板中的换行符干扰了邮件发送命令；
    2. 邮件模板中存在特殊字符，邮件程序解析出现问题 
* 我最终是重新书写了 html 模板，linux 发送邮件的问题解决了

**希望能帮助大家，给大家一个思路解决类似问题，谢谢！**
