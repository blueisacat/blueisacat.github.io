---
layout: default
title: CRUD命名
parent: Spring
---

对于业务模块 xxx 而言， CRUD 命名如下：

- 分页：GetMapping("/xxx/page")

- 列表：GetMapping("/xxx/list")

- 获取：GetMapping("/xxx/{id}")

- 新增：PostMapping("/xxx")

- 修改：PutMapping("/xxx")

- 删除（逻辑）：GetMapping("/xxx/logical/{id}")

- 删除（物理）：GetMapping("/xxx/physical/{id}")
