# resources

* **读取jar包中的resources文件**

  * 是内嵌web容器，其特点是只有一个jar文件，在容器启动后不会解压缩，项目实际访问时jar包或者war包
  * 读取jar里面的文件，只能用流去读取，不能用file

  ```java
  InputStream is = ResourcesUtil.class.getResourceAsStream("static/assets/test.txt");
  ```

* 普通的web项目，像用Tomcat容器，特点是压缩包随着容器的启动会解压缩成一个文件夹，项目访问的时候，实际是去访问文件夹，而不是jar或者war包

  ```java
  import org.springframework.util.ResourceUtils;
  File file= ResourceUtils.getFile("classpath:test.txt");
  // 或者
  ClassPathResource classPathResource = new ClassPathResource("test.txt");
  获取文件：classPathResource .getFile();
  获取文件流：classPathResource .getInputStream();
  ```

# File

* 文件复制

  ```java
  String str = "/Users/dingyuanjie/work/engine/doc/compliance_client_lib/gc/512m1g/1/2/3/5.txt";
  String sour = "/Users/dingyuanjie/work/engine/doc/compliance_client_lib/gc/gc.log.zip";
  File target = new File(str);
  File source = new File(sour);
  FileUtils.copyFile(source, target);    // 会自动创建source的父目录
  
  // 判断父目录
  if (!file.getParentFile().isDirectory()) {
    throw new Exception("dstImgPath file directory no exist");
  }
  ```

  