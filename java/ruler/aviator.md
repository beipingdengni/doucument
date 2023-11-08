

```
<dependency>
    <groupId>com.googlecode.aviator</groupId>
    <artifactId>aviator</artifactId>
    <version>5.3.3</version>
</dependency>
```



```groovy
String expression = "3 + 2 * 6";
Object result = AviatorEvaluator.execute(expression);
System.out.println(result);


// 1.定义变量
Map<String, Object> map = new HashMap<>();
map.put("name", "张三");
map.put("job", "程序员");

// 2.定义表达式
String exp = "'你好，我是'+ name + '，我的职业是' + job + '，很高兴认识你'";

// 3.使用Aviator执行表达式
Object result = AviatorEvaluator.execute(exp, map);

// 4.输出结果
System.out.println(result);
```

