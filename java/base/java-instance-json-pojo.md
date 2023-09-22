PropertyDescriptor



## Gson

```

关于null值的处理 Gson会忽略掉这个字段，不产生相关的信息

Array
Book[] books = gson.fromJson(booksJson, Book[].class);

Set
Type bookType=new TypeToken<Set<Book>>(){}.getType();
Set<Book> set=gson.fromJson(booksJson,bookType);

Map
Type mapType=new TypeToken<Map<String,String>>(){}.getType();
Map<String,String> map=gson.fromJson(mapJson,mapType);

enum枚举（ 序列化别名 ）
@SerializedName("1") B,


Gson实例默认不会去处理@Expose注解，需要通过GsonBuilder的方式
Gson gson=new GsonBuilder().excludeFieldsWithoutExposeAnnotation().create();
@Expose//序列化也反序列化
@Expose(deserialize = false)//只序列化
@Expose(serialize = false)//只反序列化
@Expose(serialize = false,deserialize = false)//既不序列化也不反序列化


```



# 类实例化操作（无参构造、有参构造）

```java
 private static Object newInstance(Class<?> cls) {
    try {
        return cls.getDeclaredConstructor().newInstance();
    } catch (Exception t) {
        Constructor<?>[] constructors = cls.getDeclaredConstructors();
        /*
            From Javadoc java.lang.Class#getDeclaredConstructors
            This method returns an array of Constructor objects reflecting all the constructors
            declared by the class represented by this Class object.
            This method returns an array of length 0,
            if this Class object represents an interface, a primitive type, an array class, or void.
            */
        if (constructors.length == 0) {
            throw new RuntimeException("Illegal constructor: " + cls.getName());
        }
        Throwable lastError = null;
        Arrays.sort(constructors, Comparator.comparingInt(a -> a.getParameterTypes().length));
        for (Constructor<?> constructor : constructors) {
            try {
                constructor.setAccessible(true);
                Object[] parameters = Arrays.stream(constructor.getParameterTypes()).map(PojoUtils::getDefaultValue).toArray();
                return constructor.newInstance(parameters);
            } catch (Exception e) {
                lastError = e;
            }
        }
        throw new RuntimeException(lastError.getMessage(), lastError);
    }
}

/**
 * return init value
 *
 * @param parameterType
 * @return
 */
private static Object getDefaultValue(Class<?> parameterType) {
    if ("char".equals(parameterType.getName())) {
        return Character.MIN_VALUE;
    }
    if ("boolean".equals(parameterType.getName())) {
        return false;
    }
    if ("byte".equals(parameterType.getName())) {
        return (byte) 0;
    }
    if ("short".equals(parameterType.getName())) {
        return (short) 0;
    }
    return parameterType.isPrimitive() ? 0 : null;
}
```

