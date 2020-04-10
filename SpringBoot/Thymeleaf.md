# Thymeleaf使用

## 1.导入命名空间

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org">
```

## 2.文本标签

th:html属性；用来替换原生的属性值

```html
<!-- p标签中会显示th:text中获得的值
	th:utext中不会转义特殊字符
	可以用于获取配置文件中值,例如：国际化
-->
<p th:text="#{home.welcome}">Welcome to our grocery store!</p>
<p th:utext="#{home.welcome}">Welcome to our grocery store!</p>

<div th:text="2+2">该标签显示计算后的结果</div>	

<!-- Link URL Expressions: th:href会在输入的路径前面加上项目名 -->
<link rel="stylesheet" type="text/css" media="all" href="../../css/gtvg.css" th:href="@{/css/gtvg.css}" />

```

## 3.表达式 -Simple expressions:

```java
@RequestMapping("/test")
	public String test(HttpServletRequest request) {
		request.setAttribute("diao","yuhang");
		request.getSession().setAttribute("zhou","ying");
		request.getServletContext().setAttribute("context","servletContext");
		return "Thymeleaf";
	}
```



```html
<!--Expression Basic Objects基本对象
	Variable Expressions: ${} 获取变量值；OGNL
	#ctx : the context object.
    #vars: the context variables.
    #locale : the context locale.
    #request : (only in Web Contexts) the HttpServletRequest object.
		${#request.getAttribute('foo')}
        ${#request.getParameter('foo')}
        ${#request.getContextPath()}
        ${#request.getRequestName()}
        ...
    #response : (only in Web Contexts) the HttpServletResponse object.
    #session : (only in Web Contexts) the HttpSession object.
        ${session.foo} // Retrieves the session atttribute 'foo'
        ${session.size()}
        ${session.isEmpty()}
        ${session.containsKey('foo')}
    #servletContext : (only in Web Contexts) the ServletContext object.
-->

<body>
	request: <h3 th:text="${diao}"></h3>
	request: <h3 th:text="${#request.getAttribute('diao')}"></h3>
	serssion:<h3 th:text="${session.zhou}"></h3>
	serssion:<h3 th:text="${session.isEmpty()}"></h3>
	serssion:<h3 th:text="${session.size()}"></h3>
	serssion:<h3 th:text="${session.containsKey('diao')}"></h3>
	
	servletContext: <h3 th:text="${#servletContext.getAttribute('context')}"></h3>	
</body>
```

#### 抽取公共标签，引入公共标签

```html
<!-- Fragment Expressions: ~{...}
th:replace --将公共的片段替换掉声明的标签中
th:insert -- 将公共的片段插入到声明的标签中
th:include -- 将公共的片段的内容包含进当前声明的标签中
-->

1、抽取声明公共片段
<div th:fragment="copy">
&copy; 2011 The Good Thymes Virtual Grocery
</div>

2、引入公共片段 footer是html文件的名称
<div th:insert="~{footer :: copy}"></div>
例如：common为文件夹的名称
<div th:replace="~{common/bar :: topbar }"></div>

3、默认效果：
insert的公共片段在div标签中
如果使用th:insert等属性进行引入，可以不用写~{}：
行内写法可以加上：[[~{}]];[(~{})]；
```





## 4、行内写法[[ ]]、[( )]的区别

1. `[[ ]]:不会转义特殊字符`
2. `[[ ]]:会转义特殊字符`

## 5、条件判断标签th:if th:unless

```html
<!-- 可以在th:if中进行条件判断 -->
<div th:if="${zhou} == null" >条件成立,显示该条信息</div>
<div th:unless="${diao} == '<h1>yuhang</h1>'" >条件不成立,显示该条信息</div>
```

## 6、声明局部变量th:with

```html
<div th:with="variable=${10%2==0?'可以整除':'不可以整除'}">
		<div th:text="${variable}">获取声明的变量</div>
</div>
```

## 7、设置属性的值Setting Attribute Values  (th:attr)

```html
<img th:attr="src=@{/idea快捷键.png}" th:width="100px" th:height="100px"/>
```



## 8、遍历集合th:each

```html
<!-- 
HashMap<String,String> map = new HashMap<>();
		map.put("key1","value1");
		map.put("key2","value2");
		map.put("key3","value3");
-->
<div th:each="entry,state:${map}">
		<div th:text="${entry.key} +'=>'+ ${entry.value}"></div>
		<div th:text="${state}"></div>
		<div th:text="${state.index}"></div>
		<div th:text="${state.count}"></div>	
</div>
```

![结果图](D:\0_LeargingSummary\SpringBoot\images\遍历map结果图.png)



## 9、Expression Utility Objects  工具对象

### Execution Info  

### Messages  

### URIs/URLs  

### Conversions  

### Dates  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Dates
* ======================================================================
*/
/*
* Format date with the standard locale format
* Also works with arrays, lists or sets
*/
${#dates.format(date)}
${#dates.arrayFormat(datesArray)}
${#dates.listFormat(datesList)}
${#dates.setFormat(datesSet)}
/*
* Format date with the ISO8601 format
* Also works with arrays, lists or sets
*/
${#dates.formatISO(date)}
${#dates.arrayFormatISO(datesArray)}
${#dates.listFormatISO(datesList)}
${#dates.setFormatISO(datesSet)}
/*
* Format date with the specified pattern
* Also works with arrays, lists or sets
*/
${#dates.format(date, 'dd/MMM/yyyy HH:mm')}
${#dates.arrayFormat(datesArray, 'dd/MMM/yyyy HH:mm')}
${#dates.listFormat(datesList, 'dd/MMM/yyyy HH:mm')}
${#dates.setFormat(datesSet, 'dd/MMM/yyyy HH:mm')}
/*
* Obtain date properties
* Also works with arrays, lists or sets
*/
${#dates.day(date)} // also arrayDay(...), listDay(...), etc.
${#dates.month(date)} // also arrayMonth(...), listMonth(...), etc.
${#dates.monthName(date)} // also arrayMonthName(...), listMonthName(...), etc.
${#dates.monthNameShort(date)} // also arrayMonthNameShort(...), listMonthNameShort(...), etc.
${#dates.year(date)} // also arrayYear(...), listYear(...), etc.
${#dates.dayOfWeek(date)} // also arrayDayOfWeek(...), listDayOfWeek(...), etc.
${#dates.dayOfWeekName(date)} // also arrayDayOfWeekName(...), listDayOfWeekName(...), etc.
${#dates.dayOfWeekNameShort(date)} // also arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...), etc.
${#dates.hour(date)} // also arrayHour(...), listHour(...), etc.
${#dates.minute(date)} // also arrayMinute(...), listMinute(...), etc.
${#dates.second(date)} // also arraySecond(...), listSecond(...), etc.
${#dates.millisecond(date)} // also arrayMillisecond(...), listMillisecond(...), etc.
/*
* Create date (java.util.Date) objects from its components
*/
${#dates.create(year,month,day)}
${#dates.create(year,month,day,hour,minute)}
${#dates.create(year,month,day,hour,minute,second)}
${#dates.create(year,month,day,hour,minute,second,millisecond)}
/*
* Create a date (java.util.Date) object for the current date and time
*/
${#dates.createNow()}
${#dates.createNowForTimeZone()}
/*
* Create a date (java.util.Date) object for the current date (time set to 00:00)
Page 91 of 104*/
${#dates.createToday()}
${#dates.createTodayForTimeZone()}
```



### Calendars  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Calendars
* ======================================================================
*/
/*
* Format calendar with the standard locale format
* Also works with arrays, lists or sets
*/
${#calendars.format(cal)}
${#calendars.arrayFormat(calArray)}
${#calendars.listFormat(calList)}
${#calendars.setFormat(calSet)}
/*
* Format calendar with the ISO8601 format
* Also works with arrays, lists or sets
*/
${#calendars.formatISO(cal)}
${#calendars.arrayFormatISO(calArray)}
${#calendars.listFormatISO(calList)}
${#calendars.setFormatISO(calSet)}
/*
* Format calendar with the specified pattern
* Also works with arrays, lists or sets
*/
${#calendars.format(cal, 'dd/MMM/yyyy HH:mm')}
${#calendars.arrayFormat(calArray, 'dd/MMM/yyyy HH:mm')}
${#calendars.listFormat(calList, 'dd/MMM/yyyy HH:mm')}
${#calendars.setFormat(calSet, 'dd/MMM/yyyy HH:mm')}
/*
* Obtain calendar properties
* Also works with arrays, lists or sets
*/
${#calendars.day(date)} // also arrayDay(...), listDay(...), etc.
${#calendars.month(date)} // also arrayMonth(...), listMonth(...), etc.
${#calendars.monthName(date)} // also arrayMonthName(...), listMonthName(...), etc.
${#calendars.monthNameShort(date)} // also arrayMonthNameShort(...), listMonthNameShort(...), etc.
${#calendars.year(date)} // also arrayYear(...), listYear(...), etc.
${#calendars.dayOfWeek(date)} // also arrayDayOfWeek(...), listDayOfWeek(...), etc.
${#calendars.dayOfWeekName(date)} // also arrayDayOfWeekName(...), listDayOfWeekName(...), etc.
${#calendars.dayOfWeekNameShort(date)} // also arrayDayOfWeekNameShort(...), listDayOfWeekNameShort(...), etc.
${#calendars.hour(date)} // also arrayHour(...), listHour(...), etc.
${#calendars.minute(date)} // also arrayMinute(...), listMinute(...), etc.
${#calendars.second(date)} // also arraySecond(...), listSecond(...), etc.
${#calendars.millisecond(date)} // also arrayMillisecond(...), listMillisecond(...), etc.
/*
* Create calendar (java.util.Calendar) objects from its components
*/
${#calendars.create(year,month,day)}
Page 92 of 104${#calendars.create(year,month,day)}
${#calendars.create(year,month,day,hour,minute)}
${#calendars.create(year,month,day,hour,minute,second)}
${#calendars.create(year,month,day,hour,minute,second,millisecond)}
${#calendars.createForTimeZone(year,month,day,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,timeZone)}
${#calendars.createForTimeZone(year,month,day,hour,minute,second,millisecond,timeZone)}
/*
* Create a calendar (java.util.Calendar) object for the current date and time
*/
${#calendars.createNow()}
${#calendars.createNowForTimeZone()}
/*
* Create a calendar (java.util.Calendar) object for the current date (time set to 00:00)
*/
${#calendars.createToday()}
${#calendars.createTodayForTimeZone()}
```

### Numbers  

### Strings  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Strings
* ======================================================================
*/
/*
* Null-safe toString()
*/
${#strings.toString(obj)} // also array*, list* and set*
/*
* Check whether a String is empty (or null). Performs a trim() operation before check
* Also works with arrays, lists or sets
*/
${#strings.isEmpty(name)}
${#strings.arrayIsEmpty(nameArr)}
${#strings.listIsEmpty(nameList)}
${#strings.setIsEmpty(nameSet)}
/*
* Perform an 'isEmpty()' check on a string and return it if false, defaulting to
* another specified string if true.
* Also works with arrays, lists or sets
*/
${#strings.defaultString(text,default)}
${#strings.arrayDefaultString(textArr,default)}
${#strings.listDefaultString(textList,default)}
${#strings.setDefaultString(textSet,default)}
/*
* Check whether a fragment is contained in a String
* Also works with arrays, lists or sets
*/
${#strings.contains(name,'ez')} // also array*, list* and set*
${#strings.containsIgnoreCase(name,'ez')} // also array*, list* and set*
/*
* Check whether a String starts or ends with a fragment
* Also works with arrays, lists or sets
*/
${#strings.startsWith(name,'Don')} // also array*, list* and set*
${#strings.endsWith(name,endingFragment)} // also array*, list* and set*
/*
* Substring-related operations
* Also works with arrays, lists or sets
*/
Page 95 of 104${#strings.indexOf(name,frag)} // also array*, list* and set*
${#strings.substring(name,3,5)} // also array*, list* and set*
${#strings.substringAfter(name,prefix)} // also array*, list* and set*
${#strings.substringBefore(name,suffix)} // also array*, list* and set*
${#strings.replace(name,'las','ler')} // also array*, list* and set*
/*
* Append and prepend
* Also works with arrays, lists or sets
*/
${#strings.prepend(str,prefix)} // also array*, list* and set*
${#strings.append(str,suffix)} // also array*, list* and set*
/*
* Change case
* Also works with arrays, lists or sets
*/
${#strings.toUpperCase(name)} // also array*, list* and set*
${#strings.toLowerCase(name)} // also array*, list* and set*
/*
* Split and join
*/
${#strings.arrayJoin(namesArray,',')}
${#strings.listJoin(namesList,',')}
${#strings.setJoin(namesSet,',')}
${#strings.arraySplit(namesStr,',')} // returns String[]
${#strings.listSplit(namesStr,',')} // returns List<String>
${#strings.setSplit(namesStr,',')} // returns Set<String>
/*
* Trim
* Also works with arrays, lists or sets
*/
${#strings.trim(str)} // also array*, list* and set*
/*
* Compute length
* Also works with arrays, lists or sets
*/
${#strings.length(str)} // also array*, list* and set*
/*
* Abbreviate text making it have a maximum size of n. If text is bigger, it
* will be clipped and finished in "..."
* Also works with arrays, lists or sets
*/
${#strings.abbreviate(str,10)} // also array*, list* and set*
/*
* Convert the first character to upper-case (and vice-versa)
*/
${#strings.capitalize(str)} // also array*, list* and set*
${#strings.unCapitalize(str)} // also array*, list* and set*
/*
* Convert the first character of every word to upper-case
*/
${#strings.capitalizeWords(str)} // also array*, list* and set*
${#strings.capitalizeWords(str,delimiters)} // also array*, list* and set*
/*
* Escape the string
*/
${#strings.escapeXml(str)} // also array*, list* and set*
${#strings.escapeJava(str)} // also array*, list* and set*
${#strings.escapeJavaScript(str)} // also array*, list* and set*
Page 96 of 104${#strings.escapeJavaScript(str)} // also array*, list* and set*
${#strings.unescapeJava(str)} // also array*, list* and set*
${#strings.unescapeJavaScript(str)} // also array*, list* and set*
/*
* Null-safe comparison and concatenation
*/
${#strings.equals(first, second)}
${#strings.equalsIgnoreCase(first, second)}
${#strings.concat(values...)}
${#strings.concatReplaceNulls(nullValue, values...)}
/*
* Random
*/
${#strings.randomAlphanumeric(count)}
```

### Objects

```java
*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Objects
* ======================================================================
*/
/*
* Return obj if it is not null, and default otherwise
* Also works with arrays, lists or sets
*/
${#objects.nullSafe(obj,default)}
${#objects.arrayNullSafe(objArray,default)}
${#objects.listNullSafe(objList,default)}
${#objects.setNullSafe(objSet,default)}
```



### Booleans  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Bools
* ======================================================================
*/
/*
* Evaluate a condition in the same way that it would be evaluated in a th:if tag
* (see conditional evaluation chapter afterwards).
* Also works with arrays, lists or sets
*/
${#bools.isTrue(obj)}
${#bools.arrayIsTrue(objArray)}
${#bools.listIsTrue(objList)}
${#bools.setIsTrue(objSet)}
/*
* Evaluate with negation
* Also works with arrays, lists or sets
*/
${#bools.isFalse(cond)}
${#bools.arrayIsFalse(condArray)}
${#bools.listIsFalse(condList)}
${#bools.setIsFalse(condSet)}
/*
* Evaluate and apply AND operator
* Receive an array, a list or a set as parameter
*/
${#bools.arrayAnd(condArray)}
${#bools.listAnd(condList)}
${#bools.setAnd(condSet)}
/*
* Evaluate and apply OR operator
* Receive an array, a list or a set as parameter
*/
${#bools.arrayOr(condArray)}
${#bools.listOr(condList)}
${#bools.setOr(condSet)}
```

### Arrays  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Arrays
* ======================================================================
*/
/*
* Converts to array, trying to infer array component class.
* Note that if resulting array is empty, or if the elements
* of the target object are not all of the same class,
* this method will return Object[].
*/
${#arrays.toArray(object)}
/*
* Convert to arrays of the specified component class.
*/
${#arrays.toStringArray(object)}
${#arrays.toIntegerArray(object)}
${#arrays.toLongArray(object)}
${#arrays.toDoubleArray(object)}
${#arrays.toFloatArray(object)}
${#arrays.toBooleanArray(object)}
/*
* Compute length
*/
${#arrays.length(array)}
/*
* Check whether array is empty
*/
${#arrays.isEmpty(array)}
/*
* Check if element or elements are contained in array
*/
${#arrays.contains(array, element)}
${#arrays.containsAll(array, elements)}
```

### Lists  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Lists
* ======================================================================
*/
/*
* Converts to list
*/
${#lists.toList(object)}
/*
* Compute size
*/
${#lists.size(list)}
/*
* Check whether list is empty
*/
${#lists.isEmpty(list)}
/*
* Check if element or elements are contained in list
*/
${#lists.contains(list, element)}
${#lists.containsAll(list, elements)}
/*
* Sort a copy of the given list. The members of the list must implement
* comparable or you must define a comparator.
*/
${#lists.sort(list)}
${#lists.sort(list, comparator)}
```

### Sets  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Sets
* ======================================================================
*/
/*
* Converts to set
*/
${#sets.toSet(object)}
/*
* Compute size
*/
${#sets.size(set)}
/*
* Check whether set is empty
*/
${#sets.isEmpty(set)}
/*
* Check if element or elements are contained in set
*/
${#sets.contains(set, element)}
${#sets.containsAll(set, elements)}
```

### Maps  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Maps
* ======================================================================
*/
/*
* Compute size
*/
${#maps.size(map)}
/*
* Check whether map is empty
*/
${#maps.isEmpty(map)}
/*
* Check if key/s or value/s are contained in maps
*/
${#maps.containsKey(map, key)}
${#maps.containsAllKeys(map, keys)}
${#maps.containsValue(map, value)}
${#maps.containsAllValues(map, value)}
```

### Aggregates  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Aggregates
* ======================================================================
*/
/*
* Compute sum. Returns null if array or collection is empty
*/
${#aggregates.sum(array)}
${#aggregates.sum(collection)}
/*
* Compute average. Returns null if array or collection is empty
*/
${#aggregates.avg(array)}
${#aggregates.avg(collection)}
```

### IDs  

```java
/*
* ======================================================================
* See javadoc API for class org.thymeleaf.expression.Ids
* ======================================================================
*/
/*
* Normally used in th:id attributes, for appending a counter to the id attribute value
* so that it remains unique even when involved in an iteration process.
*/
${#ids.seq('someId')}
/*
* Normally used in th:for attributes in <label> tags, so that these labels can refer to Ids
* generated by means if the #ids.seq(...) function.
* *
Depending on whether the <label> goes before or after the element with the #ids.seq(...)
* function, the "next" (label goes before "seq") or the "prev" function (label goes after
* "seq") function should be called.
*/
${#ids.next('someId')}
${#ids.prev('someId')}
```























