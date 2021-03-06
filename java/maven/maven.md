# 执行普通jar

* 依赖打入普通jar包中

* `maven`直接` package`，然后`java -jar xx-jar-with-dependencies.jar parameter1 parameter2`

```xml
<resources>
  <resource>
    <directory>src/main/resources</directory>
  </resource>
  <resource>
    <directory>src/main/java</directory>
    <includes>
      <include>**/*.properties</include>
      <include>**/*.xml</include>
      <include>**/*.tld</include>
    </includes>
    <filtering>false</filtering>
  </resource>
</resources>

<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
      <source>1.8</source>
      <target>1.8</target>
      <encoding>UTF-8</encoding>
    </configuration>
  </plugin>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
      <!--只保留一个带依赖的jar包-->
      <appendAssemblyId>false</appendAssemblyId>
      <archive>
        <manifest>
          <!--main方法-->
          <mainClass>com.mylicense.MyLicenseTools</mainClass>
        </manifest>
      </archive>
      <descriptorRefs>
        <descriptorRef>jar-with-dependencies</descriptorRef>
      </descriptorRefs>
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
</plugins>
```

# install插件

* 本地jar包install到仓库

```xml
<dependency>
  <groupId>com.tencent.autocloud</groupId>
  <artifactId>clientlib</artifactId>
  <version>${version}</version>
</dependency>

<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-install-plugin</artifactId>
  <version>2.5.2</version>
  <executions>
    <execution>
      <id>install-external</id>
      <phase>clean</phase>
      <goals>
        <goal>install-file</goal>
      </goals>
      <configuration>
       <file>/Users/dingyuanjie/work/engine/code/compliance_api/clientlib/target/compliance-tools-clientlib-sdk-${opencal.version}-jar-with-dependencies.jar</file>
        <repositoryLayout>default</repositoryLayout>
        <groupId>com.tencent.autocloud</groupId>
        <artifactId>clientlib</artifactId>
        <version>${version}</version>
        <packaging>jar</packaging>
        <generatePom>true</generatePom>
      </configuration>
    </execution>
  </executions>
</plugin>
```

# 添加本地依赖到SpringBoot项目

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

