# ben

# 基础bean的Property

## Java PropertyDescriptor使用

```
PropertyDescriptor propertyDescriptor =new PropertyDescriptor("name",Person.class);
Method writeMethod = propertyDescriptor.getWriteMethod();
Method readMethod = propertyDescriptor.getReadMethod();

writeMethod.invoke(obj,"参数名称");
```

## BeanInfo

```java
BeanInfo  bf = Introspector.getBeanInfo(Person.class)

// bean转map
public static Map<String, Object> beanToMap(Object obj) {
  Map<String, Object> map = new HashMap<>();
  BeanInfo beanInfo = Introspector.getBeanInfo(obj.getClass());
  PropertyDescriptor[] propertyDescriptors = beanInfo.getPropertyDescriptors();
  for (PropertyDescriptor property : propertyDescriptors) {
    String key = property.getName();
    // 过滤 class 属性
    if (!key.equals("class")) {
      // 得到 property 的 getter 方法
      Method getter = property.getReadMethod();
      Object value = getter.invoke(obj);
      map.put(key, value);
    }
  }
  return map;
}
```



## PropertyEditor

```
PropertyEditorSupport implements PropertyEditor

common jar包类
BeanUtilsBean
BeanUtils

```









BeanWrapper

​	org.springframework.beans.CachedIntrospectionResults#buildGenericTypeAwarePropertyDescriptor

​		GenericTypeAwarePropertyDescriptor

