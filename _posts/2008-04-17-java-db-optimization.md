---
layout: post
title: 几种 Java 使用数据库时的优化方法
tags:
  - java
---

看到一本书中介绍了数据库简单编程，讲了一些优化数据库使用的方法，这里记录如下：

#### 在使用时连接数据库：

{% highlight java %}
public Connection getConnection()throws SQLException,ClassNotFoundException{
    Class.forName(driver);
    Connection con=DriverManager.getConnection(url,userName,password);
    return con;
}
{% endhighlight %}

这种方法的缺点是当需要许多连接时比较麻烦，而且直接使用driver,url,userName,password字符串，以后如果程序长大后，不好更改。

#### 使用一个类文件保存数据库连接的信息：

public class DBData{
    public static final String DRIVER="com.mysql.jdbc.Driver";
    public static final String URL="jdbc:mysql://localhost:3306/test";
    public static final String USER_NAME="root";
    public static final String PASSWORD="123456";
}

这中方法较前一种方法的好处是，以后修改可以只用修改这个类中的静态常量就行了。但缺点是修改了这个文件项目必须重新部署。

#### 使用属性文件：

建立一个属性文件：DB.properties 内容如下

{% highlight java %}
driver="com.mysql.jdbc.Driver"
url="..............."
userName="..............."
password=" .............."
{% endhighlight %}

建立一个获取属性的类 DB.java 内容如下

{% highlight java %}
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;
public class DB{
    private String driver,url,userName,password;
    public DB(){
        driver=getProperty("driver");
        url=getProperty("url");
        userName=getProperty("userName");
        password=getProperty("password");
    }
    public String getProperty(String param){
        InputStream in;
        Properties p=new Properties();
        try{
            in=Class.forName("DB").getResourceAsStream("DB.properties");
            p.load(in);
        }catch(ClassNotFoundException e){
            e.printStackTrace();
        }catch(IOException e）{
            e.printStackTrace();
        }
        return p.getProperty(param);
    }
    public String getDriver(){
        return driver;
    }
...
{% endhighlight %}
