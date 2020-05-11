# JSON

JSON(JavaScript Object Notation)是轻量级的数据交换格式。

```java
//JSON定义
var 变量名 = {
“key” : value , 		// Number类型
“key2” : “value” , 		// 字符串类型
“key3” : [] , 			// 数组类型
“key4” : {}, 			// json 对象类型
“key5” : [{},{}] 		// json 数组
};
```

对于JSON对象，里面的key值就是对象的属性，因此只需要使用`对象名.属性名`就可以访问对象，比如`alart(json.key1)`

### 常用方法

`JSON.stringify(json)`把一个json对象转换为json字符串
`JSON.parse(jsonString)`将一个jaon字符串转换为json对象

### 在Java中使用

通过`gson.jar`来在Java中使用JSON。

json在Java中常见的三种操作

1. Java对象和json的转换
2. Java对象List集合和Json的转换
3. Map对戏那个和Json的转换

```java
//在json操作执勤啊必须要新建一个 gson对象
Gson gson = new Gson();
//Java对象
Person p = new Person(12, "hbw234");
//将Java对象转换成json字符串
String personJson = gson.toJson(person);
//将json字符串转换成java对象
Person p = gson.fromJson(personJson, Person.class);
//java对象list集合和jason的准换
List<Person> list = new ArrayList<Person>();
		for (int i = 0; i < 3; i++) {
			list.add(new Person(10 * i, "name-" + i));
		}
//map对象和json的转换
Map<String, Person> mapPerson = new HashMap<String, Gson.Person>();
mapPerson.put("p1", new Person(1, "person-1"));
//将map转换成json对象
String jsonMapString = gson.toJson(mapPerson);
```

# Ajax

Ajax是一种创建交互网页应用的网页开发技术`Asychronous Javascript And XML`

### Ajax请求（JavaScript）

1. 创建XMLHttpRequest对象
2. 调用open的方法设置请求参数
3. 调用send方法发送请求
4. 在send方法前绑定onreadystatechange事件，处理请求完成后的操作

用JavaScript创建一个HTML页面，发起请求

```javascript
<head>
  <meta http-equiv="pragma" content="no-cache" />
    <meta http-equiv="cache-control" content="no-cache" />
      <meta http-equiv="Expires" content="0" />
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
          <title>Insert title here</title>
<script type="text/javascript">
  function ajaxRequest() {
  // 				1、我们首先要创建XMLHttpRequest 
  var xhr = new XMLHttpRequest();
  // 				2、调用open方法设置请求参数
  xhr.open("GET","ajaxServlet?action=javaScriptAjax&a="+new Date(),true);
  // 				4、在send方法前绑定onreadystatechange事件，处理请求完成后的操作。
  xhr.onreadystatechange = function() {
    // 判断请求完成，并且成功
    if (xhr.readyState == 4 && xhr.status == 200) {
      document.getElementById("div01").innerHTML = xhr.responseText;
    } 
  }
  // 				3、调用send方法发送请求
  xhr.send();
}
</script>
</head>
<body>	
  <button onclick="ajaxRequest()">ajax request</button>
<div id="div01">
  </div>
</body>

```

创建一个AjaxServlet程序接受请求

```java
public class AjaxServlet extends BaseServlet{
  protected void javaScriptAjax(){
    request.getParameter("a");
    Random random = new Random(System.currentMillis());
  }
}
```

在XML文件中配置

```xml
<servlet>
  <servlet-name>AjaxServlet</servlet-name>
  <servlet-class>com.atguigu.servlet.AjaxServlet</servlet-class>
</servlet>
<servlet-mapping>
  <servlet-name>AjaxServlet</servlet-name>
  <url-pattern>/ajaxServlet</url-pattern>
</servlet-mapping>
```

