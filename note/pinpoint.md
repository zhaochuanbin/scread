### 简介

pinpoint 是由韩国 naver（做搜索引擎的） 网站开源的一款 APM 软件

1、实现原理

在请求header中，增加Trace数据结构，TxId：请求的唯一编号；SpanId：处理请求的服务编号（provider）；pSpanId：调用者的SpanId（Consumer），
通过这一组编号来跟踪请求

![](image/pinpoint-1.png)
