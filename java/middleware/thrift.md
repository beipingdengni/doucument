## 官方地址

https://thrift.apache.org/

https://github.com/apache/thrift



## 定义thrift文件

tutorial.thrift

```c
// Before running this file, you will need to have installed the thrift compiler
// into /usr/local/bin.

/**
 * The first thing to know about are types. The available types in Thrift are:
 *
 *  bool        Boolean, one byte
 *  i8 (byte)   Signed 8-bit integer
 *  i16         Signed 16-bit integer
 *  i32         Signed 32-bit integer
 *  i64         Signed 64-bit integer
 *  double      64-bit floating point value
 *  string      String
 *  binary      Blob (byte array)
 *  map<t1,t2>  Map from one type to another
 *  list<t1>    Ordered list of one type
 *  set<t1>     Set of unique elements of one type
 *
 * Did you also notice that Thrift supports C style comments?
 */

// 包含其他文件
include "shared.thrift"

/**
 * Thrift files can namespace, package, or prefix their output in various target languages.
 * java 域名空间、包
 */
namespace java com.zh.ch.bigdata.thrift.tutorial

/**
 * Thrift lets you do typedefs to get pretty names for your types. Standard
 * C style here.
 * Thrift 允许您执行 typedef 来为您的类型获取漂亮的名称。这里是标准的 C 风格。
 */
typedef i32 MyInteger

/**
 * 定义常量
 * Thrift 还允许您定义跨语言使用的常量。复杂类型和结构使用 JSON 表示法指定。
 */
const i32 INT32CONSTANT = 9853
const map<string,string> MAPCONSTANT = {'hello':'world', 'goodnight':'moon'}

/**
 * You can define enums, which are just 32 bit integers. Values are optional
 * and start at 1 if not supplied, C style again.
 */
enum Operation {
  ADD = 1,
  SUBTRACT = 2,
  MULTIPLY = 3,
  DIVIDE = 4
}

/**
 * Structs are the basic complex data structures. They are comprised of fields
 * which each have an integer identifier, a type, a symbolic name, and an
 * optional default value.
 *
 * Fields can be declared "optional", which ensures they will not be included
 * in the serialized output if they aren't set.  Note that this requires some
 * manual management in some languages.
 */
struct Work {
  1: i32 num1 = 0,
  2: i32 num2,
  3: Operation op,
  4: optional string comment,
}

/**
 * Structs can also be exceptions, if they are nasty.
 */
exception InvalidOperation {
  1: i32 whatOp,
  2: string why
}

/**
 * Ahh, now onto the cool part, defining a service. Services just need a name
 * and can optionally inherit from another service using the extends keyword.
 */
service Calculator extends shared.SharedService {

  /**
   * A method definition looks like C code. It has a return type, arguments,
   * and optionally a list of exceptions that it may throw. Note that argument
   * lists and exception lists are specified using the exact same syntax as
   * field lists in struct or exception definitions.
   *  
   */

   void ping(),

   i32 add(1:i32 num1, 2:i32 num2),

   i32 calculate(1:i32 logid, 2:Work w) throws (1:InvalidOperation ouch),

   /**
    * 此方法有一个 oneway 修饰符。这意味着客户端只发出请求，根本不监听任何响应。单向方法必须为空。
    */
   oneway void zip()

}
```

- `include "shared.thrift"`表示tutorial.thrift文件引入了shared.thrift
- Java语言的命名空间设置为`namespace java com.zh.ch.bigdata.thrift.tutorial` ，也就是说，使用thrift编译器编译上述文件后，生成的代码在上述包中
- `typedef i32 MyInteger` 表示用MyInteger表示i32数据类型

## 编译thrift

### 脚本编译

```sh
# 安联
brew install thrift
# 执行编译
thrift --gen <language> <Thrift filename>
```

### maven编译

```xml
<plugin>
  <groupId>org.apache.thrift.tools</groupId>
  <artifactId>maven-thrift-plugin</artifactId>
  <version>0.1.11</version>
  <configuration>
    <thriftExecutable>～/Document/soft/thrift-0.16.0/thrift</thriftExecutable>
    <thriftSourceRoot>src/main/thrift</thriftSourceRoot>
    <outputDirectory>src/main/java</outputDirectory>
    <generator>java</generator>
  </configuration>
  <executions>
    <execution>
      <id>thrift-sources</id>
      <phase>generate-sources</phase>
      <goals>
        <goal>compile</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```



## 定义java接口实现

### 添加依赖包

```xml
<dependency>
  <groupId>org.apache.thrift</groupId>
  <artifactId>libthrift</artifactId>
  <version>0.16.0</version>
</dependency>
```

### CalculatorHandler.java

```java
public class CalculatorHandler implements Calculator.Iface {

    private HashMap<Integer,SharedStruct> log;
		// 初始化
    public CalculatorHandler() {
        log = new HashMap<Integer, SharedStruct>();
    }
		// ping
    public void ping() {
        System.out.println("ping()");
    }
		// 添加
    public int add(int n1, int n2) {
        System.out.println("add(" + n1 + "," + n2 + ")");
        return n1 + n2;
    }
		// 计算
    public int calculate(int logid, Work work) throws InvalidOperation {
        System.out.println("calculate(" + logid + ", {" + work.op + "," + work.num1 + "," + work.num2 + "})");
        int val = 0;
        switch (work.op) {
            case ADD:
                val = work.num1 + work.num2;
                break;
            case SUBTRACT:
                val = work.num1 - work.num2;
                break;
            case MULTIPLY:
                val = work.num1 * work.num2;
                break;
            case DIVIDE:
                if (work.num2 == 0) {
                    InvalidOperation io = new InvalidOperation();
                    io.whatOp = work.op.getValue();
                    io.why = "Cannot divide by 0";
                    throw io;
                }
                val = work.num1 / work.num2;
                break;
            default:
                InvalidOperation io = new InvalidOperation();
                io.whatOp = work.op.getValue();
                io.why = "Unknown operation";
                throw io;
        }

        SharedStruct entry = new SharedStruct();
        entry.key = logid;
        entry.value = Integer.toString(val);
        log.put(logid, entry);
      	// 添加
        return val;
    }
  // 获取值
    public SharedStruct getStruct(int key) {
        System.out.println("getStruct(" + key + ")");
        return log.get(key);
    }
   // 只接收不返回数据
    public void zip() {
        System.out.println("zip()");
    }
}
```

### JavaServer.java

```java
import org.apache.thrift.server.TServer;
import org.apache.thrift.server.TServer.Args;
import org.apache.thrift.server.TSimpleServer;
import org.apache.thrift.transport.TServerSocket;
import org.apache.thrift.transport.TServerTransport;

public class JavaServer {

    public static CalculatorHandler handler;

    public static Calculator.Processor processor;

    public static void main(String [] args) {
        try {
            handler = new CalculatorHandler();
            processor = new Calculator.Processor(handler);

            Runnable simple = new Runnable() {
                public void run() {
                    simple(processor);
                }
            };

            new Thread(simple).start();
        } catch (Exception x) {
            x.printStackTrace();
        }
    }

    public static void simple(Calculator.Processor processor) {
        try {
            TServerTransport serverTransport = new TServerSocket(9090);
            TServer server = new TSimpleServer(new Args(serverTransport).processor(processor));

            // Use this for a multithreaded server
            // TServer server = new TThreadPoolServer(new TThreadPoolServer.Args(serverTransport).processor(processor));

            System.out.println("Starting the simple server...");
            server.serve();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### JavaClient.java

```java
import org.apache.thrift.TException;
import org.apache.thrift.transport.TSSLTransportFactory;
import org.apache.thrift.transport.TTransport;
import org.apache.thrift.transport.TSocket;
import org.apache.thrift.transport.TSSLTransportFactory.TSSLTransportParameters;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.protocol.TProtocol;

public class JavaClient {

    public static void main(String [] args) {
        try {
            TTransport transport;
            transport = new TSocket("localhost", 9090);
            transport.open();
            TProtocol protocol = new TBinaryProtocol(transport);
            Calculator.Client client = new Calculator.Client(protocol);

            perform(client);

            transport.close();
        } catch (TException x) {
            x.printStackTrace();
        }
    }
		// 执行动作和请求
    private static void perform(Calculator.Client client) throws TException
    {
        client.ping();
        System.out.println("ping()");

        int sum = client.add(1,1);
        System.out.println("1+1=" + sum);

        Work work = new Work();

        work.op = Operation.DIVIDE;
        work.num1 = 1;
        work.num2 = 0;
        try {
            int quotient = client.calculate(1, work);
            System.out.println("Whoa we can divide by 0");
        } catch (InvalidOperation io) {
            System.out.println("Invalid operation: " + io.why);
        }

        work.op = Operation.SUBTRACT;
        work.num1 = 15;
        work.num2 = 10;
        try {
            int diff = client.calculate(1, work);
            System.out.println("15-10=" + diff);
        } catch (InvalidOperation io) {
            System.out.println("Invalid operation: " + io.why);
        }

        SharedStruct log = client.getStruct(1);
        System.out.println("Check log: " + log.value);
    }
}
```

