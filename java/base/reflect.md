

获取分类中的类

```java
// 写一个生成器接口
public interface Generator<T> {
	// 返回一个T所对应的实例
	public T next();
}

// 反射获取实例
public class GeneratorImpl<T> implements Generator<T>{
	@Override
	public T next() {
    // 获取父类class的类型
		Type t=this.getClass().getGenericSuperclass();
    // 获取范性里面的参数类型
		Type actualTypeArguments=((ParameterizedType)t).getActualTypeArguments()[0];
		try {
      // 类型转换为class
			Class<?> c3= (Class<?>) actualTypeArguments;
      // 构造参数实例对象
			return (T) c3.newInstance();
		} catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		}
		return null;
	}
}

```



### JDK

```java
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;
public class PlainTest2 {    
  public static void main(String[] args) throws Exception {        
    /**         
    * Type 表示类型，java 所有的原生类型、参数化类型、变量类型、原子类型等都是该类实现。其主要实现有：         
    * java.lang.Class 类         
    * java.lang.reflect.ParameterizedType  参数化类型         
    * java.lang.reflect.WildcardType WildcardType represents a wildcard type expression, such as {@code ?}, {@code ? extends Number}, or {@code ? super Integer}         
    * java.lang.reflect.TypeVariable  List<T> 这样的类型         
    */        
    // 类        
    Type[] genericInterfaces = Wrapper.class.getGenericInterfaces();        
    // 属性        
    Field field = Wrapper.class.getDeclaredField("ids");        
    Type genericType = field.getGenericType();        
    Field field2 = Wrapper.class.getDeclaredField("names");        
    Type genericType2 = field2.getGenericType();        
    Field field3 = Wrapper.class.getDeclaredField("names2");        
    Type genericType3 = field3.getGenericType();        
    // 方法        
    Method method = PlainTest2.class.getMethod("test", 
                                               List.class, 
                                               Map.class, 
                                               String.class, 
                                               Wrapper.class, 
                                               String[].class, 
                                               int[].class);        
    // 获取方法参数        
    Type[] genericParameterTypes = method.getGenericParameterTypes();        
    // 获取返回值        
    Type genericReturnType = method.getGenericReturnType();        
    System.out.println(genericReturnType);        
    // 如果想获取实际类型，ParameterizedType 表示是一种参数化类型，
    // 也就是泛型. 返回实际类型的数组， 比如Map<String, Object> 就是两个Type        
    Type[] actualTypeArguments = ((ParameterizedType) genericReturnType).getActualTypeArguments();
    System.out.println(actualTypeArguments[0]); 
    // actualTypeArguments[0] 实际是个class 对象        
    // 获取其实际类型        
    Type rawType = ((ParameterizedType) genericReturnType).getRawType(); 
    // List.class, rawType实际是个class 对象        
    System.out.println(rawType);    
  } 
  
  // 调用方法
  public static List<String> test(List<Integer> datas,
                                  Map<String, Object> map, 
                                  String s,
                                  Wrapper wrapper, 
                                  String[] strs, 
                                  int[] ints) throws NoSuchMethodException {        
    return null;    
  }    
  private static class Wrapper<T> {        
    private String name;        
    private List<Integer> ids;        
    private List<? extends Number> names;        
    private List<T> names2;    
  }
}
```

### cglib

```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version> <!-- 请使用最新的版本号 -->
</dependency>
```

代码实现

```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.InvocationHandler;

public class CglibDynamicProxyExample {
    public static void main(String[] args) {
        // 创建目标对象
        SomeServiceImpl target = new SomeServiceImpl();
        // 创建 Enhancer 对象
        Enhancer enhancer = new Enhancer();
        // 设置父类为 SomeServiceImpl
        enhancer.setSuperclass(SomeServiceImpl.class);
        // 设置回调函数，即拦截器
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
                System.out.println("Before method: " + method.getName());
                // 调用原始方法
                Object result = proxy.invokeSuper(obj, args);
                System.out.println("After method: " + method.getName());
                return result;
            }
        });
        // 创建代理对象
        SomeServiceImpl proxyInstance = (SomeServiceImpl) enhancer.create();
        // 使用代理对象调用方法
        proxyInstance.doSomething();
    }
}
interface SomeService {
    void doSomething();
}
class SomeServiceImpl implements SomeService {
    public void doSomething() {
        System.out.println("Doing something...");
    }
}
```



### ResolvableType（spring ）

> 介绍

```java
//For example:
  private HashMap<Integer, List<String>> myMap;
 
  public void example() {
      ResolvableType t = ResolvableType.forField(getClass().getDeclaredField("myMap"));
      t.getSuperType(); // AbstractMap<Integer, List<String>>
      t.asMap(); // Map<Integer, List<String>>
      t.getGeneric(0).resolve(); // Integer
      t.getGeneric(1).resolve(); // List
      t.getGeneric(1); // List<String>
      t.resolveGeneric(1, 0); // String
  }

// 参考学习方法
forField(Field), 
forMethodParameter(Method, int), 
forMethodReturnType(Method), 
forConstructorParameter(Constructor, int), 
forClass(Class), 
forType(Type), 
forInstance(Object), 
ResolvableTypeProvider
```

