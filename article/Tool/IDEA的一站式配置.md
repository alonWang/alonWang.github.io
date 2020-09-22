

### Run/Debug Configurations添加JVM参数

![image-20200922200543748](img/run-configuration-template.png)

在VM options添加

```properties
# 不同版本JVM支持命令可能发生变化,详见 https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html

# 发生OOM时，自动生成DUMP文件
-XX:+HeapDumpOnOutOfMemoryError
# 生成DUMP文件路径,支持相对路径,默认生成文件名 java_pid{pid}.hprof
-XX:HeapDumpPath={absolute-or-relative-path}
# 禁止调用System.gc()的显式GC
-XX:+DisableExplicitGC

# 最小堆 单位默认byte,可用单位k/K,m/M
-Xms{value} 
# 最大堆 
-Xmx{value} 
# 栈容量
-Xss{value} 
# 新生代容量

-Xmn{value}
# 开启GC日志打印
-XX:+PrintGCDetails
# GC日志位置
-Xlog:gc:{filename}
# 显示所有可设置参数及默认值 
-XX:+PrintFlagsInitial


```







