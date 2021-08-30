# JEE 开发入门

## 准备介质

该实验使用 Windows 环境 ， 安装的介质都需要是 Windows 版本。

[eclipse JEE 下载页面](https://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/2021-06/R/eclipse-jee-2021-06-R-win32-x86_64.zip)
[WebLogic的下载页面](http://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html)下找到版本10.3.6下的 Generic 。对应的 JDK 版本为 jdk1.7 ，可以到[JDK的下载页面](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html#jdk-7u80-oth-JPR)中下载 jdk-7u80-windows-x64.exe 

## 创建项目
1. 启动 Eclipse ，选择 workspace (工作路径)
2. 点击 File -> New -> Dynamic Web Project
3. 设置 Project name ，本文这里设置为loginweb, Target runtime 点击 New Runtime, 选择 Oracle Weblogic Server ，这里需要下载 Weblogic 的插件，下载完成后点击 Next ，设置刚刚下载的 Weblogic 和 Jdk 目录，设置完成后点击下一步，项目创建完成。

## 创建 index.jsp 文件
在 loginweb/src/mian 下右键 webapp 目录 -> New -> JSP file , 创建 index.jsp 文件, 打开 index.jsp 可以看到:

```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Insert title here</title>
</head>
<body>
</body>
</html>
```

在 body 标签中添加一段文字，和一个超链接：

```jsp
<body>
  <p>this is index</p>
  <p><a href:="login.jsp">Login here</a></p>
</body>
```

## 创建 login.jsp 文件

在 loginweb/src/mian 下右键 webapp 目录 -> New -> JSP file , 创建 login.jsp 文件, 添加 username 和 password 文本框和提交按钮:

```jsp
<%@ page language="java" contentType="text/html; charset=ISO-8859-1"
    pageEncoding="ISO-8859-1"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="ISO-8859-1">
<title>Insert title here</title>
</head>
<body>
   <form action="LoginServlet" method="post">
	   <p><input name="username"/></p>
	   <p><input name="password" type="password"/></p>
	   <p><input name="submit" type="submit" value="submit"/></p>
   </form>
</body>
</html>
```
form 中 action 属性规定了当提交表单时，数据要发送到哪里 。 method 属性规定了以什么方式提交表单 get 或 post (默认是 get )，详细可以查看[get 和 post 区别](#get-和-post-区别)。 

## 创建 Servlet

1. 右键项目，New -> Servelet 
2. 设置 java package , 这里设置为 loginWeb ， 设置 Class name ， 这里设置为 LoginServlet 。
3. 点击完成 ， Servelet 位置在 src/main/java/test/loginServlet/ 下。

可以在 Servlet 的 doPost 方法中设置匹配规则, 如果用户名和密码相同， 则跳转到 index.jsp ，否则的话继续留在 login.jsp:

```java

protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		response.getWriter().append("Served at: ").append(request.getContextPath());
}
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub

        String user = request.getParameter("user");
        String password = request.getParameter("password");
        if(user.equals(password)){
                request.getSession().setAttribute("user", user);
                response.sendRedirect("index.jsp");
            //  RequestDispatcher  disp  =  request.getRequestDispatcher("index.jsp");
                disp.forward(request,  response);
        }else{
                response.sendRedirect("login.jsp");
            //  RequestDispatcher  disp  =  request.getRequestDispatcher("login.jsp");
            //  disp.forward(request,  response);
        }
}
```

该段代码中的 sendRedirect 和 forward 方法都可以实现页面的跳转， 两种方法的具体功能和区别，可以查看[sendRedirect 和 forward 的区别](#sendredirect-和-forward-的区别)。

## LoginServelet 添加会话信息

如果用户名密码相同， 判定登陆成功， 页面跳转到 index.jsp， 将 user 信息添加到 session 中， 有关 session 可以查看[Session 和 Cookie 的作用](#session-和-cookie-的作用)

```java
 if(user.equals(password)){
//      out.print("Login Succeed !");
        response.sendRedirect("index.jsp");
        request.getSession().setAttribute("user", user);
}
```

## index.jsp 添加会话判断

确保 session 中存入 user 的值不为空， 也不是空字符串， 然后将 user 的值显示出来 。 

```jsp
<% 
if(session.getAttribute("user")!=null && !session.getAttribute("user").equals("")){
%>
<p>Welcome user:<%=session.getAttribute("user")%></p>
<%}else{ %>
<p><a href="login.jsp">Login Here</a></p>
<%} %>
```

## 作业
1. Get 和 Post 的区别
2. Session 和 Cookie 的作用
3. sendRedirect 和 forward 的区别
4. form 怎么更改提交方式