---
layout: default
title:  "调用 Jmeter API 进行压力测试"
date:   2017-09-04 14:49:56 +0800
categories: jmeter
---

直接上调用 Jmeter API 进行压力测试的代码

```java
//JMeter Engine
StandardJMeterEngine jmeter = new StandardJMeterEngine();

// 本地 JMeter 的配置文件地址 JMeter initialization (properties, log levels, locale, etc)
JMeterUtils.loadJMeterProperties(PathConstant.JMETER_PROPERTIES.getValue());
// 本地 JMeter 目录
JMeterUtils.setJMeterHome(PathConstant.JMETER_HOME.getValue());
JMeterUtils.initLocale();
// JMeter 报告的地址
String jtlLogPath = PathConstant.JMETER_LOG.getJmeterLog();

// 支持压测实时监控
BackendListener backendListener = new BackendListener();
/*backendListener.setClassname("org.apache.jmeter.visualizers.backend.influxdb.InfluxdbBackendListenerClient");
Arguments args = new Arguments();
args.addArgument("influxdbMetricsSender", "org.apache.jmeter.visualizers.backend.influxdb.HttpMetricsSender");
args.addArgument("influxdbUrl", "http://x.x.x.x:8086/write?db=jmeter");
args.addArgument("application", "jmeter");
args.addArgument("measurement", "jmeter");
args.addArgument("summaryOnly", "true");
args.addArgument("samplersRegex", ".*");
args.addArgument("percentiles", "90;95;99");
args.addArgument("testTitle", "Test name");*/
backendListener.setClassname("org.apache.jmeter.visualizers.backend.graphite.GraphiteBackendListenerClient");
Arguments args = new Arguments();
args.addArgument("graphiteMetricsSender", "org.apache.jmeter.visualizers.backend.graphite.TextGraphiteMetricsSender");
args.addArgument("graphiteHost", "x.x.x.x");
args.addArgument("graphitePort", "2003");
args.addArgument("rootMetricsPrefix", "jmeter.");
args.addArgument("summaryOnly", "true");
args.addArgument("samplersList", "");
args.addArgument("percentiles", "90;95;99");
args.addArgument("useRegexpForSamplersList", "false");
backendListener.setArguments(args);

// Result collector
ResultCollector resultCollector = new ResultCollector();
resultCollector.setFilename(jtlLogPath);

SampleSaveConfiguration saveConfiguration = new SampleSaveConfiguration();
// 设置压测结果为 xml 格式，csv 的话就注释掉
saveConfiguration.setAsXml(true);
saveConfiguration.setCode(true);
saveConfiguration.setLatency(true);
resultCollector.setSaveConfig(saveConfiguration);

// HTTP Sampler
HTTPSampler httpSampler = new HTTPSampler();
// 压测主机地址
httpSampler.setDomain(url);
// 压测接口端口号
httpSampler.setPort(80);
// 压测接口地址
httpSampler.setPath("/");
// 压测请求方法类型
httpSampler.setMethod("GET");

// Thread Group
ThreadGroup threadGroup = new ThreadGroup();
// 线程数
threadGroup.setNumThreads(threadNum);
// 达到设定的并发数需要的时间，单位秒
threadGroup.setRampUp(1);
// 压测的总时长
threadGroup.setDuration((long) duration);
// 打开调度器开发，支持按时长压测
threadGroup.setScheduler(true);
threadGroup.setName("Group1");

// Loop Controller
LoopController loopController = new LoopController();
// 循环次数设置为 -1，可以通过 setDuration 来指定运行时间，这里是一个关键点
loopController.setLoops(-1);
threadGroup.setSamplerController(loopController);

HashTree threadGroupTree = new HashTree();
threadGroupTree.add(httpSampler);

// Test Plan
TestPlan testPlan = new TestPlan("Create JMeter Script From Java Code");

// JMeter Test Plan, basic all u JOrphan HashTree
HashTree testPlanTree = new HashTree();
// Construct Test Plan from previously initialized elements
testPlanTree.add(threadGroup, threadGroupTree);
testPlanTree.add(resultCollector);
testPlanTree.add(backendListener);

HashTree hashTree = new HashTree();
hashTree.add(testPlan, testPlanTree);

// Run Test Plan
jmeter.configure(hashTree);
logger.debug("JMeter 压测开始...");
jmeter.run();
logger.debug("JMeter 压测结束...");
```
