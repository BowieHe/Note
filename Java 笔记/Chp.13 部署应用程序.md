# JAR文件

JAR文件时压缩的，使用了熟知的ZIP压缩格式

可以使用JDK的指令创建JAR文件
`jar cvf JARFileName File1 File2 ...`
例如``jar cvf TestClass,jar *.class icon.gif`

```java
c			创建一个新的或者空的存档文件并加人文件。 如果指定的文件名是目录,jar程序将会对它们 进行递归处理
C			暂时改变H录， 例如:  jar cvf JARFileName.jar -C classes *.class.改变 classes 子目录， 以便增加这些类文件
e			在清单文件中创建一个条目
f			将 JAR 文件名指定为第二个命令行参数。如果没有这个参数,jar命令会将结果写到标准输出上(在创建 JAR 文件时)或者从标准输入中读取它 (在解压或者列出 JAR 文件内容时) 
i			建立索引文件(用于加快对大型归档的查找 )
m			将一个清单文件( manifest )添加到 JAR 文件中。清单是对存杓内容和来源的说明每个归档有一个默认的清单文件。 但是， 如果想验证归杓文件的内容， 可以提供自己的清单文件 
M			不为条目创建清笮文件
i			显示内容表
u			更新一个已有的 JAR 文件
v			生成详细的输出结果
x			解压文件。 如果提供一个或多个文件名，只解压这些文件;否则，解压所有文件
0			存储 不进行ZIPm缩
```

清单文件：每个JAR文件包含一个用于描述归档特征的清单文件(manifest)

使用 e 选项指定程序入口点
`jar cvfe MyProgram.jar com.mycompany.mypkg.MainAppClass filesToAdd`
或者`Main-Class: com.mycompany.mypkg.MainAppClass`
==不要将拓展名**.class**添加到末尾==

IDEA的 .class 文件在out的子目录下面

如果在JAR中添加清单文件，需要先自己创建`mainfest.mf`文件，再执行`jar cfm JarFileName ManifestName classFile`

密封：默认情况下JAR文件是不密封的，因此可以在清单文件里加上`Sealed: true`来密封所有的包
如果只是想单独密封一个包，前面加上`Name: com/mycoinpany/util/`

