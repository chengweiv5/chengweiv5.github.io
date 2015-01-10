---
layout: post
title: MyEclipse 连接 MySQL 数据库
tags:
  - java 笔记
---

像下面这样配置 MyEclipse：

- Window -> Open perspective -> Other -> MyEclipse Database Explorer
- 填写相应的数据库驱动，url, userName, password
- 点击 Test Connect，然后会要求输入密码，配置都正确，即能连接上

使用下面的代码连接 MySQL 数据库：

{% highlight java %}
try{
    Class.forName(driver);//Class.forName("com.mysql.jdbc.Driver");
    Connection con=DriverManager.getConnection(url,user,password);
    //url="jdbc:mysql://localhost:3306/test"注意：test是mysql自带的一个测试数据库，没有内容
    //user="root"
    //password="mima"
    Statement stm=con.createStatement();
    ResultSet set=stm.executeQuery(sql);
    //sql="select * from a_table"
    while(set.next()){
        out.println(set.getInt("id"));
        out.println(set.getString("name"));
        ....
    }
}catch(SQLException e){
    e.printStackTrace();
}
{% endhighlight %}
