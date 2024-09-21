## avro

官方网站：

https://avro.apache.org/

### pom依赖

```xml
<dependency>
  <groupId>org.apache.avro</groupId>
  <artifactId>avro</artifactId>
  <version>1.11.1</version>
</dependency>
```

### 定义schema

```json
{
 "namespace": "example.avro",
 "type": "record",
 "name": "User",
 "fields": [
     {"name": "name", "type": "string"},
     {"name": "favorite_number",  "type": ["int", "null"]},
     {"name": "favorite_color", "type": ["string", "null"]}
 ]
}
```

### 构建

#### 手动构建

```sh
# 参考使用
# java -jar /path/to/avro-tools-1.11.1.jar compile schema <schema file> <destination>
java -jar /path/to/avro-tools-1.11.1.jar compile schema user.avsc .
```

#### maven配置，编译生成avro文件

```xml
<plugin>
  <groupId>org.apache.avro</groupId>
  <artifactId>avro-maven-plugin</artifactId>
  <version>1.11.1</version>
  <executions>
    <execution>
      <phase>generate-sources</phase>
      <goals>
        <goal>schema</goal>
      </goals>
      <configuration>
        <sourceDirectory>${project.basedir}/src/main/avro/</sourceDirectory>
        <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
      </configuration>
    </execution>
  </executions>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <source>1.8</source>
    <target>1.8</target>
  </configuration>
</plugin>
```

### 基础的构建实例

```java
// 新建user三种方式
// 方式1
User user1 = new User();
user1.setName("Alyssa");
user1.setFavoriteNumber(256);
// 方式2
User user2 = new User("Ben", 7, "red");
// 方式3
User user3 = User.newBuilder().setName("Charlie").setFavoriteColor("blue").setFavoriteNumber(null).build();
```



### 文件

构建user实例

```java
Schema schema = new Schema.Parser().parse(new File("user.avsc"));
// 实例1
GenericRecord user1 = new GenericData.Record(schema);
user1.put("name", "Alyssa");
user1.put("favorite_number", 256);
// Leave favorite color null
// 实例2
GenericRecord user2 = new GenericData.Record(schema);
user2.put("name", "Ben");
user2.put("favorite_number", 7);
user2.put("favorite_color", "red");

```

序列化

```java
File file = new File("users2.avro");
DatumWriter<GenericRecord> datumWriter = new GenericDatumWriter<GenericRecord>(schema);
// 写入文件中
DataFileWriter<GenericRecord> dataFileWriter = new DataFileWriter<GenericRecord>(datumWriter);
dataFileWriter.create(schema, file);
// 添加元素
dataFileWriter.append(user1); // 添加：实例1
dataFileWriter.append(user2); // 添加：实例2
dataFileWriter.close();
```

反序列化

```java
File file = new File("users2.avro");
DatumReader<GenericRecord> datumReader = new GenericDatumReader<GenericRecord>(schema);
// 文件中读取
DataFileReader<GenericRecord> dataFileReader = new DataFileReader<GenericRecord>(file, datumReader);
GenericRecord user = null;
while (dataFileReader.hasNext()) {
	user = dataFileReader.next(user);
	System.out.println(user);
}
```

