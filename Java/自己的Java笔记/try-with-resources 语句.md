# try-with-resources 语句

## 简单应用

在Java7之前，使用`try-catch-finally`管理可关闭的资源，例如 ：

```java
public class TryCatch {
    public static void main(String[] args) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try{
            Class.forName("com.mysql.cj.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1/graduate?serverTimezone=GMT%2B8","root","root");
            preparedStatement = connection.prepareStatement("SELECT * FROM USER WHERE ID = ?");
            preparedStatement.setInt(1,1);
            resultSet = preparedStatement.executeQuery();
            while(resultSet.next()){
                System.out.println(resultSet.getString("username"));
                System.out.println(resultSet.getString("password"));
            }
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }finally {
            if(resultSet != null){
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if (preparedStatement != null){
                try {
                    preparedStatement.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(connection != null){
                try {
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```

Java7的`try-with-resource`极大地简化了这个任务 ：任何实现了`java.lang.AutoCloseable`的类都可以用于`try-with-resources`结构中 。

而 JDBC 中的`Connection`、`Statement`、`ResultSet`接口都继承了`AutoCloseable`接口 。

下面的例子使用了`try-with-resources`结构，在`try`关键字后面的圆括号中声明的资源，都会在隐式的`finally`块中自动关闭 。

```java
public class TryWithResources {
    public static void main(String[] args) {
        try(
                Connection connection = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/graduate?serverTimezone=GMT%2B8","root","root");
                PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM user WHERE id = ?");
        ){
            preparedStatement.setInt(1,1);
            try(ResultSet resultSet = preparedStatement.executeQuery()){
                while(resultSet.next()){
                    System.out.println(resultSet.getString("username"));
                    System.out.println(resultSet.getString("password"));
                }
            }
        }catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

再举一个Java读取文件流信息的例子 ，在Java7之前需要如下写法 ：

```java
BufferedReader bufferedReader = null;
String filePath = "F:\\项目练习\\JavaWeb高级编程\\介绍JavaEE平台\\try_with_resources\\naruto.txt";
try{
    bufferedReader = new BufferedReader(new FileReader(new File(filePath)));
    String s = null;
    while((s = bufferedReader.readLine()) != null){
        System.out.println(s);
    }
}catch (Exception e){
    e.printStackTrace();
}finally {
    if(bufferedReader != null){
        try {
            bufferedReader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

在Java7之后 ：

```java
String filePath = "F:\\项目练习\\JavaWeb高级编程\\介绍JavaEE平台\\try_with_resources\\naruto.txt";
try(
        BufferedReader bufferedReader = new BufferedReader(new FileReader(new File(filePath)));
        ){
    String out = null;
    while((out = bufferedReader.readLine()) != null){
        System.out.println(out);
    }
}catch (Exception e){
    e.printStackTrace();
}
```

## 动手实践

为了能够配合`try-with-resources`，资源必须要实现`AutoCloseable`接口 。该接口的实现类需要重写`close`方法 ：

```java
public interface AutoCloseable{
    void close() throws Exception;
}
```

```java
public class Connection implements AutoCloseable {
    public void sendData(){
        System.out.println("正在发送数据");
    }

    @Override
    public void close() throws IOException {
        System.out.println("正在关闭链接");
    }
}
```

调用类 ：

```java
public static void main(String[] args) {
    try(Connection connection = new Connection();){
        connection.sendData();
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

运行结构如下所示 ：

> 正在发送数据
> 正在关闭链接

通过结果我们可以看到，`close`方法被自动调用了 。

## 👏 异常屏蔽问题

在`try-catch-finally`代码块中，如果`try`、`catch`、`finally`块均有异常抛出，那么最终只能抛出`finally`块中的异常，而`try`、`catch`块中的异常会被屏蔽 。这就是**异常屏蔽**问题 。如下图所示 ：

```java
public class Connection implements AutoCloseable {
    public void sendData() throws Exception{
        System.out.println("正在发送数据");
        throw new Exception("send data");
    }
    @Override
    public void close() throws Exception {
        System.out.println("正在关闭链接");
        throw new MyException("close");
    }
}
```

调用方法 ：

```java
public class ConnectionDemo {
    public static void main(String[] args) {
        try {
            test();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    private static void test() throws Exception{
        Connection connection = null;
        try{
            connection = new Connection();
            connection.sendData();
        }finally {
            if (connection != null){
                connection.close();
            }
        }
    }
}
```

当执行`con.sendData()`时，该方法会将异常抛给调用者`main()`，但是在抛出之前会先执行`finally`块 。当执行`finally`时，也会向调用者抛出一个异常 。此时，由`try`抛出的异常会被覆盖，`main`方法中仅仅打印出`finally`块中的异常。其结果如下所示 ：

> 正在发送数据
> 正在关闭链接
> exception.MyException: close
> ​	at java7.Connection.close(Connection.java:15)
> ​	at java7.ConnectionDemo.test(ConnectionDemo.java:26)
> ​	at java7.ConnectionDemo.main(ConnectionDemo.java:12)

这就是`try-catch-finally`的异常屏蔽问题，而`try-with-resources`能很好的解决这个问题 ：

```java
public static void main(String[] args) {
    try(Connection connection = new Connection();){
        connection.sendData();
    }catch (Exception e){
        e.printStackTrace();
    }
}
```

其结果如下所示 ：

> 正在发送数据
> 正在关闭链接
> java.lang.Exception: send data
> ​	at java7.Connection.sendData(Connection.java:9)
> ​	at java7.ConnectionDemo.main(ConnectionDemo.java:6)
> ​	Suppressed: exception.MyException: close
> ​		at java7.Connection.close(Connection.java:15)
> ​		at java7.ConnectionDemo.main(ConnectionDemo.java:7)

## `try-catch-finally`执行流程

`finally`中的`return`、`throw`会覆盖`try`和`catch`中的 **`return`和`throw`** ：

```java
public int test() {
    try {
        int a = 1;
        a = a / 0;
        return a;
    } catch (Exception e) {
        return -1;
    } finally{
        return -2;
    }
}
```

函数的返回结果是 -2 ；

注意，该代码仅供演示，实际项目中，不要再`finally`中使用`return` 。

## 异常处理规约

参见阿里巴巴异常处理 ——— [异常处理](https://github.com/alibaba/p3c/blob/master/p3c-gitbook/%E5%BC%82%E5%B8%B8%E6%97%A5%E5%BF%97/%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86.md)

## 参考

- [揭秘Java异常体系中的秘密](https://zhuanlan.zhihu.com/p/34470354)

