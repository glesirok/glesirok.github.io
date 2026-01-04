---
title: SQL知识笔记
date: 2025-12-18 10:36 +0800
categories:
  - IT
  - SQL
tags:
  - SQL
comments: true
---

## 1. JOIN 和 WHERE

### 1.1 JOIN 执行后才执行WHERE吗？


实际执行：优化器会做谓词下推（Predicate Pushdown）
到扫描相关表时就过滤，而不是等 JOIN 完再过滤。

#### 1.1.1 ON vs WHERE
| 位置  | 行为                                                         |
|-------|--------------------------------------------------------------|
| ON    | 不满足条件时，右表字段返回 NULL，左表行保留                  |
| WHERE | 不满足条件时，整行被过滤掉（LEFT JOIN 变成 INNER JOIN 效果） |