---
title: MySQL - InnoDB 和 MyISAM 的区别
author:
  name: Jugg Wang
  link: https://github.com/kekaiwang
date: 2022-01-19 20:30:00 +0800
categories: [MySQL, InnoDB]
tags: [InnoDB, MyISAM]
render_with_liquid: false
---

## InnoDB 和 MyISAM 的区别

### 定义

- InnoDB：MySQL 默认的存储引擎，是一个平衡了可靠性和高性能的通用存储引擎。从 MySQL-5.5 开始做为默认存储引擎。
- MyISAM：在 MySQL-5.1 及之前的版本，MyISAM 是默认引擎。因为它不支持事务和行锁并且崩溃后无法恢复导致了它的没落，当然还是有很多优点的。

## 区别

### 事务

InnoDB 支持事物，MyISAM 不支持

### 索引

- InnoDB 使用 B+ 树聚簇索引，MyISAM 非聚簇索引

### 全文索引

InnoDB 5.6 以后开始支持全文索引，MyISAM 一直支持全文索引

### 锁

InnoDB 支持行锁、表锁，MyISAM 只支持表锁

### 外键

InnoDB 支持，MyISAM 不支持

### 表行数

InnoDB 没有保存，MyISAM 单独存储了，使用 `count(*)` 可以直接返回行数

### 唯一主键

InnoDB 必须要有唯一主键，MyISAM 可以没有

> InnoDB 当没有主键时会选择唯一索引做为主键，再会自己使用隐藏列 row_id 做为主键

### 文件存储

InnoDB 数据和索引的文件存储在 `*.ibd` 的文件中，表结构是存在以 `*.frm` 为后缀的文件里，MyISAM 的索引和文件是分开的，`*.MYD` 存储表的数据、`*.MYI` 存储表索引，`*.frm` 是表结构文件

### 总结

- InnoDB 支持事务，外键，并发量大，适合大量的 update 操作
- MyISAM 查询数据快，适合 select，不支持事务，并发量小，也没有 crash-safe
