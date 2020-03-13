---
title: 如何优雅的解析word表格
date: 2020-01-07 22:11:48
thumbnail: 
categories:
    - java
tags:
    - experience
---

## 使用POI和自定义注解解析word表格

&emsp;&emsp;对于一些报表的需求的时候，可能会对接一些系统，我们需要解析一些报表数据入库，然后再分析。报表可能是csv，可能是excel，也有可能是word。word其实比较少见，所以解析的使用会比较少，今天就先记录一下word表格如何解析成java对象list。
&emsp;&emsp;这里用到了注解反射的配合完成，不熟悉使用的可以先了解注解。简单说注释是给编码人员读的，而注解是给程序读的，单纯的注解没有什么用，需要配合反射来使用。

<!-- more -->

### 使用POI

java的poi给我们提供了解析各种文件的能力，其中就包括word，首先我们引入jar包
``` xml
<dependency>
		<groupId>org.apache.poi</groupId>
		<artifactId>poi-ooxml</artifactId>
		<version>${poi.version}</version>
</dependency>

<dependency>
		<groupId>org.apache.poi</groupId>
		<artifactId>poi-ooxml-schemas</artifactId>
		<version>${poi.version}</version>
</dependency>

<!-- 这里使用的版本是4.0.1 上面2个依赖同一版本就行了（不同可能会有冲突）-->

```

### 根据需求了解用法

我们的需求是解析word中的一个表格，返回List<Object>的形式，那么我们看看怎么用：
``` java
// 根据输入流创建解析对象
XWPFDocument xwpf = new XWPFDocument(InputSteam);
// 获取word中table
List<XWPFTable> tables = xwpf.getTables();
// 获取word中table的迭代器
Iterator<XWPFTable> tablesIterator

// 再看看XWPFTable的方法
// 获取table中的所有行
List<XWPFTableRow> rows = table.getRows();

// 获取rows中的一行
XWPFTableRow row = rows.get(i);

// 获取行中的所有单元格
List<XWPFTableCell> cells = row.getTableCells();

// 获取单元格中的内容 包括获取一个单元格
String text = cells.get(index).getText();

```

接下来我们知道了，他给我们提供一个word中表格的list，我们可以根据表格的index定位需要的表格，然后提供了获取表格中行，然后根据行中单元格的定位获取单元格中内容的功能；那么我们就能根据表格的index和单元格的index来将表格映射到类，将单元格映射到字段；

### 定义注解

``` java
/**
 *标记表格再word中位置的注解，映射到类
 */
@Inherited
@Documented
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface WordTableIndex {

    // word 中的table序号 从0开始
    int value();
}

/**
 *标记单元格在行中的位置的注解，映射到字段
 */
@Inherited
@Documented
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface WordTableCellIndex {
    /**
     * 在table中的列序号 从0开始
     */
    int value();
}

```

### 工具类

``` java

public static <E> List<E> getTableData(InputStream in, Class<E> clazz) throws IOException, IllegalAccessException, InstantiationException {
		XWPFDocument xwpf = new XWPFDocument(in);
		List<XWPFTable> tables = xwpf.getTables();
		List<E> results = new ArrayList<>();
		WordTableIndex tableIndexAnno = clazz.getAnnotation(WordTableIndex.class);
		if (tableIndexAnno == null) {
				return results;
		}
		int tableIndex = tableIndexAnno.value();
		XWPFTable table = tables.get(tableIndex);
		List<XWPFTableRow> rows = table.getRows();
		//读取每一行数据 这里从1开始是把表头给忽略；
		for (int i = 1; i < rows.size(); i++) {
				XWPFTableRow row = rows.get(i);
				//读取每一列数据
				List<XWPFTableCell> cells = row.getTableCells();
				E res = clazz.newInstance();
				Field[] fields = clazz.getDeclaredFields();
				for (Field f : fields) {
						WordTableCellIndex annotation = f.getAnnotation(WordTableCellIndex.class);
						if (annotation != null) {
								int index = annotation.value();
								String text = cells.get(index).getText();
								f.setAccessible(true);
								f.set(res, text);
						}
				}
				results.add(res);
		}

		return results;
}
```

### 如何使用

那么我们该如何使用他呢：

1. 我们需要一个类，映射的就是这个word中的一个表格
``` java

@Data
public class WordTable {

	private String column1;

	private String column2;

	...

}
```

2. 我们在类上和字段上分别注解上映射
``` java
@Data
@WordTableIndex(0)
public class WordTable {

	@WordTableCellIndex(0)
	private String column1;

	@WordTableCellIndex(1)
	private String column2;
	...

}
```
3. 最后我们调用方法
``` java

// 获得文件流
InputStream in = new FileInputStream("xxxx");

List<WordTable> tableResults = getTableData(in, WordTable.class);

```
这样我们就得到了table中的数据，这样的方式适合table中数据位置固定的情况，如果数据的位置不是固定的，但是表头中的字段是固定的，可以注解替换成表头中的内容，然后再根据表头内容和字段注解获取到但与那个在行中的位置。
