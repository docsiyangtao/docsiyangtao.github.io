## 基本规则

1. key: value，KV之间有空格
2. 大小写敏感
3. 使用缩进表示层级关系（不是tab，而是空格）
4. 注释用 #
5. '' 和 "" 表示字符串内容，可避免转义及转义



## 示例

```yaml
yaml:
  integer: 1
  aDouble: 2.2
  aBoolean: false
  string: "\t233"
  string2: '\t233'
  localDateTime: 2022-08-18 12:03:21
  date: 2022-08-18 12:03:21
#  strings: [str1, str2]
  strings:
    - str1
    - str2
#  map: {k1:v1, k2:v2}
  map:
    k1: 1
    k2: 2
  set:
    - 555
    - 666
    - 777
  listMap:
    k3: [1, 2, 3]
    k4: [4, 5, 6]
  user:
    id: 1
    name: '\tzhangsan'
```

对应Properties类

```java
@Component("yamlDemoProperties")
@ConfigurationProperties(prefix = "yaml")
@Data
@ToString
public class YamlDemoProperties {
    private Integer integer;
    private Double aDouble;
    private Boolean aBoolean;
    private String string;
    private String string2;
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime localDateTime;
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date date;
    private String[] strings;
    private List<String> list;
    private Map<String, Object> map;
    private Set<String> set;
    private Map<String, List<String>> listMap;
    private User user;
}
```



## 测试

```java
@GetMapping("/test1")
public String test1() {
    return JSON.toJSONString(yamlDemoProperties);
}
```

![image-20220716123417017](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220716123417017.png)

