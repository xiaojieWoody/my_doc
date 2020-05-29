# Java

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

* 遍历根路径下的文件和目录

  ```java
  String dirPath = "/data/mdf4-tools";
  Path dir = Paths.get(dirPath);
  Files.walkFileTree(dir, new SimpleFileVisitor<Path>(){
    @Override
    public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
      // 操作文件  file
      return super.visitFile(file, attrs);
    }
    @Override
    public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
      // 操作目录 dir
      return super.postVisitDirectory(dir, exc);
    }
  });
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

* list1 - list2

* 比较两个List<String> 是否相同

  ```java
  System.out.println(list1.stream().sorted().collect(Collectors.joining()).equals(list2.stream().sorted().collect(Collectors.joining())));
  ```

* json对象转file

  ```java
  try (FileWriter fileWriter = new FileWriter("/Users/dingyuanjie/work/engine/doc/mdf4_tools/tmp/2.json");) {
  	fileWriter.write(JSON.toJSONString(processedMdf4Info));
  } catch (Exception e ) {
    e.printStackTrace();
  }
  ```

##  Thread

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

* 异步获取结果

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
```





