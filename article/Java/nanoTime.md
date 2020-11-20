# System.nanoTime()特性

System.nanoTime()的初始值是在**本JVM实例**启动时"随机"选择的一个数字,随着JVM的运行而变化,System.currentTimeMillis表示系统时间,这就导致了的它们的几个差别:

1. System.currentTimeMillis()表示系统时间(即UTC). System.nanoTime()无法表示当前时间,本质上它是一个随机数字.
2. 在同一机器上的不同JVM上,System.currentTimeMillis是相同的,System.nanoTime()是不同的.
3. System.currentTimeMillis()系统时间敏感,System.nanoTime()系统时间不敏感.比如我们将系统时间往前调一秒,System.currentTimeMillis()相比修改前会减少1000.而System.nanoTime()不会变化. 

基于System.nanoTime()系统时间不敏感的特性,它被广泛应用在需要相对时间的场景中,如ScheduledThreadPoolExecutor,在0点0分添加一个1小时后执行的任务,那么只有在系统运行1小时后,它才会被触发,在此期间无论如何修改系统时间都不会影响.



---

 [【填坑纪事】一次用System.nanoTime()填坑System.currentTimeMills()的实例记录](https://www.cnblogs.com/andy-songwei/p/10784049.html)