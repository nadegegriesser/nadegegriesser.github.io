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

### Test Class

```java
package de.griesser.rest.services.integration;

import java.util.Collection;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import de.griesser.rest.exceptions.PersonNotFound;
import de.griesser.rest.resources.PersonResource;
import de.griesser.rest.services.PersonService;

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


