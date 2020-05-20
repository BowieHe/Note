### 为什么使用

Maven提供了开发人员构建一个完整的生命周期框架。自动完成项目的基础工具建设，Maven使用标准的目录结构和默认构建生命周期

Maven主要做两件事：1，统一开发规范和工具 2，统一管理jar包

> 1 依赖的管理：仅仅通过jar包的几个属性，就能确定唯一的jar包，在指定的文件pom.xml中，只要写入这些依赖属性，就会自动下载并管理jar包。
> 2 项目的构建：内置很多的插件与生命周期，支持多种任务，比如校验、编译、测试、打包、部署、发布...
> 3 项目的知识管理：管理项目相关的其他内容，比如开发者信息，版本等等

当构建Maven文件时，会先检查`pom.xml`文件确定依赖包下载位置。执行顺序如下

1. 从本地资源库中查找并获得依赖包，没有则执行第二步
2. 从Maven默认中央仓库查找并获得依赖包，没有则第三步
3. `pom.xml`中定义了自定义的远程仓库，会在仓库中查找并获得依赖包，如果都没有，那Maven会抛出异常

#### Maven的构建：

把动态的Web工程经过编译得到的编译结果部署到服务器上的整个过程

编译：Java源文件[.java] -> class字节码文件[.class]
部署：最终子啊servlet容器中部署的不是动态web工程，而是编译之后的文件

**构建环节**

- 清理clean:将以前编译得到的旧class文件删除
- 编译compile：将Java源程序编译成字节码文件
- 测试 test：自动测试，调用junit程序
- 报告report：测试程序执行结果
- 打包package：动态Web工程打包(War)，Java工程打Jar包
- 安装install：将打包的阿斗的文件复制到仓库中制定的位置(Maven特有)
- 部署deploy：将动态Web工程生成的War包复制到Servlet容器下，

## 创建Maven项目

所有的Maven指令都必须在项目`pom.xml`所在地址运行(mvn clean等)

1. `mvn archetype:generate`
2. 选择骨架，输入坐标:groupId, artifactId, version, package;
3. 将项目转换成IDEA：`mvn idea:idea`
4. 项目打包： `mvn package`

#### Maven目录结构

```shell
根目录：工程名 
|---src：源码 
|---|---main:存放主程序 
|---|---|---java：java源码文件 
|---|---|---resource：存放框架的配置文件 
|---|---test：存放测试程序 
|---pop.xml：maven的核心配置文件
```

## maven常用指令

- mvn clean:清除target文件夹，回到编译之前
- mvn compile:在`pom.xml`配置的依赖包导入到仓库，完成之后路径内会多一个`target`文件夹，里面主要是编译后的字节码文件
- mvn test-compile：target文件夹下厨了classes又多了一个`test-compile`文件夹
- mvn test：
- mvn package：target文件夹下多了打包好的jar包
- mvn install:

### 依赖

Java内有个环境变量`classpath`.JVM运行代码的时候，需要基于`classpath`查找需要的类文件，再加载到内存执行。

Mavan在编译项目主代码的时候，编译和执行测试代码的时候，和Maven项目具体运行的时候都有独立的`classpath`，每个都需要添加需要的依赖到不同的`classpath`。这些`classpath`就是依赖。
依赖的范围就是控制这三种`classpath`的关系
依赖范围用 `<scope>`来选择

1. compile : 编译依赖范围。对编译，测试，运行的classpath都有效
2. test : 测试依赖范围。只对测试classpath有效
3. provided : 编译，测试有效，例如 servlet ，运行时容器中自带了servlet-api，没有必要使用
4. runtime : 运行和测试有效，例如 jdbc，编译时只需相应的接口，测试和运行时才需要具体的实现
5. system : 编译，测试有效。使用此范围的依赖必须通过systemPath元素指定依赖文件的路径。该依赖不是通过Maven仓库解析的，谨慎使用。

- maven解析依赖信息时会到本地仓库中去查找被依赖的jar包
  - 本地仓库没有的会去中央仓库查找maven坐标来获取，再下载到本地仓库
  - 中央仓库找不到依赖的jar包的时候，就会编译失败
- 如果依赖是自己或者团队开发的maven工程，要先使用`mvn install`命令将maven工程的jar包导入到本地仓库中，再回到工作项目中去使用`mvn compile`成功编译

## 依赖传递性

pom.xml文件配置好依赖之后，必须要先`mvn install`后，依赖的jar包才能使用

比如：WebDemo项目依赖Service1 Service1项目依赖Service2
即WebDemo的`pom.xml`文件想通过编译，Service1必须`mvn isntall`。同理，编译Service1的`pom.xml`文件之前，需要`mvn install`Service2.

依赖版本的优先选择：

1. 路径最短优先原则
2. 路径相同声明优先原则
3. 统一管理依赖的版本：使用properties标签，自定义版本的标签名。在使用的地方通过`${自定义标签名}`来使用

## 配置build

配置好build之后，执行`mvn package`之后，在maven工程指定的target目录里war包和文件都按照配置生成了

```xml
<build>
　　<!-- 项目的名字 -->
　　<finalName>WebMavenDemo</finalName>
　　<!-- 描述项目中资源的位置 -->
　　<resources>
　　　　<!-- 自定义资源1 -->
　　　　<resource>
　　　　　　<!-- 资源目录 -->
　　　　　　<directory>src/main/java</directory>
　　　　　　<!-- 包括哪些文件参与打包 -->
　　　　　　<includes>
　　　　　　　　<include>**/*.xml</include>
　　　　　　</includes>
　　　　　　<!-- 排除哪些文件不参与打包 -->
　　　　　　<excludes>
　　　　　　　　<exclude>**/*.txt</exclude>
　　　　　　　　　　<exclude>**/*.doc</exclude>
　　　　　　</excludes>
　　　　</resource>
　　</resources>
　　<!-- 设置构建时候的插件 -->
　　<plugins>
　　　　<plugin>
　　　　　　<groupId>org.apache.maven.plugins</groupId>
　　　　　　<artifactId>maven-compiler-plugin</artifactId>
　　　　　　<version>2.1</version>
　　　　　　<configuration>
　　　　　　　　<!-- 源代码编译版本 -->
　　　　　　　　<source>1.8</source>
　　　　　　　　<!-- 目标平台编译版本 -->
　　　　　　　　<target>1.8</target>
　　　　　　</configuration>
　　　　</plugin>
　　　　<!-- 资源插件（资源的插件） -->
　　　　<plugin>
　　　　　　<groupId>org.apache.maven.plugins</groupId>
　　　　　　<artifactId>maven-resources-plugin</artifactId>
　　　　　　<version>2.1</version>
　　　　　　<executions>
　　　　　　　　<execution>
　　　　　　　　　　<phase>compile</phase>
　　　　　　　　</execution>
　　　　　　</executions>
　　　　　　<configuration>
　　　　　　　　<encoding>UTF-8</encoding>
　　　　　　</configuration>
　　　　</plugin>
　　　　<!-- war插件(将项目打成war包) -->
　　　　<plugin>
　　　　　　<groupId>org.apache.maven.plugins</groupId>
　　　　　　<artifactId>maven-war-plugin</artifactId>
　　　　　　<version>2.1</version>
　　　　　　<configuration>
　　　　　　　　<!-- war包名字 -->
　　　　　　　　<warName>WebMavenDemo1</warName>
　　　　　　</configuration>
　　　　</plugin>
　　</plugins>
</build>
```



#  Error

`Source option 5 is no longer supported. Use 6 or later.`
	在`pom.xml`中设置`maven.compiler.source`和`maven.compiler.target`版本到新版本

# 名词解释

#### 生命周期

粗略一点，过程包括：编译，测试，打包，集成测试，验证，部署

三种生命周期：

1. Clean Lifecircle：在进行真正的构建之前进行一些清理工作

   - pre-clean：执行一些需要clena之前完成的工作
   - clean移除所有上一次构建生成的文件
   - post-clean执行一些需要子啊clean之后立刻完成的工作

2. Default Lifecircle：构建的核心部分，编译，测试，打包，部署等

   - validate
   - Generate-sources
   - Process-sources
   - Generate-resources
   - Process-resources:复制并处理资源文件，至目标目录，准备打包
   - compile：编译项目的源代码
   - Process-classes
   - Generate-test-source
   - process-test-source
   - Generate-test-resources
   - Process-test-resources:复制并处理资源文件，到目标测试目录
   - Test-compile：编译测试源代码
   - Process-test-classes
   - Test：使用合适的单元测试框架运行测试。这些测试代码不会被打包和部署
   - Prepare-package
   - Package:接受编译好的代码，打包成可发布的格式，如Jar
   - Pre-integration-test
   - Integration-test
   - Post-integration-test
   - verify
   - install:将包安装至本地仓库，让其他项目依赖
   - deploy:将最终的包复制到远程仓库，让其他开发人员共享

   当执行`mvn install`的时候，其中也执行了compile和test
   **无论执行成名周期哪一个阶段，maven都是从该生命周期的开始执行**

3. Site Lifecircle：生成项目报告，站点，发布站点

   - pre-site执行一些需要在站点文档之前完成的工作
   - site：生成项目的站点文档
   - Post-site执行一些需要在生成站点文档之后完成的工作，为部署做准备
   - Site-deploy：将生成的站点文档部署到特定的服务器上

#### 版本规范

SNAPSHOT：一般用于开发过程中，表示不稳定的版本
LATEST：特定构件的最新发布，可以是发布版，也可以是snapshot版
RELEASE：仓库中最后一个非快照版本

- 同一项目中所有模块版本保持一致
- 子模块统一继承父模块版本
- 统一在顶层模块POM的<dependencyManagement/>中定义子模块的依赖版本号，子模块中添加依赖时不要添加版本号
- 开发测试阶段使用SNAPSHOT
- 生产发布时使用RELEASE
- 新版本迭代只修改顶层POM中版本

#### 坐标

- groupId , artifactId , version 三个元素是项目的坐标，唯一的标识这个项目。
- groupId 项目所在组，一般是组织或公司
- artifactId 是当前项目在组中的唯一ID；
- version 表示版本，SNAPSHOT表示快照，表示此项目还在开发中，不稳定。
