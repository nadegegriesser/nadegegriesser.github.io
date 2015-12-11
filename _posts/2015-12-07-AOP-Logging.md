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

