---
layout: post
title: How to override a status code set by CXF using a ContainerResponseFilter and name binding
---

The correct status code to return on successful resource creation is 201 Created. In this case CXF will set the response status code to 200 OK. There are many ways to override this value. One possibility is to return an instance of the Response class, so you can set headers, status and body manually. But this implies a change in the service API. We will resort to another possibility that leaves the API unchanged. 

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/4.0.0)

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)
* Apache CXF 3.1.4 (as Maven dependency)
* Apache JMeter 2.13


### NameBinding

We define a NameBinding annotation called StatusCode. It will be used to decorate our filter and the resource method to which the provider should be bound to, so that we do not affect other methods.

```java
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(value = RetentionPolicy.RUNTIME)
@NameBinding
public @interface StatusCode {
}
```

### Service interface

We decorate the resource method create with @StatusCode.

```java
@Path("/persons")
public interface PersonService {

    ...

    @POST
    @Consumes(APPLICATION_JSON)
    @Produces(APPLICATION_JSON)
    @StatusCode
    PersonResource create(PersonResource person);

   ...

}
```

### ContainerResponseFilter

In case of success (200 OK) the filter will turn the status code into 201 Created.

We decorate the filter with @StatusCode.

```java
@StatusCode
public class StatusCodeFilter implements ContainerResponseFilter {

    @Override
    public void filter(ContainerRequestContext reqCtx, ContainerResponseContext respCtx) throws IOException {
        if (respCtx.getStatus() == Status.OK.getStatusCode()) {
            respCtx.setStatus(Status.CREATED.getStatusCode());
        }
    }

}
```


### Spring Configuration

We register the new filter.

```xml
<jaxrs:server address="/rest">
    <jaxrs:serviceBeans>
        <ref bean="personService" />
    </jaxrs:serviceBeans>
    <jaxrs:providers>
        <bean class="org.codehaus.jackson.jaxrs.JacksonJaxbJsonProvider" />
        <bean class="de.griesser.rest.exceptions.PersonNotFoundExceptionMapper" />
        <bean class="de.griesser.rest.exceptions.NumberFormatExceptionMapper" />
        <bean class="de.griesser.rest.filters.StatusCodeFilter" />
    </jaxrs:providers>
</jaxrs:server>
```

### Launch

In the parent module run :

```sh
mvn clean install
```

### Output

One JMeter test should fail, as it expects a 200 on successful resource creation.

```text
[INFO] --- jmeter-maven-plugin:1.10.1:jmeter (jmeter-tests) @ rest ---
[INFO]  
[INFO] -------------------------------------------------------
[INFO]  P E R F O R M A N C E    T E S T S
[INFO] -------------------------------------------------------
[INFO]  
[INFO]  
[info]  
[debug] JMeter is called with the following command line arguments: -n -t C:\Users\ngriesser\git\code-samples\parent\rest\src\test\jmeter\REST-Service-Testplan.jmx -l C:\Users\ngriesser\git\code-samples\parent\rest\target\jmeter\results\20151229-REST-Service-Testplan.jtl -d C:\Users\ngriesser\git\code-samples\parent\rest\target\jmeter -j C:\Users\ngriesser\git\code-samples\parent\rest\target\jmeter\logs\REST-Service-Testplan.jmx.log
[info] Executing test: REST-Service-Testplan.jmx
[debug] Creating summariser <summary>
[debug] Created the tree successfully using C:\Users\ngriesser\git\code-samples\parent\rest\src\test\jmeter\REST-Service-Testplan.jmx
[debug] Starting the test @ Tue Dec 29 12:10:16 CET 2015 (1451387416809)
[debug] Waiting for possible shutdown message on port 4445
2015-12-29 12:10:17 INFO  PersonServiceImpl - getAll()
[EL Fine]: sql: 2015-12-29 12:10:17.236--ServerSession(477700359)--Connection(1122980374)--SELECT ID, FIRSTNAME, LASTNAME FROM PERSONENTITY
2015-12-29 12:10:17 INFO  PersonServiceImpl - getAll() returned [] in 0 msecs
2015-12-29 12:10:17 INFO  PersonServiceImpl - create(PersonResource [id=null, lastName=Griesser, firstName=Nadege])
[EL Fine]: sql: 2015-12-29 12:10:17.252--ClientSession(248805497)--Connection(1122980374)--INSERT INTO PERSONENTITY (ID, FIRSTNAME, LASTNAME) VALUES (?, ?, ?)
	bind => [3 parameters bound]
2015-12-29 12:10:17 INFO  PersonServiceImpl - create(PersonResource [id=null, lastName=Griesser, firstName=Nadege]) returned PersonResource [id=2, lastName=Griesser, firstName=Nadege] in 0 msecs
2015-12-29 12:10:17 INFO  PersonServiceImpl - getAll()
[EL Fine]: sql: 2015-12-29 12:10:17.267--ServerSession(477700359)--Connection(1122980374)--SELECT ID, FIRSTNAME, LASTNAME FROM PERSONENTITY
2015-12-29 12:10:17 INFO  PersonServiceImpl - getAll() returned [PersonResource [id=2, lastName=Griesser, firstName=Nadege]] in 0 msecs
2015-12-29 12:10:17 INFO  PersonServiceImpl - get(2)
2015-12-29 12:10:17 INFO  PersonServiceImpl - get(2) returned PersonResource [id=2, lastName=Griesser, firstName=Nadege] in 0 msecs
2015-12-29 12:10:17 INFO  PersonServiceImpl - delete(2)
[EL Fine]: sql: 2015-12-29 12:10:17.392--ClientSession(1676173917)--Connection(1122980374)--DELETE FROM PERSONENTITY WHERE (ID = ?)
	bind => [1 parameter bound]
2015-12-29 12:10:17 INFO  PersonServiceImpl - delete(2) returned null in 0 msecs
[debug] summary =      5 in     1s =    9,2/s Avg:    37 Min:     6 Max:   116 Err:     1 (20,00%)
[debug] Tidying up ...    @ Tue Dec 29 12:10:17 CET 2015 (1451387417392)
[debug] ... end of run
[info] Completed Test: REST-Service-Testplan.jmx
[INFO]  
[INFO] Test Results:
[INFO]  
[INFO] Tests Run: 1, Failures: 1
[INFO]  
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO] 
[INFO] persistence ........................................ SUCCESS [  2.600 s]
[INFO] rest ............................................... FAILURE [ 58.058 s]
[INFO] parent ............................................. SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:00 min
[INFO] Finished at: 2015-12-29T12:10:17+01:00
[INFO] Final Memory: 59M/715M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal com.lazerycode.jmeter:jmeter-maven-plugin:1.10.1:jmeter (jmeter-tests) on project rest: There were 1 test failures.  See the JMeter logs at 'C:\Users\ngriesser\git\code-samples\parent\rest\target\jmeter\logs' for details. -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :rest
```

In the parent module run :

```sh
mvn jetty:run-war
```

Start JMeter, open the test plan and run the tests.

![JMeter View Result Tree]({{ site.baseurl }}/images/jmeter-statuscode/01.PNG "JMeter View Result Tree")

Adapt the assertion to expect 201 instead of 200.

The tests should all be successful now.
