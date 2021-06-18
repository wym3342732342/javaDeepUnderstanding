# JAX-RS

## 一、简介

&emsp;&emsp;JAX-RS即Java API for RESTful Web Services，是JAVA EE6引入的一个新技术。JAX-RS提供了一些注解将一个资源类，一个POJO Java类，封装为Web资源。

&emsp;&emsp;基于JAX-RS实现的框架有Jersey，RESTEasy等。这两个框架创建的应用可以很方便地部署到Servlet 容器中，比如Tomcat，JBoss等。值得一提的是RESTEasy是由JBoss公司开发的，所以将用RESTEasy框架实现的应用部署到JBoss服务器上，可以实现很多额外的功能。

## 二、标注

包括：

- `@Path`：标注资源类或者方法的相对路径

- `@GET，@PUT，@POST，@DELETE`：标注方法是HTTP请求的类型。

- `@Produces`：标注返回的MIME媒体类型

- `@Consumes`：标注可接受请求的MIME媒体类型

- `@PathParam`：来自于URL的路径
- `@QueryParam`：来自于URL的查询参数
- `@HeaderParam`：来自于HTTP请求的头信息
- `@CookieParam`：来自于HTTP请求的Cookie
- `@MatrixParam`：矩阵参数
- `@FormParam`：来自于HTTP请求的表单参数



