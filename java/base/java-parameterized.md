

# 泛型



```java
public class MyTest extends TestClass<String> implements TestInterface1<Integer>,TestInterface2<Long> {
 
    private List<Integer> list;
 
    private Map<Integer, String> map;
 
    public List<String> aa() {
        return null;
    }
 
    public void bb(List<Long> list) {
 
    }
 
    public static void main(String[] args) throws Exception {
        System.out.println("== 泛型类声明的泛型类型 ==");
        ParameterizedType parameterizedType = (ParameterizedType)MyTest.class.getGenericSuperclass();
        System.out.println(parameterizedType.getTypeName() + "--------->" + parameterizedType.getActualTypeArguments()[0].getTypeName());
 
        Type[] types = MyTest.class.getGenericInterfaces();
        for (Type type : types) {
            ParameterizedType typ = (ParameterizedType)type;
            System.out.println(typ.getTypeName() + "--------->" + typ.getActualTypeArguments()[0].getTypeName());
        }
 
        System.out.println("== 成员变量中的泛型类型 ==");
        ParameterizedType parameterizedType1 = (ParameterizedType)MyTest.class.getDeclaredField("list").getGenericType();
        System.out.println(parameterizedType1.getTypeName() + "--------->" + parameterizedType1.getActualTypeArguments()[0].getTypeName());
 
        ParameterizedType parameterizedType2 = (ParameterizedType)MyTest.class.getDeclaredField("map").getGenericType();
        System.out.println(parameterizedType2.getTypeName() + "--------->" + parameterizedType2.getActualTypeArguments()[0].getTypeName()+","+parameterizedType2.getActualTypeArguments()[1].getTypeName());
 
        System.out.println("== 方法参数中的泛型类型 ==");
        ParameterizedType parameterizedType3 = (ParameterizedType)MyTest.class.getMethod("aa").getGenericReturnType();
        System.out.println(parameterizedType3.getTypeName() + "--------->" + parameterizedType3.getActualTypeArguments()[0].getTypeName());
 
        System.out.println("== 方法返回值中的泛型类型 ==");
        Type[] types1 = MyTest.class.getMethod("bb", List.class).getGenericParameterTypes();
        for (Type type : types1) {
            ParameterizedType typ = (ParameterizedType)type;
            System.out.println(typ.getTypeName() + "--------->" + typ.getActualTypeArguments()[0].getTypeName());
        }
    }
}
 
class TestClass<T> {
 
}
 
interface TestInterface1<T> {
 
}
 
interface TestInterface2<T> {
 
}
```

