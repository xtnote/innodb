java程序在启动以后，会在java.io.tmpdir指定的目录下(/tmp/hsperfdata_{userName})生成以java进程的pid为文件名的文件
jps、jconsole、jvisualvm等工具的数据来源就是这个文件（/tmp/hsperfdata_userName/pid)

### jps
```
usage:  
  jps [-help]  
  jps [-q] [-mlvV] [<hostid>]  
-q 只显示pid  
-m 输出传递给main 方法的参数  
-l 输出应用程序main class的完整package名 或者 应用程序的jar文件完整路径名  
-v 输出传递给JVM的参数  
```

### jstack
```
Usage:  
    jstack [-l] <pid>  
        (to connect to running process)  
    jstack -F [-m] [-l] <pid>  
        (to connect to a hung process)  
    jstack [-m] [-l] <executable> <core>  
        (to connect to a core file)  
    jstack [-m] [-l] [server_id@]<remote server IP or hostname>  
        (to connect to a remote debug server)  

pid 需要被打印配置信息的java进程id,可以用jps查询.   
-l长列表. 打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表.        
-F当’jstack [-l] pid’没有相应的时候强制打印栈信息  
-m打印java和native c/c++框架的所有栈信息.  
-h | -help打印帮助信息  
```

jstack用于生成java虚拟机当前时刻的线程快照,线程快照是当前java虚拟机内每一条线程正在执行的方法堆栈的集合  
线程的几种状态：  
NEW,未启动的。不会出现在Dump中。  
RUNNABLE,在虚拟机内执行的。  
BLOCKED,受阻塞并等待监视器锁。  
WATING,无限期等待另一个线程执行特定操作。  
TIMED_WATING,有时限的等待另一个线程的特定操作。  
TERMINATED,已退出的。  

locked:通过synchronized关键字,成功获取到了对象的锁,成为监视器的拥有者,在临界区内操作。对象锁是可以线程重入的。  
waiting to lock:通过synchronized关键字,没有获取到了对象的锁,线程在监视器的进入区等待。在调用栈顶出现,线程状态为Blocked。  
waiting on:通过synchronized关键字,成功获取到了对象的锁后,调用了wait方法,进入对象的等待区等待。在调用栈顶出现,线程状态为WAITING或TIMED_WATING。  
parking to wait for:park是基本的线程阻塞原语,不通过监视器在对象上阻塞。随concurrent包会出现的新的机制,与synchronized体系不同。  

runnable:状态一般为RUNNABLE。  
in Object.wait():等待区等待,状态为WAITING或TIMED_WAITING。  
waiting for monitor entry:进入区等待,状态为BLOCKED。  
waiting on condition:等待区等待、被park。  
sleeping:休眠的线程,调用了Thread.sleep()。  

### jmap
```
Usage:  
    jmap [option] <pid>  
        (to connect to running process)  
    jmap [option] <executable <core>  
        (to connect to a core file)  
    jmap [option] [server_id@]<remote server IP or hostname>  
        (to connect to remote debug server)  

<no option> 如果使用不带选项参数的jmap打印共享对象映射，将会打印目标虚拟机中加载的每个共享对象的起始地址、映射大小以及共享对象文件的路径全称。这与Solaris的pmap工具比较相似。  
-dump:[live,]format=b,file=<filename> 以hprof二进制格式转储Java堆到指定filename的文件中。live子选项是可选的。如果指定了live子选项，堆中只有活动的对象会被转储。想要浏览heap dump，你可以使用jhat(Java堆分析工具)读取生成的文件。  
-finalizerinfo 打印等待终结的对象信息。  
-heap 打印一个堆的摘要信息，包括使用的GC算法、堆配置信息和generation wise heap usage。  
-histo[:live] 打印堆的柱状图。其中包括每个Java类、对象数量、内存大小(单位：字节)、完全限定的类名。打印的虚拟机内部的类名称将会带有一个*前缀。如果指定了live子选项，则只计算活动的对象。  
-permstat 打印Java堆内存的永久保存区域的类加载器的智能统计信息。对于每个类加载器而言，它的名称、活跃度、地址、父类加载器、它所加载的类的数量和大小都会被打印。此外，包含的字符串数量和大小也会被打印。  
-F 强制模式。如果指定的pid没有响应，请使用jmap -dump或jmap -histo选项。此模式下，不支持live子选项。  
-h 打印帮助信息。  
-help 打印帮助信息。  
-J<flag> 指定传递给运行jmap的JVM的参数。    
```

### jstat
```
jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
Option — 选项，我们一般使用 -gcutil 查看gc情况  
vmid — VM的进程号，即当前运行的java进程号  
interval– 间隔时间，单位为秒或者毫秒  
count — 打印次数，如果缺省则打印无数次  

jstat –class<pid> : 显示加载class的数量，及所占空间等信息。  
jstat -compiler <pid>显示VM实时编译的数量等信息。  
jstat -gc <pid>: 可以显示gc的信息，查看gc的次数，及时间。  
jstat -gccapacity <pid>:可以显示，VM内存中三代（young,old,perm）对象的使用和占用大小  
jstat -gcutil <pid>:统计gc信息  
jstat -gcnew <pid>:年轻代对象的信息  
jstat -gcnewcapacity<pid>: 年轻代对象的信息及其占用量。  
jstat -gcold <pid>：old代对象的信息。  
stat -gcoldcapacity <pid>: old代对象的信息及其占用量。  
```

### jhat
解析Java堆dump并启动一个web服务器，然后就可以在浏览器中查看堆的dump文件了。  
```
jps -l  
jmap -dump:format=b,file=heapDump 62247  
jhat heapDump  
```
