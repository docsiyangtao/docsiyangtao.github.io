# EasyExcel简单使用

## 读

Excel表格数据

sheet1

| 字符串标题 | 日期标题         | 数字标题 | 字符串标题2 | 转换日期      | 金额     |
| ---------- | ---------------- | -------- | ----------- | ------------- | -------- |
| 字符串0    | 2020/1/1 1:01:01 | 1        | 男          | 2023年5月21日 | 2.21     |
| 字符串1    | 2020/1/2 1:01:01 | 2        |             | 2023年5月22日 | 1,233.43 |
| 字符串2    | 2020/1/3 1:01:01 | 3        |             | 2023年5月23日 | 522.00   |

sheet2

| 字符串标题 | 日期标题         | 数字标题 | 字符串标题2 | 转换日期      | 金额      |
| ---------- | ---------------- | -------- | ----------- | ------------- | --------- |
| 字符串0    | 2022/1/1 1:01:01 | 2        | 男          | 2021年5月21日 | 11,112.21 |
| 字符串1    | 2023/1/2 1:01:01 | 2        | 女          | 2023年5月22日 | 1,233.43  |
| 字符串2    | 2021/1/3 1:01:01 | 3        |             | 2022年5月23日 | 1,223.00  |

Excel数据类

```java
@Getter
@Setter
@EqualsAndHashCode
public class IndexOrNameData {
    // 强制读取第3列，不建议 index 和 name 同时用
    @ExcelProperty(index = 2)
    private Double doubleData;
    
    // 用名字去匹配，这里需要注意，如果名字重复，会导致只有一个字段读取到数据
    @ExcelProperty("字符串标题")
    private String string;
    
    @ExcelProperty("日期标题")
    private LocalDateTime dateTime;
    
    @ExcelProperty(index = 3)
    private Integer gender;
    
    @ExcelProperty("转换日期")
    @DateTimeFormat("yyyy年MM月dd日")
    private LocalDate date;
    
    @ExcelProperty("金额")
    @NumberFormat(",##0.00")
    private BigDecimal price;
}
```

常用读

```java
@Test
public void test() {
    String fileName = TestFileUtil.getPath() + "demo" + File.separator + "demo.xlsx";
    EasyExcel.read(fileName)  // 完整的文件名 行数据封装 定义Listener
            .head(IndexOrNameData.class)
            .registerReadListener(new ReadListener<IndexOrNameData>() {
                @Override
                public void invoke(IndexOrNameData data, AnalysisContext context) {
                    // 行数据处理
                    System.out.println(JSON.toJSONString(data));
                }
                @Override
                public void doAfterAllAnalysed(AnalysisContext context) {
                    // 处理完数据后
                }
            })
            // 编码，仅针对CSV文件
            .charset(StandardCharsets.UTF_8)
            // 强制使用输入流。为否则将输入流传输到临时文件以提高效率
            .mandatoryUseInputStream(false)
            // 自动关闭流，默认是
            .autoCloseStream(true)
            // 是否跳过空行
            .ignoreEmptyRow(true)
            // 密码
            .password("")
            // 设置转换器（针对所有列），使用数据类中ExcelProperty可单独对某个列进行设置
            .registerConverter(new MyConvert())
            // 扩展读取，如超链接，批注等
            .extraRead(CellExtraTypeEnum.HYPERLINK)
            // 构建ExcelReader
            .build()
            // 开始读取，参数为需要读取的Sheet页对象
            .read(new ReadSheet(0), new ReadSheet("Sheet2"));
            // 可指定sheet页（sheet无参则默认第一页），并使用doRead方法进行读取
            // .sheet(1)
            // .doRead();
            // .doReadSync();
}
```

运行结果

![image-20230617003430171](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20230617003430171.png)

## 写

数据类型

```java
@Getter
@Setter
@EqualsAndHashCode
@AllArgsConstructor
@NoArgsConstructor
public class DemoData {
    @ExcelProperty("字符串标题")
    private String string;
    // 一个指定index，则其他属性也需要指定index才会导出
    @ExcelProperty(value = "日期标题", index = 2)
    private Date date;
    @ExcelProperty("数字标题")
    private Double doubleData;
    @ExcelProperty(value = "字符串标题2", index = 0)
    private String string2;
    @ExcelProperty(value = "字符串标题3", index = 1)
    private String string3;
    // 忽略字段
    @ExcelIgnore
    private String ignore;
}
```

常用写方法

```java
@Test
public void test() {
    String fileName = TestFileUtil.getPath() + "simpleWrite" + System.currentTimeMillis() + ".xlsx";
    EasyExcel.write(fileName)
            .head(DemoData.class)
            .sheet("sheet1")
            // 导出排除对应列，参数可为Excel列名、列坐标、数据类属性名
            .excludeColumnFieldNames(Collections.singleton("字符串标题"))
            .excludeColumnFieldNames(Collections.singleton("doubleData"))
            .excludeColumnIndexes(Collections.singleton(3))
            // 只导出对应的列，如包含排除的字段，则也不导出
            .includeColumnFieldNames(Arrays.asList("string2", "doubleData", "date", "string3"))
            .doWrite(new ArrayList<DemoData>() {{
                add(new DemoData("标题1", new Date(), 1.1D, "123", "345", "1"));
                add(new DemoData("标题2", new Date(), 2.2D, "123", "345", "2"));
                add(new DemoData("标题3", new Date(), 3.3D, "123", "345", "3"));
            }});
}
```