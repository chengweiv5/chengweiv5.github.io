---
layout: post
title: Java Servlet 中上传文件要点
tags:
  - java 笔记
---

在使用 Java Servlet 编写 Web 服务器时，很有可能会提供文件上传功能，这里是我踩过的坑。

#### 页面设置
这也是最容易忽略的一点：在 JSP 页面中的 <form> 中添加 *ENCTYPE="multipart/form-data"*

#### 使用文件上传库
加入第三方文件上传的库到项目中，在这里使用 Apache 的 jakarta 项目中的 commons-* 包。

#### 修改代码

首先调用 ServletFileUpload.isMultipartContent(request) 返回一个 boolean 值，如果是
true 则表示 request 支持文件上传，这里可以加测试语句，如果是 false 就知道是在 request
出了问题，可能就是传递 request 的 JSP 中的 <form> 没有添加 *ENCTYPE="multipart/form-data"*。

其次，修改代码如下：

{% highlight java %}
FileItemFactory factory=new DiskFileItemFactory();
ServletFileUpload up=new ServletFileUpload(factory);
Iterator items;//这里使用老式的迭代器，会产生警告，因为没有用泛型，但不用理他
try {
    items=up.parseRequest(request).iterator();
    while(items.hasNext()){
        FileItem item=(FileItem)items.next();
        if(!item.isFormField()){//如果是文件上传框，则处理文件
         String name=item.getName();//获得上传框中的字符串
         name=name.subString(name.lastIndexOf('//')+1,name.length());//获得上传文件名
         String savePath=request.getRealPath("upload")+File.separatorChar+name;
         //生成保存上传文件的绝对路径，当然可以用绝对路径指定其他保存路径
         //upload是新建的一个文件夹，用来保存上传的文件
         File file=new File(savePath);//获得绝对路径后可以使用File来访问了
         item.write(file);//写入具体的文件内容
         } else if(item.isFormField()){//如果是普通文本输入框
             String data=item.getString();
             if(item.getFieldName().equals("user")){
                 user=data;
             } else if(....) {
                 ...
             }
         }
    }
}catch(IOException e){

}catch(Exception e){

}
{% endhighlight %}

注意：在有附件上传功能的页面中，不能使用 String name=request.getParameter("name");
这种简单的方式来获取表单提取的资料了。
