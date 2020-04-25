JavaScript诞生主要是完成页面的数据验证。因此在客户端运行的时候，需要浏览器来解析执行JavaScript代码

JavaScript和HTML代码结合：

1. 在head标签中，使用script标签写JS代码
   `<script type="text/javascript">alert("world");</script>`

2. 使用script标签引入JS文件
   `<script type="text/javascript" src="1.js"></script>`

## 变量

- 数值类型： number
- 字符串类型：string
- 对象类型：object
- 布尔类型：boolean
- 函数类型：function

## JS里面特殊值：

undefined：为定义，所有未赋初始变量的默认值都是undefined
null：空值
NaN： not a number，非数字，非数值

变量格式：  var 变量名;  或者 var 变量名 = 值;

### 运算符：

==：等于，只是简单的字面值比较
 = == ：全等于，除了字面比较，还会检查两个数据类型是否一致
&&：且运算
||：或运算
!：取反运算

JS中每一个变量都有一个逻辑值（真假），0,null,undefined,""都是false

## 数组

数组定义： var 数组名 = []; 定义一个空数组
通过下标对数组元素进行赋值操作的时候，会自动进行扩容操作，类似Python
var arr = [], arr.length == 0;

## 函数

第一种定义方式是使用function定义，如果要有带返回值的函数，在函数体内使用return直接返回即可

```js
fun1(12,"abc");//	带有返回值函数的定义
		function fun2(num1,num2){
			return num1 + num2;
		}
var sum = fun2(100,200);
		alert(sum);
```

第二种是 var 函数名 = function

```js
var fun = function(){
				alert("fun函数被调用了");
			}
fun();
```

**JS中函数允许重载，但是重载会直接覆盖掉上一次的定义**

函数中有隐形参数：arguements，只存在在function函数内（可变长数组）

## JS中自定义对象

```js
var obj = new Object(); //定义了一个对象实例
obj.name = "国哥";// 	变量名.属性名 = 值;	添加一个属性
obj.fun = function(){// 变量名.函数名 = function(){}	添加一个函数
  alert("国哥好帅，==>>>" + this.name + ", " + this.age);
}

var obj = {
				name:"华仔",
				age:18,
				fun:function(){
					alert("姓名：" + this.name + " , 年龄：" + this.age);
				}
		};
```

## JS中事件

常用事件

1. onload：加载完成事件，在页面加载完成后做一些初始化操作
2. onclick：单击事件，用于按钮
3. onblur：失去焦点事件，用于检验用户名是否有验
4. onchange：内容发生改变事件，用于下拉菜单，和输入框内容变化
5. onsubmit：表单提交事件，用于表单提交前的验证表单项是否合法，若不合法，则阻止表单提交

事件的注册分为静态注册和动态注册两种

- 静态注册：通过标签的事件属性直接赋值事件响应后的代码。
- 动态注册：通过代码的形式先获取标签对象。再通过标签对象，事件名=function(){}函数这种方式注册事件响应后代码

```js
动态注册事件步骤
window.onload = function(){
1、获取标签对象
2、通过标签对象.事件名 = function(){}
}
```

## onload加载完成事件

```js
<script type="text/javascript">
			function onloadFun(){
				alert('静态注册 onloadFun() 页面加载完成');
			}
			// onload事件，动态注册
			window.onload = function(){
				alert("动态注册onload事件");
			}
</script>   //在body中<body onload="onloadFun();">调用
```

## onclick单击事件

```js
function onclickFun(){
  alert("静态注册onclick");
}			
			//动态注册onclick事件
window.onload = function(){
  var btnObj = document.getElementById("btn01");
  //document是js语言提供的一个对象，getElementById方法 是通过id属性获取标签对象
  // 2 通过标签对象.事件名 = function(){}
  btnObj.onclick = function(){
    alert("这是动态注册的onclick事件");
  }
}
body中		<button onclick="onclickFun();">按钮1</button>
		<button id="btn01">按钮2</button>
```

## onblur失去焦点事件

```js
function onblurFun(){
  console.log("静态注册onblur事件");//往控制台打印信息
}
window.onload = function(){
  //1 获取标签对象
  var passObj = document.getElementById("pass");
  //2 通过标签对象.事件名 = function(){}
  passObj.onblur = function(){
    console.log("动态注册失去焦点事件onblur");
  }
}
body中调用
		用户名：<input type="text" onblur="onblurFun()"/><br/>
		密码：<input id="pass" type="password" /><br/>
```

## onchange内容发生改变事件

```js
function onChangeFun(){
  alert("静态注册的onchange事件");
}
window.onload = function(){
  //1 获取标签对象
  var selObj = document.getElementById("sel01");
  //2 通过标签对象.事件名 = function(){}
  selObj.onchange = function(){
    alert("onchange动态注册");
  }
}
body调用：<select onchange="onChangeFun()"></select>
```

## onsubmit表单提交事件

```js
function onsubmitFun(){
  alert("静态注册onsubmit事件,验证所有表单项，有不合法就阻止提交");
  return false;
}
window.onload = function(){
  //1 获取标签对象
  var formObj = document.getElementById("form01");
  //2 通过标签对象.事件名=function(){}
  formObj.onsubmit = function(){
    alert("动态注册onsubmit事件,,一切ok");
    return true;
  }
}
调用
<form action="http://localhost:8080" onsubmit="return onsubmitFun();">
  <input type="submit" value="静怸onsubmit"/>
</form>

```

## DOM模型

全称Document Object Model，即将文档中标签，属性，文本转换为对象来进行管理。

### Document对象

1. Document管理了所有的HTML文档内容
2. Document是一种树结构的文档，有层级关系
3. 让我们把所有的标签都对象化
4. 通过document访问所有的标签

```js
//HTML标签对象话类似如下代码
class Dom{
	private String id;		// id属性
	private String tagName; //表示标签名
	private Dom parentNode; //父亲
	private List<Dom> children; // 孩子结点
    private String innerHTML; // 起始标签和结束标签中间的内容
}
```

## Document对象中的方法介绍

1. document.getElementById(elementId)
   通过标签的id属性查找标签dom对象，elementId是标签的id属性值
2. document.getElementsByName(elementName)
   通过标签的name属性查找标签dom对象，elementName标签的name属性值
3. document.getElementsByTagName(tagname)
   通过标签名查找标签dom对象。tagname是标签名
4. document.createElement( tagName)
   通过给定的标签名，创建一个标签对象。tagName是要创建的标名

使用优先顺序是：Id  >  name > 标签

### getElementById

```js
function onclickFun(){
  // 当你要操作某个标签的时候。要先获取到标签对象
  var usernameObj = document.getElementById("username");
  // 输入框中的内容
  var usernameText = usernameObj.value;
  // 			验证 用户名必须由字母。数字。下划线组成。并且长度是5-12位
  // 验证字符串匹配规则，需要使用正则表达式技术。
  var patt = /^\w{5,12}$/;
  // test方法是正则表达式对象的方法，专门用来验证字符串是否匹配规则
  // 如果匹配就返回true，不匹配就返回false
  // 要操作span标签对象，就要先获取span标签对象
  var usernameSpanObj = document.getElementById("usernameSpan");
  // innerHTML 属性表示起始标签和结束标签中的内容
  // 这个属性，可读可写 
  if (patt.test(usernameText)) {
    // 				usernameSpanObj.innerHTML = "用户名合法";
    usernameSpanObj.innerHTML = '<img alt="" src="right.png" width="12" height="12">';
    // 				alert("用户名合法");
  } else {
    // 				usernameSpanObj.innerHTML = "用户名不合法";
    usernameSpanObj.innerHTML = '<img alt="" src="wrong.png" width="12" height="12">';
    // 				alert("用户名不合法");
  }
}
</script>
</head>
<body>
  用户名：<input id="username" type="text" name="username" value="abc"/>
    <span id="usernameSpan" style="color: red;">
      </span>
<br/>
      <button onclick="onclickFun()">验证</button>
</body>
```

## 标签对象常用属性和方法

getElementsByTagName() 
返回包含带有指定标签名称的所有元素的节点列表（集合/节点数组）

appendChild( oChildNode ) 
可以添加一个子节点，oChildNode是要添加的孩子节点

### 属性

childNodes：获取当前节点的所有子节点

属性，获取当前节点的所有子节点

firstChild：获取当前节点的第一个子节点

属性，获取当前节点的第一个子节点

lastChild：获取当前节点的最后一个子节点

属性，获取当前节点的最后一个子节点

parentNode：获取当前节点的父节点

属性，获取当前节点的父节点

nextSibling：获取当前节点的下一个节点

属性，获取当前节点的下一个节点

previousSibling：获取当前节点的上一个节点

属性，获取当前节点的上一个节点

className用于获取或设置标签的class属性值

innerHTML ：表示获取/设置起始标签和结束标签中的内容

属性，表示获取/设置起始标签和结束标签中的内容

innerText：表示获取/设置起始标签和结束标签中的文本