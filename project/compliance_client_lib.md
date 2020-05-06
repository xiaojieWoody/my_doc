* 难点

  * 将cpplib.zip解压到指定目录，然后加载其中so失败

    * 解决

      * ccpplib.zip中的so之间有链接关系，首先c++那边处理好so之间的链接关系，用` ldd /tmp/fullstack/cpplib/libComplianceImageProcess.so`检测
  
        ![企业微信截图_2923a077-3e77-4fa5-9c05-6816ccbb34de](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/企业微信截图_2923a077-3e77-4fa5-9c05-6816ccbb34de.png)
    
      * 解压过后，因为java.class.path在虚拟机启动的时候就已经加载其下的类，所以需要将解压目录追加到java.class.path，然后重新reload java.class.path
      
        ```java
        String currentPath = System.getProperty("java.library.path");
        System.setProperty( "java.library.path", currentPath + ":" + cppLibLoadPath );
        // this forces JVM to reload "java.library.path" property
        Field fieldSysPath = ClassLoader.class.getDeclaredField( "sys_paths" );
        fieldSysPath.setAccessible( true );
        fieldSysPath.set( null, null );
        ```
    
      * load指定so
    
        ```java
      System.load(cppLibLoadPath + "/libComplianceImageProcess.so");
        ```
  
* 简介

  * 打成jar包供其他项目依赖

  * 对外提供多个方法，对图片进行合规处理（车牌和人脸识别并打马赛克、图片进行模糊、扭曲）

    ```java
    // 3个c++方法，对图片进行合规处理
    public class CppImageProcessImpl {
        static {
            // 需先加载c++类库，ComplianceImageProcess.so
            // /usr/lib目录下（LD_LIBRARY_PATH环境变量）
            System.loadLibrary("ComplianceImageProcess");
        }
        public static native byte[] imageProcess(byte[] srcFileData) throws Exception;
        public static native List<byte[]> batchImageProcess(List<byte[]> srcFiles) throws Exception;
        // c++所需配置参数，在java compliance.properties中配置
        public static native void init(int queueNum, int imageNum, String moduleDirPath, String cppLogDirPath, int quality) throws Exception;
    }
    ```

    ```java
    // java对外接口，其中会调用c++方法来处理图片
    // 1. 传入图片路径，对图片进行合规处理
    public static boolean complianceImageProcess(String srcFilePath, String dstFilePath)
    public static byte[] complianceImageProcess(byte[] srcFileData) throws Exception
    // 批量分页多线程处理
    public static int batchComplianceImageProcess(String srcDirPath, String dstDirPath) throws Exception
    // 批量分页多线程处理
    public static int batchComplianceImageProcess(String[] srcFilePaths, String dstDirPath) throws Exception
    // 批量分页多线程处理
    public static int batchComplianceImageProcess(String[] srcFilePaths, String[] dstFilePaths) throws Exception
    // 批量分页多线程处理
    public static List<byte[]> batchComplianceImageProcess(List<byte[]> srcFilesData) throws Exception 
    ```

    ```java
    // ComplianceLibrary.java
    // 其他项目添加compliance_client_lib依赖后，需在SpringBoot的main方法中调用loadLibrary()方法来初始化，然后才能调用依赖中合规处理图片的方法
    1. 读取compliance.properties中的配置信息（线程池参数、c++所需参数）
    2. c++所需model配置文件、so都集成到项目的resource目录下，启动时需要创建临时目录，将model拷贝到临时目录、将so的zip包拷贝到临时目录然后解压到/usr/lib目录下（LD_LIBRARY_PATH环境变量）  
    3. 调用c++方法之前需要   System.loadLibrary("ComplianceImageProcess");
    ```

* 技术点
  * 创建临时目录
  * nio拷贝文件
  * log4j2
  * 批量分页多线程处理
    * submit
    * invokeall