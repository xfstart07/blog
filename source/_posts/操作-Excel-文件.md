---
title: "[Go] 操作 Excel 文件"
date: 2018-06-10 11:25:14
categories:
  - Go
tags:
  - Go
  - Excel
---

介绍两个操作 `Excel` 文件的库 `tealeg/xlsx` 和 `excelize`。

## tealeg/xlsx

### 创建文件和工作表(Sheet)
我们创建一个工作表，然后写入一些文字。

```go
package main

import (
	"github.com/tealeg/xlsx"
	"fmt"
)

func main() {
	file := xlsx.NewFile()
	sheet, err := file.AddSheet("Sheet1")
	if err != nil {
		fmt.Println("创建工作表失败", err)
		return
	}

	row := sheet.AddRow()
	cell := row.AddCell()
	cell.Value = "我是一个表格"
	err = file.Save("public/excel2.xlsx")
	fmt.Println(err)
}
```

使用 `NewFile()` 方法来创建一个Excel 文件，然后创建一个 Sheet，使用方法 `AddSheet()`，还有添加行和表格的方法 `AddRow()` 和 `AddCell()`

<!--more-->

### 打开并写入
首先我们来打开一个 `xlsx` 的文件，使用 `OpenFile()` 方法。

```go
package main

import (
	"github.com/tealeg/xlsx"
	"fmt"
)

func main () {
	filePath := "public/excel.xlsx"
	file, err := xlsx.OpenFile(filePath) // 打开一个文件
	if err != nil {
		fmt.Println(err)
		return
	}

	for _, sheet := range file.Sheets {
    	for _, rows := range sheet.Rows {
    		for _, cell := range rows.Cells {
    			fmt.Println(cell.Value)
    		}
    	}
  }
}
```

## excelize
`excelize` 也是一个支持读写 `xlsx` 文件的库。由 360 公司开源的。

## 操作 excel 文件

```go
package main

import (
	"github.com/360EntSecGroup-Skylar/excelize"
	"fmt"
)

func main() {
	xlsx, err := excelize.OpenFile("public/excel2.xlsx")
	if err != nil {
		fmt.Println(err)
		return
	}

	// 获取工作表，打印出数据
	rows := xlsx.GetRows("Sheet1")
	for _, row := range rows {
		for _, cell := range row {
			fmt.Println(cell)
		}
	}

	sheetName := xlsx.GetSheetName(1)

	cell := xlsx.GetCellValue(sheetName, "A1")
	fmt.Println(cell)

	// 合并 A1-E1
	xlsx.MergeCell(sheetName, "A1", "E1")

	// A1-E1 的内容居中
	style, err := xlsx.NewStyle(`{"alignment":{"horizontal":"center"}}`)
	if err != nil {
		fmt.Println("创建样式失败", err)
		return
	}
	xlsx.SetCellStyle(sheetName, "A1", "E1", style)

	xlsx.Save()
}
```

### 总结
目前使用的感觉是 `excelize` 更适合用来操作文件，它操作文件的特点是根据指定的位置进行操作。
感觉 `tealeg/xlsx` 比较适合写入数据。

## 资源
https://github.com/tealeg/xlsx
https://github.com/360EntSecGroup-Skylar/excelize
