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
* JMeter

### pom.xml

```xml
```




### JMeter Configuration

Here is the test scenario :

1. Get all the resources. Initially this should return an empty list.
2. Create a resource.
3. Get all the resources again. This should return a list containing one element : the resource we created previously.
4. Get a single resource by its id. This should return the resource we created in step 2.
5. Delete the resource. To leave the application in the state it was before the tests.

```java

```

### Test Automatisation

This is done in the pom.xml.

We will slightly modify the configuration of the Jetty plugin to start Jetty as a daemon in the pre-integration-test phase and stop it in the post-integration-test phase.

The Surefire plugin will be used to compile and execute the tests. We want these tests to be executed during the integration-test phase and not the test phase. We will use package inclusion / exclusion for that, ignore all tests in the integration package during the test phase (we currently have no other tests, but we will in the future) and consider only the tests in this package during the integration test phase.

```xml

```

### Launch

```sh
mvn clean install
```

### Output

```text


```


