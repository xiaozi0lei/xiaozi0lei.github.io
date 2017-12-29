---
layout: default
title:  "服务端测试流程搭建探索"
date:   2017-12-29 09:53:50 +0800
categories: testing
---

* 一、构建 Web 界面进行提测规范
* 二、搭建 Jenkins Job 进行自动化部署
* 三、搭建 Jenkins Job 进行自动化接口测试
* 四、搭建远程调试服务端，进行服务端问题定位
* 五、搭建 k8s 作为 docker 容器编排系统
* 六、搭建 docker 容器进行业务测试版本隔离

**一、构建 Web 界面进行提测规范**
通过 SpringBoot 框架开发 Web 界面制作提测表单，规范开发提测信息
* SpringBoot
* MyBatis
* MySQL
* BootStrap

**二、搭建 Jenkins Job 进行自动化部署**
Web 表单提交后，发送 POST 请求到 Jenkins 服务器，携带部署的 git tag 号，部署的项目名称，部署对应的项目
* Jenkins
* Gitlab

**三、搭建 Jenkins Job 进行自动化接口测试**
项目部署完成后，调用接口自动化测试，执行接口自动化测试，将测试结果写入数据库中，测试完成后调用微信通知，邮件通知 RPC 服务发送通知给对应的测试、开发人员
* thrift
* springboot restful 接口

**四、搭建远程调试服务端，进行服务端问题定位**
自动部署过程中，服务端的启动脚本中 Java 参数添加支持远程调试功能的参数
本地搭建对应版本的服务器代码，在 idea 中设置远程调试，本地 idea 打断点，调用远端服务时触发本地断点，定位问题

**五、搭建 k8s 作为 docker 容器编排系统**
支持部署测试环境到轻量容器中，可以模拟线上环境，分布式，负载均衡等

**六、搭建 docker 容器进行业务测试版本隔离**
部署时通过 dockerfile 模板编译环境版本，提高多环境部署的效率
