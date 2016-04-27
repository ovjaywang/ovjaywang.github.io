---
title: 关于bae的java环境下的几点提示
categories:
  - 代码狗
  - 学习log
tags:
  - bae
  - java
id: 98
date: 2013-05-08 12:09:32
---

1.要加入session的话需要在duapp.xml中加入
2.不能部署在本机tomcat的问题 以myeclipse为例
在项目根目录的 .mymetadata中id="XXX"后面 加入 context-root="/项目名字"（不带后缀，随便起名字 最好和 name="XX"中一样）![bae的本机部署](http://i1061.photobucket.com/albums/t476/ov_beeshoot/bae90E87F72.jpg)
3.log4j要参考文档进行配置。否则直接使用会有异常抛出。
4.连接云百度数据库 必须要在百度的云环境（即svn或者git在本地编辑的工程。BaeEnv是云环境的变量 不用导入包即可使用）
注意的是 连接云数库只能在云环境使用 使用tomcat等在本地的无法连接云数据库。
在java环境中为例：
在云环境基本信息中获取ApIkey 和secretKey![bae基本信息](http://i1061.photobucket.com/albums/t476/ov_beeshoot/bae768457FA672C4FE1606F.jpg)
在版本管理中新建一个版本 并在本地用svn同步（不知道什么是svn...算了）
在云数据库中新建一个数据库 可以获取免费的1G配额
数据库可以利用phpAdmin管理 也可以在本地建好导出sql脚本（最好在本地建好 否则phpAdmin比较难操作 在本地可以利用MysqlFont我比较喜欢这个）如图建立了一个User表 里面有username和password两个字段
![bae数据库](http://i1061.photobucket.com/albums/t476/ov_beeshoot/bae6570636E5E93.jpg)
在同步的javaweb中建以下几个东西
一个DBManager.java以连接数据库

'''java
import java.sql.DriverManager;
import java.sql.SQLException;
import com.mysql.jdbc.Connection;
public class  DBManager {
    private static Connection con=null;
    /**
     * 
     * 构造方法私有 外部方法不能创建对象
     * @author Wjy
     * 懒汉单态模式
     */
    private DBManager()
    {}
    public synchronized static Connection getCon() 
        throws ClassNotFoundException, SQLException
    {
//      String host = BaeEnv.getBaeHeader(BaeEnv.BAE_ENV_ADDR_SQL_IP);
//      String port = BaeEnv.getBaeHeader(BaeEnv.BAE_ENV_ADDR_SQL_PORT);
//      String username = BaeEnv.getBaeHeader(BaeEnv.BAE_ENV_AK);
//      String password = BaeEnv.getBaeHeader(BaeEnv.BAE_ENV_SK);
          String DRIVERNAME = &quot;com.mysql.jdbc.Driver&quot;;
          String URL = &quot;jdbc:mysql://sqld.duapp.com:4050/rMDtIBgOtaeTHVuKjaep&quot;;
          String USER = &quot;uK2b4AyjaNTGCcxUbR9Drmo8&quot;;
          String PWD = &quot;LGN4LjqH0bTFMGhtTMoQMCeDFQ6tzeRW&quot;;
        //String driverName = &quot;com.mysql.jdbc.Driver&quot;;
        //String dbUrl = &quot;jdbc:mysql://&quot;;
        //String serverName = host + &quot;:&quot; + port + &quot;/&quot;;   
            //从平台查询应用要使用的数据库名
        String databaseName = &quot;rMDtIBgOtaeTHVuKjaep&quot;;
        //String connName = dbUrl + serverName + databaseName; 
        Connection connection = null;   
        Class.forName(DRIVERNAME);
        connection = (Connection) DriverManager.getConnection(URL, USER,PWD); // 链接数据库  
        if(connection!=null){return connection;}
        else{return null;}//具体的数据库操作逻辑

    } 
}
'''

一个实体类User.java

'''java
public class User {
    private String userName;
    private String psw;
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getPsw() {
        return psw;
    }
    public void setPsw(String psw) {
        this.psw = psw;
    }
    public User(String userName, String psw) {
        super();
        this.userName = userName;
        this.psw = psw;
    }
    @Override
    public String toString() {
        return &quot;User [userName=&quot; + userName + &quot;, psw=&quot; + psw + &quot;]&quot;;
    }

}
'''

一个把sql查询的结果集转换成实体类的接口和实现类
EntityMapping .java

'''java
import java.sql.ResultSet;
import java.sql.SQLException;
public interface EntityMapping {
    public Object mapping(ResultSet rs)throws SQLException ;
}
'''

UserMapping.java

'''java
import java.sql.ResultSet;
import java.sql.SQLException;
public class UserMapping  implements EntityMapping{

    @Override
    public User mapping(ResultSet rs) throws SQLException {
        int i = 1;
        User user = new User(rs.getString(i++),rs.getString(i++));
        return user;
    }

}
'''

一个万能！！！的！！！操作数据库的类。。在任何地方也可以使用
JdbcTemplate.java

'''java 
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Vector;

//代替原来DBmamager功能
public class JdbcTemplate {
    public  boolean update(String sql,Object... values) throws ClassNotFoundException, SQLException
    {
        PreparedStatement psta=null;
        int row=0;
        try {
            Connection con=DBManager.getCon();
            psta= con.prepareStatement(sql);    
                for (int i = 0; i &lt; values.length; i++) {
                psta.setObject(i+1, values[i]);
            }
            row=psta.executeUpdate();
        } catch (Exception e) {
            // TODO: handle exception
        }finally
        {
            if (psta!=null) {
                psta.close();
                psta=null;
            }
        }       

        return (row==1);
    }
    public List findAll(String sql,EntityMapping EnMp,Object...Objects) throws SQLException
    {   PreparedStatement psta=null;
        ResultSet rs=null;
        List list=new Vector();
        try {
            Connection con=DBManager.getCon();
            psta= con.prepareStatement(sql);    
        for (int i = 0; i &lt; Objects.length; i++) {
             psta.setObject(i+1, Objects[i]);
        }
             rs=psta.executeQuery();
            if(rs==null)
            {System.out.println(&quot;为空！&quot;);}
        while(rs.next())
        {
             list.add(EnMp.mapping(rs));
        }

    } catch (Exception e) {
    }finally
    {
        if (rs!=null) {
            rs.close();
            rs=null;
        }
        if (psta!=null) {
            psta.close();
            psta=null;
        }
    }       
        return list;
    }
}
'''

连接数据库的servlet
GetUser.java

'''java
import java.io.IOException;
import java.sql.SQLException;
import java.util.List;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class GetUser extends HttpServlet {
    public GetUser() {
        super();
    }
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        this.doPost(request, response);
    }
    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
         JdbcTemplate template=new JdbcTemplate();
         List  list=null;        
         try {
             list=template.findAll(&quot;select * from User&quot;, new UserMapping());
        } catch (SQLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();}
         request.setAttribute(&quot;User&quot;, list);
         request.getRequestDispatcher(&quot;/User.jsp&quot;).forward(request, response);}
}
'''

最后是显示信息的页面User.jsp

'''html
&lt;%@ page language=&quot;java&quot; import=&quot;java.util.*,java.net.URL&quot; contentType=&quot;text/html; charset=UTF-8&quot;%&gt;
&lt;%@page import=&quot;你的类名！&quot;%&gt;
&lt;!DOCTYPE HTML PUBLIC &quot;-//W3C//DTD HTML 4.01 Transitional//EN&quot;&gt;
&lt;html&gt;
  &lt;head&gt;
    &lt;title&gt;User&lt;/title&gt;
  &lt;/head&gt;
  &lt;body&gt;
    &lt;%Object info=request.getAttribute(&quot;User&quot;);
    if (info!=null)
    {  List&lt;User&gt; uu=(List&lt;User&gt;)info;
        for (User user : uu) {
            %&gt;&lt;%=user%&gt;&lt;br&gt;&lt;% 
        }}
   %&gt;
  &lt;/body&gt;
&lt;/html&gt;
'''

记住的是要把web.xml中的welcome-filelist设成servlet的名字或者直接输入地址
http://1.你的app名字.duapp.com/GetUser