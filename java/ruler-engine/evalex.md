

```xml
<dependency>
    <groupId>com.udojava</groupId>
    <artifactId>EvalEx</artifactId>
    <version>2.5</version>
</dependency>
```





```java
import com.udojava.evalex.Expression; 
import java.math.BigDecimal;
 
public class EvalexTest {
 
    public static void main(String[] args) {
        //业务规则：连续登录5天，奖励2个虚拟币
        String rule = "FLOOR(loginDays/5)*2";
 
        BigDecimal result = new Expression(rule)
                //用户实际连续登录的天数，这里只是示例，实际应用中，可从db中查询出具体值
                .with("loginDays", "6")
                .eval();
        System.out.println(result.toPlainString());
    }
}
```

