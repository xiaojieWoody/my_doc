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

