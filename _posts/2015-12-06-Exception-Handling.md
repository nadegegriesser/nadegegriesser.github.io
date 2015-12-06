---
layout: post
title: Exception Handling with ExceptionMapper
---

The basic REST service has been quite fault tolerant towards incorrect ids until now. Let us add exception handling with an ExceptionMapper.

Assuming the client is not supposed to use unknown ids, we will thow an exception in this case;

```java
public class PersonNotFound extends Exception {

    private static final long serialVersionUID = 9176382068329878558L;

    public PersonNotFound(String id) {
        super("Person " + id + " not found");
    }

}
```

We slightly modify the service interface and implementation to throw this exception for GET and PUT operations.

```java
@Path("/persons")
public interface PersonService {

    @GET
    @Path("/{id}")
    @Produces(APPLICATION_JSON)
    PersonResource get(@PathParam(value = "id") String id) throws PersonNotFound;

    @PUT
    @Path("/{id}")
    @Consumes(APPLICATION_JSON)
    @Produces(APPLICATION_JSON)
    PersonResource update(@PathParam(value = "id") String id, PersonResource person) throws PersonNotFound;

  ...
  
}
```

```java
public class PersonServiceImpl implements PersonService {

    public PersonResource get(String id) throws PersonNotFound {
        PersonResource res = persons.get(id);
        if (res == null) {
            throw new PersonNotFound(id);
        }
        return res;
    }

    public PersonResource update(String id, PersonResource person) throws PersonNotFound {
        if (!persons.containsKey(id)) {
            throw new PersonNotFound(id);
        }
        person.setId(id);
        persons.put(id, person);
        return person;
    }
    
  ...

}
```

If you leave the code like that, the server will return a status code 500 Internal Server Error. 

An ExceptionMapper makes it possible to return a specific status code and body when a specific exception is thrown and the cool thing is, you do not have to change the service implementation for that.

Just implement the ExceptionMapper<T> interface:

```java
public class PersonNotFoundExceptionMapper implements ExceptionMapper<PersonNotFound> {

    @Override
    public Response toResponse(PersonNotFound ex) {
        return Response.status(NOT_FOUND).entity(ex.getMessage()).build();
    }

}
```

and do not forget to add the ExceptionMapper to the list of providers in the Spring configuration :

```xml
<jaxrs:server address="/rest">
    <jaxrs:serviceBeans>
        <ref bean="personService" />
    </jaxrs:serviceBeans>
    <jaxrs:providers>
        <bean class="org.codehaus.jackson.jaxrs.JacksonJaxbJsonProvider" />
        <bean class="de.griesser.rest.exceptions.PersonNotFoundExceptionMapper" />
    </jaxrs:providers>
</jaxrs:server>
```

Now send a GET request to http://localhost:8080/rest/persons/incorrectid

The server will respond with a status code 404 Not Found and a body containing "Person incorrectid not found"

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/2.0.0)
