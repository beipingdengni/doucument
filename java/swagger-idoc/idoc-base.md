## 自动生成文档

> GitHub: https://github.com/houbb/idoc/tree/master

```xml
<build>
    <plugins>
        <plugin>
            <groupId>com.github.houbb</groupId>
            <artifactId>idoc-core</artifactId>
            <version>0.3.0</version>
        </plugin>
    </plugins>
</build>

<!-- 可以使用内置的 MarkdownDocGenerator 输出到 markdown 文件 -->
<plugin>
    <groupId>com.github.houbb</groupId>
    <artifactId>idoc-core</artifactId>
    <version>0.3.0</version>
    <configuration>
        <generates>
            <generate>com.github.houbb.idoc.ftl.api.generator.MarkdownDocGenerator</generate>
        </generates>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>com.github.houbb</groupId>
            <artifactId>idoc-ftl</artifactId>
            <version>0.3.0</version>
        </dependency>
    </dependencies>
</plugin>
```



```java
/**
 * 查询用户服务类
 * @author binbin.hou
 * @since 0.0.1
 */
public interface QueryUserService {

    /**
     * 根据用户信息查询用户
     * @param user 用户信息
     * @return 结果
     * @since 0.0.2,2019/02/12
     */
    public User queryUser(final User user);
}

// @require 表示当前字段是否必填，作为方法入参时。
// @remark 表示当前字段的备注信息。

/**
 * 用户信息
 * @author binbin.hou
 * @since 0.0.1
 */
public class User {

    /**
     * 名称
     * @require 是
     * @remark 中文名称，请认真填写
     */
    private String name;

    /**
     * 年龄
     */
    private int age;
}
```

