# Week2：Hive基础

## 本周目标
- 在CentOS 7上跑通Hive环境
- 掌握HiveQL的核心操作：建库、建表、加载数据、查询
- 理解Hive的核心概念：内部表 vs 外部表、分区表、ORDER BY vs SORT BY

## 环境信息
- OS：CentOS 7（已预装Hadoop + Hive）
- 访问方式：SSH登录后执行 `hive` 命令

## 完成清单
- [ ] SSH登录CentOS 7
- [ ] 进入Hive命令行（`hive`）
- [ ] 创建数据库：`create database if not exists my_ai_data;`
- [ ] 创建内部表（`CREATE TABLE`）
- [ ] 创建外部表（`CREATE EXTERNAL TABLE`）
- [ ] 加载本地数据（`LOAD DATA LOCAL INPATH`）
- [ ] 执行查询（`SELECT`、`GROUP BY`、`JOIN`）
- [ ] 体验 `ORDER BY` vs `SORT BY` 的区别
- [ ] 创建分区表（`PARTITIONED BY`）
- [ ] 整理Hive vs Oracle差异笔记

## 核心命令速查
| 操作 | 命令 |
|------|------|
| 进入Hive | `hive` |
| 查看所有数据库 | `show databases;` |
| 使用数据库 | `use database_name;` |
| 查看所有表 | `show tables;` |
| 查看表结构 | `desc table_name;` |
| 加载本地数据 | `load data local inpath '路径' into table 表名;` |
| 退出Hive | `quit;` |

## 面试考点记录
- [ ] 内部表和外部表的区别（能说清楚并演示）
- [ ] ORDER BY 和 SORT BY 的区别
- [ ] 分区表的作用和用法
- [ ] HiveQL和SQL的3个主要差异

## 遇到的问题及解决
（本周执行过程中遇到问题后填写）

| 问题 | 解决方案 |
|------|----------|
| | |

## 文件说明
- `create_table.sql` - 建表脚本（内部表、外部表、分区表）
- `load_data.sql` - 数据加载脚本
- `queries.sql` - 查询示例（含窗口函数、JOIN、排序对比）
- `notes.md` - Hive vs Oracle 差异笔记 + 面试考点整理

## 下周计划
- Python基础语法
- pandas数据处理入门
