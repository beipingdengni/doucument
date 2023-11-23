



## 自定义反序列化类

```java
实现ObjectDeserializer接口，然后调用
ParserConfig.getGlobalInstance().putDeserializer(Type, ObjectDeserializer)

// 实现类
ParserConfig.getGlobalInstance().putDeserializer(clazz, 
    new JavaBeanDeserializer(ParserConfig.getGlobalInstance(), clazz) {

    @Override
    public <T> T deserialze(DefaultJSONParser parser, Type type, Object fieldName){
        //解析为JSONObject，方便业务使用并反序列化为对应的子类
        JSONObject jsonObj = this.deserialze(parser, JSONObject.class, fieldName, 0);

        //业务根据JSONObject创建对应子类对象逻辑
        T obj = ...

        return obj;
    }

    @Override
    public Object createInstance(Map<String, Object> map, ParserConfig config) //
        throws IllegalArgumentException {
        JSONObject jsonObj = null;
        if (map instanceof JSONObject){
            jsonObj = (JSONObject)map;
        }else if(map != null){
          jsonObj = new JSONObject(map);
        }

        //业务根据JSONObject创建对应子类对象逻辑
        T obj = ...

        return obj;
    }

});
```

