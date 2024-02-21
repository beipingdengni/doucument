# maven-assembly-plugin

> 使用 maven-assembly-plugin 插件打出来的包只有一个 Jar 包，这个 Jar 包中包含了项目代码以及依赖的代码。也就意味着此种方式打出来的 Jar 包可以直接通过 java -jar xxx.jar 的命令来运行。而且我们可以联合maven-compiler-plugin插件来使用。

## pom配置

### 配置文件xml

```xml
 <!-- 配置变量定义,打包时使用相应环境变量 -->
<profiles>
  <profile>
    <id>poc</id>
    <properties>
      <buildEnv>poc</buildEnv>
    </properties>
    <dependencies>
     <dependency>
        <groupId>org.apache.rocketmq</groupId>
        <artifactId>rocketmq-client</artifactId>
        <version>5.1.4</version>
    </dependency>
    </dependencies>
  </profile>
</profiles>

<build>
  <!-- 项目最终打包成的名字 -->
  <finalName>my-customer-server</finalName>
  <filters>
			<filter>src/main/buildEnv/open-analysis-${buildEnv}.properties</filter>
	</filters>
  <plugins>
    <plugin>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
        <!-- 指定maven编译的jdk版本,如果不指定,maven3默认用jdk 1.5 maven2默认用jdk1.3 -->
        <source>1.8</source> <!-- 源代码使用的JDK版本 -->
        <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->
        <encoding>UTF-8</encoding><!-- 字符集编码 -->
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-assembly-plugin</artifactId>
      <!-- 相当于在执行 package 打包时，在后面加上 assembly:single  -->
      <executions>
        <execution>
          <id>make-assembly</id>
          <phase>package</phase> <!-- 绑定到package生命周期阶段上 -->
          <goals>
            <goal>single</goal> <!-- 绑定到package生命周期阶段上 -->
          </goals>
          <configuration>
            <filters>
              <filter>${basedir}/src/assembly/my-customer-${buildEnv}.properties</filter>
            </filters>
            <descriptors>
              <descriptor>${basedir}/src/assembly/single-dist.xml</descriptor>
            </descriptors>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

#### 文件配置

> 具体行为为读取`src/assembly/single-dist.xml`下的内容进行文件的相关操作，并且通过`src/assembly/filter.properties`的规则对部分文件内容进行替换
>
> filter.properties 文件内容：username=admin
>
> mysql.properties 的文件内容： mysql.root=${username} , 将被替换

```xml
<assembly>
    <id>single-distro</id>
    <formats>
        <format>jar</format>
        <format>zip</format>
        <format>tar.gz</format>
    </formats>
<!--  指定是否包含打包层目录（比如finalName是output，当值为true，所有文件被放在output目录下，否则直接放在包的根目录下）； -->
  	<includeBaseDirectory>false</includeBaseDirectory>
    <fileSets>
        <fileSet>
          	<!-- 将项目中src/main/resources目录下的资源文件copy到target目录的conf目录下 -->
            <directory>src/main/resources</directory>
            <outputDirectory>conf</outputDirectory>
	          <filtered>true</filtered>  <!-- 过滤资源文件，对其中的maven变量进行复制 -->
            <includes>
                <include>*.xml</include>
                <include>*.properties</include>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
            </includes>
            <fileMode>0644</fileMode>
        </fileSet>
        <fileSet>
            <!-- 将项目中src/main/bin目录下的脚本文件copy到target目录的bin目录下 -->
            <directory>src/assembly/bin</directory>
            <outputDirectory>bin</outputDirectory>
            <fileMode>0755</fileMode>
        </fileSet>
    </fileSets>
    <dependencySets>
      	 <!-- 指定依赖  -->
        <!-- <dependencySet>
            <includes>
                <include>com.gee.maven:mavenEnforcerExample</include>
            </includes>
            <outputDirectory>lib</outputDirectory>
        </dependencySet>
				-->
      	<!-- 运行中的jar全部放入到lib中  -->
        <dependencySet>
          <!-- <useProjectArtifact>true</useProjectArtifact> -->
          <outputDirectory>lib</outputDirectory>
          <scope>runtime</scope>
      </dependencySet>
    </dependencySets>
</assembly>
```



### 无配置并定义启动类启动类

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-assembly-plugin</artifactId>
  <configuration>
    <archive>
      <!-- 指定启动类 -->
      <manifest>
        <mainClass>com.tbp.HelloStart</mainClass>
      </manifest>
    </archive>
    <!-- 描述后缀 -->
    <descriptorRefs>
      <descriptorRef>jar-with-dependencies</descriptorRef>
    </descriptorRefs>
  </configuration>
  <!-- 相当于在执行 package 打包时，在后面加上 assembly:single  -->
  <executions>
    <execution>
      <id>make-assembly</id>
      <phase>package</phase>
      <goals>
        <goal>single</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
