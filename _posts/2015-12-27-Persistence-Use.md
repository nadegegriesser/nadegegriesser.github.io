---
layout: post
title: Use of persistence layer 
---

Now we can replace the map used in the rest layer with the new persistence implementation.

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/3.0.1)

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)


### pom.xml

Add the persistence module as a dependency to the rest module.

```xml
<dependency>
    <groupId>de.griesser</groupId>
    <artifactId>persistence</artifactId>
    <version>${project.version}</version>
</dependency>
```

### Service implementation

We leave the service interface unchanged. So we can the the integration tests defined previously.

The map is replaced by the repository from the persistence module. 

As Spring Data JPA is used to create the repository implementation and the repository is not configured explicitely in the Spring configuration, we use autowiring to habe the repository injected.

As the repository works with an id of type Long, we have to convert the input which is of type String. We will take care of potential NumberFormatExceptions using an ExceptionMapper.

```java
public class PersonServiceImpl implements PersonService {

    @Autowired(required = true)
    private PersonRepository repository;

    public void setRepository(PersonRepository repository) {
        this.repository = repository;
    }

    public Collection<PersonResource> getAll() {
        Collection<PersonResource> res = new ArrayList<>();
        for (PersonEntity entity : repository.findAll()) {
            res.add(new PersonResource(entity));
        }
        return res;
    }

    public PersonResource get(String id) throws PersonNotFound {
        PersonEntity entity = repository.findOne(Long.parseLong(id));
        if (entity == null) {
            throw new PersonNotFound(id);
        }
        return new PersonResource(entity);
    }

    public PersonResource create(PersonResource person) {
        PersonEntity entity = repository.save(new PersonEntity(person.getFirstName(), person.getLastName()));
        return new PersonResource(entity);
    }

    public PersonResource update(String id, PersonResource person) throws PersonNotFound {
        PersonEntity entity = repository.findOne(Long.parseLong(id));
        if (entity == null) {
            throw new PersonNotFound(id);
        }
        entity.setFirstName(person.getFirstName());
        entity.setLastName(person.getLastName());
        entity = repository.save(entity);
        return new PersonResource(entity);
    }

    public void delete(String id) {
        repository.delete(Long.parseLong(id));
    }

}
```

### ExceptionMapper

```java
public class NumberFormatExceptionMapper implements ExceptionMapper<NumberFormatException> {

    @Override
    public Response toResponse(NumberFormatException ex) {
        return Response.status(BAD_REQUEST).entity("NumberFormatException " + ex.getMessage()).build();
    }

}
```

### Spring configuration

We register the new ExceptionMapper in the Spring configuration.

```xml
<jaxrs:server address="/rest">
    <jaxrs:serviceBeans>
        <ref bean="personService" />
    </jaxrs:serviceBeans>
    <jaxrs:providers>
        <bean class="org.codehaus.jackson.jaxrs.JacksonJaxbJsonProvider" />
        <bean class="de.griesser.rest.exceptions.PersonNotFoundExceptionMapper" />
        <bean class="de.griesser.rest.exceptions.NumberFormatExceptionMapper" />
    </jaxrs:providers>
</jaxrs:server>
```
