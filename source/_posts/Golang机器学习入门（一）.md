---
title: Golang机器学习入门（一）
date: 2020-05-19 00:49:45
tags:
  - 样本特征
categories:
  - 机器学习
---

本章将会讲解golang中如何读取样本文件。

<!-- more -->

## 样本文件

首先让我们来看一下什么是样本文件，以下数据摘自[kaggle伦敦房屋数据](https://www.kaggle.com/justinas/housing-in-london)

    date,area,average_price,code,houses_sold,no_of_crimes,borough_flag
    1995-01-01,city of london,91449,E09000001,17,,1
    1995-02-01,city of london,82203,E09000001,7,,1
    1995-03-01,city of london,79121,E09000001,14,,1
    1995-04-01,city of london,77101,E09000001,7,,1
    1995-05-01,city of london,84409,E09000001,10,,1
    1995-06-01,city of london,94901,E09000001,17,,1
    1995-07-01,city of london,110128,E09000001,13,,1
    1995-08-01,city of london,112329,E09000001,14,,1
    1995-09-01,city of london,104473,E09000001,17,,1

从以上内容可见，样本文件是一个多行的文件，每一行为一个样本，每一列表示该样本的一个属性（特征）。

## 特征解释

1. `date`: 该字段表示采样日期
2. `area`: 该字段表示样本所属地区
3. `average_price`: 该字段表示房屋均价
4. `code`: 该字段表示所属地区的邮政编码
5. `houses_sold`: 该字段表示所属地区出售房屋数量
6. `no_of_crimes`: 该字段表示所属地区的犯罪案件数量
7. `borough_flag`: 该字段表示所属区域是否是一个自治区

通过进一步观察发现，`date`字段为日期类型，`area`和`code`字段为字符串类型，其他所有字段均为数值类型。

## 建模

下面让我们来考虑一下如何进行建模（创建数据结构），首先根据第一行内容我们可以得到一些column（字段）

    type columnType int 

    const (
        columnTime columnType = iota
        columnString
        columnInt
        columnFloat
    )

    // Column data column
    type Column struct {
        index      int 
        name       string
        t          columnType
        timeParse  func(string) time.Time
        timeFormat func(time.Time) string
    }

根据上面的类型定义，我们需要支持`string`、`int`、`float`和`time`四种类型的字段（引入float类型仅为了方便运算）。类型中的`index`和`name`是为了定位到字段的位置与名称的，`timeParse`函数和`timeFormat`函数用于对日期时间类型的字段进行序列化与反序列化。