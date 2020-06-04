```shell
# mdf4_tools
注意处理失败场景
```

# Java

## main

* 命令行参数解析

  ```java
  // java -jar xxx.jar /data/axis,/data/can --mf4-source-root-path /data/mdf4_tools/xxx
  
  public static final String OPT_HELP = "help";
  public static final String MF4_SOURCE_ROOT_PATH = "mf4-source-root-path"
  
  public List<String> filenameList;
  public static String mdf4SourceRootPath;
  private static CommandLine commandLine;
  private static Options options;
    
  public static void main(String[] args) throws Exception {
    // 解析启动参数
    parseArgs(args);
    if (commandLine.hasOption(OPT_HELP)) {
      printHelpMessage();
      return;
    }
    // ...
  }
  // 解析启动参数
  private static void parseArgs(String[] args) throws Exception {
    CommandLineParser parser = new DefaultParser();
    options = new Options();
  
    options.addOption("h", OPT_HELP, false, "print help message");
    options.addOption(null, MF4_SOURCE_ROOT_PATH, true, "mf4 source root path");
    // ...
  
    commandLine = parser.parse(options, args);
    // 带命令名的参数值
    mdf4SourceRootPath = getMf4SourceRootPath();
    if (commandLine.getArgList().size() != 1) {
      throw new ParseException("the program expects exactly at least filename list(string seperated by `,`) " + "as first parameter");
    } else {
      // 不带命令名的参数值，多个用,分隔
      // 如/data/axis,/data/can
      filenameList = Arrays.asList(commandLine.getArgList().get(0).split(","));
    }
  }
  
  // 打印命令帮助信息
  static void printHelpMessage() {
    new HelpFormatter().printHelp("commandName [OPTIONS] <MDF4-FILE>", options);
  }
  
  // 获取命令上的参数值
  public static String getMf4SourceRootPath() throws Exception {
    String mf4SourceRootPath = null;
    if (commandLine.hasOption(MF4_SOURCE_ROOT_PATH)) {
      mf4SourceRootPath = commandLine.getOptionValue(MF4_SOURCE_ROOT_PATH);
    } else {
      mf4SourceRootPath = Mdf4Utils.getPropertyByName("mdf4.pipeline.mdf4.mdf4SourceRootPath");
    }
    if (StringUtils.isBlank(mf4SourceRootPath)) {
      throw new Exception("mf4-source-root-path should not empty!");
    }
    if (!(Paths.get(mf4SourceRootPath).toFile().exists() && Paths.get(mf4SourceRootPath).toFile().isDirectory())) {
      throw new Exception("mf4-source-root-path should exist and it shoule be a directory!");
    }
    return mf4SourceRootPath;
  }
  ```

## File

* 获取`jar`包`resource`文件

  ```java
  // resources/ffmpeg/windows/ffmpeg.exe
  InputStream resourceAsStream = FfmpegUtil.class.getClassLoader().getResourceAsStream("ffmpeg/windows/ffmpeg.exe");
  File file = new File("/Users/dingyuanjie/work/engine/doc/license/tmp/ffmpeg.exe");
  FileUtils.copyInputStreamToFile(resourceAsStream, file);
  
  
  Properties props = new Properties();
  try {
    InputStream stream = ComplianceLibrary.class.getResourceAsStream("/compliance.properties");
    System.out.println(stream);
    BufferedReader br = new BufferedReader(new InputStreamReader(stream, "UTF-8"));
    props.load(br);
  } catch (IOException e) {
    log.error("load config properties fail: {}", e);
  }
  ```

* 获取`jar`包所在绝对路径

  ```java
  String resourcePath = FfmpegUtil.class.getProtectionDomain().getCodeSource().getLocation().getPath();
  ```

## NIO

* `Path`

  ```java
  String dirPath = "/data/mdf4-tools";
  Path dir = Paths.get(dirPath);
  ```

* 创建临时目录，程序退出时自动删除，遍历根目录下的文件和目录

  ```java
  private static Path createTempDirectory() throws IOException {
    //创建临时目录，不指定根，具体位置取决于操作系统  可以通过 System.getProperty("java.io.tmpdir") 查看
    Path tempDirectory = Files.createTempDirectory("compliance_lib");
    //使用"钩子"的方式删除临时目录（在程序退出后）,一般可以不手动清理，有的系统在重启后会自动清理临时目录
    Runtime.getRuntime().addShutdownHook(new Thread(() -> {
      try {
        Files.deleteIfExists(tempDirectory);
      } catch (DirectoryNotEmptyException e) {
        try {
          // 遍历指定根路径下的文件和目录
          Files.walkFileTree(tempDirectory, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
              // 删除文件
              Files.delete(file);
              return FileVisitResult.CONTINUE;
            }
  
            @Override
            public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
              // 删除目录
              Files.delete(dir);
              return super.postVisitDirectory(dir, exc);
            }
          });
        } catch (IOException ex) {
          log.error("", e);
        }
      } catch (IOException e) {
        log.error("delete tmp file error!", e);
      }
    }));
    return tempDirectory;
  }
  ```

* 拷贝文件

  ```java
  InputStream modelBinary = ComplianceLibrary.class.getResourceAsStream(location);
  if(modelBinary == null) {
    throw new Exception("cpp model file " + location + " is not exists!");
  }
  Path destinationModel = tempDirectory.resolve("./" + location).normalize();
  Files.copy(modelBinary, destinationModel, new CopyOption[0]);
  modelBinary.close();
  ```

## 异常

* `main`方法中抛出异常地方直接终止运行

  ```java
  public static void main(String[] args) throws Exception {
    for(int i = 0; i < 10; i++) {
      System.out.println("t");
      throw new Exception("Error");
    }
    System.out.println("test");
  }
  ```

* `SpringBoot`中抛出异常，当前调用

  ```java
  @RequestMapping("/test")
  public void test() throws Exception {
    System.out.println("1：" + count);
    Test.test();  // 此处抛出异常，下面不再执行
    System.out.println("2：" + count);
    count++;
  }
  ```

## 集合

* 转换

  ```java
  // List to Array
  newSrcFilePaths.stream().toArray(String[]::new);
  // Array to List
  String[] s = new String[]{"A", "B", "C", "D","E"};
  Arrays.asList(s);
  // List to Set
  Set<String> set = new HashSet<>(list);
  // Set to List
  List<String> list_1 = new ArrayList<>(set);
  // Array to Set
  s = new String[]{"A", "B", "C", "D","E"};
  set = new HashSet<>(Arrays.asList(s));
  // Set to Array
  dest = set.toArray(new String[0]);
  ```

## Lambda

* `list1 - list2`

  ```java
  List<String> mt_re = new ArrayList<>();
  mt_re.add("1");
  mt_re.add("2");
  mt_re.add("3");
  List<String> processedMdf4 = new ArrayList<>();
  processedMdf4.add("3");
  processedMdf4.add("4");
  processedMdf4.add("5");
  
  List<String> collect = mt_re.stream().filter(item -> !processedMdf4.contains(item)).collect(Collectors.toList());
  System.out.println(JSON.toJSONString(collect));
  ```

* 比较两个`List<String>`是否相同

  ```java
  String str1 = list1.stream().sorted().collect(Collectors.joining());
  String str2 = list2.stream().sorted().collect(Collectors.joining());
  System.out.println(str1);
  System.out.println(str2);
  System.out.println(str1.equals(str2));
  ```

## maven

* `java.lang.NoClassDefFoundError`

  * 本地依赖jar包没打进去

  * 解决

    ```xml
    <dependency>
      <groupId>com.tencent.autocloud</groupId>
      <artifactId>compliance_client_lib</artifactId>
      <version>1.0-SNAPSHOT</version>
      <scope>system</scope>
      <systemPath>${basedir}/src/main/libs/compliance_client_lib-1.0-SNAPSHOT.jar</systemPath>
    </dependency>
    
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <version>2.2.6.RELEASE</version>
      <configuration>
        <!--把项目打成jar，同时把本地jar包也引入进去-->
        <includeSystemScope>true</includeSystemScope>
      </configuration>
    </plugin>
    ```

* 可执行的普通jar包

  ```xml
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>3.0.0</version>
    <configuration>
      <!--只产生一个jar包-->
      <appendAssemblyId>false</appendAssemblyId>
      <descriptorRefs>
  			<!--将所需的依赖jar包打包到jar中-->
        <descriptorRef>jar-with-dependencies</descriptorRef>
      </descriptorRefs>
      <archive>
        <manifest>
          <!--java -jar时运行的main方法-->
          <mainClass>com.tencent.ad.mdf4.pipeline.Mdf4Pipeline</mainClass>
        </manifest>
      </archive>
    </configuration>
    <executions>
      <execution>
        <phase>package</phase>
        <goals>
          <goal>single</goal>
        </goals>
      </execution>
    </executions>
  </plugin>
  
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.0</version>
    <configuration>
      <source>8</source>
      <target>8</target>
    </configuration>
  </plugin>
  ```

## 日期

* `Calendar`

  ```java
  SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
  Date date = new Date();
  Calendar calendar = Calendar.getInstance();
  calendar.setTime(date);
  // 当前日期（String）
  String currentDateStr = sdf.format(calendar.getTime());
  // 昨天
  calendar.add(Calendar.DAY_OF_MONTH, -1);
  sdf.format(calendar.getTime());
  // 上一个月
  calendar.add(Calendar.MONTH, -1);
  sdf.format(calendar.getTime());
  ```

## String

* 格式化

  ```java
  // 保留2位，不足的前面补0
  String result = String.format("%02d", cal.get(Calendar.MONTH) + 1);
  ```

## 正则表达式

* 截取字符串中某一片段

  ```java
  // /car-data/MDF4/ingest/BM60408/2020/05/19/697be47d-fb4d-448d-ab48-016df20ea8db/
  // 如获取 /2020/05/19/
  String reg = "(.*)(/\\d{4}/\\d{2}/\\d{2}/)(.*)";
  Pattern compile = Pattern.compile(reg);
  Matcher matcher = compile.matcher(dateStr);
  if (matcher.matches()) {
    return matcher.group(2);
  }  
  ```

  

## JSON

* json文件转对象

  ```xml
  <dependency>
      <groupId>com.googlecode.json-simple</groupId>
      <artifactId>json-simple</artifactId>
      <version>1.1.1</version>
  </dependency>
  ```

  ```java
  JSONParser jsonParser = new JSONParser();
  Object parse = jsonParser.parse(new FileReader("/Users/dingyuanjie/work/engine/doc/mdf4_tools/tmp/1.json"));
  List<Mdf4Info> parse1 = (List<Mdf4Info>) parse;
  ```

  ```json
  [
      {
          "mdf4AbDirPath":[
  "/Users/dingyuanjie/work/engine/doc/mdf4_tools/tmp/2020/05/14/20191021T180607_715486_BM60408_14_2_MT_RE/"
          ],
          "mdf4Name":[
              [
                  "20191021T180607_715486_BM60408_14_2_1_MT_RE.MF4",
                  "20191021T180607_715486_BM60408_14_2_3_MT_RE.MF4",
                  "20191021T180607_715486_BM60408_14_2_2_MT_RE.MF4"
              ]
          ],
          "mdf4Type":"MT_RE"
      },
      {
          "mdf4AbDirPath":[
  "/Users/dingyuanjie/work/engine/doc/mdf4_tools/tmp/2020/05/14/20191021T180607_715486_BM60408_14_2_MT_RE/111"
          ],
          "mdf4Name":[
              [
                  "20191021T180607_715486_BM60408_14_2_1_MT_RE.MF4111",
                  "20191021T180607_715486_BM60408_14_2_3_MT_RE.MF4",
                  "20191021T180607_715486_BM60408_14_2_2_MT_RE.MF41111"
              ]
          ],
          "mdf4Type":"MT_RE11111111111"
      }
  ]
  ```

* json对象转file

  ```java
  try (FileWriter fileWriter = new FileWriter("/Users/dingyuanjie/work/engine/doc/mdf4_tools/tmp/2.json");) {
  	fileWriter.write(JSON.toJSONString(processedMdf4Info));
  } catch (Exception e ) {
    logger.info("",e);
  }
  ```

##  Thread

* 分页

  ```java
  private static List<List<String>> batchMdf4Info(List<String> sourceMf4List, Integer pageSize) {
    if (sourceMf4List == null || sourceMf4List.size() < 1) {
      return null;
    }
  
    List<List<String>> res = new ArrayList<>();
    int total = sourceMf4List.size();
    int page = total % pageSize == 0 ? total / pageSize : total / pageSize + 1;
    for (int i = 0; i < page; i++) {
      int start = 0;
      int end = 0;
      if ((i * pageSize + pageSize) <= total) {
        start = i * pageSize;
        end = i * pageSize + pageSize;
      }
      if ((i * pageSize + pageSize) > total) {
        start = i * pageSize;
        end = total;
      }
      if (total < pageSize) {
        start = 0;
        end = total;
      }
      List<String> subList = sourceMf4List.subList(start, end);
      res.add(subList);
    }
  
    return res;
  }
  ```

* 周期执行线程池

  ```java
  public class ScheduleTest {
  
      public static void main(String[] args) {
          ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
                  new BasicThreadFactory.Builder().namingPattern("mf4-online-process-schedule-pool-%d").daemon(true).build());
          // 任务执行完后等待500毫秒后继续执行下一个任务
          ScheduledFuture<?> scheduledFuture = executorService.scheduleWithFixedDelay(
                  new ProcessRunnable(),
                  0,
                  500,
                  TimeUnit.MILLISECONDS);
          try {
              scheduledFuture.get();
          } catch (InterruptedException e) {
              e.printStackTrace();
          } catch (ExecutionException e) {
              e.printStackTrace();
          }
      }
  
      static class ProcessRunnable implements Runnable{
          @Override
          public void run() {
              System.out.println("________________1_________________");
              try {
                  Thread.sleep(3000);
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println("________________2_________________");
          }
      }
  }
  ```

* 多线程任务

  ```java
  public List<Future<List<String>>> mdf4CanHandle(Map<String, List<String>> canRes) throws Exception {
    ExecutorService pool = Mdf4CarmenPipeline.executorService;
    // 创建多个有返回值的任务
    List<Future<List<String>>> list = new ArrayList<>();
    if (canRes != null && canRes.size() > 0) {
      Iterator<Map.Entry<String, List<String>>> iterator = canRes.entrySet().iterator();
      while (iterator.hasNext()) {
        Map.Entry<String, List<String>> next = iterator.next();
        // 做内部heard校验
        Callable<List<String>> c = new MyCallable(next.getKey(), next.getValue());
        // 执行任务并获取Future对象
        Future<List<String>> submitTask = pool.submit(c);
        list.add(submitTask);
      }
    }
    return list;
  }
  
  // 可以是内部类
  class MyCallable implements Callable<List<String>> {
    private String fileType;
    private List<String> fileAbPath;
  
    MyCallable(String fileType, List<String> fileAbPath) {
      this.fileType = fileType;
      this.fileAbPath = fileAbPath;
    }
  
    @Override
    public List<String> call() throws Exception {
      List<String> res = new ArrayList<>();
      if (fileAbPath != null && fileAbPath.size() > 0) {
        for (String path : fileAbPath) {
          try {
            logger.info("CAN MF4 path: "+path);
             // 有问题先打印日志，慎重抛异常
             // ...
              res.add(path);
            } else {
              logger.info("CAN check failed Recorder.Name is: " + streamName + " MF4 path :"+path);
            }
          } catch (Exception e) {
            logger.error(path+"CAN check exception :"+e.getMessage());
          }
        }
      }
      return res;
    }
  }
  ```
  
* 异步获取结果（防止超时）

  ```java
  private static List<String> getSuccessCanMF4Path(List<Future<List<String>>> canHandleList) {
    List<String> successCanMF4Path = new ArrayList<>();
    do {
      Iterator<Future<List<String>>> iterator = canHandleList.iterator();
      canHandleList = new ArrayList<>();
      while (iterator.hasNext()) {
        Future<List<String>> next = iterator.next();
        try {
          List<String> fileAbPath = next.get(300, TimeUnit.MILLISECONDS);
          if (fileAbPath != null && fileAbPath.size() > 0) {
            successCanMF4Path.addAll(fileAbPath);
          }
        } catch (TimeoutException e) {
          canHandleList.add(next);
        } catch (Exception e) {
          log.info("", e);
        }
      }
    } while (canHandleList.size() > 0);
  
    return successCanMF4Path;
  }
  ```
  
* 自定义线程池

  ```java
  public static ExecutorService executorService;
  // 自定义线程池
  public static ExecutorService getExecutorService() {
    String coreThreadStr = Mdf4Utils.getPropertyByName("mdf4.pipeline.mdf4.coreThreadNum");
    int coreThread = coreThreadStr == null ? Runtime.getRuntime().availableProcessors() * 2 : Integer.valueOf(coreThreadStr);
    String maxThreadStr = Mdf4Utils.getPropertyByName("mdf4.pipeline.mdf4.maxThreadNum");
    int maxThread = maxThreadStr == null ? Runtime.getRuntime().availableProcessors() * 2 * 2 : Integer.valueOf(maxThreadStr);
    String blockQueueCapacityStr = Mdf4Utils.getPropertyByName("mdf4.pipeline.mdf4.blockQueueCapacity");
    int blockQueueCapacity = blockQueueCapacityStr == null ? 1024 : Integer.valueOf(blockQueueCapacityStr);
  
    ThreadFactory nameThreadFactory = new ThreadFactoryBuilder().setNameFormat("compliance_client_lib-pool-%d").build();
    executorService = new ThreadPoolExecutor(coreThread,
                                             maxThread,
                                             0L,
                                             TimeUnit.MILLISECONDS,
                                             new LinkedBlockingQueue<Runnable>(blockQueueCapacity),
                                             nameThreadFactory,
                                             new ThreadPoolExecutor.AbortPolicy());
  
    return executorService;
  }
  ```

## Test

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
</dependency>
  
@RunWith(SpringRunner.class)
@SpringBootTest
public class AppTest {

    @Autowired
    private MetaDatabaseService metaDatabaseService;

    @Test
    public void testSave() {
        MetaDatabase metaDatabase = new MetaDatabase();
        metaDatabase.setName("defaule");
        metaDatabase.setLocation("hdfs://hadoop000:8020/user/hive/warehouse");
        metaDatabaseService.save(metaDatabase);
    }
}
```

# DAO

## JPA

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>8.0.15</version>
</dependency>
```

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: root
    password: IamMySQL0108
    url: jdbc:mysql://127.0.0.1:3306/imooc_boot_scala?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
  jpa:
    hibernate:
      # 自动建表
      ddl-auto: update
    database: mysql
    open-in-view: true
```

```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;
/**
 * 数据库元数据
 */
@Entity
@Table
@Data
public class MetaDatabase {

    /**数据库ID*/
    @Id
    @GeneratedValue         //自增
    private Integer id;
    /**数据库名称*/
    private String name;
    /**数据库存放的文件系统地址*/
    private String location;
}
```

```java
public interface MetaDatabaseRepository extends CrudRepository<MetaDatabase, Integer> {
}
```

```java
@Service
public class MetaDatabaseService {

    @Autowired
    private MetaDatabaseRepository metaDatabaseRepository;

    @Transactional
    public void save(MetaDatabase metaDatabase) {
        metaDatabaseRepository.save(metaDatabase);
    }

    public Iterable<MetaDatabase> query() {
        return metaDatabaseRepository.findAll();
    }
}
```

# Scala

```xml
<scala.version>2.11.8</scala.version>

<dependency>
  <groupId>org.scala-lang</groupId>
  <artifactId>scala-library</artifactId>
  <version>${scala.version}</version>
</dependency>

<!--添加scala的plugin-->
<plugin>
  <groupId>net.alchim31.maven</groupId>
  <artifactId>scala-maven-plugin</artifactId>
  <version>3.2.1</version>
  <executions>
    <execution>
      <id>compile-scala</id>
      <phase>compile</phase>
      <goals>
        <goal>add-source</goal>
        <goal>compile</goal>
      </goals>
    </execution>
    <execution>
      <id>test-compile-scala</id>
      <phase>test-compile</phase>
      <goals>
        <goal>add-source</goal>
        <goal>testCompile</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <recompileMode>incremental</recompileMode>
    <scalaVersion>${scala.version}</scalaVersion>
    <args>
      <arg>-deprecation</arg>
    </args>
    <jvmArgs>
      <jvmArg>-Xms64m</jvmArg>
      <jvmArg>-Xmx1024m</jvmArg>
    </jvmArgs>
  </configuration>
</plugin>
```

```scala
import javax.persistence.{Entity, GeneratedValue, Id, Table}

import scala.beans.BeanProperty

@Entity
@Table
class MetaTable {
  @Id
  @GeneratedValue
  @BeanProperty
  var id:Integer = _

  @BeanProperty
  var name:String = _

  @BeanProperty
  var tableType:String = _

  @BeanProperty
  var dbId:Integer = _
}
```

```scala
import com.dyj.domain.MetaTable
import org.springframework.data.repository.CrudRepository

trait MetaTableRepository extends CrudRepository[MetaTable, Integer]{
}
```

```scala
@Service
class MetaTableService @Autowired()(metaTableRepository: MetaTableRepository) {
	
  @Transactional
  def save(metaTable:MetaTable) = {
    metaTableRepository.save(metaTable)
  }

  def query() = {
    metaTableRepository.findAll()
  }
}
```

```scala
@RestController
@RequestMapping(Array("/meta/table"))
class MetaTableController @Autowired()(metaTableService: MetaTableService) {

  @RequestMapping(value = Array("/"), method = Array(RequestMethod.POST))
  @ResponseBody
  def save(@ModelAttribute metaTable: MetaTable) = {
    metaTableService.save(metaTable)
    ResultVOUtils.success()      // scala 调用已有的Java代码
  }

  @RequestMapping(value = Array("/"), method = Array(RequestMethod.GET))
  @ResponseBody
  def query() = {
    ResultVOUtils.success(metaTableService.query())
  }
}
```

# MySQL

* java.sql.SQLException: Access denied for user ''@'localhost' (using password: NO)

  ```shell
  spring:
    datasource:
      driver-class-name: com.mysql.cj.jdbc.Driver
      username: root                      # data-username 改为 username
      password: IamMySQL0108							# data-password 改为 password
      url: jdbc:mysql://127.0.0.1:3306/imooc_boot_scala?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
  ```

# Linux

* tar

  ```shell
  # 压缩
  tar -zcvf mdf4_tools_0528_v1.tar.gz mdf4_tools
  # 解压
  tar -zxvf mdf4_tools_0528_v1.tar.gz
  ```

* 

# Docker

```shell
# 复制
docker cp  /xxx/x/.txt containerName:/tmp
# 创建容器
docker run -idockt --name test2 --net host 182.61.181.127:5000/tad_centos_7.6:v1.0
# exited 状态变为 running
docker start containerId
```





