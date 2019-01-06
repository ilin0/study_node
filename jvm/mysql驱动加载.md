### mysql驱动加载.md

```java
 public static void main(String[] args) throws Exception{
    Class.forName("com.mysql.jdbc.Driver");// 这段代码可以有，也可以没有，spi默认为加载

    Connection conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/mytestdb","username", "password");


 }
```

### 加载初始化
1、首先com.mysql.jdbc.Driver
```
// 它会初始化--静态方法会被执行
static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
```

2、java.sql.DriverManager
```
// 它的静态方法也会被执行
static {
        loadInitialDrivers();
        println("JDBC DriverManager initialized");
    }
```
3、loadInitialDrivers采用ServiceLoader加载（spi机制）加载了所有的驱动。
    Class.forName("")//加载驱动的这行方法都可以省略掉。
4、DriverManager的getConnection方法
    它有if(isDriverAllowed(aDriver.driver, callerCL))判断
    判断加载driver的加载器与调用这个类的加载器是不是同一个加载器。如果是才可能用，不是则不能尝试连接。这个提现在命名空间的问题，因为使用者可以很轻易的修改当前线程的加载器。
