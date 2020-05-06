# Docker容器中

* 查看当前所用的GC回收器

  * `java -XX:+PrintCommandLineFlags -version`

  * `free -h`、`free -m`

  * 堆大小
    
  * `xxx(GB)=/1024/1024/1024`
    
  * ![image-20200420222504831](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200420222504831.png)

  * docker容器和主机间复制

    ```shell
    docker cp cpu:/home/compliance-tools-cpp/build/gc_log/gc_g1_512_1.log /data/compliance_client_lib_gc_log
    ```

* 启动jar包时添加参数改变GC回收器类型

  * ```shell
    # 使用SerialGC添加参数(单线程):       
    -XX:+UseSerialGC
    # 使用ParallelGC添加参数(并行的多线程收集器，目标是达到一个可控制的吞吐量):     
    -XX:+UseParallelGC 
    # 使用CMSGC添加参数(并发收集，目标是尽量减少停顿时间,基于标记-清除算法):          
    -XX:+UseConcMarkSweepGC
    # 使用G1GC添加参数(并发与并行、分代收集、空间整合、可预测的停顿,兼顾吞吐量和停顿时间):           
    -XX:+UseG1GC
    
    nohup java -jar -XX:+UseG1GC xxx.jar &
    
    # jdk1.8 官方建议
    java -jar -XX:+UseG1GC -XX:MaxGCPauseMillis=200 xxx.jar
    ```

* 开启GC日志,将GC日志导出到指定文件夹下,以便之后利用工具分析和参考

  * ```shell
    # #/root/outp是自己指定的路径,可以灵活指定,命名也是可以随便取的,只要后缀为.log即可
    -Xloggc:/root/outp/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps
    
    nohup java -jar -Xloggc:/root/outp/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps xxx.jar &
    
    # 启动后可以在root/outp文件夹下看到gc.log
    ```

* 借助工具分析GC日志

  * [在线的](https://gceasy.io/)
  
  * ==gcviewer工具==
  
    ```shell
    # 可以识别的日志输出命令
    nohup java -jar -Xloggc:/root/outp/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps  outp.jar &
    # 具体的存放目录/root/outp可以随意指定,gc.log命名也可随便取,后缀.log不要变
    ```

* jmeter

  ```shell
  /data/performance/fullstcksdk/CPU_test/oneBatchImageProcess_test.jmx
  
  /opt/apache-jmeter-5.2.1/bin/jmeter -n -t /data/performance/fullstcksdk/CPU_test/oneBatchImageProcess_test.jmx  -l /data/test.jtl
  ```

  ![企业微信截图_15874349141061](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/企业微信截图_15874349141061.png)

* nohup

  ```shell
  # linux下利用nohup后台运行jar文件包程序
  # 方式一：所有输出被重定向到nohup.out的文件中
  nohup java -jar XXX.jar &
  # 方式二：输出重定向到out.file文件
  nohup java -jar XXX.jar >temp.txt &
  # 通过jobs命令查看后台运行任务
  jobs
  # 将某个作业调回前台控制，只需要 fg + 编号即可
  fg 23
  ```

* top命令

  ```shell
  PID — 进程id
  USER — 进程所有者
  PR — 进程优先级
  NI — nice值。负值表示高优先级，正值表示低优先级
  VIRT — 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
  RES — 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
  SHR — 共享内存大小，单位kb
  S — 进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
  %CPU — 上次更新到现在的CPU时间占用百分比
  %MEM — 进程使用的物理内存百分比
  TIME+ — 进程使用的CPU时间总计，单位1/100秒
  COMMAND — 进程名称（命令名/命令行）
  ```

* 场景

  * 5线程

    * 512M1G

    ![image-20200421105751188](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421105751188.png)

    ```java
    nohup java -jar -Xms512m -Xmx1024m -Xloggc:/home/compliance-tools-cpp/build/gc_log/gc_default_512_1.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps ./demo-0.0.1-SNAPSHOT.jar  >./java_log/temp.txt &
    ```

    ![image-20200421110616332](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421110616332.png)

    ![image-20200421110639084](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421110639084.png)

    ![image-20200421112147567](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421112147567.png)

* 512m1g，15线程

  * ```shell
    nohup java -jar -Xms512m -Xmx1024m -Xloggc:/home/compliance-tools-cpp/build/gc_log/gc_512_1_15thread_1.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps ./demo-0.0.1-SNAPSHOT.jar  >./java_log/temp.txt &
    ```

    ![image-20200421115421940](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421115421940.png)

  * ![image-20200421122937826](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421122937826.png)

  * ![image-20200421123540880](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421123540880.png)

* 场景

  * G1收集器

    * 512M1G，5线程

      ```shell
      nohup java -jar -Xms512m -Xmx1024m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -Xloggc:/home/compliance-tools-cpp/build/gc_log/gc_g1_512_1.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps ./demo-0.0.1-SNAPSHOT.jar  >./java_log/temp.txt &
      ```

      ![image-20200421113144282](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421113144282.png)

    ![image-20200421114531209](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421114531209.png)

    

    ![image-20200421120133322](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421120133322.png)

  * 512M1G，15线程

    ```shell
    nohup java -jar -Xms512m -Xmx1024m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -Xloggc:/home/compliance-tools-cpp/build/gc_log/gc_g1_512_1_15thread.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps ./demo-0.0.1-SNAPSHOT.jar  >./java_log/temp.txt &
    ```

    ![image-20200421123352115](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421123352115.png)

    ![image-20200421130612677](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421130612677.png)

    ![image-20200421131237475](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200421131237475.png)

* 压测

```shell
启动jar
nohup java -jar -Xms1024m -Xmx3096m -Xloggc:/root/outp/gc_1g_3g_423_oldthread.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps  -XX:+UseG1GC -XX:MaxGCPauseMillis=200 demo-0.0.1-SNAPSHOT_oldthread.jar >./nohup_oldthread.log 2>&1 &
```

```shell
在容器外面启动jmeter压测：
 /opt/apache-jmeter-5.2.1/bin/jmeter -n -t /data/performance/wendingxing/oneBatchImageProcess_CPU_sdk.jmx -l /data/performance/wendingxing/test_3.jtl
```

* ==服务没挂、没接收请求、但是jmeter挂了==

![img](https://img-blog.csdn.net/2018092818225795?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdmV4aWFvdGFvemk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180928172321237?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdmV4aWFvdGFvemk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180928171804236?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xvdmV4aWFvdGFvemk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)









