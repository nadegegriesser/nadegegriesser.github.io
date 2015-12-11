---
layout: post
title: Logging with Spring AOP
---

Aspect Oriented Programming allows you to implement transversal functionalities such as logging, monitoring or transaction handling without having to modifiy existing code.

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/2.1.0)

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)
* Log4j 


### pom.xml
First of all we add Spring AOP, Aspectj and Log4j as dependencies.

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.2.3.RELEASE</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.8.7</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.7</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

### Aspect

The aspect is in charge of logging all the service calls.

Before a method call, it will display the method name and the arguments.

After a method call it will display the method name, the arguments, the result and the time taken.

In case of an exception, it will display the method name, the arguments, the exception and the time taken.

It is important to return the result of the proceed method and to rethrow an exception, otherwise it will change the behaviour of your program. This is the dangerous this about aspects.

It is also possible to log the exception - without a try catch block - by using the AfterThrowing advice. But as it does not wrap around the method call, it is not possible to measure the time taken.

Calling joinPoint.getTarget() you can access the target object and use it to get the corresponding logger.

```java
@Aspect
public class LoggingAspect {

    @Around("execution(* de.griesser.rest.services.*.*(..))")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        Logger log = Logger.getLogger(joinPoint.getTarget().getClass());
        String methodAndArguments = getMethodAndArgumentsAsString(joinPoint);
        log.info(methodAndArguments);
        long start = System.currentTimeMillis();
        Object result;
        try {
            result = joinPoint.proceed();
        } catch (Throwable ex) {
            log.error(methodAndArguments + getExceptionAsString(ex, getDuration(start)), ex);
            throw ex;
        }
        log.info(methodAndArguments + getResultAsString(result, getDuration(start)));
        return result;
    }

    protected long getDuration(long start) {
        return System.currentTimeMillis() - start;
    }

    protected String getMethodAndArgumentsAsString(ProceedingJoinPoint joinPoint) {
        return Arrays.stream(joinPoint.getArgs()).map(arg -> arg.toString())
                .collect(Collectors.joining(", ", getMethodName(joinPoint) + "(", ")"));
    }

    protected String getMethodName(ProceedingJoinPoint joinPoint) {
        return MethodSignature.class.cast(joinPoint.getSignature()).getMethod().getName();
    }

    protected String getResultAsString(Object result, long duration) {
        return new StringBuilder(" returned ").append(result).append(" in ").append(duration).append(" msecs")
                .toString();
    }

    protected String getExceptionAsString(Throwable ex, long duration) {
        return new StringBuilder(" threw ").append(ex.getClass().getSimpleName()).append(" after ").append(duration)
                .append(" msecs with message ").append(ex.getMessage()).toString();
    }

}
```

### beans.xml

After that we need to declare the aspect and enable bean autoproxying in the Spring configuration.

```xml
<aop:aspectj-autoproxy />

<bean class="de.griesser.rest.logging.LoggingAspect" />
```

### log4j.properties

The following configuration write logs to a file and to the console.

The format is "[time] [log level] [class name] - [message][eol]"

```xml
# Root logger option
log4j.rootLogger=INFO, file, console

# Direct log messages to a log file
log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.File=${project.build.directory}/${project.build.finalName}.log
log4j.appender.file.MaxFileSize=10MB
log4j.appender.file.MaxBackupIndex=10
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n
 
# Direct log messages to stdout
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Target=System.out
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n
```

### output

This is the output generated for a request to get a resource collection followed by a get request with an incorrect id.

```java
2015-12-11 08:05:37 INFO  PersonServiceImpl - getAll()
2015-12-11 08:05:37 INFO  PersonServiceImpl - getAll() returned [] in 0 msecs
2015-12-11 08:05:58 INFO  PersonServiceImpl - get(incorrectid)
2015-12-11 08:05:58 ERROR PersonServiceImpl - get(incorrectid) threw PersonNotFound after 1 msecs with message Person incorrectid not found
de.griesser.rest.exceptions.PersonNotFound: Person incorrectid not found
	at de.griesser.rest.services.PersonServiceImpl.get(PersonServiceImpl.java:26)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.aop.support.AopUtils.invokeJoinpointUsingReflection(AopUtils.java:302)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.invokeJoinpoint(ReflectiveMethodInvocation.java:190)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:157)
	at org.springframework.aop.aspectj.MethodInvocationProceedingJoinPoint.proceed(MethodInvocationProceedingJoinPoint.java:85)
	at de.griesser.rest.logging.LoggingAspect.logAround(LoggingAspect.java:23)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethodWithGivenArgs(AbstractAspectJAdvice.java:621)
	at org.springframework.aop.aspectj.AbstractAspectJAdvice.invokeAdviceMethod(AbstractAspectJAdvice.java:610)
	at org.springframework.aop.aspectj.AspectJAroundAdvice.invoke(AspectJAroundAdvice.java:68)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.interceptor.ExposeInvocationInterceptor.invoke(ExposeInvocationInterceptor.java:92)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:179)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:208)
	at com.sun.proxy.$Proxy28.get(Unknown Source)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:497)
	at org.apache.cxf.service.invoker.AbstractInvoker.performInvocation(AbstractInvoker.java:180)
	at org.apache.cxf.service.invoker.AbstractInvoker.invoke(AbstractInvoker.java:96)
	at org.apache.cxf.jaxrs.JAXRSInvoker.invoke(JAXRSInvoker.java:200)
	at org.apache.cxf.jaxrs.JAXRSInvoker.invoke(JAXRSInvoker.java:99)
	at org.apache.cxf.interceptor.ServiceInvokerInterceptor$1.run(ServiceInvokerInterceptor.java:59)
	at org.apache.cxf.interceptor.ServiceInvokerInterceptor.handleMessage(ServiceInvokerInterceptor.java:96)
	at org.apache.cxf.phase.PhaseInterceptorChain.doIntercept(PhaseInterceptorChain.java:308)
	at org.apache.cxf.transport.ChainInitiationObserver.onMessage(ChainInitiationObserver.java:121)
	at org.apache.cxf.transport.http.AbstractHTTPDestination.invoke(AbstractHTTPDestination.java:251)
	at org.apache.cxf.transport.servlet.ServletController.invokeDestination(ServletController.java:234)
	at org.apache.cxf.transport.servlet.ServletController.invoke(ServletController.java:208)
	at org.apache.cxf.transport.servlet.ServletController.invoke(ServletController.java:160)
	at org.apache.cxf.transport.servlet.CXFNonSpringServlet.invoke(CXFNonSpringServlet.java:180)
	at org.apache.cxf.transport.servlet.AbstractHTTPServlet.handleRequest(AbstractHTTPServlet.java:293)
	at org.apache.cxf.transport.servlet.AbstractHTTPServlet.doGet(AbstractHTTPServlet.java:217)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:687)
	at org.apache.cxf.transport.servlet.AbstractHTTPServlet.service(AbstractHTTPServlet.java:268)
	at org.eclipse.jetty.servlet.ServletHolder.handle(ServletHolder.java:821)
	at org.eclipse.jetty.servlet.ServletHandler.doHandle(ServletHandler.java:583)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:143)
	at org.eclipse.jetty.security.SecurityHandler.handle(SecurityHandler.java:548)
	at org.eclipse.jetty.server.session.SessionHandler.doHandle(SessionHandler.java:226)
	at org.eclipse.jetty.server.handler.ContextHandler.doHandle(ContextHandler.java:1158)
	at org.eclipse.jetty.servlet.ServletHandler.doScope(ServletHandler.java:511)
	at org.eclipse.jetty.server.session.SessionHandler.doScope(SessionHandler.java:185)
	at org.eclipse.jetty.server.handler.ContextHandler.doScope(ContextHandler.java:1090)
	at org.eclipse.jetty.server.handler.ScopedHandler.handle(ScopedHandler.java:141)
	at org.eclipse.jetty.server.handler.ContextHandlerCollection.handle(ContextHandlerCollection.java:213)
	at org.eclipse.jetty.server.handler.HandlerCollection.handle(HandlerCollection.java:109)
	at org.eclipse.jetty.server.handler.HandlerWrapper.handle(HandlerWrapper.java:119)
	at org.eclipse.jetty.server.Server.handle(Server.java:517)
	at org.eclipse.jetty.server.HttpChannel.handle(HttpChannel.java:308)
	at org.eclipse.jetty.server.HttpConnection.onFillable(HttpConnection.java:242)
	at org.eclipse.jetty.io.AbstractConnection$ReadCallback.succeeded(AbstractConnection.java:261)
	at org.eclipse.jetty.io.FillInterest.fillable(FillInterest.java:95)
	at org.eclipse.jetty.io.SelectChannelEndPoint$2.run(SelectChannelEndPoint.java:75)
	at org.eclipse.jetty.util.thread.strategy.ExecuteProduceConsume.produceAndRun(ExecuteProduceConsume.java:213)
	at org.eclipse.jetty.util.thread.strategy.ExecuteProduceConsume.run(ExecuteProduceConsume.java:147)
	at org.eclipse.jetty.util.thread.QueuedThreadPool.runJob(QueuedThreadPool.java:654)
	at org.eclipse.jetty.util.thread.QueuedThreadPool$3.run(QueuedThreadPool.java:572)
	at java.lang.Thread.run(Thread.java:745)
```
