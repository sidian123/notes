# 引言

## 介绍

* Apacke POI同时支持`.xls`和`.xlsx`格式的Excel表单.
* `Workbook`, `Sheet`, `Row`, `Cell`接口分别建模Excel对应概念(字面意思)
* 对于不同格式的文件, 使用不同接口实现. 有一特点: 处理`.xlsx`文件的类有前缀`XSSF`; 处理`.xls`文件的类有前缀`HSSF`
* Excel中, 不同Cell都是有类型的, 有不同的访问方法.

## Maven依赖

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.2</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.2</version>
</dependency>
```

# 使用

## 读

例子

```java
FileInputStream file = new FileInputStream(new File(System.getProperty("user.home")+"/Desktop/全部事件.xlsx"));
Workbook workbook = new XSSFWorkbook(file);//excel文件
Sheet sheet = workbook.getSheetAt(0);//某一页
Row row = sheet.getRow(2);//某行
for (Cell cell : row) {
    switch (cell.getCellType()) {//判断某列类型
        case STRING://字符串
            System.out.println("String Type, Value:"+cell.getRichStringCellValue());
            break;
        case NUMERIC:
            if(DateUtil.isCellDateFormatted(cell)){//日期
                System.out.println("Date Type, Value:"+cell.getDateCellValue());
            }else{//数字
                System.out.println("Num Type, Value:"+cell.getNumericCellValue());
            }
            break;
        case BOOLEAN://布尔
            System.out.println("Boolean Type, Value:"+cell.getBooleanCellValue());
            break;
        case FORMULA://公式
            System.out.println("Formula Type, Value:"+cell.getCellFormula());
            break;
        default://不知道什么类型,应该取不出吧
            System.out.println("Other Type, Value");
    }
}
```

## 写

传送门: [Writing to Excel](https://www.baeldung.com/java-microsoft-excel#2-writing-to-excel)

# 参考

* [Apache POI](https://poi.apache.org/) 官网
* [Working with Microsoft Excel in Java](https://www.baeldung.com/java-microsoft-excel) 教程

