---
title: TODO
tag: TODO
date: 2021-01-18 12:06:48
updated: 2021-01-18 12:06:48
categories: 
keywords: TODO
description: 日常todoList
---
# 2021-01-18
- [ ] 标签体系联动物料表更新、删除 [xdp-39](https://jira.datastory.com.cn/projects/XDP/issues/XDP-39)
- [ ] 熟悉海纳、工场源码，架构
- [ ] 过期标签清理问题，只能设置label_default下的字段对应值为null
- [ ] 删除标签后，label_default里面无法删除字段。字段无限膨胀问题。另： 一次性标签。
- [ ] 标签更新问题，当前逻辑是增对增量数据，每次打标签只会upsert，不会 删除旧标签，无法满足 分区全量的物料表场景
- [ ] 标签规则设置问题 如统一的p_date前置条件
- [ ] 实时标签？基于工场？（spark streaming）