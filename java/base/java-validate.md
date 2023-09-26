### http 中自定义验证


##### [hibernate 官方文档指导](http://hibernate.org/validator/documentation/getting-started/)
##### [官方api 详细指导](https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/#validator-gettingstarted-createmodel)



时区

```
TimeZone timeZone = TimeZone.getTimeZone("GMT+8");

SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
sdf.setTimeZone(timeZone);
String formattedTime = sdf.format(currentTime);
```



#### 导入包

``` xml
 <dependency>  
        <groupId>javax.el</groupId>  
        <artifactId>javax.el-api</artifactId>  
        <version>2.2.4</version>  
    </dependency>  
      
  <dependency>  
        <groupId>org.hibernate</groupId>  
        <artifactId>hibernate-validator</artifactId>  
        <version>5.1.3.Final</version>  
    </dependency>  
```

#### 新建bean
``` java
 public class StudentInfo {  
    @NotBlank(message="用户名不能为空")  
    private String userName;  
    @NotBlank(message="年龄不能为空")  
    @Pattern(regexp="^[0-9]{1,2}$",message="年龄是整数")  
    private String age;  
    /** 
     * 如果是空，则不校验，如果不为空，则校验 
     */  
    @Pattern(regexp="^[0-9]{4}-[0-9]{2}-[0-9]{2}$",message="出生日期格式不正确")  
    private String birthday;   
    @NotBlank(message="学校不能为空")  
    private String school;  
 }
```

#### 新建 ValidatorUtil.java
``` java
public class ValidatorUtil {  
    private static Validator validator = Validation.buildDefaultValidatorFactory()  
            .getValidator();  
      
    public static <T> Map<String,StringBuffer> validate(T obj){  
        Map<String,StringBuffer> errorMap = null;  
        Set<ConstraintViolation<T>> set = validator.validate(obj,Default.class);  
        if(set != null && set.size() >0 ){  
            errorMap = new HashMap<String,StringBuffer>();  
            String property = null;  
            for(ConstraintViolation<T> cv : set){  
                //这里循环获取错误信息，可以自定义格式  
                property = cv.getPropertyPath().toString();  
                if(errorMap.get(property) != null){  
                    errorMap.get(property).append("," + cv.getMessage());  
                }else{  
                    StringBuffer sb = new StringBuffer();  
                    sb.append(cv.getMessage());  
                    errorMap.put(property, sb);  
                }  
            }  
        }  
        return errorMap;  
    }    
}  
```