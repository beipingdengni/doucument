```xml
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
            </includes>
        </resource>

        <resource>
            <directory>src/main/lib</directory>
            <targetPath>BOOT-INF/lib/</targetPath>
            <includes>
                <include>**/*.jar</include>
            </includes>
        </resource>
    </resources>
    <extensions>
        <extension>
          	<!-- 检测系统，目前使用grpc 比较多 -->
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.5.0.Final</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>com.github.eirslett</groupId>
            <artifactId>frontend-maven-plugin</artifactId>
            <version>1.9.1</version>
            <executions>
                <!-- 检查是否安装node npm -->
                <execution>
                    <id>install node and npm</id>
                    <goals>
                        <goal>install-node-and-npm</goal>
                    </goals>
                    <phase>generate-resources</phase>
                </execution>
                <!-- npm install -->
                <execution>
                    <id>npm install</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <phase>generate-resources</phase>
                    <configuration>
                        <arguments>install</arguments>
                    </configuration>
                </execution>
                <!-- build -->
                <execution>
                    <id>npm run build</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <phase>generate-resources</phase>
                    <configuration>
                        <arguments>run build</arguments>
                    </configuration>
                </execution>
            </executions>
            <configuration>
                <nodeVersion>v16.15.1</nodeVersion>
                <npmVersion>8.11.0</npmVersion>
                <!-- node安装路径 -->
                <installDirectory>dist</installDirectory>
                <!-- 前端代码路径 -->
                <workingDirectory>resources-front-end</workingDirectory>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>
        <plugin>
            <artifactId>maven-resources-plugin</artifactId>
            <executions>
                <execution> <!-- 复制配置文件 -->
                    <id>copy-resources</id>
                    <phase>compile</phase>
                    <goals>
                        <goal>copy-resources</goal>
                    </goals>
                    <configuration>
                        <resources>
                            <resource>
                                <directory>resources-front-end/dist</directory>
                                <includes>
                                    <include>static/**</include>
                                    <include>index.html</include>
                                    <include>FATE_logo.png</include>
                                </includes>
                            </resource>
                        </resources>
                        <outputDirectory>${project.build.directory}/classes/static</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <version>3.1.1</version>
            <configuration combine.self="override">
                <!--                    <excludes>-->
                <!--                        <exclude>/*.*</exclude>-->
                <!--                    </excludes>-->
                <includes>
                    <include>**/*</include>
                </includes>
            </configuration>
        </plugin>

        <!--            <plugin>-->
        <!--                <groupId>org.apache.maven.plugins</groupId>-->
        <!--                <artifactId>maven-assembly-plugin</artifactId>-->
        <!--                <version>2.5.5</version>-->
        <!--                <configuration>-->
        <!--                    <appendAssemblyId>false</appendAssemblyId>-->
        <!--                    <descriptor>src/assembly/descriptor.xml</descriptor>-->
        <!--                </configuration>-->
        <!--                <executions>-->
        <!--                    <execution>-->
        <!--                        <goals>-->
        <!--                            <goal>single</goal>-->
        <!--                        </goals>-->
        <!--                        <phase>package</phase>-->
        <!--                    </execution>-->
        <!--                </executions>-->
        <!--            </plugin>-->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
                <mainClass>com.webank.ai.fate.board.bootstrap.Bootstrap</mainClass>
            </configuration>

            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

