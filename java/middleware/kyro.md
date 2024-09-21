

## kryo

### maven

```xml
<dependency>
    <groupId>com.esotericsoftware</groupId>
    <artifactId>kryo</artifactId>
    <version>4.0.2</version>
</dependency>
```

需要注意的是，由于kryo使用了较高版本的asm，可能会与业务现有依赖的asm产生冲突，这是一个比较常见的问题。只需将依赖改成：

```xml
<dependency>
    <groupId>com.esotericsoftware</groupId>
    <artifactId>kryo-shaded</artifactId>
    <version>4.0.2</version>
</dependency>
```

### 使用模版类

```java
public class KryoSerializer {
    private static final ThreadLocal<Kryo> kryoLocal = ThreadLocal.withInitial(() -> {
		Kryo kryo = new Kryo();
		kryo.setReferences(true);//检测循环依赖，默认值为true,避免版本变化显式设置
		kryo.setRegistrationRequired(false);//默认值为true，避免版本变化显式设置
		((DefaultInstantiatorStrategy) kryo.getInstantiatorStrategy())
			.setFallbackInstantiatorStrategy(new StdInstantiatorStrategy());//设定默认的实例化器
		return kryo;
	});

	public byte[] serialize(Object obj) {
		Kryo kryo = getKryo();
		ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
		Output output = new Output(byteArrayOutputStream);
		kryo.writeClassAndObject(output, obj);
		output.close();
		return byteArrayOutputStream.toByteArray();
	}

	public <T> T deserialize(byte[] bytes) {
		Kryo kryo = getKryo();
		Input input = new Input(new ByteArrayInputStream(bytes));
		return (T) kryo.readClassAndObject(input);
	}

	private Kryo getKryo() {
		return kryoLocal.get();
	}
}
```

注意：

> 但是由于现实原因，同样的代码，同样的Class在不同的机器上注册编号任然不能保证一致，所以多机器部署时候反序列化可能会出现问题。所以kryo默认会开启类注册(version:5.0.2)，可以通过kryo.setRegistrationRequired(false)关闭, 关闭后Kryo会根据类型去loadClass关联
>
> kryo.setRegistrationRequired(false);*//一般设置为false* 