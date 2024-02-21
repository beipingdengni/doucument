





## spring 基础使用

> 参考文章：https://cloud.tencent.com/developer/article/1676200

### 字符串处理

```java
ExpressionParser parser = new SpelExpressionParser();
Expression expression = parser.parseExpression("('Hello' + ' World').concat(#end)");
EvaluationContext context = new StandardEvaluationContext();
context.setVariable("end", "!");
System.out.println(expression.getValue(context));
```

### 定制解析模版(#{})

```java
//创建解析器
SpelExpressionParser parser = new SpelExpressionParser();
//创建解析器上下文
ParserContext context = new TemplateParserContext("%{", "}");
Expression expression = parser.parseExpression("你好:%{#name},我们正在学习:%{#lesson}", context);

//创建表达式计算上下文
EvaluationContext evaluationContext = new StandardEvaluationContext();
evaluationContext.setVariable("name", "路人甲java");
evaluationContext.setVariable("lesson", "spring高手系列!");
//获取值
String value = expression.getValue(evaluationContext, String.class);
System.out.println(value); // 你好:路人甲java,我们正在学习:spring高手系列!
```

### 类型处理、比较大小

```java
String str1 = parser.parseExpression("'Hello World!'").getValue(String.class);
int int1 = parser.parseExpression("1").getValue(Integer.class);
long long1 = parser.parseExpression("-1L").getValue(long.class);
float float1 = parser.parseExpression("1.1").getValue(Float.class);
double double1 = parser.parseExpression("1.1E+2").getValue(double.class);
int hex1 = parser.parseExpression("0xa").getValue(Integer.class);
long hex2 = parser.parseExpression("0xaL").getValue(long.class);
boolean true1 = parser.parseExpression("true").getValue(boolean.class);
boolean false1 = parser.parseExpression("false").getValue(boolean.class);
Object null1 = parser.parseExpression("null").getValue(Object.class);

// SpEL同样提供了等价的“EQ” 、“NE”、 “GT”、“GE”、 “LT” 、“LE”来表示等于、不等于、大于、大于等于、小于、小于等于，不区分大小写
boolean v1 = parser.parseExpression("1>2").getValue(boolean.class);
boolean between1 = parser.parseExpression("1 between {1,2}").getValue(boolean.class);
boolean result1 = parser.parseExpression("2>1 and (!true or !false)").getValue(boolean.class);
boolean result2 = parser.parseExpression("2>1 && (!true || !false)").getValue(boolean.class);
// 字符 NOT OR AND
boolean result3 = parser.parseExpression("2>1 and (NOT true or NOT false)").getValue(boolean.class);
boolean result4 = parser.parseExpression("2>1 && (NOT true || NOT false)").getValue(boolean.class);
```

### 类类型表达式、类实例化

```java
ExpressionParser parser = new SpelExpressionParser();
//java.lang包类访问
Class<String> result1 = parser.parseExpression("T(String)").getValue(Class.class);
System.out.println(result1);

//其他包类访问
String expression2 = "T(com.tbp.spel.MySpelTest)";
Class<SpelTest> value = parser.parseExpression(expression2).getValue(Class.class);
System.out.println(value == SpelTest.class);

//类静态字段访问
int result3 = parser.parseExpression("T(Integer).MAX_VALUE").getValue(int.class);
System.out.println(result3 == Integer.MAX_VALUE);

//类静态方法调用
int result4 = parser.parseExpression("T(Integer).parseInt('1')").getValue(int.class);
System.out.println(result4);

// 类实例化
String result1 = parser.parseExpression("new String('路人甲java')").getValue(String.class);
System.out.println(result1);
// 非 java.lang 下要写类全名
Date result2 = parser.parseExpression("new java.util.Date()").getValue(Date.class);
System.out.println(result2);
// instanceof表达式
Boolean value = parser.parseExpression("'路人甲' instanceof T(String)").getValue(Boolean.class);
System.out.println(value);
```

### 变量定义及引用

```java
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = new StandardEvaluationContext();
context.setVariable("name", "路人甲java");
context.setVariable("lesson", "Spring系列");

//获取name变量，lesson变量
String name = parser.parseExpression("#name").getValue(context, String.class);
System.out.println(name);
String lesson = parser.parseExpression("#lesson").getValue(context, String.class);
System.out.println(lesson);

//StandardEvaluationContext构造器传入root对象，可以通过#root来访问root对象
context = new StandardEvaluationContext("我是root对象");
String rootObj = parser.parseExpression("#root").getValue(context, String.class);
System.out.println(rootObj);

//#this用来访问当前上线文中的对象
String thisObj = parser.parseExpression("#this").getValue(context, String.class);
System.out.println(thisObj);
```

### 自定义函数

```java
//定义2个函数,registerFunction和setVariable都可以，不过从语义上面来看用registerFunction更恰当
StandardEvaluationContext context = new StandardEvaluationContext();
Method parseInt = Integer.class.getDeclaredMethod("parseInt", String.class);
context.registerFunction("parseInt1", parseInt);
context.setVariable("parseInt2", parseInt);

ExpressionParser parser = new SpelExpressionParser();
System.out.println(parser.parseExpression("#parseInt1('3')").getValue(context, int.class));
System.out.println(parser.parseExpression("#parseInt2('3')").getValue(context, int.class));

String expression1 = "#parseInt1('3') == #parseInt2('3')";
boolean result1 = parser.parseExpression(expression1).getValue(context, boolean.class);
System.out.println(result1);
```

### 表达式赋值、数组、字典

```java
//user为root对象
ExpressionParser parser = new SpelExpressionParser();
EvaluationContext context = new StandardEvaluationContext(user);
parser.parseExpression("#root.name").setValue(context, "路人甲java");
System.out.println(parser.parseExpression("#root").getValue(context, user.getClass()));

//使用安全访问符号?.，可以规避null错误
System.out.println(parser.parseExpression("#user?.car?.name").getValue(context, String.class));

//1.测试集合或数组
List<Integer> list = new ArrayList<Integer>();
list.add(1);
list.add(4);
list.add(5);
list.add(7);
EvaluationContext context1 = new StandardEvaluationContext();
context1.setVariable("list", list);
//
parser.parseExpression("#list[1]").setValue(context1, 4);
int result1 = parser.parseExpression("#list[1]").getValue(context1, int.class);
// 投影，类似list stream map
Collection<Integer> result1 = parser.parseExpression("#list.![#this+1]").getValue(context1, Collection.class);
// 过滤
Collection<Integer> result1 = parser.parseExpression("#list.?[#this>4]").getValue(context1, Collection.class);

//2.测试字典
Map<String, Integer> map = new HashMap<String, Integer>();
map.put("a", 1);
map.put("b", 2);
map.put("c", 3);
EvaluationContext context2 = new StandardEvaluationContext();
context2.setVariable("map", map);
// 
parser.parseExpression("#map['a']").setValue(context2, 4);
Integer result2 = parser.parseExpression("#map['a']").getValue(context2, int.class);
// Map投影最终只能得到List结果，如上所示，对于投影表达式中的“#this”将是Map.Entry，所以可以使用“value”来获取值，使用“key”来获取键
List<Integer> result2 = parser.parseExpression("#map.![value+1]").getValue(context2, List.class);

// 过滤
//  key
Map<String, Integer> result2 = parser.parseExpression("#map.?[key!='a']").getValue(context2, Map.class);
result2.forEach((key, value) -> {
    System.out.println(key + ":" + value);
});
// value
List<Integer> result3 = parser.parseExpression("#map.?[key!='a'].![value+1]").getValue(context2, List.class);
result3.forEach(System.out::println);

// 过滤 key！= 'a' ,然后给value值加 +1
List<Integer> result3 = parser.parseExpression("#map.?[key!='a'].![value+1]").getValue(context2, List.class);


```

### spring bean 应用使用

> SpEL支持使用“@”符号来引用Bean，在引用Bean时需要使用BeanResolver接口实现来查找Bean，Spring提供BeanFactoryResolver实现。

```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// 创建bean
User user = new User();
Car car = new Car();
car.setName("保时捷");
user.setId(1L);
user.setCar(car);
// 注册入容器
factory.registerSingleton("user", user);
// 构建bean
StandardEvaluationContext context = new StandardEvaluationContext();
context.setBeanResolver(new BeanFactoryResolver(factory));
// 
ExpressionParser parser = new SpelExpressionParser();
User userBean = parser.parseExpression("@user").getValue(context, User.class);
// 调用bean里面方法
User userBean = parser.parseExpression("@userService.getInfoById(#user.id)").getValue(context, User.class);
System.out.println(userBean);
System.out.println(userBean == factory.getBean("user"));
```

## spring修改引擎

```java
@Component
public class SpelBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanExpressionResolver beanExpressionResolver = beanFactory.getBeanExpressionResolver();
        if (beanExpressionResolver instanceof StandardBeanExpressionResolver) {
            StandardBeanExpressionResolver resolver = (StandardBeanExpressionResolver) beanExpressionResolver;
            resolver.setExpressionPrefix("%{"); // 默认: #{
            resolver.setExpressionSuffix("}");  // 默认: }
        }
    }
}
```

### 自定义处理，调用bean中方法

1. SpEL支持使用@符号来引用Bean。 
2. 在引用Bean时需要使用BeanResolver接口来查找Bean， Spring会提供BeanFactoryResolver的实现。

> #### `code: #{@AopController.test(#order.purchaseName)}`;

```java
@RestController("AopController")
public class AopController implements BeanFactoryAware {

    //获取到Spring容器的beanFactory对象
    private BeanFactory beanFactory;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    @GetMapping("spel")
    public String spel() {
        Order order = new Order();
        order.setPurchaseName("张三");
        order.setDeliveryOrderNo("100000001"); 
        // 自定义方法
        String spELString = "新增配置参数，code: #{@AopController.test(#order.purchaseName)}";
        //定义模板。默认是以#{开头，以#结尾
        TemplateParserContext PARSER_CONTEXT = new TemplateParserContext();
        ExpressionParser parser = new SpelExpressionParser();
        StandardEvaluationContext context = new StandardEvaluationContext();
        //填充evaluationContext对象的`BeanFactoryResolver`。
        context.setBeanResolver(new BeanFactoryResolver(beanFactory));
        context.setVariable("order",order);
        // 构建表达式
        Expression expression = parser.parseExpression(spELString,PARSER_CONTEXT);
        // 解析
        String value = expression.getValue(context, String.class);
      	// value值等于： 新增配置参数，code: 自定义方法
        return value;
    }

    public String test(String Str){
        System.out.println("自定义方法:"+Str);
        return "自定义方法";
    }
}
```

