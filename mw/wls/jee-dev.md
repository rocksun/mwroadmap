# JEE 开发入门

- [准备介质](#准备介质)
- [安装介质](#安装介质)
- [创建项目](#创建项目)
- [创建 index.jsp 文件](#创建-indexjsp-文件)
- [创建 login.jsp 文件](#创建-loginjsp-文件)
- [创建 Servlet](#创建-servlet)
- [LoginServelet 添加会话信息](#loginservelet-添加会话信息)
- [index.jsp 添加会话判断](#indexjsp-添加会话判断)
- [运行程序](#运行程序)
- [思考](#思考)

## 准备介质

该实验使用 Windows 环境， 安装的介质都需要是 Windows 版本， 在连接中找到对应的版本下载即可。 

 * Eclipse [下载页面](https://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/2021-06/R/eclipse-jee-2021-06-R-win32-x86_64.zip)
* WebLogic 10.3.6 Generic [下载页面](http://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html)
* jdk-7u80-windows-x64 [下载页面](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html#jdk-7u80-oth-JPR)

## 安装介质

1. JDK 安装：

安装 Weblogc 之前需要先安装 JDK， 选择安装路径， 其余配置点击下一步即可，安装过程中会出现两次安装提示：第一次是安装 jdk ，第二次是安装 jre ， 本文安装路径为 D:\Java\jdk 和 D:\Java\jre 。

2. 配置环境变量：

右击“我的电脑” -> "高级" -> "环境变量"

* JAVA_HOME:
在系统变量中新建 JAVA_HOME 环境变量。值设置为 jdk 的安装目录 D:\Java\jdk，Eclipse 通过 JAVA_HOME 变量来找到并使用安装好的 jdk
* Path 
在系统变量里找到Path变量，这是系统自带的，不用新建。双击Path，由于原来的变量值已经存在，在已有的变量后加上“;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin”, 变量之间用分号隔开
* 配置完成后， 可以在命令行窗口(cmd)中输入 java -version 如果配置成功， 会输出对应的 java 版本。

3. Eclipse 安装：

运行安装包， 弹出安装界面， 选择 Eclipse IDE for Enterprise Java Developers， 选则安装路径， 本文路径为 D:\Eclipse\ (目录中最好不要有中文和空格)， 接受安装协议， 点击安装， 等待安装完成

4. WebLogic 安装:

运行安装包， 创建中间件主目录， 本文中路径为 D:\Oracle\Middleware\， 注册安全更新中去掉默认的勾选， 选择 weblogic 默认的配置安装， 之后配置全部下一步， 等待安装完成 

## 创建项目

1. 启动 Eclipse，选择 workspace (工作目录)
2. 点击 File -> New -> Dynamic Web Project
3. 设置 Project name ，本文这里设置为 loginweb
    Target runtime 点击 New Runtime, 选择 Oracle Weblogic Server 
    这里需要下载 Weblogic 的插件，下载完成后点击 Next 
4. 设置刚刚下载的 Weblogic 和 Jdk 目录，设置完成后点击下一步
5. 项目创建完成。

## 创建 index.jsp 文件

在 loginweb/src/mian 下右键 webapp 目录 -> New -> JSP file, 创建 index.jsp 文件, 打开 index.jsp 可以看到:

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

form 中 action 属性规定了当提交表单时，数据要发送到哪里 。 method 属性规定了以什么方式提交表单 get 或 post (默认是 get )。

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
            //  disp.forward(request,  response);
        }else{
                response.sendRedirect("login.jsp");
            //  RequestDispatcher  disp  =  request.getRequestDispatcher("login.jsp");
            //  disp.forward(request,  response);
        }
}
```

## LoginServelet 添加会话信息

如果用户名密码相同， 判定登陆成功， 页面跳转到 index.jsp， 将 user 信息添加到 session 中 。

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

## 运行程序

1. 点击上边菜单栏运行按钮中的 Run As -> Run On Servers  
2. 选择 Oracle WebLogic Server 之后点击 NEXT 
3. 选择域目录， 如果没有创建域， 可以点击后边的创建域目录，设置域目录名称和路径， 点击完成
4. 程序运行后， 可以到本地打开浏览器， 输入本地的 IP 端口和页面名称进行访问， 比如本文中的: http://localhost:7001/index.jsp

## 思考

1. Get 和 Post 的区别
2. Session 和 Cookie 的作用
3. sendRedirect 和 forward 的区别
4. form 怎么更改提交方式