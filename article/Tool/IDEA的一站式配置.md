

### Run/Debug Configurations添加JVM启动参数



推荐网站http://jvmmemory.com/

![image-20200922200543748](img/run-configuration-template.png)

在VM options添加

```properties
# 不同版本JVM支持命令可能发生变化,详见 https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html

# 打印命令行参数 推荐添加
-XX:+PrintCommandLineFlags
# 打印Flag参数 推荐添加
-XX:+PrintFlagsFinal

# 发生OOM时，自动生成DUMP文件  推荐添加
-XX:+HeapDumpOnOutOfMemoryError
# 生成DUMP文件路径,支持相对路径,默认生成文件名 java_pid{pid}.hprof
-XX:HeapDumpPath=./dump
# 禁止调用System.gc()的显式GC
-XX:+DisableExplicitGC

# 开启GC日志打印 按需添加
-verbose:gc
# GC日志位置
-Xlog:gc:./gc.log


```



### maven

* 将maven的bin目录添加到系统环境变量Path中

* 修改setting.xml 修改本地仓库地址 添加国内镜像,默认profile

```xml


<mirror>
<id>nexus-aliyun</id>
<mirrorOf>central</mirrorOf>
<name>Nexus aliyun</name>
<url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>


<profile>
  <id>jdk-1.8</id>

  <activation>
    <activeByDefault>true</activeByDefault>
    <jdk>1.8</jdk>
  </activation>

  <properties>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
  </properties>
</profile>



```

* 
* User settings file 和Local repository都使用默认的{user-path}/.m2/路径,无需override
  * 假如指定了别的路径.   可能导致IDE和命令行执行相同mvn命令得出的结果不同
    *  例如在命令行执行 mvn clean   ,使用的setting file 和repository默认就是 .m2路径下的,而IDE用的是自定义的.  如果不了解原理,会导致很多莫名其妙的问题
  *  
* 开启并行编译 Thread count 添加1C   实测编译速度提升一倍.

![image-20200924185104979](img/maven.png)



### 编码

* 全局改为UTF-8编码(无BOM).再也不用担心乱码问题

* 开启 Transparent native-to-ascii conversion功能, IDE会自动将ASCII编码数据转换为UTF-8展示.详见[IDEA中properties文件的 Transparent native-to-ascii 功能探究](https://www.jianshu.com/p/8409237e0114) 

![image-20200927154930143](img/idea-encoding.png)

### 修改IDEA properties

>  Help => Edit Custom Properties 

修改IDEA的一些配置,配置项参见[官方文档](https://www.jetbrains.com/help/idea/tuning-the-ide.html#common-platform-properties)

```properties
# 强制IDEA对大文件进行代码分析
idea.max.intellisense.filesize=99999
```

### 修改IDEA VM Options

> Help => Edit Custom VM Options

IDEA也是一个JAVA项目,可以配置其JVM启动参数以获得更好体验

```properties
# 加载字节码时,不进行验证. 在这个场景下,可以减少不必要的性能损耗
-Xverify:none
# 全局UTF-8编码 从此远离乱码
-Dfile.encoding=UTF-8
```



### 导包

> Editor => General => Auto Import

开启自动导包和导包优化. 并将一些不常用但容易冲突的包排除在外

![image-20200927180804724](img/idea-auto-import.png)



### Code Style

调整 Class count to use import with '*' 改为99999, 间接禁用 import *

![image-20200927200804220](img/idea-import-n.png)

代码排列(rearrange),勾选 Grouping rules, 个人项目我一般都会勾上.团队项目就不要用了.

![image-20200927201011484](img/idea-rearrange.png)



### live template

放个效果图,这个比较复杂,具体使用参见我的另一篇文章[idea中live template的详细使用教程](https://www.jianshu.com/p/3974df6572af)

![](img/idea-live-template.gif)

### file and code template

给你的新建文件添加作者时间等信息,这里还可以使用Apache Velocity引擎进行更高级的操作,详见[idea中file template的较高级使用](https://www.jianshu.com/p/189ce7ea7ba6)

![image-20200928213752129](img/idea-file-template.png)

```java
/**
* 
*@author alonwang
*@date ${DATE} ${TIME}
*/
```



### 插件篇

* Codota AI代码提示+相关代码示例
* Material Theme UI 超好看的UI,我习惯使用Arch风格
* Rainbow Brackets  让你的括号变得五颜六色
* VisualVM Launcher  随时随时打开 VisualVM