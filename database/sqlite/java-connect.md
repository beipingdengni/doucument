

# java 连接sqlite

## 官方网站

https://www.sqlite.org/index.html

## java连接

```properties
#SQLiteDataSource dataSource = new SQLiteDataSource();
#dataSource.setUrl("jdbc:sqlite:./demo.db");

spring.datasource.driver-class-name=org.sqlite.JDBC # 数据驱动
spring.datasource.url=jdbc:sqlite:./demo.db  # 数据库地址
spring.datasource.username=
spring.datasource.password=
```



## java 内存

```java
SQLiteConnection source = new SQLiteConnection("Data Source=c:\\test.db");
source.Open();
 
using (SQLiteConnection destination = new SQLiteConnection("Data Source=:memory:"))
{
  destination.Open();               
 
  // copy db file to memory
  source.BackupDatabase(destination, "main", "main",-1, null, 0);
  source.Close();
 
  // insert, select ,...        
  using (SQLiteCommand command = new SQLiteCommand())
  {
    command.CommandText = "INSERT INTO t1 (x) VALUES('some new value');";
 
    command.Connection = destination;
    command.ExecuteNonQuery();
  }             
 
  source = new SQLiteConnection("Data Source=c:\\test.db");
  source.Open();
 
  // save memory db to file
  destination.BackupDatabase(source, "main", "main",-1, null, 0);
  source.Close();               
}
```



### 步骤 1：添加SQLite JDBC依赖

如果你使用的是Maven，可以在`pom.xml`中添加以下依赖：

```xml
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.36.0.3</version> <!-- 使用最新版本 -->
</dependency>
```

对于Gradle，可以在`build.gradle`文件中添加：

```gradle
dependencies {
    implementation 'org.xerial:sqlite-jdbc:3.36.0.3' // 使用最新版本
}
```

如果不使用构建工具，则需要手动下载JDBC驱动的jar包，并将其添加到项目的类路径中。

### 步骤 2：注册JDBC驱动并建立连接

在Java代码中，你需要注册SQLite JDBC驱动，并建立到SQLite数据库文件的连接：

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class SQLiteExample {
    public static void main(String[] args) {
        // 连接SQLite的URL，test.db是数据库文件名
        String url = "jdbc:sqlite:test.db";

        // 创建一个数据库连接
        try (Connection conn = DriverManager.getConnection(url)) {
            if (conn != null) {
                System.out.println("连接到SQLite数据库成功！");
                // 在这里进行数据库操作
                // ...
            }
        } catch (SQLException e) {
            System.err.println(e.getMessage());
        }
    }
}
```

### 步骤 3：执行SQL语句

一旦建立了连接，你就可以创建`Statement`或`PreparedStatement`对象来执行SQL语句了：

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class SQLiteExample {
    public static void main(String[] args) {
        String url = "jdbc:sqlite:test.db";

        // 创建一个数据库连接
        try (Connection conn = DriverManager.getConnection(url);
             Statement stmt = conn.createStatement()) {

            // 创建一个新表
            String sql = "CREATE TABLE IF NOT EXISTS students (" +
                         "id INTEGER PRIMARY KEY AUTOINCREMENT," +
                         "name TEXT NOT NULL," +
                         "age INTEGER)";
            stmt.execute(sql);
            System.out.println("表创建成功或已存在。");

            // 插入数据
            sql = "INSERT INTO students (name, age) VALUES ('Alice', 21)";
            stmt.executeUpdate(sql);
            System.out.println("数据插入成功。");

            // 查询数据
            sql = "SELECT * FROM students";
            try (ResultSet rs = stmt.executeQuery(sql)) {
                while (rs.next()) {
                    System.out.println(rs.getInt("id") + "\t" +
                                       rs.getString("name") + "\t" +
                                       rs.getInt("age"));
                }
            }

        } catch (SQLException e) {
            System.err.println(e.getMessage());
        }
    }
}
```

在上面的代码中，我们创建了一个名为`students`的新表，然后向其中插入了一条数据，并查询了所有数据。

### 注意事项

- 在实际应用中，应该避免硬编码SQL语句，特别是在插入或更新数据时，应该使用`PreparedStatement`来防止SQL注入攻击。
- 确保在操作完成后关闭数据库连接、语句和结果集对象，以释放资源。在上面的示例中，我们使用了try-with-resources语句，它会自动关闭资源。
- SQLite JDBC驱动默认支持多线程的串行化访问，你可以通过数据库URL的参数来调整并发模式。
- 如果你的应用程序需要处理大量的并发写操作，SQLite可能不是最佳选择，应该考虑使用更强大的数据库系统。
