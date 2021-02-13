# ERROR NOTES

## 一、版本问题

## 1.1、java.lang.NoSuchMethodError: javax.servlet.http.HttpServletResponse.getStatus()I

检查 SpringMVC 与 Jetty 容器的版本兼容问题

```java
java.lang.NoSuchMethodError: javax.servlet.http.HttpServletResponse.getStatus()I
	at org.springframework.web.servlet.FrameworkServlet.publishRequestHandledEvent(FrameworkServlet.java:1145)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1022)
	at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:897)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:707)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:882)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:820)
	at org.mortbay.jetty.servlet.ServletHolder.handle(ServletHolder.java:487)
	at org.mortbay.jetty.servlet.ServletHandler.handle(ServletHandler.java:362)
	at org.mortbay.jetty.security.SecurityHandler.handle(SecurityHandler.java:216)
	at org.mortbay.jetty.servlet.SessionHandler.handle(SessionHandler.java:181)
	at org.mortbay.jetty.handler.ContextHandler.handle(ContextHandler.java:712)
	at org.mortbay.jetty.webapp.WebAppContext.handle(WebAppContext.java:405)
	at org.mortbay.jetty.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:211)
	at org.mortbay.jetty.handler.HandlerCollection.handle(HandlerCollection.java:114)
	at org.mortbay.jetty.handler.HandlerWrapper.handle(HandlerWrapper.java:139)
	at org.mortbay.jetty.Server.handle(Server.java:313)
	at org.mortbay.jetty.HttpConnection.handleRequest(HttpConnection.java:506)
	at org.mortbay.jetty.HttpConnection$RequestHandler.headerComplete(HttpConnection.java:830)
	at org.mortbay.jetty.HttpParser.parseNext(HttpParser.java:514)
	at org.mortbay.jetty.HttpParser.parseAvailable(HttpParser.java:211)
	at org.mortbay.jetty.HttpConnection.handle(HttpConnection.java:381)
	at org.mortbay.io.nio.SelectChannelEndPoint.run(SelectChannelEndPoint.java:396)
	at org.mortbay.thread.BoundedThreadPool$PoolThread.run(BoundedThreadPool.java:442)
```

