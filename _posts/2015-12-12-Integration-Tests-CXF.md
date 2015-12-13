---
layout: post
title: Integration tests with CXF client
---

One convenient possibility to write integration tests is to use the proxy-based CXF client. It makes it possible to reuse interfaces on the client side.

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/2.2.0)

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)
* Apache CXF 3.1.4 (as Maven dependency)
* JUnit 4.12

### pom.xml

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>4.2.3.RELEASE</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.apache.cxf</groupId>
    <artifactId>cxf-rt-rs-client</artifactId>
    <version>3.1.4</version>
    <scope>test</scope>
</dependency>
```

### Test Client Spring Configuration

We will use dependency injection to set our test client. We need to specify the interface and to register a JSON provider for serialization / deserialization.

```xml
<jaxrs:client id="testClient" address="http://localhost:8080/rest"
        serviceClass="de.griesser.rest.services.PersonService">
        <jaxrs:providers>
            <bean class="org.codehaus.jackson.jaxrs.JacksonJaxbJsonProvider" />
        </jaxrs:providers>
</jaxrs:client>
```


### Test Class

Here is the test scenario :

1. Get all the resources. Initially this should return an empty list.
2. Create a resource.
3. Get all the resources again. This should return a list containing one element : the resource we created previously.
4. Get a single resource by its id. This should return the resource we created in step 2.
5. Delete the resource. To leave the application in the state it was before the tests.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "classpath:test-client.xml" })
public class PersonServiceTest {

    private static final String LAST_NAME = "Griesser";
    private static final String FIRST_NAME = "Nadege";
    @Autowired
    private PersonService sut;

    @Test
    public void test() throws PersonNotFound {
        testInitialGetAll();

        String id = testCreate();

        testGetAllAfterCreate(id);

        testGetAfterCreate(id);

        testDelete(id);
    }

    protected void testInitialGetAll() {
        // given

        // when
        Collection<PersonResource> res = sut.getAll();

        // then
        Assert.assertNotNull(res);
        Assert.assertTrue(res.isEmpty());
    }

    protected String testCreate() {
        // given
        PersonResource person = new PersonResource();
        person.setFirstName(FIRST_NAME);
        person.setLastName(LAST_NAME);

        // when
        person = sut.create(person);

        // then
        Assert.assertNotNull(person);
        Assert.assertNotNull(person.getId());
        Assert.assertEquals(FIRST_NAME, person.getFirstName());
        Assert.assertEquals(LAST_NAME, person.getLastName());
        return person.getId();
    }

    protected void testGetAllAfterCreate(String id) {
        // given

        // when
        Collection<PersonResource> res = sut.getAll();

        // then
        Assert.assertNotNull(res);
        Assert.assertEquals(1, res.size());
        PersonResource person = res.iterator().next();
        checkPerson(person, id);
    }

    protected void testGetAfterCreate(String id) throws PersonNotFound {
        // given

        // when
        PersonResource person = sut.get(id);

        // then
        checkPerson(person, id);
    }

    protected void testDelete(String id) throws PersonNotFound {
        // given

        // when
        sut.delete(id);

        // then
    }

    protected void checkPerson(PersonResource person, String id) {
        Assert.assertNotNull(person);
        Assert.assertEquals(id, person.getId());
        Assert.assertEquals(FIRST_NAME, person.getFirstName());
        Assert.assertEquals(LAST_NAME, person.getLastName());
    }
}
```

### Test Automatisation

This is done in the pom.xml.

We will slightly modify the configuration of the Jetty plugin to start Jetty as a daemon in the pre-integration-test phase and stop it in the post-integration-test phase.

The Surefire plugin will be used to compile and execute the tests. We want these tests to be executed during the integration-test phase and not the test phase. We will use package inclusion / exclusion for that, ignore all tests in the integration package during the test phase (we currently have no other tests, but we will in the future) and consider only the tests in this package during the integration test phase.

```xml
<plugins>
    <plugin>
        <groupId>org.eclipse.jetty</groupId>
        <artifactId>jetty-maven-plugin</artifactId>
        <version>9.3.6.v20151106</version>
        <configuration>
            <stopPort>11079</stopPort>
            <stopKey>stop</stopKey>
        </configuration>
        <executions>
            <execution>
                <id>start-jetty</id>
                <phase>pre-integration-test</phase>
                <goals>
                    <goal>start</goal>
                </goals>
                <configuration>
                    <scanintervalseconds>0</scanintervalseconds>
                    <daemon>true</daemon>
                </configuration>
            </execution>
            <execution>
                <id>stop-jetty</id>
                <phase>post-integration-test</phase>
                <goals>
                    <goal>stop</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.19</version>
        <executions>
            <execution>
                <id>default-test</id>
                <phase>test</phase>
                <goals>
                    <goal>test</goal>
                </goals>
                <configuration>
                    <excludes>
                        <exclude>**/integration/*.java</exclude>
                    </excludes>
                </configuration>
            </execution>
            <execution>
                <id>integration-test</id>
                <phase>integration-test</phase>
                <goals>
                    <goal>test</goal>
                </goals>
                <configuration>
                    <includes>
                        <include>**/integration/*.java</include>
                    </includes>
                </configuration>
            </execution>
        </executions>
    </plugin>
</plugins>
```

### Launch

```sh
mvn clean install
```

### Output

```sh
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building rest 2.2.0-SNAPSHOT
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
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to C:\Users\ngriesser\git\code-samples\rest\target\test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.19:test (default-test) @ rest ---
[INFO] 
[INFO] --- maven-war-plugin:2.2:war (default-war) @ rest ---
[INFO] Packaging webapp
[INFO] Assembling webapp [rest] in [C:\Users\ngriesser\git\code-samples\rest\target\rest]
[INFO] Processing war project
[INFO] Copying webapp resources [C:\Users\ngriesser\git\code-samples\rest\src\main\webapp]
[INFO] Webapp assembled in [187 msecs]
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
[INFO] Logging initialized @6351ms
[INFO] Context path = /
[INFO] Tmp directory = C:\Users\ngriesser\git\code-samples\rest\target\tmp
[INFO] Web defaults = org/eclipse/jetty/webapp/webdefault.xml
[INFO] Web overrides =  none
[INFO] web.xml file = file:///C:/Users/ngriesser/git/code-samples/rest/src/main/webapp/WEB-INF/web.xml
[INFO] Webapp directory = C:\Users\ngriesser\git\code-samples\rest\src\main\webapp
[INFO] jetty-9.3.6.v20151106
[INFO] No Spring WebApplicationInitializer types detected on classpath
[INFO] Initializing Spring root WebApplicationContext
2015-12-13 07:47:40 INFO  ContextLoader - Root WebApplicationContext: initialization started
2015-12-13 07:47:40 INFO  XmlWebApplicationContext - Refreshing Root WebApplicationContext: startup date [Sun Dec 13 07:47:40 CET 2015]; root of context hierarchy
2015-12-13 07:47:41 INFO  XmlBeanDefinitionReader - Loading XML bean definitions from ServletContext resource [/WEB-INF/beans.xml]
Dez 13, 2015 7:47:41 AM org.apache.cxf.endpoint.ServerImpl initDestination
INFORMATION: Setting the server's publish address to be /rest
2015-12-13 07:47:41 INFO  ContextLoader - Root WebApplicationContext: initialization completed in 1109 ms
[INFO] Started o.e.j.m.p.JettyWebAppContext@625d9132{/,file:///C:/Users/ngriesser/git/code-samples/rest/src/main/webapp/,AVAILABLE}{file:///C:/Users/ngriesser/git/code-samples/rest/src/main/webapp/}
[INFO] Started ServerConnector@6db328f8{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}
[INFO] Started @9488ms
[INFO] Started Jetty Server
[INFO] 
[INFO] --- maven-surefire-plugin:2.19:test (integration-test) @ rest ---

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
2015-12-13 07:47:43 INFO  XmlBeanDefinitionReader - Loading XML bean definitions from class path resource [test-client.xml]
2015-12-13 07:47:43 INFO  GenericApplicationContext - Refreshing org.springframework.context.support.GenericApplicationContext@23e028a9: startup date [Sun Dec 13 07:47:43 CET 2015]; root of context hierarchy
2015-12-13 07:47:43 INFO  PersonServiceImpl - getAll()
2015-12-13 07:47:43 INFO  PersonServiceImpl - getAll() returned [] in 0 msecs
2015-12-13 07:47:44 INFO  PersonServiceImpl - create(PersonResource [id=null, lastName=Griesser, firstName=Nadege])
2015-12-13 07:47:44 INFO  PersonServiceImpl - create(PersonResource [id=null, lastName=Griesser, firstName=Nadege]) returned PersonResource [id=5479f146-b159-4476-98ce-19045ee8cf3b, lastName=Griesser, firstName=Nadege] in 0 msecs
2015-12-13 07:47:44 INFO  PersonServiceImpl - getAll()
2015-12-13 07:47:44 INFO  PersonServiceImpl - getAll() returned [PersonResource [id=5479f146-b159-4476-98ce-19045ee8cf3b, lastName=Griesser, firstName=Nadege]] in 0 msecs
2015-12-13 07:47:44 INFO  PersonServiceImpl - get(5479f146-b159-4476-98ce-19045ee8cf3b)
2015-12-13 07:47:44 INFO  PersonServiceImpl - get(5479f146-b159-4476-98ce-19045ee8cf3b) returned PersonResource [id=5479f146-b159-4476-98ce-19045ee8cf3b, lastName=Griesser, firstName=Nadege] in 0 msecs
2015-12-13 07:47:44 INFO  PersonServiceImpl - delete(5479f146-b159-4476-98ce-19045ee8cf3b)
2015-12-13 07:47:44 INFO  PersonServiceImpl - delete(5479f146-b159-4476-98ce-19045ee8cf3b) returned null in 0 msecs
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.297 sec - in de.griesser.rest.services.integration.PersonServiceTest
2015-12-13 07:47:44 INFO  GenericApplicationContext - Closing org.springframework.context.support.GenericApplicationContext@23e028a9: startup date [Sun Dec 13 07:47:43 CET 2015]; root of context hierarchy

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] 
[INFO] --- jetty-maven-plugin:9.3.6.v20151106:stop (stop-jetty) @ rest ---
[INFO] 
[INFO] --- maven-install-plugin:2.4:install (default-install) @ rest ---
[INFO] Stopped ServerConnector@6db328f8{HTTP/1.1,[http/1.1]}{0.0.0.0:8080}
[INFO] Installing C:\Users\ngriesser\git\code-samples\rest\target\rest.war to C:\Users\ngriesser\.m2\repository\de\griesser\rest\2.2.0-SNAPSHOT\rest-2.2.0-SNAPSHOT.war
[INFO] Installing C:\Users\ngriesser\git\code-samples\rest\pom.xml to C:\Users\ngriesser\.m2\repository\de\griesser\rest\2.2.0-SNAPSHOT\rest-2.2.0-SNAPSHOT.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 11.125 s
[INFO] Finished at: 2015-12-13T07:47:45+01:00
[INFO] Final Memory: 35M/458M
[INFO] ------------------------------------------------------------------------
```


