---
layout: post
title: "监控——我是这么玩"
description: 
image: 
category: 'blog'
tags:
- grafana
- 日志
- webhook
---

监控的大致架构

源数据——>数据收集——>数据建模——>数据展示——>告警通知 ------> 源数据链

#### 用到的技术
- 数据源
    - 系统监控：日志（ERROR）
    - 业务监控：业务监控规则
- 数据收集
    - kafka，logstash，loghub
- 数据建模
    - elasticsearch
- 数据展示
    - grafana，kibana
- 告警通知
    - webhook，wxAPI，短信api





-----
