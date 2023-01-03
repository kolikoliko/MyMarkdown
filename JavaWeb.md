# javaweb



## JDBC

用java语言操作关系型数据库的一套API，全称（java database connectitvity）java数据库连接

在java里面使用数据库，需要导入对应数据库的包，这里使用的数据库式mysql

一般步骤如下

在此之前，需要导入java的mysql包

```java
/**
 * JDBC的快速入门
 */
public class JDBCDemo {
    public static void main(String[] args) throws Exception {
        //1.注册驱动，一般是不写的
        Class.forName("com.mysql.cj.jdbc.Driver");

        //2.获取链接
        String url="jdbc:mysql://localhost:3306/jdbcdemo";
        String user="koliko";
        String password="abc123456";
        Connection conn = DriverManager.getConnection(url, user, password);

        //3.定义sql语句
        String sql="INSERT into student (stu_id,stu_class,stu_name ) VALUES (5,3,\"zgm\")";

        //4.获取sql的执行对象
        Statement stmt = conn.createStatement();

        //5.执行对象
        int count = stmt.executeUpdate(sql);//返回的结果是行数

        //6.处理结果
        System.out.println(count);

        //7.释放资源
        stmt.close();
        conn.close();
    }
}
```



### DriverManger

（驱动管理类）作用：

1. 注册驱动
2. 获取数据库的连接



### Connection

(数据库连接对象)作用：

1. 获取sql的对象

   ![image-20230103211024787](images/image-20230103211024787.png)

2. 管理事务

   ![image-20230103211129641](images/image-20230103211129641.png)

在事务处理的时候，通常使用java里的try catch语句，在抛出异常后进行回滚，无异常则提交，以达到事务的完成。



### Statement

被用来执行sql语句

![image-20230103213603558](images/image-20230103213603558.png)



### ResultSet

![image-20230103220204626](images/image-20230103220204626.png)

ResultSet属于数据格式

代码参考如下，利用集合来实现对数据库行内容的提取

```java
public class JDBC_ResultSet {
    public static void main(String[] args) throws Exception {
        //1.注册驱动
        //Class.forName("com.mysql.cj.jdbc.Driver");

        //2.获取链接
        String url="jdbc:mysql://localhost:3306/jdbcdemo";
        String user="koliko";
        String password="abc123456";
        Connection conn = DriverManager.getConnection(url, user, password);

        //3.定义sql语句
        String sql="select * from student";

        //4.获取sql的执行对象
        Statement stmt = conn.createStatement();

        //5.执行对象
        ResultSet rs=stmt.executeQuery(sql);//返回的结果是ResultSet格式的对象

        List<student> list=new ArrayList<>();
        //6.处理结果
     	//当rs.next()指向为非空的时候不断地赋值
        while(rs.next()){
            student stu=new student();

            int stu_id=rs.getInt("stu_id");
            String stu_name=rs.getString("stu_name");
            String stu_class=rs.getString("stu_class");

            stu.setStu_id(stu_id);
            stu.setStu_name(stu_name);
            stu.setStu_class(stu_class);

            list.add(stu);
        }

        System.out.println(list);
        //7.释放资源
        rs.close();
        stmt.close();
        conn.close();
    }
}
```



### PrepareStatement

作用和Statement一样，能做到sql防注入

防注入的原理在于，在输入字符串的时候进行了转义

![image-20230103225600577](images/image-20230103225600577.png)