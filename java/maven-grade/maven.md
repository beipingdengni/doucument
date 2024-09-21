# maven管理

## 发布配置

#### pom.xml配置

```xml
<distributionManagement>
    <repository>
      	<!-- id值要和setting.xml中server值要对应上 -->
        <id>my_central</id> 
        <name>JD maven2 repository-releases</name>
        <url>部署的服务地址URL -- 地址 </url>
    </repository>
</distributionManagement>
```

#### setting.xml配置

```xml
<servers>
  <server>
    <id>my_central</id> 
    <username>admin</username>
    <password>jd1234</password>
  </server>
</servers>
```

## 指定下载jar仓库地址

```xml
<!-- 项目中配置 pom.xml ， <repositories></repositories>节点中加入对应的仓库使用地址  -->
<repository>
  <id>central</id>
  <url>https://maven.aliyun.com/repository/central</url>
  <releases>
    <enabled>true</enabled>
  </releases>
  <snapshots>
    <enabled>true</enabled>
  </snapshots>
</repository>

<!-- 全局指令 conf/settings.xml -->
<mirror>
  <id>aliyunmaven</id>
  <mirrorOf>*</mirrorOf>
  <name>阿里云公共仓库</name>
  <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

## 强制更新包

```shell
mvn clean install -U

mvn clean install -U -s ～/m2/settins.xml
```

## 查看依赖

> `-Dverbose`参数会把被忽略的jar，即相同jar包的不同版本引入也列出来

```shell
# 列出项目的所有jar包
mvn dependency:list -Dverbose
# 列出项目的包依赖树
# -Dincludes=aop #列出指定要求的jar，   excludes用法跟includes是一样的
mvn dependency:tree -Dverbose
# 查看包冲突
mvn dependency:tree -Dverbose -Dincludes=commons-collections

# 分析依赖
# 找出项目中依赖有如下情况的：
#。  声明了并且使用了的依赖
没有声明但是使用了的依赖
声明了但是没有使用的依赖
mvn dependency:analyze-only 

# 列出所有的远程repositories
mvn dependency:list-repositories
```

## 清理包依赖

```shell
# 删除本地仓库的元数据
rm -rf ~/.m2/repository/org/apache/maven/plugins
# 清除 Maven 的依赖项解析缓存
#项目全部清除 该命令可以清除本地仓库中所有的缓存，包括依赖库和插件等
mvn dependency:purge-local-repository

#命令行删除本地依赖； 如果只需要清除指定的依赖库缓存，将com.example:example-artifact替换为需要清除的依赖库的坐标
mvn dependency:purge-local-repository -DreResolve=false -DmanualInclude=groupId:artifactId
#或
mvn dependency:purge-local-repository -DgroupId=com.example -DartifactId=myapp -Dversion=1.0.0
```

## 项目配置拉取最新包

```xml
<repositories>
    <repository>
        <id>central</id>
        <name>libs-releases</name>
        <url>maven仓库中心URL-发布地址</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>snapshots</id>
        <name>libs-snapshots</name>
        <url>maven仓库中心URL-快照地址</url>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
        </snapshots>
    </repository>
</repositories>
```

# 包上传

### 手动上传jar

```sh
mvn clean install deploy:deploy-file \
-DgroupId=com.iflytek.icourt \
-DartifactId=common-service \
-Dversion=1.0.0-SNAPSHOT \
-Dpackaging=jar \
-Dfile=target/common-service-1.0.0-SNAPSHOT.jar \
-DpomFile=pom.xml \
-Durl=http://pl.maven.iflytek.com/nexus/content/repositories/court-snapshot/ \
-DrepositoryId=court-snapshot \
-Dmaven.test.skip=true -e

```

### 源码上传

> maven-source-plugin 
>
> 手打打包不上传： mvn source:jar

```xml
<build>
  <plugins>
    <!-- 要将源码放上去，需要加入这个插件 -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-source-plugin</artifactId>
      <configuration>
        <attach>true</attach>
      </configuration>
      <executions>
        <execution>
          <phase>compile</phase>
          <goals>
            <goal>jar</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```



### Maven多模块父项目Pom发布到Maven私服

> 一个父项目下Father面有3个子项目A、B、C。A和B是JAR包服务,C是接口sdk。当C提供给外部使用时对方也需要Father这个父POM(因为C指定了Father作为parent),这时候需要把Father父POM上传的私服供对方下载。
>
> 下面两种方式可以灵活控制deploy的模块，避免不需要上传的模块也上传到私服上去

#### Maven命令 -N 参数

使用下面的命令可以只对父POM进行deploy，不递归子模块。

> `mvn deploy -N`

#### maven-deploy-plugin插件skip参数

例如上面A和B都是JAR服务不需要上传私服，在A和Bmodule的pom.xml中添加如下配置，可以实现在父pom节点Deploy时不对A和B做上传操作。

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-deploy-plugin</artifactId>
  <configuration>
  	<skip>true</skip>
  </configuration>
</plugin>
```

# 基础操作

### 打包jar

> 参考博客：`idea`、`maven-jar-plugin`、`maven-dependency-plugin`、`maven-assembly-plugin`
>
> https://blog.csdn.net/m0_55868614/article/details/125329582

#### maven-compiler-plugin

```xml
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
```

#### maven-jar-plugin、maven-dependency-plugin

> resources/META-INF/MANIFEST.MF 文件内容如下：
>
> ```java
> Manifest-Version:1.0
> Class-Path:lib/fastjson-1.2.22.jar
> Main-Class:com.tbp.community.CommunityApplication
> ```

```xml
<build>
    <!-- 项目最终打包成的名字 -->
    <finalName>community</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <archive>
                    <manifest>
                        <!-- 会在 MANIFEST.MF 中生成 Class-Path 项 -->
                        <!-- 系统会根据 Class-Path 项配置的路径加载依赖 -->
                        <addClasspath>true</addClasspath>
                        <!-- 指定依赖包所在目录，相对于项目最终 Jar 包的路径 -->
                        <classpathPrefix>lib/</classpathPrefix>
                        <!-- 指定 MainClass -->
                        <mainClass>com.tbp.community.CommunityApplication</mainClass>
                    </manifest>
                  	 <manifestEntries>
                       <!-- 配合 buildnumber-maven-plugin 每次产生不同的版本号 -->
											 <!-- <Implementation-Version>
                        ${project.version}-${buildNumber}-${maven.build.timestamp}
                      </Implementation-Version> -->
			                <Class-Path>conf/</Class-Path>
										</manifestEntries>
                </archive>
            </configuration>
        </plugin>
 
        <!-- 配置依赖包 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <!-- 相当于执行 mvn 命令，将依赖打包到指定目录 -->
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <!--将依赖打包至 target 下的 lib 目录-->
                        <outputDirectory>${project.build.directory}/lib</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

#### (推荐)maven-assembly-plugin(单独介绍)

> 使用 maven-assembly-plugin 插件打出来的包只有一个 Jar 包，这个 Jar 包中包含了项目代码以及依赖的代码。也就意味着此种方式打出来的 Jar 包可以直接通过 java -jar xxx.jar 的命令来运行。而且我们可以联合maven-compiler-plugin插件来使用。

```xml
<build>
  <!-- 项目最终打包成的名字 -->
  <finalName>my-customer-server</finalName>
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
  </plugins>
</build>
```

#### maven-shade-plugin

```xml
<build>
  <!-- 项目最终打包成的名字 -->
  <finalName>test</finalName>
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
      <artifactId>maven-shade-plugin</artifactId>
      <version>3.2.4</version>
      <executions>
        <execution>
          <phase>package</phase>
          <goals>
            <goal>shade</goal>
          </goals>
          <configuration>
            <transformers>
              <!-- 指定启动类 -->
              <transformer
                           implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.tbp.HelloStart</mainClass>
              </transformer>

              <!-- 下面的配置仅针对存在同名资源文件的情况，如没有则不用配置-->
              <!-- 有些项目包可能会包含同文件名的资源文件（例如属性文件）-->
              <!-- 为避免覆盖，可以将它们的内容合并到一个文件中 -->
              <transformer
                           implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.handlers</resource>
              </transformer>
              <transformer
                           implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.schemas</resource>
              </transformer>
            </transformers>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```



## PMD、checkstyle



### 忽略检查

```shell
mvn clean package \
-Dpmd.skip=true\
-Dcheckstyle.skip=true\
-Dmaven.test.skip=true\
-U -f pom.xml 
-s ~/.m2/settings.xml
```

