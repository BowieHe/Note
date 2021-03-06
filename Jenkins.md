## 项目开发效率优化

1. 持续部署：项目功能部署至服务器后可以运行，为下一步测试环节或者最终用户正式使用做好准备
2. 持续集成：经常讲所有的模块集成到一起进行测试，尽早发现项目整体运行的问题，尽早解决。
3. 持续交付：项目各个版本之间间隔时间过长，对用户反馈感知迟钝，无法精确改善用户体验。应该用小版本快速迭代，不断手机用户反馈信息，用最快的速度改进，优化。

**优点**：

1. 降低风险：

   多次集成和相应测试，利于检查缺陷，了解软件将康状况

2. 减少重复过程：

   重复过程产生主要有两个方面：一是编译，测试，打包部署等固定过程；二是一个缺陷没有及时发现，会导致后续代码的开发方向错误

3. 任何时间，地点生成可部署的软件

4. 增强项目的可见性

   持续集成可以注意到趋势并且有效的进行决策；为项目构建状态和品质指标提供有效决策；可以看到构建成功，失败，总体品质等项目信息的趋势

# Jenkins + SVN持续集成环境搭建

## 系统结构总述

- 创建虚拟机安装Linux系统
- 版本控制子系统
  - Subversion服务器
  - 项目对应版本库
  - 版本库中子程序
- 持续集成子系统
  - JDK
  - Tomcat
  - Maven
  - Jenkins
    - 主体程序
    - SNV插件
    - Maven插件
    - Deploy to Web Container插件
- 应用发布子系统
  - JDK
  - Tomcat

