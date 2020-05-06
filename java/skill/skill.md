* 获取jar包resource文件

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

* 获取jar包所在绝对路径

  ```java
  String resourcePath = FfmpegUtil.class.getProtectionDomain().getCodeSource().getLocation().getPath();
  ```
  
* 异常

  * main方法中抛出异常地方直接终止运行

    ```java
    public static void main(String[] args) throws Exception {
      for(int i = 0; i < 10; i++) {
        System.out.println("t");
        throw new Exception("Error");
      }
      System.out.println("test");
    }
    ```

  * SpringBoot中抛出异常，当前调用

    ```java
    @RequestMapping("/test")
    public void test() throws Exception {
      System.out.println("1：" + count);
      Test.test();  // 此处抛出异常，下面不再执行
      System.out.println("2：" + count);
      count++;
    }
    ```

* List转array

  ```java
  newSrcFilePaths.stream().toArray(String[]::new)
  ```

* java.lang.NoClassDefFoundError:

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

    



