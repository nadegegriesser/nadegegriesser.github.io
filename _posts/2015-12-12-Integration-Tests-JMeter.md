---
layout: post
title: Integration tests with JMeter
---

JMeter is a tool that can be used to perform integration or last tests on REST services. We will configure the same test scenario as with the proxy-based CXF client.

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/2.3.0)

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)
* Apache CXF 3.1.4 (as Maven dependency)
* Apache JMeter 2.13


### JMeter Configuration

The following test plan file has to be saved unter src/test/jmeter.

Here is the test scenario :

1. Get all the resources. Initially this should return an empty list.
2. Create a resource.
3. Get all the resources again. This should return a list containing one element : the resource we created previously.
4. Get a single resource by its id. This should return the resource we created in step 2.
5. Delete the resource. To leave the application in the state it was before the tests.

We will configure some user defined variables : protocol, server, port, path. These will make it possible to run the tests on different environments or will make a change in the URL less painful.

As you can see the test plan is structured like the test scenario.

![Test plan]({{ site.baseurl }}/images/jmeter/01.PNG "Test plan")

The first this you have to do is to configure a thread group. JMeter is initially intended to perform last tests. In this case you may want to change the thread properties.

![Thread group]({{ site.baseurl }}/images/jmeter/02.PNG "Thread group")

### Initial get all 

A simple controller is used to regroup header, HTTP request and assertions. Simply specify a meaningful name.

![Simple controller]({{ site.baseurl }}/images/jmeter/03.PNG "Simple controller")

#### <a name="accept_json"></a>Accept json

A HTTP header manager is used to set the headers that will be sent. These can be set globally or on a specific request like here.

Click the Add button and type name and value.

![HTTP header]({{ site.baseurl }}/images/jmeter/04.PNG "HTTP header")

#### <a name="get_persons"></a>GET /persons

This is the request to get all the resources.

Set a meaningful name, protocol, server, port, method and path. User defined variables have been defined previously for that. Access them with ${var_name}.

![GET /persons]({{ site.baseurl }}/images/jmeter/05.PNG "GET /persons")

#### <a name="status_200"></a>Status code = 200

With assertions it is possible to check the status code that has been returned.

Select Response Code, equals and enter the expected status code.

![Status code assertion]({{ site.baseurl }}/images/jmeter/06.PNG "Status code assertion")

#### Body =  []

It is also possible to check the content of the response. An empty list is expected here.

Select Text response, equals and enter the expected body.

![Empty body list assertion]({{ site.baseurl }}/images/jmeter/07.PNG "Empty body list assertion")

### Create

#### Content-Type json Accept json

Content-Type and Accept headers are set here as in [Accept json](#accept_json).

#### POST /persons

Protocol, server, port and path are set here as in [GET /persons](#get_persons).

Method is set to POST.

First name and last name are specified in the body data.

![POST /persons]({{ site.baseurl }}/images/jmeter/08.PNG "POST /persons")

#### Status code = 200

Similar to [Status code = 200](#status_200).

#### Body = {...}

This assertion checks that an id has been generated and that first name and last name are correct. 

It uses a regular expression, as the id varies (Pattern Matching Rules "Matches").

Writing the assertion that way makes it readable, but not flexible as far as the field order is concerned. One can use @JsonPropertyOrder annotation in the resource class to force a fixed order.

![Body single assertion]({{ site.baseurl }}/images/jmeter/09.PNG "Body single asertion")

#### Id extractor

A regular expression extractor is used to get the id of the new resource.

Reference name is the variable to save the extracted result in.

The regular expression contains one set of parentheses to extract one group.

Template $1$ refers to the first group without performing any transformation.

Match No. 1 matches the first occurence (we only have one).

![Id extractor]({{ site.baseurl }}/images/jmeter/10.PNG "Id extractor")

### Get all after create

#### Accept json

Similar to [Accept json](#accept_json).

#### GET /persons

Similar to [GET /persons](#get_persons).

#### Status code = 200

Similar to [Status code = 200](#status_200).

#### <a name="body"></a>Body = [{...}]

This assertion checks that the list contains exactly one element that matches with the newly generated resource.

As we know which id to expected, we can use a simple Equals check, the id variable in the pattern will be substituted when running the tests.

![Body list assertion]({{ site.baseurl }}/images/jmeter/11.PNG "Body list assertion")

### Get after create

#### Accept json

Similar to [Accept json](#accept_json).

#### GET /persons/{id}

The extracted id is used there to directly access the single resource. It is added at the end of the path with a / separator.

![GET /persons/{id}]({{ site.baseurl }}/images/jmeter/12.PNG "GET /persons/{id}")

#### Status code = 200

Similar to [Status code = 200](#status_200).

#### Body = {...}

Id, first name and last name are checked similar to [Body = [{...}]](#body) but without the array.

### Delete

#### DELETE /persons/{id}

This request deletes the newly created resource.

![DELETE /persons/{id}]({{ site.baseurl }}/images/jmeter/13.PNG "DELETE /persons/{id}")

#### Status code = 204

Similar to [Status code = 200](#status_200) but with 204.

#### View tree result

A view tree result listener nicely displays all the requests that have been fired.

These are marked green or red depending on the assertion result. 

Sampler result, Request and Response data tabs provide detailed information.

![View results tree]({{ site.baseurl }}/images/jmeter/14.PNG "View results tree")

When an assertion fails, the request is marked red and you can expand it to get more details on the failed assertion (expected vs actual result).

For instance when jetty has been stopped :

![View results tree]({{ site.baseurl }}/images/jmeter/15.PNG "View results tree")


### Run the tests manually

Start the web application

```sh
mvn clean install jetty:run-war
```

In JMeter select "Run" > "Start" in the top menu or click the button with a green arrow pointing to the right.


### Test Automatisation

The JMeter plugin will be used to run the tests during the integration-test phase. It will run all the test plans found under src/test/jmeter.

```xml
 <plugin>
    <groupId>com.lazerycode.jmeter</groupId>
    <artifactId>jmeter-maven-plugin</artifactId>
    <version>1.10.1</version>
    <executions>
        <execution>
            <id>jmeter-tests</id>
            <phase>integration-test</phase>
            <goals>
                <goal>jmeter</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Launch

```sh
mvn clean install
```

### Output

```text
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building rest 2.3.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ rest ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ rest ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ rest ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ rest ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-surefire-plugin:2.19:test (default-test) @ rest ---
[INFO] 
[INFO] --- maven-war-plugin:2.2:war (default-war) @ rest ---
[INFO] Packaging webapp
[INFO] Assembling webapp [rest] in [C:\Users\ngriesser\git\code-samples\rest\target\rest]
[INFO] Processing war project
[INFO] Copying webapp resources [C:\Users\ngriesser\git\code-samples\rest\src\main\webapp]
[INFO] Webapp assembled in [152 msecs]
[INFO] Building war: C:\Users\ngriesser\git\code-samples\rest\target\rest.war
[INFO] WEB-INF\web.xml already added, skipping
[INFO] 
[INFO] >>> jetty-maven-plugin:9.3.6.v20151106:start (start-jetty) > validate @ rest >>>
[INFO] 
[INFO] <<< jetty-maven-plugin:9.3.6.v20151106:start (start-jetty) < validate @ rest <<<
[INFO] 
[INFO] --- jetty-maven-plugin:9.3.6.v20151106:start (start-jetty) @ rest ---
[INFO] Configuring Jetty for project: rest
[INFO] webAppSourceDirectory not set. Trying src\main\webapp
[INFO] Reload Mechanic: automatic
[INFO] Classes = C:\Users\ngriesser\git\code-samples\rest\target\classes
[INFO] Logging initialized @4775ms
[INFO] Context path = /
[INFO] Tmp directory = C:\Users\ngriesser\git\code-samples\rest\target\tmp
[INFO] Web defaults = org/eclipse/jetty/webapp/webdefault.xml
[INFO] Web overrides =  none
[INFO] web.xml file = file:///C:/Users/ngriesser/git/code-samples/rest/src/main/webapp/WEB-INF/web.xml
[INFO] Webapp directory = C:\Users\ngriesser\git\code-samples\rest\src\main\webapp
[INFO] jetty-9.3.6.v20151106
[INFO] No Spring WebApplicationInitializer types detected on classpath
[INFO] Initializing Spring root WebApplicationContext
2015-12-15 08:34:03 INFO  ContextLoader - Root WebApplicationContext: initialization started
2015-12-15 08:34:03 INFO  XmlWebApplicationContext - Refreshing Root WebApplicationContext: startup date [Tue Dec 15 08:34:03 CET 2015]; root of context hierarchy
2015-12-15 08:34:03 INFO  XmlBeanDefinitionReader - Loading XML bean definitions from ServletContext resource [/WEB-INF/beans.xml]
Dez 15, 2015 8:34:04 AM org.apache.cxf.endpoint.ServerImpl initDestination
INFORMATION: Setting the server's publish address to be /rest
2015-12-15 08:34:04 INFO  ContextLoader - Root WebApplicationContext: initialization completed in 1014 ms
[INFO] Started o.e.j.m.p.JettyWebAppContext@5a00eb1e{/,file:///C:/Users/ngriesser/git/code-samples/rest/src/main/webapp/,AVAILABLE}{file:///C:/Users/ngriesser/git/code-samples/rest/src/main/webapp/}
[INFO] Started ServerConnector@6ecc02bb{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}
[INFO] Started @8366ms
[INFO] Started Jetty Server
[INFO] 
[INFO] --- jmeter-maven-plugin:1.10.1:jmeter (jmeter-tests) @ rest ---
[INFO]  
[INFO] -------------------------------------------------------
[INFO]  P E R F O R M A N C E    T E S T S
[INFO] -------------------------------------------------------
[INFO]  
[INFO]  
[info]  
[debug] JMeter is called with the following command line arguments: -n -t C:\Users\ngriesser\git\code-samples\rest\src\test\jmeter\REST-Service-Testplan.jmx -l C:\Users\ngriesser\git\code-samples\rest\target\jmeter\results\20151215-REST-Service-Testplan.jtl -d C:\Users\ngriesser\git\code-samples\rest\target\jmeter -j C:\Users\ngriesser\git\code-samples\rest\target\jmeter\logs\REST-Service-Testplan.jmx.log
[info] Executing test: REST-Service-Testplan.jmx
[debug] Creating summariser <summary>
[debug] Created the tree successfully using C:\Users\ngriesser\git\code-samples\rest\src\test\jmeter\REST-Service-Testplan.jmx
[debug] Starting the test @ Tue Dec 15 08:34:09 CET 2015 (1450164849874)
[debug] Waiting for possible shutdown message on port 4445
2015-12-15 08:34:10 INFO  PersonServiceImpl - getAll()
2015-12-15 08:34:10 INFO  PersonServiceImpl - getAll() returned [] in 0 msecs
2015-12-15 08:34:10 INFO  PersonServiceImpl - create(PersonResource [id=null, lastName=Griesser, firstName=Nadege])
2015-12-15 08:34:10 INFO  PersonServiceImpl - create(PersonResource [id=null, lastName=Griesser, firstName=Nadege]) returned PersonResource [id=be5816e6-c6c2-459e-ac51-28830cf7e4ce, lastName=Griesser, firstName=Nadege] in 0 msecs
2015-12-15 08:34:10 INFO  PersonServiceImpl - getAll()
2015-12-15 08:34:10 INFO  PersonServiceImpl - getAll() returned [PersonResource [id=be5816e6-c6c2-459e-ac51-28830cf7e4ce, lastName=Griesser, firstName=Nadege]] in 0 msecs
2015-12-15 08:34:10 INFO  PersonServiceImpl - get(be5816e6-c6c2-459e-ac51-28830cf7e4ce)
2015-12-15 08:34:10 INFO  PersonServiceImpl - get(be5816e6-c6c2-459e-ac51-28830cf7e4ce) returned PersonResource [id=be5816e6-c6c2-459e-ac51-28830cf7e4ce, lastName=Griesser, firstName=Nadege] in 0 msecs
2015-12-15 08:34:10 INFO  PersonServiceImpl - delete(be5816e6-c6c2-459e-ac51-28830cf7e4ce)
2015-12-15 08:34:10 INFO  PersonServiceImpl - delete(be5816e6-c6c2-459e-ac51-28830cf7e4ce) returned null in 0 msecs
[debug] summary =      5 in   0,5s =   10,4/s Avg:    27 Min:     4 Max:    66 Err:     0 (0,00%)
[debug] Tidying up ...    @ Tue Dec 15 08:34:10 CET 2015 (1450164850421)
[debug] ... end of run
[info] Completed Test: REST-Service-Testplan.jmx
[INFO]  
[INFO] Test Results:
[INFO]  
[INFO] Tests Run: 1, Failures: 0
[INFO]  
[INFO] 
[INFO] --- jetty-maven-plugin:9.3.6.v20151106:stop (stop-jetty) @ rest ---
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ rest ---
[INFO] Stopped ServerConnector@6ecc02bb{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}
[INFO] Installing C:\Users\ngriesser\git\code-samples\rest\target\rest.war to C:\Users\ngriesser\.m2\repository\de\griesser\rest\2.3.0-SNAPSHOT\rest-2.3.0-SNAPSHOT.war
[INFO] Installing C:\Users\ngriesser\git\code-samples\rest\pom.xml to C:\Users\ngriesser\.m2\repository\de\griesser\rest\2.3.0-SNAPSHOT\rest-2.3.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 13.687 s
[INFO] Finished at: 2015-12-15T08:34:11+01:00
[INFO] Final Memory: 32M/426M
[INFO] ------------------------------------------------------------------------
```


