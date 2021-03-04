#### 频繁Full GC

生产上频繁出现Full GC，重启后短时间没问题，超过一定时间后，又开始频繁Full GC，通过mat分析dump文件，发现有一个叫ScheduledThreadPoolExecutor的类下面有个叫DelayedWorkQueue的类下面的RunnableScheduledFuture数组特别大，有898656个ScheduledFutureTask元素，占用了很多空间。

![](/assets/jvm/mat.png)

通过查看代码发现ScheduledFutureTask是在长连接建立时用来处理心跳续期的，在开启该任务时会存到一个map（用来存储长连接对应的任务，以便长连接断开时停止该任务）中，但在长连接断掉后关闭该任务时没从map中remove掉，所以这部分对象一直没有被回收，至此定位频繁Full GC的原因就是内存泄漏，增加remove的逻辑，问题解决。

#### MQ消息积压
