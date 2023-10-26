---
layout: default
title: OLAP、OLTP、HTAP
parent: Other
---

# 1. OLTP(On-Line Transaction Processing)联机事务处理

事件驱动、面向应用，主要特性：

- 数据是应用系统产生的

- 每次处理的数据量很小

- 相应时间要求高

- 用户量大，并发度高

- 各种数据操作主要基于索引进行

# 2. OLAP(on-Line Analytic Processing)联机分析处理

- 主要用来分析处理数据仓库的数据，主要用来查询数据

- 数据来源是OLTP系统中的操作数据

- 查询的数据量大，而且会涉及到多表连接、全表扫描等复杂查询

- 相应时间与具体的查询有很大的关系

- 用户数量相对较小，并发度低，主要面向业务人员和管理人员

# 3. HTAP(Hybrid Transactional/Analytical Processing)混合事务和分析处理

既可以应用于事务型数据库场景，亦可以应用于分析型数据库场景