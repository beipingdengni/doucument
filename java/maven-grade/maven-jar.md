

```xml
<build>
  <plugins>
    <!-- 编译为1.8 -->
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
   <!-- jar 生产 -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <executions>
        <execution>
          <id>copy</id>
          <phase>install</phase>
          <goals>
            <goal>jar</goal>
          </goals>
        </execution>
      </executions>
      <configuration>
        <classesDirectory>target/classes/</classesDirectory>
        <archive>
          <manifest>
            <!-- 主函数的入口 -->
            <mainClass>com.jd.jimi3.jmdb.CommonLine</mainClass>
            <!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
            <useUniqueVersions>false</useUniqueVersions>
            <addClasspath>true</addClasspath>
            <classpathPrefix>lib/</classpathPrefix>
          </manifest>
          <manifestEntries>
            <Class-Path>.</Class-Path>
          </manifestEntries>
        </archive>
      </configuration>
    </plugin>
    <!-- 复制jar -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <executions>
        <execution>
          <id>copy</id>
          <phase>install</phase>
          <goals>
            <goal>copy-dependencies</goal>
          </goals>
          <configuration>
            <outputDirectory>${project.build.directory}/lib</outputDirectory>
          </configuration>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

