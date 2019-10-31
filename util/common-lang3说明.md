
## StringUtils
-  处理方式：安全（所有处理前会做null判断）
-  处理对象：String,StringBuffer，StringBuilder等实现CharSequence接口的类

  
方法|说明|应用场景
---|---|---
**equals(str,str)**|比较两字符串|在不确定字符串对象是否有null时
**equalsIgnoreCase(str,str)**|比较字符串，忽视大小写
equalsAny(Str...)|比较多个字符串，忽视大小写
equalsAnyIgnoreCase(Str...)|比较多个字符串，忽视大小写
Blank| 空字符或null或内容全部为空格 |
**isBlank()** | 为blank返回true | 参数校验
**isNotBlank()** | 不为blank返回true | 参数校验
**isAnyBlank()** | 多个字符串，有一个为blank时返回true | 多参数校验
isNoneBlank(Str...) | 多个字符串，都不为blank时返回true
isAllBlank(Str...) | 多个字符串，都为blank时返回true
**defaultIfBlank()**|当字符串为blank时，返回传入的默认值 | 
Empty| 空字符或null |
isEmpty() | 为Empty返回true
isNotEmpty() | 不为Empty返回true
isAnyEmpty(Str...) | 多个字符串，有一个为Empty时返回true 
isNoneEmpty(Str...) | 多个字符串，都不为Empty时返回true
isAllEmpty(Str...) | 多个字符串，都为Empty时返回true
defaultIfEmpty()|当字符串为Empty时，返回传入的默认值
truncate(Str,max)|保留指定字段串最大长度，超过的被切除| 防止提交过长字段引起数据库报错
truncate（Str,offset,max）|保留指定字段串从下标起始后最大长度|提取身份证中的生日示例：truncate(idCard,6,8)
indexOf(Str,searchStr,star)|获取指定字符串在指定开始位后的出现位置
indexOfIgnoreCase|获取指定字符串在指定开始位后的出现位置，忽视大小写
indexOfAny(Str,searchStr..)|获取指定字符串组合是否出现过|简单密码校验，示例密码中不能出现123456
containsIgnoreCase()|检查源字符串是否包含搜索字符，忽视大小写
containsOnly(Str,Str)|检查源字符串是否只包含某些字符|密码符号限制
containsNone(Str,Str)|检查源字符串是否不包含某些字符|防止用户输入违规符号
left(str,len)|获取字符串最左边的指定长度字符|手机号段
right(str,len)|获取字符串最右边的指定长度字符|手机后4位
substringBefore(str,separator)|获取分隔符第一次出现之前的子字符串
**substringAfterLast(str,separator)**|获取分隔符最后一次出现之前的子字符串|获取上传文件类型示例：substringAfterLast("xxxx.mp4",".") = mp4
**mid(str,pol,len)**|获取字符串指定位后的指定长度字符|提取身份证中的生日示例：mid(idCard,6,8)
**substringsBetween(str,openstr,closeStr)**|搜索字符串以开始和结束标记分隔的子字符串，返回数组中所有匹配的字符串数组|提取文本括号内内容。示例substringsBetween("[中国][美国]","[","]")={"中国","美国"}
**join(obj[],separator)**|连接对象(long,int,char)数组并插入指定分隔符
**replaceIgnoreCase(str,searchStr,replaceStr)**|替换字符串，忽视大小写
**overlay(str,overlayStr,start,end)**|覆盖源字符串指定部分内容|手机号隐藏中间几位。示例overlay("13424093400","\*\*\*\*",3,7)=134\*\*\*\*3400
repeat(str,separator,repeat)|把源字符复制指定次数
**rightPad(str,size,padStr)**|设置源字符大小，余位用指定字符填充|部分银行接口对接
capitalize(Str)|源字符第一个字符串大写
**countMatches(str，subStr)**|统计字符串出现次数|计算词频
isAlpha(str)|检查源字符串是否只包含Unicode字母|姓名校验
isAlphaSpace(str)|检查源字符串是否只包含Unicode字母和空格|姓名校验
isAlphanumeric(str)|检查源字符串是否只包含Unicode字母和数字|
**isNumeric(str)**|判断字段串是否为整数(不支持+-号，小数点)
getDigits(str)|把字符串中的数字提取出来
reverse(str)|把字符串反转
**abbreviate(str,maxsize)**|指定字符串长度，剩余部份用...替代|超长数据展示
difference(str,str)|消除两个字符串中重复部分|数据去重
getCommonPrefix(str...)|获取字符串数组中起始重复部分
startsWithIgnoreCase(str,suffixStr)|判读字符串前缀，忽视大小写
endsWithAny(str,str..)|判断字符串后缀是否在限制范围内|判断上传文件类型是否在限制内。示例：endsWithAny("xxxxxx123.jpeg","jpeg","gif","jpg","png")


## RandomUtils
-  获取随机数（int,long,double,booble,float,byte）


## RandomStringUtils
-  获取随机字符串

  
方法|说明|应用场景
---|---|---
**random(int,str)**|根据设定的字符串因子，随机生成指定长度字符串|初始化密码salt


## NumberUtils
-  数字操作

  
方法|说明|应用场景
---|---|---
**toInt(str,int)**|把字符串转成数字，失败默认返回0|
**isCreatable(str)**|判读是否为数字，支持小数、+-号


## DateFormatUtils
-  格式化日期

  
方法|说明|应用场景
---|---|---
**format(date|long|calendar,pattern)**|根据指定模板格式代日期|format(Date(),"yyyy-MM-dd")


## DateUtils
-  日期操作

  
方法|说明|应用场景
---|---|---
**add\*\*(date,int)**|日期增减
**parseDate(String,pattern)**|转化字符串为日期
