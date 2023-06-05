## 聚合函数

* `AVG([DISTINCT] expr)`：（分组）对列的数据求平均值
* `SUM([DISTINCT] expr)`：（分组）对列的数据求总和
*  `MIN([DISTINCT] expr)`：（分组）求列数据的最小值
*  `MAX([DISTINCT] expr)`：（分组）求列数据的最大值
* `COUNT([DISTINCT] expr)`：（分组）求数据行数



## 流程控制

* `IF(expr1,expr2,expr3)`：如果expr1的值为TRUE，返回expr2， 否则返回expr3
* `IFNULL(expr1,expr2)`：如果expr1不为NULL，返回expr1，否则返回expr2
* `CASE WHEN when_value THEN statement_list WHEN when_value THEN statement_list .... [ELSE resultn] END`：相当于Java的if...else if...else... 
* `CASE case_value WHEN when_value THEN statement_list ELSE statement_list END CASE;`：相当于Java的switch...case...
* `BENCHMARK(count,expr)`：将表达式expr重复执行count次
* REPEAT statement_list UNTIL search_condition END REPEAT;：循环执行直至达成条件



## 数据库信息

* `VERSION()`：返回当前MySQL的版本号
* `CONNECTION_ID()`：返回当前MySQL服务器的连接数
* `DATABASE()`，`SCHEMA()`：返回MySQL命令行当前所在的数据库
* `USER()`、`CURRENT_USER()`、`SYSTEM_USER()`、`SESSION_USER()`：返回当前连接MySQL的用户名，返回结果格式为 “主机名@用户名” 
* `CHARSET(str)`：返回字符串str自变量的字符集
* `COLLATION(str)`：返回字符串str的比较规则
* `CONVERT(expr USING transcoding_name)`：将expr所使用的字符编码修改为transcoding_name
* `ROW_COUNT()`：返回上一句sql增删改影响的记录数，查询返回-1
* `FOUND_ROWS()`：上一句select或show语句的结果集的记录数



## 计算/比较

* `ABS(X)`：返回x的绝对值
* `SIGN(X)`：返回x的符号，正数为1，负数为-1，0返回0
* `PI()`：返回圆周率
* `CEIL(X)` / `CEILING(X)`：返回大于或等于某个值的最小整数
* `FLOOR(X)`：返回小于或等于某个值的最大整数
* `LEAST(value1,value2,...)`：返回列表中的最小值，列表含null则返回null
* `GREATEST(value1,value2,...)`：返回列表中的最大值，列表含null则返回null
* `MOD(N,M)`：返回X除以Y后的余数
* `RAND()`：返回0~1的随机值
* `RAND(X)`：返回0~1的随机值，其中x的值用作种子值，相同的X值会产生相同的随机数
* `ROUND(X)`：返回一个对x的值进行四舍五入后，最接近于X的整数
* `ROUND(X,D)`：返回一个对x的值进行四舍五入后最接近X的值，并保留到小数点后面Y位
* `TRUNCATE(X,D)`：返回数字x截断为y位小数的结果



## 字符串

* `ASCII(str)`：返回字符串str中的第一个字符的ASCII码值 
* `CHAR_LENGTH(str)`：返回字符串str的字符数。作用与CHARACTER_LENGTH(str)相同 
* `LENGTH(str)`：返回字符串str的字节数，和字符集有关 
* `CONCAT(str1,str2,..)`：连接str1,str2,..为一个字符串 
* `CONCAT_WS(separator,str1,str2,...)`：同CONCAT(str1,str2,..)函数，但是每个字符串之间要加上separator
* `INSERT(str,pos,len,newstr)`：将字符串str从第pos位置开始，len个字符长的子串替换为字符串newstr
* `REPLACE(str,from_str,to_str)`：用字符串to_str替换字符串str中所有出现的字符串from_str
* `UPPER(str)` / `UCASE(str)`：将字符串str的所有字母转成大写字母 
* `LOWER(str)` / `LCASE(str)`：将字符串str的所有字母转成小写字母 
* `LEFT(str,len)`：返回字符串str最左边的len个字符 
* `RIGHT(str,len)`：返回字符串str最右边的len个字符 
* `LPAD(str,len,padstr)`：用字符串padstr对str最左边进行填充，直到str的长度为len个字符 
* ``RPAD(str,len,padstr)`：用字符串padstr对str最右边进行填充，直到str的长度为len个字符 
* `TRIM(str)`：去掉字符串str开始与结尾的空格 
* `LTRIM(str)`：去掉字符串str左侧的空格 
* `RTRIM(str)`：去掉字符串str右侧的空格 
* `TRIM([{BOTH | LEADING | TRAILING} [remstr] FROM] str)`：去掉str两边/开始/结尾处的remstr
* `REPEAT(str,count)`：返回str重复count次的结果 
* `SPACE(N)`：返回N个空格 
* `STRCMP(expr1,expr2)`：比较字符串expr1,expr2的ASCII码值的大小 
* `SUBSTR(str,pos,len)`：返回从字符串str的pos位置其len个字符，作用与`SUBSTRING(str,pos,len)`、 `MID(str,pos,len)`相同 
* `LOCATE(substr,str)`：返回字符串substr在字符串str中首次出现的位置，作用于`POSITION(substr IN str)`、`INSTR(str,substr)`相同。未找到，返回0 
* `ELT(N,str1,str2,str3,...)`：返回指定位置的字符串，如果N=1，则返回str1，如果N=2，则返回str2...，与`MAKE_SET(bits,str1,str2,...)`作用相同
* `FIELD(str,str1,str2,str3,...)`：返回字符串str在字符串列表中第一次出现的位置
* `FIND_IN_SET(str,strlist)`：返回字符串str在字符串strlist中出现的位置。其中，字符串strlist是一个以逗号分隔的字符串 
* `REVERSE(str)`：返回str反转后的字符串 
* `NULLIF(expr1,expr2)`：比较两个字符串，如果expr1与expr2相等，则返回NULL，否则返回expr1



## 日期和时间

* `CURDATE()` / `CURRENT_DATE()`：返回当前日期，只包含年、 月、日
* `CURTIME()` / `CURRENT_TIME()`：返回当前时间，只包含时、 分、秒 
* `NOW()` / `SYSDATE()` / `CURRENT_TIMESTAMP()` / `LOCALTIME()` / `LOCALTIMESTAMP()`：返回当前系统日期和时间 
* `UTC_DATE()`：返回UTC（世界标准时间）日期 
* `UTC_TIME()`：返回UTC（世界标准时间）时间
* `UNIX_TIMESTAMP()`：以UNIX时间戳的形式返回当前时间
* `UNIX_TIMESTAMP(date)`：将时间date以UNIX时间戳的形式返回
* `FROM_UNIXTIME(unix_timestamp)`：将UNIX时间戳转换为普通格式的时间
* `FROM_UNIXTIME(unix_timestamp,format)`：将UNIX时间戳转换为指定格式的时间
* `YEAR(date)` / `MONTH(date)` / `DAY(date)`：返回具体的日期值 
* `HOUR(time)` / `MINUTE(time)` / `SECOND(time)`：返回具体的时间值 
* `MONTHNAME(date)`：返回月份：January，... 
* `DAYNAME(date)`：返回星期几：MONDAY，TUESDAY，...，SUNDAY 
* `WEEKDAY(date)`：返回周几，注意，周1是0，周2是1，...，周日是6 
* `QUARTER(date)`：返回日期对应的季度，范围为1～4 
* `WEEK(date)` / `WEEKOFYEAR(date)`：返回一年中的第几周 
* `DAYOFYEAR(date)`：返回日期是一年中的第几天
* `DAYOFMONTH(date)`：返回日期位于所在月份的第几天
* `DAYOFWEEK(date)`：返回周几，注意：周日是1，周一是2，...，周六是7
* `EXTRACT(unit FROM date)`：返回指定日期中的特定部分或组合，如SECOND、HOUR、YEAR、...、DAY_HOUR
* `TIME_TO_SEC(time)`：将 time 转化为秒并返回结果值。转化的公式为： 小时*3600+分钟 *60+秒 
* `SEC_TO_TIME(seconds)`：将 seconds 描述转化为包含小时、分钟和秒的时间
* `DATE_ADD(date,INTERVAL expr unit)` / `ADDDATE(date,INTERVAL expr unit)`：返回与给定日期时间date相差INTERVAL时间段的日期时间（单位为unit）
* `DATE_SUB(date,INTERVAL expr unit)` / `SUBDATE(date,INTERVAL expr unit)`：返回与date相差INTERVAL时间间隔的 日期（单位为unit）
* `ADDTIME(expr1,expr2)`：返回expr1加上expr2的时间。当expr2为一个数字时，代表的是秒，可以为负数 
* `SUBTIME(expr1,expr2)`：返回expr1减去expr2后的时间。当expr2为一个数字时，代表的是秒 ，可以为负数 
* `DATEDIFF(expr1,expr2)`：返回expr1- expr2的日期间隔天数
* `TIMEDIFF(expr1,expr2)`：返回expr1- expr2的时间间隔
* `FROM_DAYS(N)`：返回从0000年1月1日起，N天以后的日期
* `TO_DAYS(date)`：返回日期date距离0000年1月1日的天数
* `LAST_DAY(date)`：返回date所在月份的最后一天的日期
* `MAKEDATE(year,dayofyear)`：返回给定年份year的第dayofyear天的日期
* `MAKETIME(hour,minute,second)`：将给定的小时、分钟和秒组合成时间并返回
* `PERIOD_ADD(P,N)`：返回P加上N后的时间，如20221203+3=20221206，202212+3=202303
* `PERIOD_DIFF(P1,P2)`：计算P1-P2，参数格式为%YY%m%s或%YY%m，结果单位为%s或%m



## 格式化

* `FORMAT(X,D)`：返回对数字X进行格式化后的结果数据，保留小数D位（四舍五入）
* `DATE_FORMAT(date,format)`：按照字符串format格式化日期date值 
* `TIME_FORMAT(time,format)`：按照字符串format格式化时间time值
* `GET_FORMAT({DATE|TIME|DATETIME}, {'EUR'|'USA'|'JIS'|'ISO'|'INTERNAL'})`：返回日期字符串的显示格式
* `STR_TO_DATE(str, format)`：按照字符串format对str进行解析，解析为一个日期，%Y-%m-%d %H:%i:%s



## 类型转换

* `CAST(any AS datatype)`：将任何类型的值转换为指定类型的值，目标类型可能是BINARY、CHAR、DATE、DATETIME、TIME、DECIMAL、SIGNED、UNSIGNED，如不能进行转换则只为null
* `CONVERT(str, cast_datatype)`：将字符串转换为指定类型的值，大多数情况下与CAST函数功能重叠
* `CONVERT(str USING charset)`：将字符串转换为指定的字符集



## 指数/对数

* `SQRT(X)`：返回x的平方根。当X的值为负数时，返回NULL
* `POW(X,Y)` / `POWER(X,Y)`：返回x的y次方 
* `EXP(X)`：返回e的X次方，其中e是一个常数，2.718281828459045
* `LN(X)` / `LOG(X)`：返回以e为底的X的对数，当X <= 0 时，返回的结果为NULL 
* `LOG10(X)`：返回以10为底的X的对数，当X <= 0 时，返回的结果为NULL 
* `LOG2(X)`：返回以2为底的X的对数，当X <= 0 时，返回NULL
* `LOG(B,X)`：以B为底X的对数，当X <= 0 时，返回NULL



## 进制

* `BIN(X)`：返回X的二进制编码 
* `HEX(X)`：返回X的十六进制编码 
* `OCT(X)`：返回X的八进制编码 
* `CONV(N,from_base,to_base)`：将N从from_base进制转为to_base进制



## 角度

* `RADIANS(X)`：将角度转化为弧度，其中，参数X为角度值
* `DEGREES(X)`：将弧度转化为角度，其中，参数X为弧度值



## IP地址

* `INET_ATON(expr)`：将以点分隔的IP地址转化为一个数字，值为字符串类型
* `INET_NTOA(expr)`：将数字形式的IP地址转化为以点分隔的IP地址，值为字符串类型



## 三角函数

* `SIN(X)`：返回X的正弦值，其中，参数X为弧度值 
* `ASIN(X)`：返回X的反正弦值，即获取正弦为X的值。如果X的值不在-1到1之间，则返回NULL 
* `COS(X)`：返回X的余弦值，其中，参数X为弧度值 
* `ACOS(X)`：返回X的反余弦值，即获取余弦为x的值。如果X的值不在-1到1之间，则返回NULL 
* `TAN(X)`：返回X的正切值，其中，参数X为弧度值 
* `ATAN(X)`：返回X的反正切值，即返回正切值为X的值 
* `ATAN2(Y,X)`：返回两个参数的反正切值 
* `COT(X)`：返回X的余切值，其中，X为弧度值



## 编码/压缩

* `PASSWORD(str)`：返回字符串str的哈希加密，41位长的字符串。加密结果不可逆，常用于用户的密码加密
* `MD5(str)`：返回字符串str的md5加密后的值，也是一种加密方式。若参数为 NULL，则会返回NULL
* `SHA(str)`：从原明文密码str计算并返回加密后的密码字符串，当参数为NULL时，返回NULL。SHA加密算法比MD5更加安全
* `ENCODE(str,pass_str)`：返回使用pass_str作为加密密码加密str
* `DECODE(str,pass_str)`：返回使用pass_str作为加密密码解密str
* `TO_BASE64(str)`：返回str的Base64编码
* `FROM_BASE64(str)`：返回str的Base64的解码结果



## 日期格式化附录

| 格式 | 描述                                           |
| ---- | ---------------------------------------------- |
| %a   | 缩写星期名                                     |
| %b   | 缩写月名                                       |
| %c   | 月，数值                                       |
| %D   | 带有英文前缀的月中的天                         |
| %d   | 月的天，数值(00-31)                            |
| %e   | 月的天，数值(0-31)                             |
| %f   | 微秒                                           |
| %H   | 小时 (00-23)                                   |
| %h   | 小时 (01-12)                                   |
| %I   | 小时 (01-12)                                   |
| %i   | 分钟，数值(00-59)                              |
| %j   | 年的天 (001-366)                               |
| %k   | 小时 (0-23)                                    |
| %l   | 小时 (1-12)                                    |
| %M   | 月名                                           |
| %m   | 月，数值(00-12)                                |
| %p   | AM 或 PM                                       |
| %r   | 时间，12-小时（hh:mm:ss AM 或 PM）             |
| %S   | 秒(00-59)                                      |
| %s   | 秒(00-59)                                      |
| %T   | 时间, 24-小时 (hh:mm:ss)                       |
| %U   | 周 (00-53) 星期日是一周的第一天                |
| %u   | 周 (00-53) 星期一是一周的第一天                |
| %V   | 周 (01-53) 星期日是一周的第一天，与 %X 使用    |
| %v   | 周 (01-53) 星期一是一周的第一天，与 %x 使用    |
| %W   | 星期名                                         |
| %w   | 周的天 （0=星期日, 6=星期六）                  |
| %X   | 年，其中的星期日是周的第一天，4 位，与 %V 使用 |
| %x   | 年，其中的星期一是周的第一天，4 位，与 %v 使用 |
| %Y   | 年，4 位                                       |
| %y   | 年，2 位                                       |
