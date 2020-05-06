* 106.13.112.109
* 106.13.109.155
* 106.13.144.31

```java
package com.tencent.autocloud.compliance.client.lib;

import com.google.common.util.concurrent.ThreadFactoryBuilder;
import com.tencent.autocloud.compliance.client.impl.CppImageProcessImpl;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;

import java.io.*;
import java.nio.file.CopyOption;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.*;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;

/**
 * @ClassName LinkLibrary
 * @Author datumpeng
 * @Date 2020/4/6 下午4:24
 * @Description TODO
 * @Version 1.0
 **/
@Slf4j
public class ComplianceLibrary {

    public static ExecutorService executorService = null;
    public static int imageNum = 0;

    public static boolean loadCppSoStatus = false;

    private static final String cppLibResourcePath = "/cpp/lib/";
    private static final String cppLibZipName = "cpplib.zip";
    private static final String unzipCppDir = "/cpp/";

    /**
     * 暂时就支持liunx
     * 如果加载异常，则使用 loadLibrary ，链接库要放到对应的
     */
    public static void loadLibrary() throws Exception {
        Properties props = new Properties();
        String cppModelDirPath = null;
        try {
            InputStream stream = ComplianceLibrary.class.getResourceAsStream("/compliance.properties");
            BufferedReader br = new BufferedReader(new InputStreamReader(stream, "UTF-8"));
            props.load(br);
        } catch (IOException e) {
            log.error("load config properties fail: {}", e);
        }
        try {
            //创建临时目录
            Path tempDirectory = createTempDirectory();

            // cpp model tmp dir
            cppModelDirPath = copyCppModelFile2Temp(tempDirectory);

            // cpp lib
            Path cppLibDestination = tempDirectory.resolve("./" + unzipCppDir).normalize();
            Files.createDirectories(cppLibDestination);

            InputStream cppLibBinary = ComplianceLibrary.class.getResourceAsStream(cppLibResourcePath + cppLibZipName);
            Path cppLibPath = cppLibDestination.resolve("./" + cppLibZipName);
            // 拷贝cpplib 到临时目录
            Files.copy(cppLibBinary, cppLibPath, new CopyOption[0]);
            cppLibBinary.close();
            File cppLib = new File(cppLibPath.toUri().getPath());
            // 解压临时目录中的cpplib.zip
            String cppLibDestinationPath = cppLibDestination.toUri().getPath();
            unZip(cppLib, cppLibDestinationPath);
            // load tmp cpp lib
            String cppLibTmpZipPath = cppLibPath.toUri().getPath();
            String cppLibUnzipDir = cppLibTmpZipPath.substring(0, cppLibTmpZipPath.lastIndexOf("."));
            loadCppLib(cppLibUnzipDir);
        } catch (Exception e) {
            log.error("load resource lib fail: {}",e);
            //最好是通过环境变量
            // export LD_LIBRARY_PATH=/home/compliance-tools-cpp/build:/usr/loca/cuda-9.2/lib64:$LD_LIBRARY_PATH
            System.setProperty("java.library.path", props.getProperty("java.library.path"));
            System.loadLibrary("ComplianceImageProcess");
            throw new Exception("load resource lib fail", e);
        }

        try {
            String cpuNum = props.getProperty("complianceImageProcess.threadNum");
            String handleImageNum = props.getProperty("complianceImageProcess.handleImageNum");
            String qualityStr = props.getProperty("complianceImageProcess.quality");
            String cppLogDir = props.getProperty("complianceImageProcess.cppLogDirPath");

            String coreThreadNum = props.getProperty("complianceImageProcess.coreThreadNum");
            String maxThreadNum = props.getProperty("complianceImageProcess.maxThreadNum");
            String blockQueueNum = props.getProperty("complianceImageProcess.blockQueueCapacity");
            int coreThread = coreThreadNum == null ? Runtime.getRuntime().availableProcessors() * 2 : Integer.valueOf(coreThreadNum);
            int maxThread = maxThreadNum == null ? Runtime.getRuntime().availableProcessors() * 2 * 2 : Integer.valueOf(maxThreadNum);
            int blockQueueCapacity = blockQueueNum == null ? 2048 : Integer.valueOf(blockQueueNum);

            int num = cpuNum == null ? Runtime.getRuntime().availableProcessors() * 2 : Integer.valueOf(cpuNum);

            //init threadPool
            ThreadFactory nameThreadFactory = new ThreadFactoryBuilder().setNameFormat("compliance_client_lib-pool-%d").build();
            executorService = new ThreadPoolExecutor(coreThread,
                    maxThread,
                    0L,
                    TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue<Runnable>(blockQueueCapacity),
                    nameThreadFactory,
                    new ThreadPoolExecutor.AbortPolicy());

            imageNum = handleImageNum == null ? 1 : Integer.valueOf(handleImageNum);
            int quality = qualityStr == null ? 60 : Integer.valueOf(qualityStr);
            String cppLogDirPath = cppLogDir == null ? "" : cppLogDir;
            //init model
            CppImageProcessImpl.init(num + 2, imageNum, cppModelDirPath, cppLogDirPath, quality);
            // 动态加载成功
            loadCppSoStatus = true;
            log.info("load cpp library success!");
        } catch (Exception e) {
            log.error("init c++ model fail: {}", e);
            throw new Exception("init c++ model fail",e);
        }
    }

    private static Path createTempDirectory() throws IOException {
        //创建临时目录，不指定根，具体位置取决于操作系统  可以通过 System.getProperty("java.io.tmpdir") 查看
        Path tempDirectory = Files.createTempDirectory("compliance_lib");
        //使用"钩子"的方式删除临时目录（在程序退出后）,一般可以不手动清理，有的系统在重启后会自动清理临时目录
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            try {
                Files.delete(tempDirectory);
            } catch (IOException e) {

            }
        }));
        return tempDirectory;
    }

    /**
     * Copy Cpp model file to TempDir
     * @param tempDirectory
     * @return cppModelDirPath   cpp model tempdir path
     * @throws IOException
     */
    private static String copyCppModelFile2Temp(Path tempDirectory) throws Exception {
        String cppModelDirPath  = null;

        String cppModelResourcePath = "/cpp/model/";
        List<String> fileLocation = new ArrayList<>();
        fileLocation.add(cppModelResourcePath + "face.bin");
        fileLocation.add(cppModelResourcePath + "face.mapping");
        fileLocation.add(cppModelResourcePath + "face.xml");
        fileLocation.add(cppModelResourcePath + "licenseplate.bin");
        fileLocation.add(cppModelResourcePath + "licenseplate.mapping");
        fileLocation.add(cppModelResourcePath + "licenseplate.xml");

        // cpp model tmp dir
        Path destinationModelDir = tempDirectory.resolve("./" + cppModelResourcePath).normalize();
        Files.createDirectories(destinationModelDir);
        cppModelDirPath = destinationModelDir.toUri().getPath();

        for (String location : fileLocation) {
            InputStream modelBinary = ComplianceLibrary.class.getResourceAsStream(location);
            if(modelBinary == null) {
                throw new Exception("cpp model file " + location + " is not exists!");
            }
            Path destinationModel = tempDirectory.resolve("./" + location).normalize();
            Files.copy(modelBinary, destinationModel, new CopyOption[0]);
            modelBinary.close();
        }

        return cppModelDirPath;
    }

    /**
     * 解压 cppLib.zip
     * @param srcFile        zip源文件
     * @param destDirPath     解压后的目标文件夹
     * @throws RuntimeException 解压失败会抛出运行时异常
     */
    private static void unZip(File srcFile, String destDirPath) throws Exception {
        if (!srcFile.exists()) {
            throw new Exception(srcFile.getPath() + "Zip file is not exists!");
        }

        ZipFile zipFile = null;
        try {
            zipFile = new ZipFile(srcFile);
            Enumeration<?> entries = zipFile.entries();
            while (entries.hasMoreElements()) {
                ZipEntry entry = (ZipEntry) entries.nextElement();
                if (entry.isDirectory()) {
                    String dirPath = destDirPath + "/" + entry.getName();
                    File dir = new File(dirPath);
                    dir.mkdirs();
                } else {
                    File targetFile = new File(destDirPath + "/" + entry.getName());
                    if(!targetFile.getParentFile().exists()){
                        targetFile.getParentFile().mkdirs();
                    }
                    targetFile.createNewFile();
                    InputStream is = zipFile.getInputStream(entry);
                    FileOutputStream fos = new FileOutputStream(targetFile);
                    int len;
                    byte[] buf = new byte[2048];
                    while ((len = is.read(buf)) != -1) {
                        fos.write(buf, 0, len);
                    }
                    fos.close();
                    is.close();
                }
            }
        } catch (Exception e) {
            log.error("unzip error!", e);
            throw new RuntimeException("unzip error!", e);
        } finally {
            if(zipFile != null){
                try {
                    zipFile.close();
                } catch (IOException e) {
                    log.error("unzip cpplib error!", e);
                    throw new Exception("unzip cpplib error!",e);
                }
            }
        }
    }


    private static void loadCppLib(String destDirPath) throws Exception {

        if(StringUtils.isBlank(destDirPath)) {
            log.error("cpp lib is not exists!");
            throw new Exception("cpp lib is not exists!");
        }
        File cppLibDir = new File(destDirPath);
        if(!cppLibDir.exists() || !cppLibDir.isDirectory()) {
            log.info("cpp lib is not exists!");
            throw new Exception("cpp lib is not exists!");
        }
        File[] cppLibs = cppLibDir.listFiles();
        if(cppLibs == null || cppLibs.length < 1) {
            log.error("cpp lib is not exists!");
            throw new Exception("cpp lib is not exists!");
        }
        for (File cppLib : cppLibs) {
            String fileName = cppLib.getName();
            // 动态加载so
            if(fileName.contains(".so")) {
                log.info("load cpp so :" + cppLib.getAbsolutePath());
                //加载动态链接库
                System.load(cppLib.getAbsolutePath());
            }
        }
    }
}
```

