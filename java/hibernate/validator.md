

## maven

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.2.0.Final</version>
</dependency>
<dependency>
    <groupId>org.glassfish</groupId>
    <artifactId>jakarta.el</artifactId>
    <version>3.0.3</version>
</dependency>

<!-- 低版本可以直接使用 -->
<dependency>
  <groupId>org.hibernate</groupId>
  <artifactId>hibernate-validator</artifactId>
  <version>4.3.2.Final</version>
</dependency>
```

## 代码

### 代码案例

```java
import javax.validation.constraints.Size;
import javax.validation.constraints.NotNull;
import javax.validation.Valid;
public class ArrayContainer {
 
    @NotNull
    @Valid
    @Size(min = 1, max = 10)
    private int[] numbers;
 
    // getter and setter methods
    public int[] getNumbers() {
        return numbers;
    }
 
    public void setNumbers(int[] numbers) {
        this.numbers = numbers;
    }
}
```

### 自定义验证

```java
import javax.validation.Validation;
import javax.validation.Validator;
import javax.validation.ValidatorFactory;
import javax.validation.ConstraintViolation;
import java.util.Set;
 
public class Main {
    public static void main(String[] args) {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        Validator validator = factory.getValidator();
      	/**
      	Validator validator = Validation.byProvider(HibernateValidator.class)
                .configure()
                .failFast(false)
                .buildValidatorFactory()
                .getValidator();
      	*/
 
        ArrayContainer container = new ArrayContainer();
        container.setNumbers(new int[]{1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11}); // 错误，数组太大
 
        Set<ConstraintViolation<ArrayContainer>> violations = validator.validate(container);
 
        if (!violations.isEmpty()) {
           String errorMsg = violations.stream()
             .map(item -> item.getPropertyPath().toString() + item.getMessage())
             .collect(Collectors.joining(";"));
            throw new RuntimeException(errorMsg);
        } else {
            System.out.println("Array is valid.");
        }
    }
}
```

