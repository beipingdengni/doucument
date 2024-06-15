



## java 1.8  tensorflow 需要的包

```xml
 <dependency>
   <groupId>commons-io</groupId>
   <artifactId>commons-io</artifactId>
</dependency>
<dependency>
  <groupId>commons-beanutils</groupId>
  <artifactId>commons-beanutils</artifactId>
</dependency>
<dependency>
  <groupId>com.yesup.oss</groupId>
  <artifactId>tensorflow-client</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>grpc-protobuf</artifactId>
      <groupId>io.grpc</groupId>
    </exclusion>
    <exclusion>
      <artifactId>grpc-core</artifactId>
      <groupId>io.grpc</groupId>
    </exclusion>
    <exclusion>
      <artifactId>slf4j-api</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>com.example.tensorflow</groupId>
  <artifactId>tensorflow-serve-client</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>grpc-protobuf</artifactId>
      <groupId>io.grpc</groupId>
    </exclusion>
    <exclusion>
      <artifactId>grpc-stub</artifactId>
      <groupId>io.grpc</groupId>
    </exclusion>
    <exclusion>
      <artifactId>grpc-netty-shaded</artifactId>
      <groupId>io.grpc</groupId>
    </exclusion>
  </exclusions>
</dependency>
<dependency>
  <groupId>tw.edu.ntu.csie</groupId>
  <artifactId>libsvm</artifactId>
</dependency>
<dependency>
  <groupId>org.tensorflow</groupId>
  <artifactId>tensorflow</artifactId>
</dependency>
<dependency>
  <groupId>org.tensorflow</groupId>
  <artifactId>proto</artifactId>
</dependency>
<!--  如果你的设备支持GPU功能，还可以添加以下依赖    -->
<dependency>
  <groupId>org.tensorflow</groupId>
  <artifactId>libtensorflow</artifactId>
</dependency>
<!-- GPU资源打开
        <dependency>
            <groupId>org.tensorflow</groupId>
            <artifactId>libtensorflow_jni_gpu</artifactId>
        </dependency>
        -->

<dependency>
  <groupId>com.huaban</groupId>
  <artifactId>jieba-analysis</artifactId>
</dependency>
```