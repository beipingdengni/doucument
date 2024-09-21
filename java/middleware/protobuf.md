## 本地按照执行编译

下载 protoc.exe 工具 ，下载地址：https://github.com/protocolbuffers/protobuf/releases

```shell
# 安装
brew install proto
# 执行
protoc -I=./ --java_out=./ ./SearchInfo.proto
# -I 用于指定 .proto 文件所在的路径（也可以用-proto_path代替）
# --java_out 用于指定生成的 java 文件的路径
#./AQin.proto 就是需要编译的.proto文件
```

## 定义proto文件

放的位置默认为：src/main/proto 目录下

> src/main/proto/SearchInfo.proto

```protobuf
syntax = "proto3";//声明proto语法版本

// protoc -I=./ --java_out=./ ./AQin.proto
//     -I 用于指定 .proto 文件所在的路径（也可以用-proto_path代替）
//     --java_out 用于指定生成的 java 文件的路径
//     ./AQin.proto 就是需要编译的.proto文件

package com.tbp.pb;//定义该类生成的包位置
option java_multiple_files = true;//是否再使用protobuf编译工具的时候将该proto文件编译成一个类;


message SearchInfo{
  int32 num = 1;//数字
  string uuid = 2;//字符串
  Sex sex = 3;//枚举
  PosInfo baseInfo = 4;//对象
  repeated Hobby hobbies = 5;//集合
}

message PosInfo{
  string address = 1;
  string desc = 2;
}
message Hobby{
  string name = 1;
  string desc = 2;
}
enum Sex{
  BOY = 0;
  GIRL = 1;
}
```

## 定义maven 构建

maven组件： https://www.xolstice.org/protobuf-maven-plugin/index.html

```xml
<dependencies>
  <!--    gRPC    -->
  <!--        <dependency>-->
  <!--            <groupId>io.grpc</groupId>-->
  <!--            <artifactId>grpc-netty-shaded</artifactId>-->
  <!--            <version>1.14.0</version>-->
  <!--        </dependency>-->
  <!--        <dependency>-->
  <!--            <groupId>io.grpc</groupId>-->
  <!--            <artifactId>grpc-protobuf</artifactId>-->
  <!--            <version>1.14.0</version>-->
  <!--        </dependency>-->
  <!--        <dependency>-->
  <!--            <groupId>io.grpc</groupId>-->
  <!--            <artifactId>grpc-stub</artifactId>-->
  <!--            <version>1.14.0</version>-->
  <!--        </dependency>-->

  <!--    protobuf    -->
  <dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.24.0</version>
  </dependency>
  <dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java-util</artifactId>
    <version>3.24.0</version>
  </dependency>
</dependencies>

<build>
  <extensions>
    <extension>
      <groupId>kr.motd.maven</groupId>
      <!--
         os-maven-plugin 是设置各种有用属性（从 OS 中检测的 ${os.name} 和 ${os.arch} 属性）
         os.detected.name
         os.detected.arch
       -->
      <artifactId>os-maven-plugin</artifactId>
      <version>1.5.0.Final</version>
    </extension>
  </extensions>
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
      <groupId>org.xolstice.maven.plugins</groupId>
      <artifactId>protobuf-maven-plugin</artifactId>
      <version>0.6.1</version>
      <configuration>
        <!--   <protocArtifact>com.google.protobuf:protoc:3.5.1-1:exe:${os.detected.classifier}</protocArtifact> -->
        <!-- 默认 mac 的 arm 芯片 osx-x86_64 -->
        <protocArtifact>com.google.protobuf:protoc:3.5.1-1:exe:osx-x86_64</protocArtifact>
        <!--注意：此处是你电脑安装的protoc.exe的目录 -->
        <!--  <protocExecutable>/opt/homebrew/bin/protoc</protocExecutable> -->
        <!-- gRPC 处理 -->
        <!-- <pluginId>grpc-java</pluginId>-->
        <!-- <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.14.0:exe:${os.detected.classifier}</pluginArtifact>-->
        <!-- <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.14.0:exe:osx-x86_64</pluginArtifact> -->
        <!-- 原始proto文件的路径 -->
        <protoSourceRoot>${project.basedir}/src/main/proto</protoSourceRoot> 
        <!-- 编译后生成.java文件的路径录 -->
        <outputDirectory>${project.basedir}/src/main/java</outputDirectory>
        <!--设置是否在生成java文件之前清空outputDirectory的文件，默认值为true，设置为false时也会覆盖同名文件-->
        <clearOutputDirectory>false</clearOutputDirectory>
        <!--
           更多配置信息可以查看https://www.xolstice.org/protobuf-maven-plugin/compile-mojo.html
					 protocArtifact： protoc编译器工具的格式规范
           protocExecutable：配置protoc的编译执行程序的路径
           protocPluginDirectory: protoc的执行文件目录
           excludes：排除不需要编译的文件列表
           includes：包含需要编译的proto文件列表， 默认： <include>**/*.proto</include>
        -->
      </configuration>
      <executions>
        <execution>
          <!--在执行mvn compile的时候会执行以下操作-->
          <phase>compile</phase>
          <goals>
            <!--生成OuterClass类-->
            <goal>compile</goal>
            <!--生成Grpc类-->
            <!--  <goal>compile-custom</goal> -->
          </goals>
        </execution>
      </executions>
    </plugin>

  </plugins>
</build>
```

## 代码演示序列化及JSON

```java
//客户端和服务端使用相同的pb进行编译
//1、客户端构造数据
SearchInfo build = SearchInfo.newBuilder()
  .setNum(117).setUuid("sadasdadsacf-xxxq").setSex(Sex.BOY)
  .setBaseInfo(PosInfo.newBuilder().setAddress("yanan").setDesc("luochuan").build())
  .addAllHobbies(Lists.newArrayList(
    Hobby.newBuilder().setName("basketball").build(),
    Hobby.newBuilder().setName("chicken").build()
  )).build();
// 生成字节
byte[] bytes = build.toByteArray();

//2、客户端数据发送数据，发送数据的形式不限于是tcp，udp，mq等传输字节组的形式
//xxxxx


//3、服务端解析数据
// 解析字节成对象
SearchInfo searchInfo = SearchInfo.parseFrom(bytes);
System.out.println(searchInfo);

// json 序列化
JsonFormat.Printer printer = JsonFormat.printer();
String print = printer.print(searchInfo);
System.out.println(print);

// json 反序列化
String json="{\n" +
  "  \"num\": 117,\n" +
  "  \"uuid\": \"tianbeiping yan zheng\",\n" +
  "  \"baseInfo\": {\n" +
  "    \"address\": \"xiugai\",\n" +
  "    \"desc\": \"kankan\"\n" +
  "  },\n" +
  "  \"hobbies\": [{\n" +
  "    \"name\": \"basketball===\"\n" +
  "  }, {\n" +
  "    \"name\": \"chicken===\"\n" +
  "  }]\n" +
  "}";
JsonFormat.Parser parser = JsonFormat.parser();
SearchInfo.Builder builder = SearchInfo.newBuilder();
parser.merge(json,builder);
SearchInfo build1 = builder.build();
System.out.println(build1);
```

