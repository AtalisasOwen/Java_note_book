```java
import java.sql.Connection;  
import java.sql.DriverManager;  
import java.sql.SQLException;  
  
public class ConnectionDemo{  
    //定义MySQL驱动程序  
    public static final String DBDRIVER = "org.gjt.mm.mysql.Driver";  
    //定义MySQL数据库的连接地址  
    public static final String DBURL = "jdbc:mysql://localhost:3306/xiaowei";  
    //MySQL数据库的连接用户名  
    public static final String DBUSER = "root";  
    //MySQL数据库的连接密码  
    public static final String DBPASS = "android";  
    public static void main(String[] args){  
        //数据库连接  
        Connection con = null;  
        try{  
            //加载驱动  
            Class.forName(DBDRIVER);  
        }catch(ClassNotFoundException e){  
            e.printStackTrace();  
        }  
        try{  
            con = DriverManager.getConnection(DBURL, DBUSER, DBPASS);  
        }catch(SQLException e){  
            e.printStackTrace();  
        }  
        System.out.println(con);  
        try{  
            //数据库关闭  
            con.close();  
        }catch(SQLException e){  
            e.printStackTrace();  
        }  
    }  
}  
```



