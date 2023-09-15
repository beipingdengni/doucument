## Okhttp





JSON POST 

```java
 private OkHttpClient okHttpClient = new OkHttpClient.Builder()
            .connectTimeout(3, TimeUnit.MINUTES)
            .readTimeout(30, TimeUnit.MINUTES).build();
            
 Request request = new Request.Builder()
                .url(requestURL)
                .post(RequestBody.create(
     MediaType.parse(APPLICATION_JSON_UTF8_VALUE), "jsondata"))
   .build();

okHttpClient.newCall(request).enqueue(new Callback() {
            @Override
            public void onFailure(Call call, IOException e) {
               log.info("failure");
            }

            @Override
            public void onResponse(Call call, Response response) throws IOException {
                log.info("Success");
            }
        }); 
```

