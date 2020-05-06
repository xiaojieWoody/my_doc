```shell
# JAVA Dump
就是虚拟机运行时的快照，将虚拟机运行时的状态和信息保存到文件中
线程dump：包含所有线程的运行状态，纯文本格式
堆dump：包含所有堆对象的状态，二进制格式
```

## jps

* 查看本机java进程信息

  ```shell
  # jps -l 
  输出应用程序main.class的完整package名或者应用程序jar文件完整路径名
  # jps -v 
  输出传递给JVM的参数
  
  如果jps命令失效，而我们又要获取pid，还可以使用以下两种方法：
  1、top | grep java
  2、ps -ef |grep java
  ```

## jstack

* `jstack pid`打印线程的**栈**信息，制作线程dump文件
* 主要用于生成指定进程当前时刻的线程快照，线程快照是当前java虚拟机每一条线程正在执行的方法堆栈的集合，生成线程快照的主要目的是用于定位线程出现长时间停顿的原因，如线程间死锁、死循环、请求外部资源导致长时间等待

## jmap

* 打印内存映射，制作**堆**dump文件

* 主要用于打印指定java进程的共享对象内存映射或堆内存细节

* 堆Dump是反映堆使用情况的内存镜像，其中主要包括系统信息、虚拟机属性、完整的线程Dump、所有类和对象的状态等。一般在内存不足，GC异常等情况下，我们会去怀疑内存泄漏，这个时候就会去打印堆Dump

* ```shell
  # jmap -heap pid
  查看堆使用情况
  # jmap -histo pid
  查看堆中对象数量和大小
  # jmap -dump:format=b,file=heapdump pid
  将内存使用的详细情况输出到文件
  然后使用jhat命令查看该文件：jhat -port 4000 文件名 ，在浏览器中访问http:localhost:4000/
  ```

* 该命令适用的场景是程序内存不足或者GC频繁，这时候很可能是内存泄漏。通过以上命令查看堆使用情况、大量对象被持续引用等情况

## jstat

* 性能监控工具

* 主要是对java应用程序的资源和性能进行实时的命令行监控，包括了对heap size和垃圾回收状况的监控

  ```shell
  jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]
  option：经常使用的选项有gc、gcutil
  vmid：java进程id
  interval：间隔时间，单位为毫秒
  count：打印次数
  ```

* **jstat -gc PID 5000 20**

  * ![image-20200427140851581](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200427140851581.png)

    ```shell
    S0C:年轻代第一个survivor的容量（字节）
    S1C：年轻代第二个survivor的容量（字节）
    S0U：年轻代第一个survivor已使用的容量（字节）
    S1U：年轻代第二个survivor已使用的容量（字节）
    EC：年轻代中Eden的空间（字节）
    EU：年代代中Eden已使用的空间（字节）
    OC：老年代的容量（字节）
    OU:老年代中已使用的空间（字节）
    PC：永久代的容量
    PU：永久代已使用的容量
    YGC：从应用程序启动到采样时年轻代中GC的次数
    YGCT:从应用程序启动到采样时年轻代中GC所使用的时间（单位：S）
    FGC：从应用程序启动到采样时老年代中GC（FULL GC）的次数
    FGCT：从应用程序启动到采样时老年代中GC所使用的时间（单位：S）
    ```

* **jstat -gcutil PID 5000 20**

  ![image-20200427141106523](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200427141106523.png)

  ```shell
  s0:年轻代中第一个survivor已使用的占当前容量百分比
  s1:年轻代中第二个survivor已使用的占当前容量百分比
  E:年轻代中Eden已使用的占当前容量百分比
  O:老年代中已使用的占当前容量百分比
  P:永久代中已使用的占当前容量百分比
  ```

## jhat

* 内存分析工具

* 主要用来解析java堆dump并启动一个web服务器，然后就可以在浏览器中查看堆的dump文件了

* **jhat heapdump**

  * 这个命令将heapdump文件转换成html格式，并且启动一个http服务，默认端口为7000。

  * 如果端口冲突，可以使用以下命令指定端口：**jhat -port 4000 heapdump**

## jinfo

* jinfo可以用来查看正在运行的java运用程序的扩展参数，甚至支持在运行时动态地更改部分参数

  ```shell
  基本使用语法如下： jinfo -< option > < pid > ，其中option可以为以下信息：
  -flag< name >: 打印指定java虚拟机的参数值
  -flag [+|-]< name >：设置或取消指定java虚拟机参数的布尔值
  -flag < name >=< value >：设置指定java虚拟机的参数的值
  ```

  ```shell
  # 显示了新生代对象晋升到老年代对象的最大年龄。在运行程序运行时并没有指定这个参数，但是通过jinfo，可以查看这个参数的当前的值
  >jinfo -flag MaxTenuringThreshold 676
  -XX:MaxTenuringThreshold=6
  # 是否打印gc详细信息
  >jinfo -flag PrintGCDetails 676
  -XX:-PrintGCDetails
  ```

## ==jcmd==

* 一个多功能工具，可以用来导出堆，查看java进程，导出线程信息，执行GC等。jcmd拥有jmap的大部分功能，Oracle官方建议使用jcmd代替jmap
* 使用 `jcmd -l` 命令列出当前运行的所有虚拟机

## jconsole

* 简易的可视化控制台

## ==JVisualVM==

* 功能强大的控制台

