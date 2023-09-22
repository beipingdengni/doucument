



# Java PropertyDescriptor使用

```
PropertyDescriptor propertyDescriptor =new PropertyDescriptor("name",Person.class);
Method writeMethod = propertyDescriptor.getWriteMethod();
Method readMethod = propertyDescriptor.getReadMethod();

writeMethod.invoke(obj,"参数名称");
```

## BeanInfo

```
BeanInfo  bf = Introspector.getBeanInfo(Person.class)

Introspector
```



# PropertyEditor

```
PropertyEditorSupport implements PropertyEditor


common jar包类
BeanUtilsBean
BeanUtils

```









BeanWrapper

​	org.springframework.beans.CachedIntrospectionResults#buildGenericTypeAwarePropertyDescriptor

​		GenericTypeAwarePropertyDescriptor

