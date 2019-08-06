---
title: Hbase数据模型

date: 2017-07-03 20:41:50

tags: [Hbase]

categories: 大数据


---

介绍列数据库Hbase的数据模型

<!-- more --> 

# 列数据库
## 什么是列数据库？what

### 行/列数据库的区别
* 行式数据库：按照行存储的，行式数据库擅长随机读操作不适合用于大数据。像SQL server,Oracle，mysql等传统的是属于行式数据库范畴。
* 列数据库：列式数据库从一开始就是面向大数据环境下数据仓库的数据分析而产生。

## 列数据库如何存储数据？how

## 为什么选择列数据库？why
列数据库的存储优势：空间利用率、列数限制

# Hbase的数据模型
## RowKey
## Column Family
## Column Qualifier
