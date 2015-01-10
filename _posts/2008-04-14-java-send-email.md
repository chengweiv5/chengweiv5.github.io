---
layout: post
title: 使用 JavaMail 发送邮件
tags:
  - java 笔记
---

使用 JavaMail 发送邮件：

首先，jsp 中 <form action="servlet" method="post" ENCTYPE="multipart/form-data">
必须添加 ENCTYPE 属性才能支持附件上传功能。

其次，编写两个类：

- FormHandler.java 处理表单
- SenMail.java 发送邮件

其中，SendMail.java 中关键代码如下：

{% highlight java %}
import javax.mail.internet.*;
import java.util.*;
import javax.activation.FileDataSource;
import javax.activation.DataHandler;
public class SendMail(){
    private String user,password,host;
    private Session session;
    private Properties props;
    priavet MimeMultipart mp;
    private MimeMessage message;

    public SendMail(String host,String user,String password){
        props=System.getProperties();
        props.put("mail.smtp.host",host);
        props.put("mail.smtp.auth","true");//身份验证
        MyAuthentication auth=new MyAuthentication(user,password);
        session=Session.getDefaultInstance(props,auth);
        session.setDebug(true);//调试，可在控制台看到运行过程
        message=new MimeMessage(session);
        mp=new MimeMultipart();
    }
    public void setContent(String content){
        try{
            MimeBodyPart bp=new MimeBodyPart();
            bp.setContent("<meta http-equiv=Content-Type content=text/html;charset=gb2312>"+ content,"text/html;charset=gb2312");//注意这里不要写错
            mp.addBodyPart(bp);
        }catch(MessagingException e){

        }
    }
    public void addAttathment(String filePath){
        try{
            MimeBodyPart bp=new MimeBodyPart();
            FileDataSource source=new FileDataSource(filePath);
            bp.setDataHandler(new DataHandler(source));
            bp.setFileName(source.getName);
            mp.addBodyPart(bp);
        }catch(MessagingException e{

        }
    }

    public boolean send(){//发送邮件
        message.addMultipart(mp);
        message.saveChanges();//如果发送简单邮件不用写这两行
        ...
    }
}
{% endhighlight %}

注意：如果发送简单文本邮件，则使用 message.setText(content) 就可以完成了。
