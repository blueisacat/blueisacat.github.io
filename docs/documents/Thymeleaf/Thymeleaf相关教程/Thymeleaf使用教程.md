# Thymeleaf使用教程

!!! tip 使用流程

    1. 编写模板
    2. 准备数据
    3. 渲染模板

!!! note 模板标签

    1. 模板使用 ***${...}*** 标签获取变量
    2. 模板使用 ***[(...)]*** 标签输出参数
    3. 模板使用 ***[# ...][/]*** 标签实现迭代和条件评估

## 1 示例

* 模板

    ```text
    Dear [(${customer.name})],

    This is the list of our products:

    [# th:each="prod : ${products}"]
        - [(${prod.name})]. Price: [(${prod.price})] EUR/kg
    [/]

    Thanks,
        The Thymeleaf Shop
    ```

* 数据

    ```json
    {
        "customer": {
            "name": "Mary Ann Blueberry"
        },
        "products": [
            {
                "name": "Apricots",
                "price": 1.12
            },
            {
                "name": "Bananas",
                "price": 1.78
            },
            {
                "name": "Apples",
                "price": 0.85
            },
            {
                "name": "Watermelon",
                "price": 1.91
            }
        ]
    }
    ```

* 渲染

    ```text
    Dear Mary Ann Blueberry,

    This is the list of our products:

        - Apricots. Price: 1.12 EUR/kg
        - Bananas. Price: 1.78 EUR/kg
        - Apples. Price: 0.85 EUR/kg
        - Watermelon. Price: 1.91 EUR/kg

    Thanks,
        The Thymeleaf Shop
    ```

## 2 Literals（字面量）

### 2.1 Text

* 模板

    ```text
    text is [(${text})]
    ```

* 数据

    ```json
    {
        "text": "hello world"
    }
    ```

* 渲染

    ```text
    text is hello world
    ```

### 2.2 Number

* 模板

    ```text
    number is [(${number})]
    ```

* 数据

    ```json
    {
        "number": 123456
    }
    ```

* 渲染

    ```text
    number is 123456
    ```

### 2.3 Boolean

* 模板

    ```text
    boolean is [(${boolean})]
    ```

* 数据

    ```json
    {
        "boolean": true
    }
    ```

* 渲染

    ```text
    boolean is true
    ```

### 2.4 Null

* 模板

    ```text
    null is [(${text})]
    ```

* 数据

    ```json
    {
        "text": null
    }
    ```

* 渲染

    ```text
    null is 
    ```

## 3 ArithmeticOperations(算术运算)

* 模板

    ```text
    [(${number})] + [(${number})] = [(${number} + ${number})]
    [(${number})] - [(${number})] = [(${number} - ${number})]
    [(${number})] * [(${number})] = [(${number} * ${number})]
    [(${number})] / [(${number})] = [(${number} / ${number})]
    [(${number})] % [(${number})] = [(${number} % ${number})]
    ```

* 数据

    ```json
    {
        "number": 7
    }
    ```

* 渲染

    ```text
    7 + 7 = 14
    7 - 7 = 0
    7 * 7 = 49
    7 / 7 = 1
    7 % 7 = 0
    ```

## 4 ComparatorsAndEquality(比较器与等式)

* 模板

    ```text
    [(${a})] > [(${b})] : [(${a} > ${b})]
    [(${a})] >= [(${b})] : [(${a} >= ${b})]
    [(${a})] < [(${b})] : [(${a} < ${b})]
    [(${a})] <= [(${b})] : [(${a} <= ${b})]
    [(${a})] != [(${b})] : [(${a} != ${b})]
    [(${a})] == [(${b})] : [(${a} == ${b})]
    ![(${c})] : [(!${c})]
    (${c})] and (${d})] : [(${c} and ${d})]
    (${c})] or (${d})] : [(${c} or ${d})]
    ```

* 数据

    ```json
    {
        "a": 1,
        "b": 2,
        "c": true,
        "d": false
    }
    ```

* 渲染

    ```text
    1 > 2 : false
    1 >= 2 : false
    1 < 2 : true
    1 <= 2 : true
    1 != 2 : true
    1 == 2 : false
    !true : false
    true and false : false
    true or false : true
    ```

## 5 ConditionalExpressions(条件表达式)

* 模板

    ```text
    boolean ? 1 : 0 → [(${boolean} ? 1 : 0)]
    boolean ? 1 → [(${boolean} ? 1)]
    text ?: 1 → [(${text} ?: 1)]
    boolean ?: 1 → [(${boolean} ?: 1)]
    ```

* 数据

    ```json
    {
        "boolean": true,
        "text": null
    }
    ```

* 渲染

    ```text
    boolean ? 1 : 0 → 1
    boolean ? 1 → 1
    text ?: 1 → 1
    boolean ?: 1 → true
    ```

## 6 Iteration(迭代)

### 6.1 list遍历/取值

* 模板

    ```text
    [# th:each="data,stat : ${list}"]
        [(${stat})]
        [(${data})]
    [/]
    [(${list[0]})]
    ```

* 数据

    ```json
    {
        "list": [
            "a",
            "b",
            "c",
            "d"
        ]
    }
    ```

* 渲染

    ```text
    {index = 0, count = 1, size = 4, current = a}
    a
    {index = 1, count = 2, size = 4, current = b}
    b
    {index = 2, count = 3, size = 4, current = c}
    c
    {index = 3, count = 4, size = 4, current = d}
    d
    a
    ```

### 6.2 map使用Key获取Value

* 模板

    ```text
    [(${map[k1]})]
    [(${map.get(k2)})]
    [(${map.k3})]
    [(${map.k4})]
    ```

* 数据

    ```json
    {
        "map": {
            "k1": "v1",
            "k2": "v2",
            "k3": "v3",
            "k4": "v4"
        }
    }
    ```

* 渲染

    ```text
    v1
    v2
    v3
    v4
    ```

### 6.3 map遍历entry,获取stat、key、value

* 模板

    ```text
    [# th:each="entry,stat : ${map}"]
    [(${stat})]
    [(${entry.key})]
    [(${entry.value})]
    [/]
    ```

* 数据

    ```json
    {
        "map": {
            "k1": "v1",
            "k2": "v2",
            "k3": "v3",
            "k4": "v4"
        }
    }
    ```

* 渲染

    ```text
    {index = 0, count = 1, size = 4, current = k1=v1}
    k1
    v1
    {index = 1, count = 2, size = 4, current = k2=v2}
    k2
    v2
    {index = 2, count = 3, size = 4, current = k3=v3}
    k3
    v3
    {index = 3, count = 4, size = 4, current = k4=v4}
    k4
    v4
    ```

### 6.4 map遍历value，获取stat、value

* 模板

    ```text
    [# th:each="value,stat : ${map.values}"]
    [(${stat})]
    [(${value})]
    [/]
    ```

* 数据

    ```json
    {
        "map": {
            "k1": "v1",
            "k2": "v2",
            "k3": "v3",
            "k4": "v4"
        }
    }
    ```

* 渲染

    ```text
    {index = 0, count = 1, size = 4, current = v1}
    v1
    {index = 1, count = 2, size = 4, current = v2}
    v2
    {index = 2, count = 3, size = 4, current = v3}
    v3
    {index = 3, count = 4, size = 4, current = v4}
    v4
    ```

### 6.5 map遍历key，获取stat、key、value

* 模板

    ```text
    [# th:each="key,stat : ${map.keys}"]
    [(${stat})]
    [(${key})]
    [(${map.get(key)})]
    [/]
    ```

* 数据

    ```json
    {
        "map": {
            "k1": "v1",
            "k2": "v2",
            "k3": "v3",
            "k4": "v4"
        }
    }
    ```

* 渲染

    ```text
    {index = 0, count = 1, size = 4, current = k1}
    k1
    v1
    {index = 1, count = 2, size = 4, current = k2}
    k2
    v2
    {index = 2, count = 3, size = 4, current = k3}
    k3
    v3
    {index = 3, count = 4, size = 4, current = k4}
    k4
    v4
    ```

## 7 ConditionalEvaluation(条件评估)

### 7.1 if/unless

* 模板

    ```text
    [# th:if=${boolean}]
    boolean is true
    [/]
    [# th:unless=${boolean}]
    boolean is false
    [/]
    ```

* 数据

    ```json
    {
        "boolean": false
    }
    ```

* 渲染

    ```text
    boolean is false
    ```

### 7.2 switch/case

* 模板

    ```text
    [# th:switch=${boolean}]
    [# th:case=true]
    boolean is true
    [/]
    [# th:case=false]
    boolean is false
    [/]
    [/]
    ```

* 数据

    ```json
    {
        "boolean": false
    }
    ```

* 渲染

    ```text
    boolean is false
    ```

## 8 ExpressionUtilityObjects(表达式实用程序对象)

### 8.1 Dates

* 模板

    ```text
    默认格式 : [(${#dates.format(date)})]
    ISO8601格式 : [(${#dates.formatISO(date)})]
    自定义格式 : [(${#dates.format(date, yyyy-MM-dd HH:mm:ss)})]
    日 : [(${#dates.day(date)})]
    月 : [(${#dates.month(date)})]
    月名称 : [(${#dates.monthName(date)})]
    月简称 : [(${#dates.monthNameShort(date)})]
    年 : [(${#dates.year(date)})]
    星期（周日-周六） : [(${#dates.dayOfWeek(date)})]
    星期名称 : [(${#dates.dayOfWeekName(date)})]
    星期简称 : [(${#dates.dayOfWeekNameShort(date)})]
    时 : [(${#dates.hour(date)})]
    分 : [(${#dates.minute(date)})]
    秒 : [(${#dates.second(date)})]
    毫秒 : [(${#dates.millisecond(date)})]
    创建时间(年月日) : [(${#dates.format(#dates.create(2000,1,1))})]
    创建时间(年月日时分) : [(${#dates.format(#dates.create(2000,1,1,1,1))})]
    创建时间(年月日时分秒) : [(${#dates.format(#dates.create(2000,1,1,1,1,1))})]
    创建时间(年月日时分秒毫秒) : [(${#dates.format(#dates.create(2000,1,1,1,1,1,1))})]
    当前时间 : [(${#dates.format(#dates.createNow())})]
    当前日期 : [(${#dates.format(#dates.createToday())})]
    ```

* 数据

    ```json
    {
        "date": "Jan 11, 2022 10:30:21 AM"
    }
    ```

* 渲染

    ```text
    默认格式 : 2022年1月11日 上午10时30分21秒
    ISO8601格式 : 2022-01-11T10:30:21.602+08:00
    自定义格式 : 2022-01-11 10:30:21
    日 : 11
    月 : 1
    月名称 : 一月
    月简称 : 一月
    年 : 2022
    星期（周日-周六） : 3
    星期名称 : 星期二
    星期简称 : 星期二
    时 : 10
    分 : 30
    秒 : 21
    毫秒 : 602
    创建时间(年月日) : 2000年1月1日 上午12时00分00秒
    创建时间(年月日时分) : 2000年1月1日 上午01时01分00秒
    创建时间(年月日时分秒) : 2000年1月1日 上午01时01分01秒
    创建时间(年月日时分秒毫秒) : 2000年1月1日 上午01时01分01秒
    当前时间 : 2022年1月11日 上午10时42分50秒
    当前日期 : 2022年1月11日 上午12时00分00秒
    ```

### 8.2 Calendars

* 模板

    ```text
    默认格式 : [(${#calendars.format(cal)})]
    ISO8601格式 : [(${#calendars.formatISO(cal)})]
    自定义格式 : [(${#calendars.format(cal, yyyy-MM-dd HH:mm:ss)})]
    日 : [(${#calendars.day(cal)})]
    月 : [(${#calendars.month(cal)})]
    月名称 : [(${#calendars.monthName(cal)})]
    月简称 : [(${#calendars.monthNameShort(cal)})]
    年 : [(${#calendars.year(cal)})]
    星期（周日-周六） : [(${#calendars.dayOfWeek(cal)})]
    星期名称 : [(${#calendars.dayOfWeekName(cal)})]
    星期简称 : [(${#calendars.dayOfWeekNameShort(cal)})]
    时 : [(${#calendars.hour(cal)})]
    分 : [(${#calendars.minute(cal)})]
    秒 : [(${#calendars.second(cal)})]
    毫秒 : [(${#calendars.millisecond(cal)})]
    创建时间(年月日) : [(${#calendars.format(#calendars.create(2000,1,1))})]
    创建时间(年月日时分) : [(${#calendars.format(#calendars.create(2000,1,1,1,1))})]
    创建时间(年月日时分秒) : [(${#calendars.format(#calendars.create(2000,1,1,1,1,1))})]
    创建时间(年月日时分秒毫秒) : [(${#calendars.format(#calendars.create(2000,1,1,1,1,1,1))})]
    当前时间 : [(${#calendars.format(#calendars.createNow())})]
    当前日期 : [(${#calendars.format(#calendars.createToday())})]
    ```

* 数据

    ```json
    {
        "cal": {
            "year": 2022,
            "month": 0,
            "dayOfMonth": 11,
            "hourOfDay": 10,
            "minute": 30,
            "second": 21
        }
    }
    ```

* 渲染

    ```text
    默认格式 : 2022年1月11日 上午10时41分25秒
    ISO8601格式 : 2022-01-11T10:41:25.492+08:00
    自定义格式 : 2022-01-11 10:41:25
    日 : 11
    月 : 1
    月名称 : 一月
    月简称 : 一月
    年 : 2022
    星期（周日-周六） : 3
    星期名称 : 星期二
    星期简称 : 星期二
    时 : 10
    分 : 41
    秒 : 25
    毫秒 : 492
    创建时间(年月日) : 2000年1月1日 上午12时00分00秒
    创建时间(年月日时分) : 2000年1月1日 上午01时01分00秒
    创建时间(年月日时分秒) : 2000年1月1日 上午01时01分01秒
    创建时间(年月日时分秒毫秒) : 2000年1月1日 上午01时01分01秒
    当前时间 : 2022年1月11日 上午10时41分25秒
    当前日期 : 2022年1月11日 上午12时00分00秒
    ```

### 8.3 Numbers

* 模板

    ```text
    最小整数位数5(不足补0): [(${#numbers.formatInteger(num,5)})]
    最小整数位数5(不足补0),千分位分隔符. : [(${#numbers.formatInteger(num,5,POINT)})]
    最小整数位数5(不足补0),千分位分隔符, : [(${#numbers.formatInteger(num,5,COMMA)})]
    最小整数位数5(不足补0),千分位分隔符  : [(${#numbers.formatInteger(num,5,WHITESPACE)})]
    最小整数位数5(不足补0),小数位数5(不足补0) : [(${#numbers.formatDecimal(num,5,5)})]
    最小整数位数5(不足补0),千分位分隔符,,小数位数5(不足补0),小数分隔符  : [(${#numbers.formatDecimal(num,5,COMMA,5,WHITESPACE)})]
    货币格式 : [(${#numbers.formatCurrency(num)})]
    最小整数位5(不足补0),小数位数5(不足补0),百分比格式 : [(${#numbers.formatPercent(num,5,5)})]
    生成序列 :
    [# th:each="number : ${#numbers.sequence(1,10)}"]
    [(${number})]
    [/]
    生成序列,指定步长 : 
    [# th:each="number : ${#numbers.sequence(1,10,2)}"]
    [(${number})]
    [/]
    ```

* 数据

    ```json
    {
        "num": 123.456
    }
    ```

* 渲染

    ```text
    最小整数位数5(不足补0): 00123
    最小整数位数5(不足补0),千分位分隔符. : 00.123
    最小整数位数5(不足补0),千分位分隔符, : 00,123
    最小整数位数5(不足补0),千分位分隔符  : 00 123
    最小整数位数5(不足补0),小数位数5(不足补0) : 00123.45600
    最小整数位数5(不足补0),千分位分隔符,,小数位数5(不足补0),小数分隔符  : 00,123 45600
    货币格式 : ￥123.46
    最小整数位5(不足补0),小数位数5(不足补0),百分比格式 : 12,345.60000%
    生成序列 : 
    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    生成序列,指定步长 : 
    1
    3
    5
    7
    9
    ```

### 8.4 Strings

* 模板

    ```text
    空安全 : [(${#strings.toString(null)})]
    是否为空 : [(${#strings.isEmpty(null)})]
    默认值 : [(${#strings.defaultString(null,text)})]
    是否包含 : [(${#strings.contains(text,hello)})]
    是否包含(忽略大小写) : [(${#strings.containsIgnoreCase(text,Hello)})]
    是否开头 : [(${#strings.startsWith(text,hello)})]
    是否结尾 : [(${#strings.endsWith(text,hello)})]
    索引 : [(${#strings.indexOf(text,world)})]
    截取 : [(${#strings.substring(text,3,5)})]
    截取(指定字符之后) : [(${#strings.substringAfter(text,hello)})]
    截取(指定字符之前) : [(${#strings.substringBefore(text,world)})]
    替换 : [(${#strings.replace(text,hello,goodbye)})]
    前缀 : [(${#strings.prepend(text,prefix)})]
    后缀 : [(${#strings.append(text,suffix)})]
    大写 : [(${#strings.toUpperCase(text)})]
    小写 : [(${#strings.toLowerCase(text)})]
    拼接(数组) : [(${#strings.arrayJoin(#numbers.sequence(1,4),,)})]
    拼接(列表) : [(${#strings.listJoin(#lists.toList(#numbers.sequence(1,4)),,)})]
    拼接(集合) : [(${#strings.setJoin(#sets.toSet(#numbers.sequence(1,4)),,)})]
    拆分(数组) : 
    [# th:each=\"data :${#strings.arraySplit(1,2,3,4,,)}\"]
    [(${data})]
    [/]
    拆分(列表) : 
    [# th:each=\"data :${#strings.listSplit(1,2,3,4,,)}\"]
    [(${data})]
    [/]
    拆分(集合) : 
    [# th:each=\"data :${#strings.setSplit(1,2,3,4,,)}\"]
    [(${data})]
    [/]
    去除首尾空格 : [(${#strings.trim(  hello world     )})]
    长度 : [(${#strings.length(text)})]
    超过长度5,裁剪为... : [(${#strings.abbreviate(text,5)})]
    首字母大写 : [(${#strings.capitalize(text)})]
    首字母小写 : [(${#strings.unCapitalize(text)})]
    单词首字母大写 : [(${#strings.capitalizeWords(text)})]
    单词首字母大写,指定分隔符 : [(${#strings.capitalizeWords(text, )})]
    空安全,相等 : [(${#strings.equals(text, null)})]
    空安全,忽略大小写相等 : [(${#strings.equalsIgnoreCase(text, null)})]
    拼接 : [(${#strings.concat(a,b,c)})]
    随机5位字符串 : [(${#strings.randomAlphanumeric(5)})]
    ```

* 数据

    ```json
    {
        "text": "hello world"
    }
    ```

* 渲染

    ```text
    空安全 : 
    是否为空 : true
    默认值 : hello world
    是否包含 : true
    是否包含(忽略大小写) : true
    是否开头 : true
    是否结尾 : false
    索引 : 6
    截取 : lo
    截取(指定字符之后) :  world
    截取(指定字符之前) : hello 
    替换 : goodbye world
    前缀 : prefixhello world
    后缀 : hello worldsuffix
    大写 : HELLO WORLD
    小写 : hello world
    拼接(数组) : 1,2,3,4
    拼接(列表) : 1,2,3,4
    拼接(集合) : 1,2,3,4
    拆分(数组) : 
    1
    2
    3
    4
    拆分(列表) : 
    1
    2
    3
    4
    拆分(集合) : 
    1
    2
    3
    4
    去除首尾空格 : hello world
    长度 : 11
    超过长度5,裁剪为... : he...
    首字母大写 : Hello world
    首字母小写 : hello world
    单词首字母大写 : Hello World
    单词首字母大写,指定分隔符 : Hello World
    空安全,相等 : false
    空安全,忽略大小写相等 : false
    拼接 : abc
    随机5位字符串 : RZ1FU
    ```

### 8.5 Booleans

* 模板

    ```text
    是否为真 : [(${#bools.isTrue(boolean)})]
    是否为假 : [(${#bools.isFalse(boolean)})]
    ```

* 数据

    ```json
    {
        "boolean": true
    }
    ```

* 渲染

    ```text
    是否为真 : true
    是否为假 : false
    ```

### 8.6 Arrays

* 模板

    ```text
    转数组 : [(${#arrays.toArray(#numbers.sequence(1,4))})]
    [# th:each="data : ${#arrays.toArray(#numbers.sequence(1,4))}"]
    [(${data})]
    [/]
    数组大小 : [(${#arrays.length(#arrays.toArray(#numbers.sequence(1,4)))})]
    是否为空 : [(${#arrays.isEmpty(#arrays.toArray(#numbers.sequence(1,4)))})]
    是否包含元素 : [(${#arrays.contains(#arrays.toArray(#numbers.sequence(1,4)),1)})]
    ```

* 渲染

    ```text
    转数组 : 
    1
    2
    3
    4
    数组大小 : 4
    是否为空 : false
    是否包含元素 : true
    ```

### 8.7 Lists

* 模板

    ```text
    转列表 : 
    [# th:each="data : ${#lists.toList(#numbers.sequence(1,4))}"]
    [(${data})]
    [/]
    列表大小 : [(${#lists.size(#lists.toList(#numbers.sequence(1,4)))})]
    是否为空 : [(${#lists.isEmpty(#lists.toList(#numbers.sequence(1,4)))})]
    是否包含元素 : [(${#lists.contains(#lists.toList(#numbers.sequence(1,4)),1)})]
    排序 : [(${#lists.sort(#lists.toList(#numbers.sequence(1,4)))})]
    ```

* 渲染

    ```text
    转列表 : 
    1
    2
    3
    4
    列表大小 : 4
    是否为空 : false
    是否包含元素 : true
    排序 : [1, 2, 3, 4]
    ```

### 8.8 Sets

* 模板

    ```text
    转集合 : 
    [# th:each="data : ${#sets.toSet(#numbers.sequence(1,4))}"]
    [(${data})]
    [/]
    集合大小 : [(${#sets.size(#sets.toSet(#numbers.sequence(1,4)))})]
    是否为空 : [(${#sets.isEmpty(#sets.toSet(#numbers.sequence(1,4)))})]
    是否包含元素 : [(${#sets.contains(#sets.toSet(#numbers.sequence(1,4)),1)})]
    ```

* 渲染

    ```text
    转集合 : 
    1
    2
    3
    4
    集合大小 : 4
    是否为空 : false
    是否包含元素 : true
    ```

### 8.9 Maps

* 模板

    ```text
    映射大小 : [(${#maps.size(map)})]
    是否为空 : [(${#maps.isEmpty(map)})]
    是否包含key : [(${#maps.containsKey(map,k1)})]
    是否包含value : [(${#maps.containsValue(map,v1)})]
    ```

* 数据

    ```json
    {
        "map": {
            "k1": "v1",
            "k2": "v2",
            "k3": "v3"
        }
    }
    ```

* 渲染

    ```text
    映射大小 : 3
    是否为空 : false
    是否包含key : true
    是否包含value : true
    ```

### 8.10 Aggregates

* 模板

    ```text
    求和 : [(${#aggregates.sum(#numbers.sequence(1,4))})]\\
    求平均值 : [(${#aggregates.avg(#numbers.sequence(1,4))})]
    ```

* 渲染

    ```text
    求和 : 10
    求平均值 : 2.5
    ```

### 8.11 IDs

* 模板

    ```text
    计数器值 : [(${#ids.seq(someId)})]
    下一个值 : [(${#ids.next(someId)})]
    上一个值 : [(${#ids.prev(someId)})]
    ```

* 渲染

    ```text
    计数器值 : someId1
    下一个值 : someId2
    上一个值 : someId1
    ```
