## 普通

1. get方法获取参数

   ```java
   @GetMapping("/test1")
   public String test1(@RequestParam("param") String param) {
       return param;
   }
   ```

   ![image-20220717024905156](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717024905156.png)

2. get方法获取数组参数

   ```java
   @GetMapping("/test2")
   public String test2(@RequestParam("ids") List<Integer> ids) {
       return JSON.toJSONString(ids);
   }
   ```

   ![image-20220717025325924](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717025325924.png)

3. get方法封装多个参数

   ```java
   @GetMapping("/test3")
   public String test3(@RequestParam Map<String, String> params) {
       return JSON.toJSONString(params);
   }
   ```

   ![image-20220717025424881](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717025424881.png)

4. 路径参数

   ```java
   @GetMapping("/test4/{param}")
   public String test4(@PathVariable("param") String param) {
       return param;
   }
   ```

   ![image-20220717025549407](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717025549407.png)

5. 路径参数封装

   ```java
   @GetMapping("/test5/{param1}/{param2}")
   public String test5(@PathVariable Map<String, String> pathVarMap) {
       return JSON.toJSONString(pathVarMap);
   }
   ```

   ![image-20220717025704559](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717025704559.png)

6. 请求头参数

   ```java
   @GetMapping("/test6")
   public String test6(@RequestHeader("User-Agent") String userAgent) {
       return userAgent;
   }
   ```

   ![image-20220717025822890](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717025822890.png)

7. 请求头参数封装

   ```java
   @GetMapping("/test7")
   public String test7(@RequestHeader Map<String, String> headerMap) {
       return JSON.toJSONString(headerMap);
   }
   ```

   ![image-20220717025953278](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717025953278.png)

8. 获取请求Cookie值

   ```java
   @GetMapping("/test8")
   public String test8(@CookieValue("ck1") String _ga) {
       return _ga;
   }
   ```

   ![image-20220717031111652](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717031111652.png)

9. 获取整个Cookie

   ```java
   @GetMapping("/test9")
   public String test9(@CookieValue("ck1") Cookie cookie) {
       return JSON.toJSONString(cookie);
   }
   ```

   ![image-20220717031258384](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717031258384.png)

10. 矩阵变量参数

    ```java
    @GetMapping("/test10/{path1}/{path2}")
    public String test10(@MatrixVariable(value = "p1v1", pathVar = "path1") String path1Var1,
                         @MatrixVariable(value = "p1v2", pathVar = "path1") String path1Var2,
                         @MatrixVariable(value = "p2v1", pathVar = "path2") String path2Var1) {
        Map<String, Object> map = new HashMap<>();
        map.put("p1v1", path1Var1);
        map.put("p1v2", path1Var2);
        map.put("p2v1", path2Var1);
        return JSON.toJSONString(map);
    }
    ```

    SpringBoot默认移除了请求路径中的分号，因此不支持矩阵变量，首先需要进行WebMvc的设置（重写方法或重新声明组件）

    ```java
    @Configuration
    public class MyWebMvcConfig implements WebMvcConfigurer {
    
        /*
        @Override
        public void configurePathMatch(PathMatchConfigurer configurer) {
            UrlPathHelper urlPathHelper = new UrlPathHelper();
            // 不移除分号，用于开启矩阵变量接收参数
            urlPathHelper.setRemoveSemicolonContent(false);
            configurer.setUrlPathHelper(urlPathHelper);
        }
         */
    
        @Bean
        public WebMvcConfigurer webMvcConfigurer() {
            return new WebMvcConfigurer() {
                @Override
                public void configurePathMatch(PathMatchConfigurer configurer) {
                    UrlPathHelper urlPathHelper = new UrlPathHelper();
                    // 不移除分号，用于开启矩阵变量接收参数
                    urlPathHelper.setRemoveSemicolonContent(false);
                    configurer.setUrlPathHelper(urlPathHelper);
                }
            };
        }
    
    }
    ```

    ![image-20220717035208970](https://picgo-1304850123.cos.ap-guangzhou.myqcloud.com/image-20220717035208970.png)