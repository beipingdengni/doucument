

## 自定义处理类转化

### 时间处理

```java
Gson gson = new GsonBuilder().registerTypeAdapter(Date.class, new TypeAdapter<Date>(){

        @Override
        public void write(JsonWriter out, Date value) throws IOException {
            if (value == null){
                out.nullValue();
            }else{
                out.value(value.getTime());
            }
        }

        @Override
        public Date read(JsonReader in) throws IOException {
            if (in != null){
                return new Date(in.nextLong());
            }else{
                return null;
            }
        }
    }).create();
```

