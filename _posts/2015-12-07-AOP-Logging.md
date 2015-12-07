---
layout: post
title: Logging with Spring AOP
---

Aspect Oriented Programming allows you to implement transversal functionalities such as logging or transaction handling without having to modifiy existing code.

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/2.1.0)

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)


### pom.xml

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-aop</artifactId>
	<version>4.2.3.RELEASE</version>
	<scope>runtime</scope>
</dependency>
<dependency>
	<groupId>org.aspectj</groupId>
	<artifactId>aspectjtools</artifactId>
	<version>1.0.6</version>
	<scope>runtime</scope>
</dependency>
```
