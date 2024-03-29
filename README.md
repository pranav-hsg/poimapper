
# Poimapper

It is a lightweight library designed to convert Excel rows into Java beans, offering the additional capability to generate Excel sheets based on specified mappings

## Table of contents

* [Getting started](#gs)
* [Description & Features](#desc)
* [Why use poimapper](#why-use)
* [Usage](#usage)
  * [Mapping excel row to bean](#usage-mapping)
  * [Generating excel sheet with headers](#usage-generate-excel)
* [Customizing Configuration](#cust-conf) 
  * [PoiConfig](#cust-conf-poi)
  * [PoiBuilderConfig](#cust-conf-build)
* [Type Casting](#type-conversion)
* [Customizing Type Cast](#customized-type-conversion)
* [License](#license)

## Getting Started <a name="gs"></a>
Maven import
``` xml
<dependency>
    <groupId>io.github.pranav-hsg</groupId>
    <artifactId>poimapper</artifactId>
    <version>2.0.1</version>
</dependency
```
Gradle import
```properties
implementation group: 'io.github.pranav-hsg', name: 'poimapper', version: '2.0.1'
```

For the most up-to-date information, check out the latest details [here](https://central.sonatype.com/artifact/io.github.pranav-hsg/poimapper/1.0.0).

## Description <a name="desc"></a>
The poimapper library is a powerful Java tool designed to simplify Excel sheet mapping to java object. It eliminates the need for manual index updates when rearranging columns, thanks to its intelligent use of a column map. This map, represented by a LinkedHashMap, organizes columns logically and automatically handles index updates.

### Features

* **Header Arrangement**
  In the traditional approach, you need to manually update indices. However, with this library, there's no need for manual index management. The column order is determined automatically by adding entries to the linked hash map.
  

* **DataType Detection**
Automatically detects various data types, including String, Integer, Double, Boolean, LocalDate, LocalDateTime, ZonedDateTime, BigDecimal, Enum, and more.
Handles nested types by calling the constructor based on the field type and sets value directly. Also you can pass a customized type conversion logic, Please refer to [see more](#customized-type-conversion).


* **Excel Sheet Generation**
Makes it easy to create Excel sheet headers by using the provided column map. This saves you from having to manually map headers each time you rearrange them in your code and helps prevent errors related to manual index adjustments.[see more](#usage-generate-excel).

## Why use poimapper <a name="why-use"></a>

This library streamlines the mapping of Excel rows to Java beans by introducing a columnMap mechanism, eliminating the need for manual index adjustments when altering the Excel column order. Unlike traditional methods that require meticulous index updates, the columnMap relies on a LinkedHashMap to maintain the specified order of columns.

Consider a scenario where a new column 'Address' needs to be added before the existing 2nd column. In the traditional approach, manual index updates are necessary. However, with this library, you can effortlessly group related columns in the columnMap, avoiding manual adjustments.

```java  
// Traditional method with manual index updates
Cell cell0 = row.getCell(0);
String name = cell0.getStringCellValue();
Cell cell1 = row.getCell(1); // Manually updated index
String birthDate = cell1.getStringCellValue();
.
.
.
Cell cell100 = row.getCell(100);
String amount = cell2.getStringCellValue();
```
To
```java
Cell cell0 = row.getCell(0);
String name = cell0.getStringCellValue();
Cell cell1 = row.getCell(1); 
String address = cell1.getStringCellValue();
Cell cell2 = row.getCell(2); // Manually updated index
String birthDate = cell1.getStringCellValue();
.
.
.
Cell cell101 = row.getCell(101); // Manually updated index all the way upto last column
String amount = cell2.getStringCellValue();

```
With the poimapper, the columnMap specifies the order of columns, providing a more flexible and maintainable solution:

```java 
columnMap.put("Name", new PoiConfig("name"));
columnMap.put("Address", new PoiConfig( "address"));
columnMap.put("BirthDate", new PoiConfig( "birthDate"));
.
.
.
columnMap.put("Amount", new PoiConfig("amount"));
```
No more manual index update.This approach simplifies the mapping process and enhances flexibility. Additionally, the library allows the generation of Excel sheets directly from the columnMap, offering a comprehensive solution for diverse Excel structures. See more on excel generation feature in usage section.

## Usage <a name="usage"></a>

### To convert excel row to java object <a name="usage-mapping"></a>

```java

import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import com.poimapper.ExcelRowBeanMapper;
import com.poimapper.config.PoiConfig;
import java.util.LinkedHashMap; 
import java.util.Map;

// Declare the column mapping note that order in which it was appended will map to order of cell.
LinkedHashMap<String,PoiConfig> columnMap = new LinkedHashMap<>();
// Default value is used when cell is empty
columnMap.put("Name", new PoiConfig("name", null, "Beta"));
// Pattern is used to override default date format
columnMap.put("Birth Date",new PoiConfig("birthDate", "yyyy-MM-dd"));
// Colon is used in field mapping is used to indicate nested field
columnMap.put("Amount", new PoiConfig( "amount:value"));

ExcelRowBeanMapper mapper = new ExcelRowBeanMapper.Builder().setRowMapping(columnMap).build();
Sheet sheet = workbook.getSheetAt(1);
for (int i = 1; i < sheet.getLastRowNum(); i++) {
    Row row = sheet.getRow(i);
    SimpleTestDto userInfo = mapper.fromExcelRow(row, new SimpleTestDto());
    System.out.println(userInfo);
}

public class SimpleTestDto {
    private String name;
    private LocalDate birthDate;
    private Amount amount;
    public static class Amount {
        private BigDecimal value;
    }
}
```

### To generate excel sheet from excel mapping <a name="usage-generate-excel"></a>

Based on passed column map, it uses key and based on what order item is pushed to map, headers are generated

```java
import com.poimapper.util.ExcelSheetGeneratorUtil;

ExcelSheetGeneratorUtil excelSheetGenerator = new ExcelSheetGeneratorUtil
                .Builder()
                .setRowMapping(columnMap)
                .setPath("src/test/resources/generated")
                .build();
Boolean isSuccess = excelSheetGenerator.generate("GeneratedExcel");
```
Sample Output: 

| Name  | Birth Date | Amount |
| ------------- |:-------------:|:-------------:|
## Customizing configuration  <a name="cust-conf"></a>

```java
ExcelRowBeanMapper mapper = new ExcelRowBeanMapper.Builder()
                .setRowMapping(columnMap) //  mandatory field which accepts LinkedHashMap<String,PoiConfig> row mapping which defines field related info. 
                .setMapperSettings(new PoiBuilderConfig(false, true)) // Optional field that Accepts PoiBuilderConfig
                .setDateFormat("dd/yyyy/MM") // Optional field that accepts date format pattern
                .setDateTimeFormat("dd/yyyy/MM HH:mm:ss") // Optional field that accepts datetime format
                .setCustomStringCastingFunc(customCastFunc) // Optional field that accepts custom cast.
                .build();
```

| Method | Mandatory |     Expected Type      |     Default Value      |
|--------|:---------:|:----------------------:|:----------------------:|
| setRowMapping   |   true    |          LinkedHashMap<String, PoiConfig>          |          null          |
| setMapperSettings   |   false   | PoiBuilderConfig | new PoiBuilderConfig() |
| setDateFormat   |   false    |     String    |      "yyyy-MM-dd"      |
| setDateTimeFormat   |   false    | String  | "yyyy-MM-dd HH:mm:ss"  |
| setCustomStringCastingFunc   |   false    |   CastString    |   DefaultCastString    |

### PoiConfig <a name="cust-conf-poi"></a>
List of constructors

```java
import com.poimapper.config.PoiConfig;

public PoiConfig();

public PoiConfig(String fieldName);

public PoiConfig(String fieldName, String pattern);

public PoiConfig(String fieldName, String pattern, Object defaultValue);

// In addition to the traditional constructor methods, an alternative approach for constructing
// - instances is provided through the use of the Builder pattern.

PoiConfig.builder()
        .fieldName("name")
        .defaultValue("default")
        .pattern(null)
        .build();

// fieldName   : field name to which mapper maps to
// pattern     : Pattern to override for date and time
// defaultValue: Default value to use if Excel cell is empty
```
### PoiBuilderConfig <a name="cust-conf-build"></a>
List of constructors
```java
import com.poimapper.config.PoiBuilderConfig;

public PoiBuilderConfig();

public PoiBuilderConfig(boolean suppressWarnings, boolean strictMode);

// suppressWarning : whether to suppress warnings
// strictMode : whether to throw error if failed to cast value / if field specified in rowMapping is not present 
```
| Field                      | Expected Type |     Default Value      |
|----------------------------|:-------------:|:----------------------:|
| suppressWarnings              |    boolean    |          true          |
| strictMode          |    boolean    |           false         |
## Type casting <a name="type-conversion"></a>

By default, the library automatically converts these types from the string type (extracted from the cell) to the corresponding field type of the bean.

* String
* Integer and int
* Double and double
* Boolean and boolean
* LocalDate
* LocalDateTime
* ZonedDateTime
* BigDecimal
* Enum
* Number

For nested types specified with the "fieldMapping" using a colon (":"), the library automatically invokes the constructor based on the type. It creates an instance of the nested type and directly assigns the extracted value to the respective field.

For instance, in the provided example, the "Amount" class is a nested type. The library calls the no-argument constructor of the "Amount" class and sets the 'value' field. This is done based on the default casting types defined or a custom cast method if one is provided.

## Customizing the default cast implementation  <a name="customized-type-conversion"></a>
You can even customize your mapping based on your own implementation. This will override the default method.
```java
import com.poimapper.util.ExcelSheetGeneratorUtil;

// Declare a custom cast method
public Object customCast(Field field, String value,PoiConfig config){
  Class<?> fieldType = field.getType();
  if (fieldType == String.class) {
    return value;
  }else if(fieldType == Boolean.class || fieldType == boolean.class){
    return  true;
  }
  return null;
}

public void test(){
  ExcelRowBeanMapper mapper = new ExcelRowBeanMapper.
          Builder().
          setCustomStringCastingFunc(this::customCast). // Set custom cast implementation
          setRowMapping(columnMap).
          build();
  Sheet sheet = workbook.getSheetAt(1);
  for (int i = 1; i < sheet.getLastRowNum(); i++) {
    Row row = sheet.getRow(i);
    SimpleTestDto userInfo = mapper.fromExcelRow(row, new SimpleTestDto());
    System.out.println(userInfo);
  }
}
```
## License <a name="license"></a>

[MIT](https://choosealicense.com/licenses/mit/)

