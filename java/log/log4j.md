# 普通Java项目

```xml
<!--log4j start-->
<dependency>
  <groupId>log4j</groupId>
  <artifactId>log4j</artifactId>
  <version>1.2.17</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
  <version>1.7.26</version>
</dependency>
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.26</version>
</dependency>
<!--log4j end-->

<build>
  <resources>
    <resource>
      <directory>src/main/resources</directory>
      <includes>
        <include>**/*.*</include>
      </includes>
      <!--是否替换资源中的属性-->
      <filtering>false</filtering>
    </resource>
  </resources>
</build>
```

```properties
# resource目录下 log4j.properties
###\u8BBE\u7F6E###
log4j.rootLogger = INFO,stdout,D,E

### \u8F93\u51FA\u4FE1\u606F\u5230\u63A7\u5236\u53F0 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

### \u8F93\u51FADEBUG \u7EA7\u522B\u4EE5\u4E0A\u7684\u65E5\u5FD7\u5230=\u9879\u76EE\u6839\u76EE\u5F55\u4E0Blog_debug.log ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = log_info.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = INFO
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n

### \u8F93\u51FAERROR \u7EA7\u522B\u4EE5\u4E0A\u7684\u65E5\u5FD7\u5230=\u9879\u76EE\u6839\u76EE\u5F55\u4E0Blog_error.log ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File = log_error.log
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

# SpringBoot项目

```xml
<!--log4j2-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

```yaml
# resources目录下log4j2.yml
# 共有8个级别，按照从低到高为：ALL < TRACE < DEBUG < INFO < WARN < ERROR < FATAL < OFF。
# https://blog.csdn.net/u010598111/article/details/80556437
# https://www.cnblogs.com/buguge/p/10256769.html

Configuration:
  status: warn
  monitorInterval: 30

  Properties: # 定义全局变量
    Property: # 缺省配置（用于开发环境）。其他环境需要在VM参数中指定，如下：
      #测试：-Dlog.level.console=warn -Dlog.level.xjj=trace
      #生产：-Dlog.level.console=warn -Dlog.level.xjj=info
      - name: log.level.console
        value: info
      - name: log.path
#        value: /Users/dingyuanjie/work/engine/doc/compliance_api/test/log/
        value: /data/log/
      - name: project.name
        value: opencal-calculation
      - name: log.pattern
        value: "%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %F[%L] [%p] - %m%n"      # 日期，线程名，类路径及名称，行数，级别，内容 分隔符

  Appenders:
    Console:  #输出到控制台
      name: CONSOLE
      target: SYSTEM_OUT
      PatternLayout:
        pattern: ${log.pattern}

    RollingFile:
      - name: ROLLING_FILE
        fileName: ${log.path}/${project.name}.log
        filePattern: "${log.path}/$${date:yyyy-MM}/${project.name}-%d{yyyy-MM-dd}-%i.log.gz"
        PatternLayout:
          pattern: ${log.pattern}
        Filters:
#        一定要先去除不接受的日志级别，然后获取需要接受的日志级别
          ThresholdFilter:  #该配置 只接受debug及以上级别的日志
            - level: info     #此日志级别或以上的过滤方式
              onMatch: ACCEPT   # DENY / ACCEPT
              onMismatch: NEUTRAL
        Policies:
          TimeBasedTriggeringPolicy:  # 按天分类
            modulate: true
            interval: 1
        DefaultRolloverStrategy:     # 文件最多100个
          max: 100

  Loggers:
    Root:
      level: info
      AppenderRef:
        - ref: CONSOLE
        - ref: ROLLING_FILE

#    监听具体包下面的日志
#    Logger: # 为com.xjj包配置特殊的Log级别，方便调试
#      - name: com.xjj
#        additivity: false
#        level: ${sys:log.level.xjj}
#        AppenderRef:
#          - ref: CONSOLE
#          - ref: ROLLING_FILE
```

