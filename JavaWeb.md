# Java Web

## 1.Getting Started

## 2.开发环境

## 3.控制层Web开发

### Servlet继承体系

**Servlet**

Servlet 类提供了五个方法，其中三个生命周期方法和两个普通方法。

```java
//Servlet的生命周期:从Servlet被创建到Servlet被销毁的过程
//一次创建，到处服务
//一个Servlet只会有一个对象，服务所有的请求
/*
 * 1.实例化（使用构造方法创建对象）
 * 2.初始化  执行init方法
 * 3.服务     执行service方法
 * 4.销毁    执行destroy方法
 */
public class ServletDemo1 implements Servlet {

    //public ServletDemo1(){}

     //生命周期方法:当Servlet第一次被创建对象时执行该方法,该方法在整个生命周期中只执行一次
    public void init(ServletConfig arg0) throws ServletException {
                System.out.println("=======init=========");
        }

    //生命周期方法:对客户端响应的方法,该方法会被执行多次，每次请求该servlet都会执行该方法
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {
        System.out.println("hehe");

    }

    //生命周期方法:当Servlet被销毁时执行该方法
    public void destroy() {
        System.out.println("******destroy**********");
    }
//当停止tomcat时也就销毁的servlet。
    public ServletConfig getServletConfig() {

        return null;
    }

    public String getServletInfo() {

        return null;
    }
}
```

**GenericServlet**

GenericServlet 是一个抽象类，实现了 Servlet 接口，并且对其中的 init() 和 destroy() 和 service() 提供了默认实现。在 GenericServlet 中，主要完成了以下任务：

- 将 init() 中的 ServletConfig 赋给一个类级变量，可以由 getServletConfig 获得；
- 为 Servlet 所有方法提供默认实现；
- 可以直接调用 ServletConfig 中的方法

如果继承这个类的话，我们必须重写 service() 方法来对处理请求。

**HttpServlet**

HttpServlet 也是一个抽象类，它进一步继承并封装了 GenericServlet，使得使用更加简单方便，由于是扩展了 Http 的内容，所以还需要使用 HttpServletRequest 和 HttpServletResponse，这两个类分别是 ServletRequest 和 ServletResponse 的子类。

HttpServlet 中对原始的 Servlet 中的方法都进行了默认的操作，不需要显式的销毁初始化以及 service()，在 HttpServlet 中，自定义了一个新的 service() 方法，其中通过 getMethod() 方法判断请求的类型，从而调用 doGet() 或者 doPost() 处理 get,post 请求，使用者只需要继承 HttpServlet，然后重写 doPost() 或者 doGet() 方法处理请求即可。

### 第一个Servlet程序

通常是继承HttpServlet类，至少要重写doGet方法

```java
package org.mooc;

import java.io.*;

import javax.servlet.*;
import javax.servlet.http.*;

/**
 * Servlet implementation class HelloServlet
 */
public class HelloServlet extends HttpServlet {
	private static final long serialVersionUID = 1L;
       
    /**
     * @see HttpServlet#HttpServlet()
     */
    public HelloServlet() {
        super();
        // TODO Auto-generated constructor stub
    }

	/**
	 * @see HttpServlet#doGet(HttpServletRequest request, HttpServletResponse response)
	 */
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		PrintWriter out = response.getWriter();
		out.println("<html><head><title>HelloWorldServlet</title></head>");
		out.println("<body><h1>Hello World!!!</h1>");
		out.println("</body></html");
		out.close();
	}
}
```

#### 部署Servlet

对于web2.0，需要修改web.xml文件

```xml
<web-app>
	<servlet>
	    <servlet-name>HelloServlet</servlet-name>
    	<servlet-class>org.mooc.HelloServlet</servlet-class>
	</servlet>
	<servlet-mapping>
	    <servlet-name>HelloServlet</servlet-name>
    	<url-pattern>/HelloServlet</url-pattern>
	</servlet-mapping>
</web-app>
```

对于web3.0，只需添加注解

`@WebServlet("/HelloServlet")`



