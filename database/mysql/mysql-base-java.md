





```java
public class Main {
    public static void main(String[] args) throws Exception {
        System.out.println("Hello world!");
        Class.forName("com.mysql.jdbc.Driver");
        //2.获得连接
//        String url = "jdbc:mysql://127.0.0.1:3306/demo?characterEncoding=utf-8&usessL=false";
        String url = "jdbc:mysql://127.0.0.1:3306/demo?characterEncoding=utf-8&serverTimezone=Asia/Shanghai";
        String username = "tian";
        String password = "123456";
        Connection connection = DriverManager.getConnection(url, username, password);
        //3.获取发送SQL语句对象
        Statement statement = connection.createStatement();
        //4.编写SQL语句
//        String sql = "insert into sys_user(name,age) value('" + UUID.randomUUID().toString() + "','18')";
//        statement.executeUpdate(sql);
        // 查询集合
        ResultSet resultSet = statement.executeQuery("select * from  sys_user");
        while (resultSet.next()) {
            long id = resultSet.getLong(1);
            String name = resultSet.getString(2);
            int age = resultSet.getInt(3);
            System.out.println(String.format("id: %s,name: %s,age:%s", id, name, age));
        }
        System.out.println();
        //5.释放资源
        statement.close();
        connection.close();
    }
}
```

