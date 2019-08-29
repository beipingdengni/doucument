## Druid



配置加密：

```

<property name="password" value="${password}" />
<property name="connectionProperties" value="config.decrypt=true;config.decrypt.key=${publickey}" />

java -cp druid-1.0.16.jar com.alibaba.druid.filter.config.ConfigTools you_password

打印出来数据
privateKey:.............
publicKey:.........
password:..........

```

