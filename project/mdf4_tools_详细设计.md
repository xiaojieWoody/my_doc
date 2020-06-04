## 需求

<img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200514093159344.png" alt="image-20200514093159344" style="zoom:80%;" />

## 处理流程

<img src="/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200518161929739.png" alt="image-20200518161929739"  />

## 详细设计

* 目录中的月和日都是两位数，如是1月，则01

### 目录规范

```shell
# /maprposix/dp.stg.tj/ad-vantage/data/store/collected
			/car-data/MDF4/ingest/BM60408/2019/10/22/697be47d-fb4d-448d-ab48-016df20ea8db/3100082/
			BN_CALIFR/20191022T074708_20191022T074728_715486_BM60408_BN_CALIFR.MF4
# 程序所能读取路径
/car-data/MDF4/ingest/BM60408/2019
		/10
				/01/697be47d-fb4d-448d-ab48-016df20ea8db/3100082
						/MT_RE
								20191022T074708_20191022T074728_715486_BM60408_MT_RE.MF4
								20191022T074708_20191022T074728_715486_BM60408_MT_RE.MF4
								...
						/BN_CALIFR
								20191022T074708_20191022T074728_715486_BM60408_BN_CALIFR.MF4
								...
						/BN_FASDLT
						/BN_IUKETH	
				...				
				/31/697be47d-fb4d-448d-ab48-016df20ea8db/3100082
						/BN_CALIFR
								20191022T074708_20191022T074728_715486_BM60408_BN_CALIFR.MF4
								20191022T074708_20191022T074728_715486_BM60408_BN_CALIFR.MF4
								...
						/BN_CALIFR
								20191022T074708_20191022T074728_715486_BM60408_BN_CALIFR.MF4
								...
						/BN_FASDLT
						/BN_IUKETH		
```

### 配置文件

```properties
# 合规一次处理MF4个数
mdf4.pipeline.mdf4.processNum=8
# MF4未处理文件所在根目录
mdf4.pipeline.mdf4.mdf4SourceRootPath=/ad-vantage/data/store/collected/car-data/MDF4
# MF4处理后文件存入根目录
mdf4.pipeline.mdf4.mdf4DstRootPath=/ad-china/data/store/compliance/c-processed/collected/car-data/MDF4
# 记录已处理日志的json文件的根目录
mdf4.pipeline.mdf4.jsonPath=/data/home/mdf4_tools/data
# 月json文件名
mdf4.pipeline.mdf4.monthProcessedJsonName=processed_mdf4_dir_list.json
# 日json文件名
mdf4.pipeline.mdf4.dayProcessedJsonName=processed_mdf4_file_list.json
# 处理失败存的json文件名字
mdf4.pipeline.mdf4.processingFailedMdf4Json=/processing_failed_mdf4_list.json
# 处理时间重复最多次数
mdf4.pipeline.mdf4.failedMdf4MaxNumber=3
# 目标车ID，--car-ids any不校验car id
mdf4.pipeline.mdf4.carID=BM60408
# 扫描当天及前几天，扫描当天及前几天的数据，如果为3，今天05/24，则会扫描05/24，05/23，05/22，05/21的数据
mdf4.pipeline.mdf4.forwardDayNums=6
# MF4文件后缀名
mdf4.pipeline.mdf4.mf4Suffix=.MF4
# Axis类型
mdf4.pipeline.mdf4.pipeline.axis=TVRfUkU=
#mdf4.pipeline.mdf4.pipeline.axis=UkVGRVRI
# CAN类型
mdf4.pipeline.mdf4.pipeline.can=Qk5fQ0FMSUZSLEJOX0ZBU0RMVCxCTl9JVUtFVEg=
# 定期扫描频率 minute
mdf4.pipeline.mdf4.processLoopRate=60
# 合规处理类型，串行SINGLE和并行BATCH
mdf4.pipeline.mdf4.processType=SINGLE,BATCH
# 线程池参数
mdf4.pipeline.mdf4.coreThreadNum=8
mdf4.pipeline.mdf4.maxThreadNum=24
mdf4.pipeline.mdf4.blockQueueCapacity=2048
```

### 扫描文件及增量判断

* 说明

  * /A/B/C/D目录结构，当D目录中增加或者减少文件后，只会修改D目录的最后修改时间，A、B、C的最后修改时间不会改变

* 记录某个月的所有已处理MF4所在目录的绝对路径及其最后修改时间，一个月生成一个

  ```json
  # 202005_processed_mf4_dir_list.json   
  {
      "/car-data/MDF4/ingest/BM60408/2019/10/01/697be47d-fb4d-448d-ab48-016df20ea8db/3100082/BN_CALIFR":156789283112,
      "/car-data/MDF4/ingest/BM60408/2019/10/01/697be47d-fb4d-448d-ab48-016df20ea8db/3100081/BN_CALIFR":156789283112
  }
  ```

* 某天已处理的MF4文件绝对路径，一天生成一个

  ```json
  # 20200501_processed_mdf_file_list.json
  [
      "/car-data/MDF4/ingest/BM60408/2019/10/01/697be47d-fb4d-448d-ab48-016df20ea8db/3100082/MT_RE/20191022T074708_20191022T074728_715486_BM60408_MT_RE.MF4",
      "/car-data/MDF4/ingest/BM60408/2019/10/01/697be47d-fb4d-448d-ab48-016df20ea8db/3100082/BN_CALIFR/20191022T074708_20191022T074728_715486_BM60408_BN_CALIFR.MF4"
  ]
  ```

* 记录处理失败的json文件，存放在json文件所在目录的根目录，处理次数超过3次则后面循环不再处理

  ```json
  # processing_failed_mdf4_list.json
  {
      "/car-data/MDF4/ingest/BM60408/2019/10/01/697be47d-fb4d-448d-ab48-016df20ea8db/3100082/BN_CALIFR/XXX.MF4":1,
      "/car-data/MDF4/ingest/BM60408/2019/10/01/697be47d-fb4d-448d-ab48-016df20ea8db/3100081/BN_CALIFR/XXX.MF4":4
  }
  ```

* 遍历根目录

  ```shell
  # 扫描当前天及前几天数据  
  配置文件中配置mdf4.pipeline.mdf4.forwardDayNums=4，即如果当前日期在一个月的1-4号之内，则还需查找上个月的已处理目录的json文件(月json文件)，所以会有当月和上月两个已处理目录的json文件；如果当前日期大于4号，则只查找当前月的已处理文件目录的json文件
  例如：
  	当前2020/05/01，则也会扫描2020/04/30,2020/04/29,2020/04/28,2020/04/27这些天的数据，然后查找4月和5月的已处理目录的json文件
  	当前2020/05/20，则会扫描2020/05/19,2020/05/18,2020/05/17,2020/05/16这些天的数据，然后只查找5月的已处理目录的json文件
  # 遍历配置的根目录
  1. 配置文件中配置扫描的根目录，/car-data/MDF4/
  2. 遍历根目录
  				/BM60408/，为目标车ID，目前从配置文件中获取然后遍历根目录时过滤掉非目标车ID数据
  				/2020/05/20/，遍历时只筛选出当前及前几天的数据
  				/MT_RE/，配置文件中配置需要扫描的MF4类型，过滤掉非目标类型数据
  3. 遍历判断
  		 只获取目录中有MF4文件的目录，拿到其绝对路径及最后修改时间
  		 根据绝对路径值到202005_processed_mf4_dir_list.json中查找，并对比各自的最后修改时间，如果存在且相同，则过滤掉该目录，否则接着遍历
  		 遍历完后，可以获取未处理或有更新的MF4文件目录的绝对路径（待处理的MF4目录路径）
  4. 遍历待处理的MF4目录路径	
  		待处理的MF4目录路径中获取该目录生成的日期（如/2020/05/20/），然后根据日期（如/2020/05/20/）查找对应日期（如/2020/05/20/）的202005xx_processed_mdf_file_list.json
  		获取待处理的MF4目录中的MF4文件绝对路径 和 202005xx_processed_mdf_file_list.json中对比，过滤掉相同数据(MF4绝对路径)，剩下即最终待处理的MF4文件绝对路径 
  ```

### MF4处理

* 目前已能获取一次扫描并已做增量判断的各种类型的`MF4`文件

  ```java
  // MF4文件绝对路径
  List<String> targetMT_RE = ...
  List<String> targetBN_CALIFR = ...
  List<String> targetBN_FASDLT = ...
  List<String> targetBN_IUKETH = ...
  ```

* `stream_name`验证

  * 此时已能拿到各种`MF4`文件，合并成一个`List<List<String> multipleMF4`

  * `stream_name`验证封装成一个方法，提交到线程池中去执行（多线程方式处理），带返回值的任务类型Future，返回处理成功的MF4绝对路径`List<List<String>> resMF4`

  * 方法中传入各种类型`MF4`的`multipleMF4`，验证完后，获取配置文件中`{dstMF4RootPath}`，替换掉MF4文件绝对路径中的`{sourceMF4RootPath}`，组成一个MF4文件目标绝对路径，然后将验证后的MF4复制到该目标绝对路径（需先判断父目录如果不存在，则先创建父目录）

    ```shell
    # 原绝对路径
    /{sourceDirPath}/{year}/{month}/{day}/mdf4_dir_name/xxx.MF4
    # 目标绝对路径
    /{dstMF4RootPath}/{year}/{month}/{day}/mdf4_dir_name/xxx.MF4
    ```

* `合规处理`

  * 为避免一次读入过多`MF4`到内存导致内存溢出，采取分批处理

    * 获取项目配置文件中配置的一次处理`MF4`文件个数`processNum`，例如20个
    * 然后`List<String> target`根据`processNum`来分批得到`List<List<String>> batchList`，然后根据`batchList`的`size`来多次调用`pipeline`中的合规处理

  * 合规处理流程封装成一个方法，根据`batchList`的`size`来决定调用次数

    * 方法中传入`List<String> mf4AbPath`

* 获取配置文件中`{dstMF4RootPath}`，替换掉MF4文件绝对路径中的`{sourceMF4RootPath}`，组成一个MF4文件目标绝对路径，合规处理完成生成新的MF4文件存储到该路径下（需先判断父目录如果不存在，则先创建父目录）

  ```shell
  # 原绝对路径
  /{sourceDirPath}/{year}/{month}/{day}/mdf4_dir_name/xxx.MF4
  # 目标绝对路径
  /{dstMF4RootPath}/{year}/{month}/{day}/mdf4_dir_name/xxx.MF4
  ```

* 获取本次循环处理失败的MF4文件信息（扫描后待处理的 - 处理成功的），读取记录失败MF4的JSON文件并筛选出MF4文件已经处理不超过3次的MF4，然后这一批再次进行处理，如果处理成功，则记录到成功的List中，如果还是失败，则失败处理记录再加1，然后将失败的数据更新到失败json文件中，替换掉上一个失败的json文件

### 更新json文件

* 当合规处理完成后，记录处理成功的MF4文件`List<String> complianceRes`，以及根据提交到线程池的可以获取返回值任务的`Future.get()`获取`stream_name`验证处理的结果`List<List<String>> verifyRes`
* 将`complianceRes`和`verifyRes`的值追加到对应日期（天）的`20200501_processed_mdf_file_list.json`的对应结构中
  * 如果当天的`json`文件不存在，则新增，否则追加
* 如果当前月的`202005_processed_mf4_dir_list.json` 不存在，则新增，否则将这次处理后的目录绝对路径及最后修改时间追加到该`json`文件中

## 代码串讲

![image-20200513105305444](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200513105305444.png)

![image-20200513100705722](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200513100705722.png)

![image-20200513100721981](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200513100721981.png)



![image-20200513101814810](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200513101814810.png)

* records根据ts来分组，同一个ts组内的record组成一张jpeg

```shell
# 提取mf4中图片
java -jar mdf4-pipeline-1.0.0-jar-with-dependencies.jar /home/woody/dev_env/mdf4-tools/mf4/20191021T181434_20191021T181454_715486_BM60408_REFETH.MF4 
--ext_jpeg_to /home/woody/dev_env/mdf4-tools/dst_jpegs 
--output_jpeg

# offline 合规处理并生成新的mf4
/Users/dingyuanjie/work/engine/doc/mdf4_tools/20191021T181434_20191021T181454_715486_BM60408_REFETH.MF4
--mdf4_trans_output
--compliance_type OFFLINE
--trans_jpeg_dir /Users/dingyuanjie/work/engine/doc/mdf4_tools/pipeline/jpegs/dst_images
--ext_jpeg_to /Users/dingyuanjie/work/engine/doc/mdf4_tools/pipeline/jpegs/t_1 
--output_jpeg

# online 合规处理并生成新的mf4
java -jar mdf4-pipeline-1.0.0-jar-with-dependencies.jar /home/woody/dev_env/mdf4-tools/mf4/20191021T181434_20191021T181454_715486_BM60408_REFETH.MF4 
--mdf4_trans_output 
--compliance_type ONLINE 
```

* setup
  * 根据参数来注册processor
* preProcess
  * 读取MF4中records
* process
  * 执行注册的processor
* postProcess
  * 根据参数决定是否生成新mf4文件
* 目前MF4文件ts值前面多了-

## 测试

* 106.13.82.229

* 非batch

  * 1个mf4，36s
  * 2个mf4，73s
  * 5个mf4，188s
  * 10个mf4，382s

* batch

  * 1个mf4，13s
  * 2个mf4，21s
  * 5个mf4，46s
  * 10个mf4，86s
  * 15个mf4，130s
  * 20个mf4，170s
  * 设置JVM参数-XX:+UseG1GC -XX:MaxGCPauseMillis=200，堆内存默认128M - 2G
    * 30个mf4，270s，
    * 35个mf4，OutOfMemoryError
  * 设置JVM参数-XX:+UseG1GC -XX:MaxGCPauseMillis=200 -Xms1024m -Xmx4096m
    * 40个mf4，498s

## 部署

### 处理Axis数据

* 只合规处理Axis的MF4文件，用于测试串行和并行合规处理速度
* `106.13.82.229`服务器上`/data/mdf4-tools/online_pipeline` 目录中有
  * 串行的`jar(mdf4-pipeline-1.0.0_pipeline_axis_single.jar)`
  * 并行的`jar(mdf4-pipeline-1.0.0_pipeline_axis_batch.jar)`
* ![image-20200524173100405](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200524173100405.png)

```shell
# 参数说明
java -jar xxx.jar 
# /xxx/raw/mdf4_directory为待处理MF4目录，/xxx/rst/mdf4_directory处理后MF4存放目录
/xxx/raw/mdf4_directory,/xxx/rst/mdf4_directory
--mdf4_trans_output
--compliance_type ONLINE
# 要处理的Car ID的数据，多个用,分隔
--car_id BM60408
# 要处理的MF4类型，多个用,分隔
--mf4_type REFETH
# 多个MF4文件批量处理，每次处理的MF4文件个数，如果一次处理太多MF4文件则可能导致内存溢出，非必须，默认8，
--batch_size 8


--mdf4_trans_output --compliance_type ONLINE --car_id BM60408 --mf4_type REFETH --batch_size 8 /data/mdf4-tools/online_pipeline/car-data,/data/mdf4-tools/online_pipeline/car-data-target-0527_01
```

```shell
# 并行
# mdf4-pipeline-1.0.0_pipeline_axis_batch.jar
java -jar mdf4-pipeline-1.0.0_pipeline_axis_batch.jar
--mdf4_trans_output --compliance_type ONLINE --car_id BM60408 --mf4_type REFETH --batch_size 8 /data/mdf4-tools/online_pipeline/car-data/MDF4/ingest/BM60408/2020/04/29/697be47d-fb4d-448d-ab48-016df20ea7db,/data/mdf4-tools/online_pipeline/car-data/MDF4/ingest/BM60408/2020/04/29/rst/697be47d-fb4d-448d-ab48-016df20ea7db
```

```shell
# 串行
# mdf4-pipeline-1.0.0_pipeline_axis_single.jar
java -jar mdf4-pipeline-1.0.0_pipeline_axis_single.jar
--mdf4_trans_output --compliance_type ONLINE --car_id BM60408 --mf4_type REFETH --batch_size 8 /data/mdf4-tools/online_pipeline/car-data/MDF4/ingest/BM60408/2020/04/29/697be47d-fb4d-448d-ab48-016df20ea7db,/data/mdf4-tools/online_pipeline/car-data/MDF4/ingest/BM60408/2020/04/29/rst/697be47d-fb4d-448d-ab48-016df20ea7db
```

* 处理结果

![image-20200524173554836](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200524173554836.png)

###处理Axis和CAN

```shell
# Axis
MT_RE
# CAN
BN_CALIFR、BN_FASDLT、BN_IUKETH
# car-id
BM60408
```

```shell
# 命令参数在配置文件中都有默认值
java -jar xxx.jar
# 待处理的MF4目录 
--mf4-source-root-path /ad-vantage/data/store/collected/car-data/MDF4 
# 处理后存放MF4目录
--mf4-target-root-path /ad-china/data/store/compliance/c-processed/collected/car-data/MDF4 
# 记录已处理MF4文件的日志目录
--processed-log-root-path /data/mdf4-tools/pipeline/json_file
# 一次读取MF4文件个数，如果设置过大，则可能导致内存溢出，配置文件中默认值为8
--batch-process-num 8
# 扫描当天及前几天的数据，如果为3，今天05/24，则会扫描05/24，05/23，05/22，05/21的数据
--forward-day-num 6 
# 定时扫描间隔时间，即一次循环过后，等待多长时间开始下次循环
--process-loop-minute-rate 1 
# 车ID，多个用,分隔，配置文件中默认BM60408
--car-ids BM60408
# 只有两个值SINGLE（串行）和BATCH（并行），默认BATCH，
--process-type SINGLE
# Axis类型（多个用,分隔，Base64编码）MT_RE
--mf4-axis-type TVRfUkU=
# CAN类型（多个用,分隔，Base64编码）BN_CALIFR,BN_FASDLT,BN_IUKETH
--mf4-can-type Qk5fQ0FMSUZSLEJOX0ZBU0RMVCxCTl9JVUtFVEg=
```

```shell
# 串行
java -jar mdf4-pipeline-1.0.0_axis_can.jar --mf4-source-root-path /data/mdf4-tools/online_pipeline/car-data/MDF4 --mf4-target-root-path /data/mdf4-tools/online_pipeline/car-data/MDF4/rst --processed-log-root-path /data/mdf4-tools/online_pipeline/car-data/MDF4/rst/json_file --batch-process-num 8  --forward-day-num 60 --process-loop-minute-rate 2 --car-ids BM60408 --process-type SINGLE
# 并行
java -jar mdf4-pipeline-1.0.0_axis_can.jar --mf4-source-root-path /data/mdf4-tools/online_pipeline/car-data/MDF4 --mf4-target-root-path /data/mdf4-tools/online_pipeline/car-data/MDF4/rst --processed-log-root-path /data/mdf4-tools/online_pipeline/car-data/MDF4/rst/json_file --batch-process-num 8  --forward-day-num 60 --process-loop-minute-rate 2 --car-ids BM60408
```

* 4T   4-5G Axis  10G(Axis CAN)

* 一个容器只能运行一个mdf4-pipeline的jar，第二个运行的jar会导致第一个运行的jar停止

* 并行
  * 2063文件 处理前82G 处理后79G 
  * 17:09:26 - 20:19:59   11433s
  * 5.5s/个 MF4

* 串行
  * 开始时间：2020-05-25 22:21:59
  * 结束时间：2020-05-26 13:27:15
  * 54300s，54300 / 1730 = 31.4 s/个
  * 生成1730个MF4文件，共66G
    * 由于启动第二个jar导致第一个运行的jar停止，共2063个MF4文件，还有333个MF4没处理完

### 提取MF4中图片

```shell
# 使用的是 Mdf4Pipeline类中的main方法
# 提取mdf4中图片
java -jar xxx.jar /home/woody/dev_env/mdf4-tools/mf4/20191021T181434_20191021T181454_715486_BM60408_REFETH.MF4.trans --ext_jpeg_to /home/woody/dev_env/mdf4-tools/dst_jpegs --output_jpeg
```

```shell
--car-ids any
```









