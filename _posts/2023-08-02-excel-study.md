---
layout: default
title:  "Excel学习总结"
date:   2023-08-02 10:30:50 +0800
categories: study
---

# 飞书表格函数学习

* 统计非空单元格个数

```excel
# 第一个参数是sheet的名称，叹号后面是统计的单元格范围
=COUNTA('高风险查询统计'!B2:B206)
```

* 对指定区域中符合指定条件的[单元格](https://baike.baidu.com/item/单元格/2825816?fromModule=lemma_inlink)计数

```excel
# 第一个参数是sheet的名称，叹号后面是统计的单号元歌范围，第二个参数是统计的单元格等于的值
# 本例统计已完成的单元格的数量
=COUNTIF('高风险作业监控'!F2:F200, "已完成")
```

* 1

# Excel原生函数学习

