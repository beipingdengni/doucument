# git-commit-id-plugin

> 将git提交打包到jar包中

## pom配置

```xml
 <groupId>pl.project13.maven</groupId>
 <artifactId>git-commit-id-plugin</artifactId>
 <version>4.9.10</version>
```



### 配置文件xml

```xml
<build>
        <!--        <resources>-->
        <!--            <resource>-->
        <!--                <directory>${basedir}/src/main/resources</directory>-->
        <!--                <filtering>true</filtering>-->
        <!--                <excludes>-->
        <!--                    <exclude>*.xml</exclude>-->
        <!--                    <exclude>*.yaml</exclude>-->
        <!--                    <exclude>mappers/**.xml</exclude>-->
        <!--                </excludes>-->
        <!--            </resource>-->
        <!--        </resources>-->
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
                <version>3.2.0</version>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <descriptors>
                        <descriptor>${basedir}/src/main/assembly/config.xml</descriptor>
                    </descriptors>
                    <outputDirectory>target</outputDirectory>
                </configuration>
            </plugin>
            <plugin>
                <groupId>pl.project13.maven</groupId>
                <artifactId>git-commit-id-plugin</artifactId>
                <version>4.9.10</version>
                <executions>
                    <execution>
                        <id>get-the-git-infos</id>
                        <goals>
                            <goal>revision</goal>
                        </goals>
                        <phase>initialize</phase>
                    </execution>
                </executions>
                <configuration>
                    <!-- 项目中 .git目录位置 -->
                    <dotGitDirectory>${project.basedir}/.git</dotGitDirectory>
                    <!-- 是否创建git.properties 文件 -->
                    <generateGitPropertiesFile>true</generateGitPropertiesFile>
                    <!-- 创建文件地址 -->
                    <generateGitPropertiesFilename>${project.basedir}/target/git.properties
                    </generateGitPropertiesFilename>
                    <!-- 定义插件中所有时间格式，默认值：yyyy-MM-dd’T’HH:mm:ssZ -->
                    <dateFormat>yyyy-MM-dd HH:mm:ss</dateFormat>
                    <!-- 生成git属性文件格式，默认值properties，可以修改为json -->
                    <!-- <format>properties</format> -->
                </configuration>
            </plugin>
        </plugins>
    </build>
```

#### 文件配置

> 具体行为为读取`${basedir}/src/main/assembly/config.xml`下的内容进行文件的相关操作
>

```xml
<assembly>
    <id>assembly</id>
    <formats>
        <format>dir</format>
        <format>zip</format>
    </formats>
    <includeBaseDirectory>false</includeBaseDirectory>

<!--    <dependencySets>-->
<!--        <dependencySet>-->
<!--            <unpack>false</unpack>-->
<!--            <scope>runtime</scope>-->
<!--            <outputDirectory>lib</outputDirectory>-->
<!--        </dependencySet>-->
<!--    </dependencySets>-->

    <fileSets>
        <fileSet>
            <directory>target/</directory>
            <outputDirectory>./</outputDirectory>
            <includes>
                <include>jimi-jsf-proxy-boot*.jar</include>
            </includes>
            <fileMode>0755</fileMode>
        </fileSet>
        <fileSet>
            <directory>${basedir}/src/main/assembly/bin</directory>
            <outputDirectory>bin</outputDirectory>
            <includes>
                <include>*.*</include>
            </includes>
            <fileMode>0755</fileMode>
        </fileSet>
        <fileSet>
            <directory>src/main/resources</directory>
            <outputDirectory>config</outputDirectory>
            <includes>
                <include>*.xml</include>
                <include>*.yaml</include>
            </includes>
            <fileMode>0644</fileMode>
        </fileSet>
        <fileSet>
            <directory>target</directory>
            <outputDirectory>./</outputDirectory>
            <includes>
                <include>git.properties</include>
            </includes>
            <fileMode>0755</fileMode>
        </fileSet>
    </fileSets>
</assembly>
```


