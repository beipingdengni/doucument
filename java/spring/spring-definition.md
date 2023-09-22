



# Java PropertyDescriptor使用

```
PropertyDescriptor propertyDescriptor =new PropertyDescriptor("name",Person.class);
Method writeMethod = propertyDescriptor.getWriteMethod();
Method readMethod = propertyDescriptor.getReadMethod();

writeMethod.invoke(obj,"参数名称");
```



BeanWrapper

​	org.springframework.beans.CachedIntrospectionResults#buildGenericTypeAwarePropertyDescriptor

​		GenericTypeAwarePropertyDescriptor

