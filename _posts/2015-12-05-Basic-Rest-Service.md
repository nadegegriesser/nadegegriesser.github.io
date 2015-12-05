---
layout: post
title: Basic Rest Service with Apache CXF
---

Let's build a basic rest service, that wil serve as a basis for all the following posts.

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)
* Apache CXF 3.1.4 (as Maven dependency)

The fundamental concept in REST is the resource. Let us start with that :

```java
public class PersonResource {

    private String id;
    private String lastName;
    private String firstName;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    @Override
    public String toString() {
        return "PersonResource [id=" + id + ", lastName=" + lastName + ", firstName=" + firstName + "]";
    }

}
```

Now we would like to do something with these resources, for example typical CRUD operations: list all the resources, return a single resource, create a resource, update or delete it...
One good point of CXF is that you can define all the REST relevant annotations on you interface.
```java
@Path("/persons")
public interface PersonService {

    @GET
    @Produces(APPLICATION_JSON)
    Collection<PersonResource> getAll();

    @GET
    @Path("/{id}")
    @Produces(APPLICATION_JSON)
    PersonResource get(@PathParam(value = "id") String id);

    @POST
    @Consumes(APPLICATION_JSON)
    @Produces(APPLICATION_JSON)
    PersonResource create(PersonResource person);

    @PUT
    @Path("/{id}")
    @Consumes(APPLICATION_JSON)
    @Produces(APPLICATION_JSON)
    PersonResource update(@PathParam(value = "id") String id, PersonResource person);

    @DELETE
    @Path("/{id}")
    void delete(@PathParam(value = "id") String id);

}
```
