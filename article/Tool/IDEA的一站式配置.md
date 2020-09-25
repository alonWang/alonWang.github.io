

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

#### 开启并行编译

Thread count 添加1C

#### 

![image-20200924185104979](img/maven.png)

