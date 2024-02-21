

# jackson-xml

```xml
<dependency>
  <groupId>com.fasterxml.jackson.dataformat</groupId>
  <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

## XmlUtil

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.dataformat.xml.XmlMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import lombok.extern.slf4j.Slf4j;

import java.text.SimpleDateFormat;
import java.util.TimeZone;

@Slf4j
public class XmlUtil {
  
    private static final XmlMapper XM = new XmlMapper();

    static {
        // 对象的所有字段全部列入，还是其他的选项，可以忽略null等
        XM.setSerializationInclusion(JsonInclude.Include.ALWAYS);
        // 设置Date类型的序列化及反序列化格式
        XM.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        // 忽略空Bean转json的错误
        XM.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        // 忽略未知属性，防止json字符串中存在，java对象中不存在对应属性的情况出现错误
        XM.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // 注册一个时间序列化及反序列化的处理模块，用于解决jdk8中localDateTime等的序列化问题
        XM.registerModule(new JavaTimeModule());
        //设置时区
        XM.setTimeZone(TimeZone.getTimeZone("GMT+8"));
    }
    public static <T> T xmlToJavaBean(String xmlStr, Class<T> clz) {
        try {
            return XM.readValue(xmlStr, clz);
        } catch (Exception e) {
            log.error("XML序列化失败", e);
            return null;
        }
    }

    public static <T> T xmlToJavaBean(String xmlStr, TypeReference<T> valueTypeRef) {
        try {
            return XM.readValue(xmlStr, valueTypeRef);
        } catch (Exception e) {
            log.error("XML序列化失败", e);
            return null;
        }
    }

    public static String javaBeanToXml(Object o) {
        try {
            return XM.writeValueAsString(o);
        } catch (Exception e) {
            log.error("XML序列化失败", e);
            return null;
        }
    }
}
```

## 注解类

| 注解                      | 作用域                    | 说明                                   | 实现时机           |
| ------------------------- | ------------------------- | -------------------------------------- | ------------------ |
| @JacksonXmlRootElement    | 类上                      | 指定 XML 根标签的名字                  | 序列化时           |
| @JacksonXmlProperty       | 属性或getter/setter方法上 | 指定属性对应 XML 映射的名称            | 序列化和反序列化时 |
| @JacksonXmlText           | 属性或getter/setter方法上 | 将属性直接作为未被标签包裹的普通文本   | 序列化和反序列化时 |
| @JacksonXmlCData          | 属性或getter/setter方法上 | 将属性值包裹在 CDATA 标签中            | 序列化时           |
| @JacksonXmlElementWrapper | 属性或getter/setter方法上 | 类中有集合的属性时，是否生成包裹的标签 | 序列化时           |

```java
import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlProperty;
import com.fasterxml.jackson.dataformat.xml.annotation.JacksonXmlRootElement;

@JacksonXmlRootElement(localName = "root")
public class RequesttParam {
    @JacksonXmlProperty(localName = "EXECUTE_ORDER")
    private String excuTeOrder;
    @JacksonXmlProperty(localName = "TT_ID")
    private String ttId;
    @JacksonXmlProperty(localName = "SrcFlag")
    private String srcFlag;
    @JacksonXmlProperty(localName = "FinishTime")
    private String finishTime;
    @JacksonXmlProperty(localName = "Longitude")
    private String longitude;
    @JacksonXmlProperty(localName = "Latitude")
    private String latitude;
    @JacksonXmlProperty(localName = "OperatorName")
    private String operatorName;
}

// 自定义表情的空间
@Data
@ToString
@JacksonXmlRootElement(namespace = "http://www.w3.org/TR/html4/school/", localName = "school:grade")
public class Grade {
    @JacksonXmlElementWrapper(useWrapping = false)
    @JacksonXmlProperty(localName = "student", namespace = "http://www.w3.org/TR/html4/school/")
    List<Student> students;
}

// 对于节点上有元素点的，比如Device上有元素ID，这种情况只需要在之前注解上加个属性即可
// @JacksonXmlProperty(localName = "ID",isAttribute = true)
// private String deviceId;
// 
// 而对于节点下有多个节点的情况，JackSon一样提供了解决办法，如下
// @JacksonXmlElementWrapper(useWrapping=false)
// @JacksonXmlProperty(localName = "SPID")
// private List<String> spid;
```

