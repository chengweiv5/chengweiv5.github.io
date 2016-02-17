---
layout: post
title: Java Servlet 中下载文件
tags:
  - java
---

在使用 Java Servlet 编写 Web 服务器时，很有可能会提供文件下载功能，这里记录了一些要点。

首先，将文件放在项目的根目录下。

其次，使用如下代码来提供下载功能。

{% highlight java %}
try {
    String name="下载.txt";//注意这里是中文名，所以要特别处理
    String realPath=request.getRealPath(name);
    String fileName=java.net.URLEncoder.encode(name,"utf-8");//采用utf-8编码文件名
    response.setCharacterEncoding("utf-8");
    response.setHeader("Content-Disposition","attachment;filename="+fileName);
    response.setContentType("text/plain");//MIME类型
    PrintWriter out=response.getWriter();
    String realPath=request.getRealPath(name);
    realPath=realPath.substring(0,realPath.lastIndexOf('//'))+File.separatorChar+name;//注意这里
    //千万不能在用file，因为它已经进过编码了，所以会抛出找不到文件异常。
    FileReader reader=new FileReader(new File(realPath));
    BufferedReader bf=new BufferedReader(reader);
    while((name=bf.readLine())!=null){//从文件中读出内容
        out.println(name);//读出文件并写入输出流中
    }
}catch(IOException e}{

}
{% endhighlight %}
