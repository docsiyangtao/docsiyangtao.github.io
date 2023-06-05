## 序列化

```java
// 普通序列化
String toJSONString(Object object, SerializerFeature... features);

// 指定过滤器的序列化
String toJSONString(Object object, SerializeFilter filter, SerializerFeature... features);

// 序列化成utf-8编码的byte数组
byte[] toJSONBytes(Object object, SerializerFeature... features);

// 序列化到Writer中
int writeJSONString(Writer writer, Object object, SerializerFeature... features);

// 序列化到OutputStream中
int writeJSONString(OutputStream os, Object object,SerializerFeature... features);
```

* SerializerFeature：
  * QuoteFieldNames：输出key时使用双引号
  * UseSingleQuotes：输出key时使用单引号
  * WriteMapNullValue：输出值为null的字段（null）
  * WriteEnumUsingToString：枚举字段输出名称
  * UseISO8601DateFormat：日期使用ISO8601格式输出（2022-09-19T23:08:38.098+08:00）
  * WriteNullListAsEmpty：输出空列表（[]）
  * WriteNullStringAsEmpty：输出值为null的字符串（""）
  * WriteNullNumberAsZero：输出值为null的数值型字段（0）
  * WriteNullBooleanAsFalse：输出值为null的布尔类型字段（false）
  * SkipTransientField：序列化时将会被忽略类中的Get方法对应transient的字段
  * SortField：将字段名称排序（a-z）
  * WriteTabAsSpecial：把\t做转义输出（已弃用）
  * PrettyFormat：美化格式（换行、缩进）
  * WriteClassName：序列化时写入类型信息（@type）
  * DisableCircularReferenceDetect：消除对同一对象的循环引用
  * WriteSlashAsSpecial：对斜杠’/’进行转义  
  * BrowserCompatible：将中文序列化为\uXXXX格式，字节数会多一些，但是能兼容IE 6
  * WriteDateUseDateFormat：修改日期格式（yyyy-MM-dd HH:mm:ss）
  * BeanToArray：将对象的值组装成列表
  * WriteNonStringKeyAsString：map的键为null时，是否输出为 "null"



## 反序列化

```java
// 普通反序列化
JSONObject parseObject(String text);

// 反序列化成列表
JSONArray parseArray(String text);
<T> List<T> parseArray(String text, Class<T> clazz);

// 反序列化到指定类型
<T> T parseObject(String jsonStr, Class<T> clazz, Feature... features);

// byte数组反序列化
<T> T parseObject(byte[] bytes, Type clazz, Feature... features);

// 带泛型的反序列化
<T> T parseObject(String text, TypeReference<T> type, Feature... features);
List<Person> persons2 = JSON.parseObject(refJson2, 
                                         new TypeReference<List<Person>>() {}.getType());

// 从流中反序列化
<T> T parseObject(InputStream is, Charset charset, Type type, Feature... features);
```



## JSONObject

JSONObject的常见用法

```java
// 将JSON解析成JSONObject
JSONObject parseObject(String text);

// JSONObject设置：JSONObject实现了Map<String, Object>接口，可以像操作Map一样操作JSONObject

// JSONObject转换成JavaBean
<T> T toJavaObject(Class<T> clazz);
```





## @JSONField

```java
public @interface JSONField {
    
    // 指定序列化后的字段顺序
    int ordinal() default 0;

    // 指定字段名称，序列和反序列化均有效，指定后原字段名称值无法反序列化
    String name() default "";

    // 指定日期格式，序列和反序列化均有效
    String format() default "";

    // 是否序列化
    boolean serialize() default true;

    // 是否反序列化
    boolean deserialize() default true;

    // 序列化特性
    SerializerFeature[] serialzeFeatures() default {};

  	// 反序列化特性
    Feature[] parseFeatures() default {};
    
    String label() default "";
    
    // 如果字段值为JSON字符串，则直接进行输出，不反序列化
    boolean jsonDirect() default false;
    
    // 定制属性的序列化类，如示例
    Class<?> serializeUsing() default Void.class;
    
   	// 定制反序列化类
    Class<?> deserializeUsing() default Void.class;

    // 反序列化时可以使用多个不同的字段名称
    String[] alternateNames() default {};

    
    boolean unwrapped() default false;
}
```

定制序列化类实例

```java
@Data
@Accessors(chain = true)
public class Pet {
    @JSONField(serializeUsing = IdValueSerializer.class)
    private Long id;
    private String name;
}

public class IdValueSerializer implements ObjectSerializer {
    @Override
    public void write(JSONSerializer serializer, 
                      Object object, 
                      Object fieldName, 
                      Type fieldType, 
                      int features) throws IOException {
        Long value = (Long) object;
        String text = "宠物编号_" + value;
        serializer.write(text);
    }
}

class Test {
    @Test
    public void test6() {
        Pet pet = new Pet().setId(1L);
        System.out.println(JSON.toJSONString(pet));		// {"id":"宠物编号_1"}
    }
}
```

定制反序列化类示例

```java
@Data
@Accessors(chain = true)
public class Person {
    @JSONField(deserializeUsing = PetNameDeserializer.class)
}

public class PetNameDeserializer implements ObjectDeserializer {
    @Override
    public Date deserialze(DefaultJSONParser parser, Type type, Object fieldName) {
        String input = parser.getInput();
        return new Date(Long.parseLong(input));
    }

    @Override
    public int getFastMatchToken() {
        return 0;
    }
}

class Tset {
    @Test
    public void test8() {
        Person person = new Person().setRegisterTime(new Date());
        System.out.println(JSON.toJSONString(person));
		String json = "{\"id\":0,\"registerTime\":1663604387182}";
        Person person1 = JSON.parseObject(json, Person.class);
        System.out.println(person1);
        // Person(registerTime=Tue Sep 20 00:19:47 CST 2022)
    }
}
```



## 大数据量处理

1. 超大JSON数组序列化

   ```java
   JSONWriter writer = new JSONWriter(new FileWriter("/tmp/huge.json"));
   writer.startArray();
   for (int i = 0; i < 1000 * 1000; ++i) {
       writer.writeValue(new VO());
   }
   writer.endArray();
   writer.close();
   ```

2. 超大JSON对象序列化

   ```java
   JSONWriter writer = new JSONWriter(new FileWriter("/tmp/huge.json"));
   writer.startObject();
   for (int i = 0; i < 1000 * 1000; ++i) {
       writer.writeKey("x" + i);
       writer.writeValue(new VO());
   }
   writer.endObject();
   writer.close();
   ```

3. 反序列化

   ```java
   JSONReader reader = new JSONReader(new FileReader("/tmp/huge.json"));
   reader.startArray();
   while(reader.hasNext()) {
       VO vo = reader.readObject(VO.class);
       // handle vo ...
   }
   reader.endArray();
   reader.close();
   ```

   ```java
   JSONReader reader = new JSONReader(new FileReader("/tmp/huge.json"));
   reader.startObject();
   while(reader.hasNext()) {
       String key = reader.readString();
       VO vo = reader.readObject(VO.class);
       // handle vo ...
   }
   reader.endObject();
   reader.close();
   ```

   

## SerializeFilter

定制序列化，部分接口是函数式接口

测试序列化对象

```java
Map<String, Object> map = new HashMap<>();
map.put("k1", 700);
map.put("k2", "v2");
```

1. PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化

   ```java
   // 只序列化值大于500的键k1
   String json2 = JSON.toJSONString(map, new PropertyFilter() {
       @Override
       public boolean apply(Object object, String name, Object value) {
           // object: 是整个对象，即 map -> {k1=700, k2=v2}
           // name: k1 / k2
           // value: 700 / v2
           return "k1".equals(name) && (Integer) value > 500;
       }
   });
   
   System.out.println(json2);		// {"k1":700}
   ```

2. PropertyPreFilter 根据PropertyName判断是否序列化

   ```java
   // 只序列化key名称为“k2”的属性
   String json3 = JSON.toJSONString(map, (PropertyPreFilter) (serializer, object, name) -> "k2".equals(name));
   
   System.out.println(json3);		// {"k2":"v2"}

3. NameFilter 修改Key，如果需要修改Key，process返回值则可

   ```java
   // 将键进行大写转换
   String json4 = JSON.toJSONString(map, (NameFilter) (object, name, value) -> name.toUpperCase());
   
   System.out.println(json4);		// {"K1":700,"K2":"v2"}
   ```

4. ValueFilter 修改Value

   ```java
   // 如果是数值类型的键，则将值增大100
   String json5 = JSON.toJSONString(map, (ValueFilter) (object, name, value) -> {
       if ("key".equals(name) && value instanceof Integer) {
           return (Integer) value + 100;
       }
       return value;
   });
   
   System.out.println(json5);		// {"k1":800,"k2":"v2"}
   ```

5. BeforeFilter 序列化前执行操作

   ```java
   // 序列化前设置RegisterTime
   Person p = new Person().setId(1L).setName("Peter");
   String json = JSON.toJSONString(p, new BeforeFilter() {
       @Override
       public void writeBefore(Object object) {
           Person p = (Person) object;
           p.setRegisterTime(new Date());
       }
   });
   
   System.out.println(json);	// {"id":1,"name":"Peter","registerTime":1663692023085} 
   ```

6. AfterFilter 序列化之后执行操作

   ```java
   // 序列化之后设置RegisterTime
   Person p = new Person().setId(1L).setName("Peter");
   String json = JSON.toJSONString(p, new AfterFilter() {
       @Override
       public void writeAfter(Object object) {
           Person p = (Person) object;
           p.setRegisterTime(new Date());
       }
   });
   
   System.out.println(json);	// {"id":1,"name":"Peter"}
   ```

注意：BeforeFilter和AfterFilter目前对Map类型的对象进行操作时有BUG，不能正常操作。



## BeanToArray

JavaBean映射为json array，由于省略了key的输出，因此json字符串更小，性能更好

```java
Person person = new Person()
        .setId(1L)
        .setName("John")
        .setRegisterTime(new Date())
        .setBirthday(LocalDate.now());
String s = JSON.toJSONString(person, 
                             SerializerFeature.BeanToArray, 
                             SerializerFeature.SortField);
System.out.println(s);	// ["2022-09-21",1,"John",1663692880196]
```

通过json array反序列化成一个对象

```java
String jsonArray = "[\"2022-09-21\",1,\"John\",null,1663692880196]";
Person p = JSON.parseObject(jsonArray, Person.class, Feature.SupportArrayToBean);
```

局部使用

```java
@Data
@Accessors(chain = true)
@JSONType(serialzeFeatures = SerializerFeature.BeanToArray, parseFeatures = Feature.SupportArrayToBean)
public class Pet {
    private Long id;
    private String name;
}

@Data
@Accessors(chain = true)
public class Person {
    private Long id = 0L;
    private String name;
    private List<Pet> pets;
}

class Test {
    @Test
    public void test1() {
        Pet pet1 = new Pet()
                .setId(1L)
                .setName("旺财");
        Pet pet2 = new Pet()
                .setId(2L)
                .setName("喵喵");
        Person person = new Person()
                .setId(1L)
                .setName("张三")
                .setPets(Arrays.asList(pet1, pet2));
        String s = JSON.toJSONString(person);
        System.out.println(s);	// {"id":1,"name":"张三","pets":[[1,"旺财"],[2,"喵喵"]]}
    }
    
    @Test
    public void test2() {
        String json = "{\"id\":1,\"name\":\"张三\",\"pets\":[[1,\"旺财\"],[2,\"喵喵\"]]}";
        Person person = JSON.parseObject(json, Person.class);
        System.out.println(person);
        // Person(id=1, name=张三, registerTime=null, birthday=null, localDateTime=null, pet=null, json=null, weekend=null, map=null, pets=[Pet(id=1, name=旺财), Pet(id=2, name=喵喵)])
    }
}
```

