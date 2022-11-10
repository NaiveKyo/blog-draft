# 一、简介

## 1、介绍

Apache POI 是 Java 程序员用来创建、修改和显示 MS Office 文件的一个流行的开源 Java 类库。它包含了一系列类和方法用于将用户输入的数据转换为 MS Office 文档，或者从文档中解析数据。

POI 即 Poor Obfuscation Implementation 的缩写；

官方文档参考：https://poi.apache.org/components/index.html

POI 版本：`org.apache.poi 5.2.2`

## 2、组件

> Apachec POI 包含的组件

通常我们将 MS Office 文档称为复合文档（Compound Document），它不仅包含文本而且包含图形、电子表格数据、声音、视频图像及其他信息。

Apache POI 提供的一系列工具类和方法正是为了服务于微软提出的 OLE2 复合文档格式，POI 主要包含以下几个组件（命名取自首字母）：

（1）基础组件：

- `POIFS`（Poor Obfuscation Implementation File System）：该组件是其他 POI 元素的基础，也是 POI 组件中最古老且最稳定的模块，主要用于显式读取不同的文件；	
- `HPSF`（Horrible Property Set Format）：主要用于提取 MS-Office 文件的属性集，将 MS-Office 文件的各项属性拆分并存储，如标题、作者、数据等等；

（2）excel 文件；

- `HSSF`（Horrible Spreadsheet Format）：主要用于读写 `xls` 后缀的 MS-Excel 文件；
- `XSSF`（XML Spreadsheet Format）：主要用于读写 `xlsx` 格式的 MS-Excel 文件；
  - 提示：xls 是 excel2003 及以前版本生成的文件格式，xlsx 是 excel 2007 及以后版本生成的文件格式；
  - xlsx 是向下兼容的；

（3）word 文件；

- `HWPF`（Horrible Word Processor Format）：主要用于读写 `doc` 后缀的 MS-Word 文件；
- `XWPF`（XML Word Processor Format）：主要用于读写 `docx` 后缀的 MS-Word 文件；
  - 提示：doc 是 word97 - 2003 版本的文件格式，2007 版本以后就是 docx；
  - docx 是被压缩过的文档，体积更小，能够处理更复杂的内容，访问速度更快；
  - 高版本的 word 是向下兼容的；
  - 可以将 docx 文件后缀改为 rar，之后便可以以压缩包的形式打开，ppt 同理；

（4）ppt 文件；

- `HSLF`（Horrible Slide Layout Format）：主要用于读写、编辑 `ppt` 后缀的 PowerPoint 演示文档；
- `XSLF`（XML Slide Layout Format）：主要用于读写、编辑 `pptx` 后缀的 PowerPoint 演示文档；
  - 提示：ppt 后缀是 97-2003 版本的文件格式，pptx 是 2007 及后续版本的文件后缀；

（5）visio 文件；

- `HDGF`（Horrible DiaGram Format）：包含的类和方法用于解析 MS-Visio 文件，读取效率较低，且只提供见到那的文本读取功能；
- `XDGF`（XML DiaGram Format）：相比 HDGF 提供了更多的支持；
  - 提示：HDGF 提供 visio 97-2003 版本的支持，XDGF 支持后续版本的以 `.vsdx` 后缀的文件；

（5）publisher 文档；

- `HPBF`（Horrible PuBlisher Format）：主要用于读写 MS-Publisher 文件，读取效率低且仅提供简单文本提取；

注意，POI 的版本问题：

- Version 3.5 以前的支持 doc、xls、ppt、etc；
- 3.5 以后的支持 docx、xlsx、pptx、etc；

更多信息参考：https://poi.apache.org/components/index.html

## 3、Maven 依赖

Apache POI 项目包含了对多种文档文件类型的支持，并提供了多种 jar 包，但我们要针对特定的文档进行操作时，可以引入对应的依赖，下面的表格中展示了常用 POI 组件和 Maven 依赖的对应关系：

（参考官方文档，Component Map 模块，下表列出了部分关系）

| 组件      | 文档类型                | Maven artifactId                                           | 注意事项                                                     |
| --------- | ----------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| POIFS     | OLE2 Filesystem         | poi                                                        | 适用于 OLE2/POIFS 类型的文件                                 |
| HPSF      | OLE2 Property Sets      | poi                                                        |                                                              |
| HSSF      | Excel XLS               | poi                                                        | 仅用于 HSSF 文件，如果是 Common SS 参考下文                  |
| HSLF      | PowerPoint PPT          | poi-scratchpad                                             |                                                              |
| HWPF      | Word DOC                | poi-scratchpad                                             |                                                              |
| HDGF      | Visio VSD               | poi-scratchpad                                             |                                                              |
| HPBF      | Publisher PUB           | poi-scratchpad                                             |                                                              |
| HSMF      | Outlook MSG             | poi-scratchpad                                             |                                                              |
| DDF       | Escher common drawings  | poi                                                        |                                                              |
| HWMF      | WMF drawings            | poi-scratchpad                                             |                                                              |
| OpenXML4J | OOXML                   | poi-ooxml plus either poi-ooxml-lite or<br/>poi-ooxml-full | See notes below for differences between these options        |
| XSSF      | Excel XLSX              | poi-ooxml                                                  |                                                              |
| XSLF      | PowerPoint PPTX         | poi-ooxml                                                  |                                                              |
| XWPF      | Word DOCX               | poi-ooxml                                                  |                                                              |
| XDGF      | Visio VSDX              | poi-ooxml                                                  |                                                              |
| Common SL | PowerPoint PPT and PPTX | poi-scratchpad and poi-ooxml                               | SL code is in the core POI jar, but implementations are in poi-scratchpad and poi-ooxml. |
| Common SS | Excel XLS and XLSX      | poi-ooxml                                                  | WorkbookFactory and friends all require poi-ooxml, not just core poi |

除此之外，官方还给出了不同 jar 包所依赖的其他类库，更多信息参考官网；





# 二、Excel 操作

## 1、简介

参考官方文档：https://poi.apache.org/components/spreadsheet/

Excel 文件的读写编辑，应该是平时使用场景最多的，下面以 Excel 相关操作为例。

XLS 后缀的 Excel 文件是 MS Office 97-2007 版本使用的文件格式，XLSX 后缀的 Excel 文件则是 MS Office 2007 以后版本使用的文件格式，而 HSSF 和 XSSF 都提供了对 Excel 文件的读写、创建、修改的支持。

主要包含以下功能：

- low level structures for those with special needs
- an eventmodel api for efficient read-only access
- a full usermodel api for creating, reading and modifying XLS files

从中可以看出，两者提供对文件低层次结构的抽象、事件模型  api、用户模型 api；

（1）如果要从 HSSF 向 XSSF 转换，则需要使用 SS 提供的一系列类或方法进行转换，相关 api 参考：https://poi.apache.org/components/spreadsheet/converting.html

（2）补充一点，生成 excel 文件的另一种方式是使用 [Cocoon](https://cocoon.apache.org/) 这个框架结合 HSSF 相关工具，该方式可以从任意的 XML 数据源通过序列化器应用到我们的 excel 样式表中；

（3）如果仅仅是想要读取 excel 中的数据，则可以使用 `eventusermodel` 包下的 eventmodel api 来进行读取；

（4）如果要修改或者生成 excel，则可以使用 usermodel api 进行相关操作。

注意 `usermodel api` 使用比较简单，但是要比 `low level eventusermodel` 占用更多的内存，而且 MS 2007 版本以后的 xlsx 类型的文件是基于 XML 的，解析这种类型也需要耗费更多的内存资源。

> 流式解析

从 poi 3.8 beta3 版本后，POI 在 XSSF 之上提供了一种内存占用更低的 SXSSF API（流式 XSSF）。

SXSSF 对 XSSF 做了兼容处理，当要处理的电子表格非常大并且 java 堆内存大小又被限制，我们可以使用 SXSSF；

SXSSF 之所有占用内存比 XSSF 低，是通过限制对 `sliding window` 中的 rows 的访问数量，且滑动窗口已访问的旧的 rows 将不会存在于窗口中，因为它们会被持久化到磁盘上，而 XSSF 则允许访问文档中的所有行。

在 `auto-flush mode` 下，可以指定访问窗口的大小，在内存保存一定数量的行，处理完某一行后，如果读入新的行到窗口中，索引最低的那个行将会被移除并写入磁盘中；也可以将窗口的大小设置为自动增长，在必要的时候调用 `flushRows(int keepRows)` 方法设置新的大小。

虽然 SXSSF 内存占用比 XSSF 低，但是也有一定的限制：

- 在某个时间点只能访问指定数量的行；
- 不支持 `Sheet.clone()`；
- 不支持公式评估；

使用 SXSSF 可以参考：https://poi.apache.org/components/spreadsheet/how-to.html#sxssf

（`package: org.apache.poi.xssf.streaming`）

## 2、XLSX 后缀

由于电脑上没有老版本的 MS Office 了，所以就以新版的 xlsx 后缀 Excel 文件为例。

### 依赖：

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.2.2</version>
</dependency>

<!-- poi 使用 log4j-api 作为日志框架, 这里引入其中 api 的实现 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.17.1</version>
</dependency>
```

为了更方便的查看提示信息，而且 poi 内部使用的是 log4j-api，所以这里引入了其实现 log4j-api，并提供以下简单的配置：

```xml
<!-- 文件名: log4j2.xml 更多信息参考官网 https://logging.apache.org/log4j/2.x/ -->
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="INFO">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

测试类准备工作：

```java
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class QuickStart {
    
    private static final Logger logger = LogManager.getLogger(QuickStart.class);
 	
    // 测试方法...
    
    /**
     * 根据路径和文件名构建完整文件路径
     * @param path  目录
     * @param name  文件名
     * @return      full path
     */
    private static String getFilePath(String path, String name) {
        String target = null;
        if (!path.endsWith(File.separator)) {
            path += File.separator;
        }

        if (!name.endsWith(".xlsx")) {
            name += ".xlsx";
        }

        target = path + name;
        return target;
    }
    
}
```

### 创建和填充数据

（1）创建一个工作簿（一个 excel 文件）：

```java
/**
 * 指定名称和路径新建一个工作簿
 * @param path 目录
 * @param name 文件名称
 */
public static void newWorkBook(String path, String name) {
    Workbook wb = new XSSFWorkbook();
    // do something ...
    String target = getFilePath(path, name);

    try (OutputStream fos = new FileOutputStream(target)) {
        // 写入到磁盘
        wb.write(fos);
        logger.info("成功创建 excel");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

注意这样创建了一个 .xlsx 后缀的工作簿，但是没有创建表格，所以打开文件会提示错误。

（2）下面创建带 sheet 的 excel（注：通常 excel 文件都应该有一个默认的 Sheet）：

```java
/**
 * 创建一个 excel 文件, 并写入指定数量的 sheet 
 * @param path          目录
 * @param name          文件名
 * @param sheetNames    sheet 名字(至少传入一个)
 *                      注意 Excel sheet 的名称不能超过 31 个字符并且不允许存在一些字符
 */
public static void createWorkBookWithSheet(String path, String name, String... sheetNames) {
    if (sheetNames.length <= 1) {
        logger.error("sheetNames 参数不能为空, 至少传入一个");
        logger.error("excel 创建失败");
        return;
    }

    Workbook wb = new XSSFWorkbook();

    String target = getFilePath(path, name);

    // 这里可以使用 org.apache.poi.ss.util.WorkbookUtil 生成安全的 sheet 名称, 该工具会将非法字符替换为 ' '
    List<String> safeSheetNames = Arrays.stream(sheetNames).map(WorkbookUtil::createSafeSheetName).collect(Collectors.toList());
    for (String safeSheetName : safeSheetNames) {
        wb.createSheet(safeSheetName);
    }

    try (OutputStream fos = new FileOutputStream(target)) {
        wb.write(fos);
        logger.info("成功创建 excel");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

（3）创建单元格并填充常规数据；

向 Sheet 中写入数据：

```java
/**
 * 测试为 excel 的 sheet 创建单元格
 */
public static void createCells(String path, String name) {
    Workbook wb = new XSSFWorkbook();
    CreationHelper creationHelper = wb.getCreationHelper();
    Sheet sheet = wb.createSheet("new sheet");

    // 创建一行, 并填充几个单元格数据, 注意行号从 0 开始
    Row row = sheet.createRow(0);
    // 创建一个单元格并填充值
    Cell cell = row.createCell(0);
    cell.setCellValue(1);

    // 也可以使用链式调用
    row.createCell(1).setCellValue(1.2);
    row.createCell(2).setCellValue(
        creationHelper.createRichTextString("This is a string")
    );
    row.createCell(3).setCellValue(true);

    // 写入磁盘
    try (OutputStream fos = new FileOutputStream(getFilePath(path, name))) {
        wb.write(fos);
        logger.info("成功创建 excel");
    } catch (IOException e) {
        e.printStackTrace();
    } 
}
```

（4）创建单元格并填充日期对象；

```java
/**
 * 单元格填充日期对象
 */
public static void createCellWithDate(String path, String name) {
    Workbook wb = new XSSFWorkbook();
    CreationHelper creationHelper = wb.getCreationHelper();
    Sheet sheet = wb.createSheet("new sheet");

    // 创建一行, 并填充几个单元格数据, 注意行号从 0 开始
    Row row = sheet.createRow(0);

    // 第一个单元格是文本形式的常规日期对象
    row.createCell(0).setCellValue(new Date()); 

    // 第二个单元格格式为 date(带 time) 类型
    // 从 WorkBook 创建风格对象是很重要的
    // 如果不这样做就会影响其他单元格的格式
    CellStyle cellStyle = wb.createCellStyle();
    cellStyle.setDataFormat(
        creationHelper.createDataFormat().getFormat("m/d/yy h:mm")
    );
    Cell cell = row.createCell(1);
    cell.setCellValue(new Date());
    cell.setCellStyle(cellStyle);

    // 也可以通过 java.util.Calender 构建日期
    Cell cell2 = row.createCell(2);
    cell2.setCellValue(Calendar.getInstance());
    cell2.setCellStyle(cellStyle);

    // 写入磁盘
    try (OutputStream fos = new FileOutputStream(getFilePath(path, name))) {
        wb.write(fos);
        logger.info("成功创建 excel");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

（5）其他类型：

```java
public static void createDifferenceTypeCells(String path, String name) {
    Workbook wb = new XSSFWorkbook();
    CreationHelper creationHelper = wb.getCreationHelper();
    Sheet sheet = wb.createSheet("new sheet");

    // 填充各种数据
    Row row = sheet.createRow(0);

    row.createCell(0).setCellValue(1.1);
    row.createCell(1).setCellValue(new Date());
    row.createCell(2).setCellValue(Calendar.getInstance());
    row.createCell(3).setCellValue("a string");
    row.createCell(4).setCellValue(true);
    row.createCell(5).setCellErrorValue(FormulaError.VALUE.getCode());

    // 写入磁盘
    try (OutputStream fos = new FileOutputStream(getFilePath(path, name))) {
        wb.write(fos);
        logger.info("成功创建 excel");
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 文件对象或输入流

当打开一个 Excel 文件时，要么是 .xls 后缀的要么是 .xlsx 后缀的，同时加载该文件的方式也有两种：

- 以 `File` 对象的形式，占用内存更少；
- 以 `InputStream` 方式，占用更多的内存，因为要缓存整个文件；

通过 `WorkbookFactory` 可以选择任一方式：

```java
/**
 * 读取文件的两种方式
 *  - File
 *  - InputStream
 */
public static void fileOrInputStream() throws IOException {
    // 通过 WorkbookFactory 可以选择其中任一方式
    Workbook wb1 = WorkbookFactory.create(new File("excel.xls"));
    Workbook wb2 = WorkbookFactory.create(new FileInputStream("excel.xlsx"));
}
```

> If using *HSSFWorkbook* or *XSSFWorkbook* directly, you should generally go through *POIFSFileSystem* or *OPCPackage*, to have full control of the lifecycle (including closing the file when done):

```java

// HSSFWorkbook, File
POIFSFileSystem fs = new POIFSFileSystem(new File("file.xls"));
HSSFWorkbook wb = new HSSFWorkbook(fs.getRoot(), true);
....
fs.close();
// HSSFWorkbook, InputStream, needs more memory
POIFSFileSystem fs = new POIFSFileSystem(myInputStream);
HSSFWorkbook wb = new HSSFWorkbook(fs.getRoot(), true);
// XSSFWorkbook, File
OPCPackage pkg = OPCPackage.open(new File("file.xlsx"));
XSSFWorkbook wb = new XSSFWorkbook(pkg);
....
pkg.close();
// XSSFWorkbook, InputStream, needs more memory
OPCPackage pkg = OPCPackage.open(myInputStream);
XSSFWorkbook wb = new XSSFWorkbook(pkg);
....
pkg.close();
```

### 对齐方式

```java
/**
 * 行高及单元格对齐方式设置
 */
public static void createWorkbookWithDiffAlignmentOptions(String path, String name) {
    Workbook wb = new XSSFWorkbook();
    CreationHelper creationHelper = wb.getCreationHelper();
    Sheet sheet = wb.createSheet("new sheet");

    // 填充各种数据
    Row row = sheet.createRow(0);
    row.setHeightInPoints(30f);

    // 创建多个单元格并设置对齐方式
    createCell(wb, row, 0, HorizontalAlignment.CENTER, VerticalAlignment.BOTTOM);
    createCell(wb, row, 1, HorizontalAlignment.CENTER_SELECTION, VerticalAlignment.BOTTOM);
    createCell(wb, row, 2, HorizontalAlignment.FILL, VerticalAlignment.CENTER);
    createCell(wb, row, 3, HorizontalAlignment.GENERAL, VerticalAlignment.CENTER);
    createCell(wb, row, 4, HorizontalAlignment.JUSTIFY, VerticalAlignment.JUSTIFY);
    createCell(wb, row, 5, HorizontalAlignment.LEFT, VerticalAlignment.TOP);
    createCell(wb, row, 6, HorizontalAlignment.RIGHT, VerticalAlignment.TOP);

    // 写入磁盘
    try (OutputStream fos = new FileOutputStream(getFilePath(path, name))) {
        wb.write(fos);
        logger.info("成功创建 excel");
    } catch (IOException e) {
        e.printStackTrace();
    }

    try {
        wb.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}

/**
 * 创建一个单元格, 并设置对齐方式
 * 
 * @param wb        工作簿
 * @param row       行
 * @param column    单元格
 * @param hAlign    水平对齐方式
 * @param vAlign    垂直对齐方式
 */
private static void createCell(Workbook wb, Row row, int column, HorizontalAlignment hAlign, VerticalAlignment vAlign) {
    Cell cell = row.createCell(column);
    cell.setCellValue("Align it");

    CellStyle cellStyle = wb.createCellStyle();
    cellStyle.setAlignment(hAlign);
    cellStyle.setVerticalAlignment(vAlign);

    cell.setCellStyle(cellStyle);
}
```

### 边框设置

```java
/**
 * 设置不同的边框
 */
private static void createWorkbookWithBorders(String path, String name) {
    Workbook wb = new XSSFWorkbook();
    Sheet sheet = wb.createSheet("new sheet");

    Row row = sheet.createRow(1);

    Cell cell = row.createCell(1);
    cell.setCellValue(4);

    // 为该单元格设置四周边框
    CellStyle cellStyle = wb.createCellStyle();

    cellStyle.setBorderBottom(BorderStyle.THIN);
    cellStyle.setBottomBorderColor(IndexedColors.BLACK.getIndex());

    cellStyle.setBorderLeft(BorderStyle.THIN);
    cellStyle.setLeftBorderColor(IndexedColors.GREEN.getIndex());

    cellStyle.setBorderRight(BorderStyle.THIN);
    cellStyle.setRightBorderColor(IndexedColors.BLUE.getIndex());

    cellStyle.setBorderTop(BorderStyle.THIN);
    cellStyle.setTopBorderColor(IndexedColors.BLACK.getIndex());

    cell.setCellStyle(cellStyle);

    // 写入磁盘
    try (OutputStream fos = new FileOutputStream(getFilePath(path, name))) {
        wb.write(fos);
        logger.info("成功创建 excel");
    } catch (IOException e) {
        e.printStackTrace();
    }

    try {
        wb.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

### 迭代

有时候需要迭代一个 workbook 中的所有 sheet，一个 sheet 中的所有 row，一个 row 中的所有 cell，可以这样做：

```java
/**
 * 迭代
 */
public static void iteratorWorkbook(Workbook wb) {

    Iterator<Sheet> sheetIterator = wb.sheetIterator();

    while (sheetIterator.hasNext()) {
        Sheet curSheet = sheetIterator.next();
        Iterator<Row> rowIterator = curSheet.rowIterator();
        while (rowIterator.hasNext()) {
            Row curRow = rowIterator.next();
            Iterator<Cell> cellIterator = curRow.cellIterator();
            while (cellIterator.hasNext()) {
                Cell curCell = cellIterator.next();
                // do something...
            }
        }
    }

    // 或者使用增强 for
    for (Sheet sheet : wb) {
        for (Row row : sheet) {
            for (Cell cell : row) {
                // do something...
            }
        }
    }
}
```

### ......

更多用法及 api 参考官网：

- https://poi.apache.org/components/spreadsheet/quick-guide.html
- https://poi.apache.org/components/spreadsheet/how-to.html

# 三、Word 操作

参考：https://poi.apache.org/components/document/index.html

