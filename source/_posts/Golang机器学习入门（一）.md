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

下面我们需要定义一种数据结构来保存每一个维度的特征（表示为一个单元格）

    // Cell data cell
    type Cell struct {
        t          columnType
        s          string
        i          int
        f          float64
        ts         time.Time
        timeFormat func(time.Time) string
        empty      bool
    }

该类型支持`string`、`int`、`float`和`time`这四种类型的数据，当`empty`字段为true时表示当前单元格的内容为空。对于空单元格需要进行内容的填充，将会在下个章节中进行介绍。

最后我们定义一种结构来保存所有单元格的数据，为了方便数据读取我们通过两种方式来保存数据：
  - 通过index来找到对应的column或cell
  - 通过name来找到对应的column或cell

        // Data data
        type Data struct {
            columnsByIndex map[int]*Column
            columnsByName  map[string]*Column

            cellsByIndex []map[int]*Cell
            cellsByName  []map[string]*Cell

            loaded bool
        }

## 载入数据

现在我们已经有了所有的数据结构，下面让我们来看一下如何加载样本文件

    // AddColumn add column definition
    func (d *Data) AddColumn(col Column) *Column {
        d.columnsByIndex[col.index] = &col
        d.columnsByName[col.name] = &col
        return &col
    }

    // LoadFromCSV read data from csv
    func (d *Data) LoadFromCSV(r io.Reader, skipHeader bool) error {
        if len(d.columnsByIndex) == 0 {
            return constant.ErrNoColumns
        }
        cr := csv.NewReader(r)
        var rowIndex int
        for {
            row, err := cr.Read()
            if err != nil {
                if err == io.EOF {
                    d.loaded = true
                    return nil
                }
                return err
            }
            rowIndex++
            if rowIndex == 1 && skipHeader {
                continue
            }
            d.addRow(row)
        }
    }

    func (d *Data) addRow(row []string) {
        index := make(map[int]*Cell)
        name := make(map[string]*Cell)
        for idx, col := range d.columnsByIndex {
            str := row[idx]
            var cell Cell
            cell.t = col.t
            if len(str) == 0 {
                cell.empty = true
            } else {
                switch col.t {
                case columnString:
                    cell.s = str
                case columnInt:
                    n, _ := strconv.ParseInt(str, 10, 64)
                    cell.i = int(n)
                case columnFloat:
                    n, _ := strconv.ParseFloat(str, 10)
                    cell.f = n
                case columnTime:
                    cell.ts = col.timeParse(str)
                    cell.timeFormat = col.timeFormat
                }
            }
            index[col.index] = &cell
            name[col.name] = &cell
        }
        d.cellsByIndex = append(d.cellsByIndex, index)
        d.cellsByName = append(d.cellsByName, name)
    }

AddColumn函数用于添加字段定义，LoadFromCSV函数则总一个reader中读取csv文件内容，其中skipHeader参数用于表示是否跳过第一行的表头。

## 代码

以上内容的代码可从 https://github.com/lwch/ml/tree/master/data 得到。