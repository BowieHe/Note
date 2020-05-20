Java Web开发有两种模型，一种是**页面中心**，适合小应用开发。还有一种是MVC模式，推荐的Java Web应用架构

一个实现MVC模式的应用包括模型，视图和控制器三块。

- 视图负责应用展示
- 模型封装了应用的数据和业务逻辑
- 控制器负责接受用户输入，改变模型以及调整视图的显示

-----

- SpringMVC使用Servlet作为控制器，大部分的应用视图是JSP页面，模型采用POJO。在实践中会采用一个JavaBean来持有模型状态，并将业务逻辑放到一个Action中。
- 每个HTTP请求都会发送给控制器，请求中的URI标识出对应的Action。Action代表了应用可执行的一个操作。一个提供了Action的Java对象称为Action对象，一个Action类可以支持多个Action。