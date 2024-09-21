```xml
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.62</version>
</dependency>
```

### Hessian2 工具类

```java
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import com.caucho.hessian.io.Hessian2Input;
import com.caucho.hessian.io.Hessian2Output;


public class Hessian2Utils {
		// JavaBean序列化.
    public static <T> byte[] serialize(T javaBean) throws Exception {
        Hessian2Output ho = null;
        ByteArrayOutputStream baos = null;

        try {
            baos = new ByteArrayOutputStream();
            ho = new Hessian2Output(baos);
            ho.writeObject(javaBean);
            ho.flush();
            return baos.toByteArray();
        } catch (Exception ex) {
            System.out.println("[模拟日志记录]HessianUtils.serialize.异常." + ex.getMessage());
            throw new Exception("HessianUtils.serialize.异常.", ex);
        } finally {
            if (null != ho) {
                ho.close();
            }
        }
    }
  	// JavaBean反序列化.
    public static <T> T deserialize(byte[] serializeData) throws Exception {
        T javaBean = null;
        Hessian2Input hi = null;
        ByteArrayInputStream bais = null;

        try {
            bais = new ByteArrayInputStream(serializeData);
            hi = new Hessian2Input(bais);
            javaBean = (T) hi.readObject();
            return javaBean;
        } catch (Exception ex) {
            System.out.println("[模拟日志记录]HessianUtils.deserialize.异常." + ex.getMessage());
            throw new Exception("HessianUtils.deserialize.异常.", ex);
        } finally {
            if (null != hi) {
                hi.close();
            }
        }
    }
}
```

### 构建需要转换到类

```java
import java.io.IOException;
import java.io.Externalizable;
import java.io.ObjectInput;
import java.io.ObjectOutput;

@data
public class Car implements Externalizable {
	// 名称
	private String name;
	// 价格
	private Integer price;
	// 颜色
	private String color;
	// 长度
	private transient int length;

	@Override
	public void writeExternal(ObjectOutput out) throws IOException {
		out.writeUTF(name);
		out.writeInt(price);
		out.writeUTF(color);
		out.writeInt(length);
	}

	@Override
	public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
		System.out.println("[模拟日志记录]readExternal.");
		name = in.readUTF();
		price = in.readInt();
		color = in.readUTF();
		length = in.readInt();
	}
}
```

### 用例运行

```java
public class Hessian2UtilsTester {

    public static void main(String[] args) throws Exception {
        byte[] hessian2Data = null;
        Car deserializeCar = null;

        Car car = new Car();
        car.setName("法拉力");
        car.setPrice(12);
        car.setColor("红色");
        car.setLength(2980);

      	// 序列化
      	// 第一种
        hessian2Data = Hessian2Utils.serialize(car);
        System.out.println("Hessian序列化数据：" + hessian2Data);
      	// 第二种
	      serializeData = SerializableUtils.serialize(car);
        System.out.println("Serializable序列化数据：" + serializeData);
        System.out.println("Serializable序列化数据大小：" + serializeData.length);
      
      	// 反序列化
        deserializeCar = Hessian2Utils.deserialize(hessian2Data);
        System.out.println("Hessian反序列化数据：" + deserializeCar);
    }
}
```

