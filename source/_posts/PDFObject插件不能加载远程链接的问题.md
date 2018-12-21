---
title: PDFObject插件不能加载远程链接的问题
date: 2018-12-21 20:38:42
tags: 
- PDFObject
- 文件下载
categories: 
- 七七八八
---

# PDFObject插件

PDFObject插件是一款前端阅读pdf的便捷插件，简单好用，官方网址：[https://pdfobject.com/](https://pdfobject.com/)，官方提供的简单用例：

```html
<script src="/js/pdfobject.js"></script>
```

```javascript
var options = {
   width: "20rem",
   height: "20rem"
};
PDFObject.embed("myfile.pdf", "#my-container", options);
```

详细的使用可以参考官方API，这里不做说明。

# PDF以链接请求的方式获取

一般情况下，当使用PDFObject的时候，传入的pdf都是路径的方式去应用，当我们的PDF不在本地或者在远端的时候我们该怎么引用呢？

```javascript
PDFObject.embed("xxx/xxx.do?id=xxxxx", "#my-container", options);
```

一般我们都会直接替换路径为请求就可以了，但是这样有时候是行不通的，F12没报错，后台的请求也确实执行了，但是容器里就是不显示PDF，而且我们将这个url贴到浏览器地址里是可以正常下载的。然后就开始百度，开始谷歌，发现大部分人都是有这种问题的，但是都没有一个好的解决方案。

查看后台下载的代码：

```java
/**
 * @param response 
 * @param filePath		//文件完整路径(包括文件名和扩展名)
 * @param fileName		//下载后看到的文件名
 * @return  文件名
 */
public static void fileDownload(final HttpServletResponse response, String filePath, 	String fileName) throws Exception{ 
    //将文件转换为字节数组
    byte[] data = FileUtil.toByteArray2(filePath);  
    fileName = URLEncoder.encode(fileName, "UTF-8");  
    response.reset();  
    response.setHeader("Content-Disposition", "attachment; filename=\"" + fileName + "\"");  
    response.addHeader("Content-Length", "" + data.length);  
    response.setContentType("application/octet-stream;charset=UTF-8");  
    OutputStream outputStream = new BufferedOutputStream(response.getOutputStream());  
    outputStream.write(data);  
    outputStream.flush();  
    outputStream.close();
    response.flushBuffer();
}
```

这段下载代码下载任何文件都是没有问题的，在项目当中也是经常用到，但是用于PDFObject就行不通了，也不是这段代码的问题，这里做个稍微的改动：

```java
/**
 * Jackrabbit库中文件设定类型的去显示
 * 针对pdfObject的修改
 * @param response
 * @param data
 * @throws Exception
 */
public static void  pdfObjectViewer(final HttpServletResponse response, byte[] data) 	throws Exception{
     response.reset();
     response.setContentType("application/pdf;charset=UTF-8");
     OutputStream outputStream = new BufferedOutputStream(response.getOutputStream());
     outputStream.write(data);
     outputStream.flush();
     outputStream.close();
     response.flushBuffer();
}
```

大家不用在意这个data字节数组，看自己的逻辑要求去修改就可以，主要是设定这个MIME类型，原先的下载MIME类型是octet-stream，这里修改为指定的文件类型pdf，文件名的设定不需要也可以。

> application：某二进制的一个附件

> octet-stream：子类型，不确定下载文件的通用指定

> pdf：pdf类型的文件

# 后记

希望大家遇到同样的问题后能看到这篇笔记，有不对的地方请指正，让他人少走弯路。