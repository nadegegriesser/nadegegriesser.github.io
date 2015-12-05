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

The implementations simply uses a map to store the resources.
```java
public class PersonServiceImpl implements PersonService {

    private Map<String, PersonResource> persons;

    public PersonServiceImpl() {
        persons = new HashMap<String, PersonResource>();
    }

    public Collection<PersonResource> getAll() {
        return persons.values();
    }

    public PersonResource get(String id) {
        return persons.get(id);
    }

    public PersonResource create(PersonResource person) {
        String id = UUID.randomUUID().toString();
        person.setId(id);
        persons.put(id, person);
        return person;
    }

    public PersonResource update(String id, PersonResource person) {
        if (persons.containsKey(id)) {
            person.setId(id);
            persons.put(id, person);
            return person;
        }
        return null;
    }

    public void delete(String id) {
        persons.remove(id);
    }

}
```

beans.xml

Spring configuration to publish the service under /rest and be able to serialize / deserialize JSON messages :
```xml
<jaxrs:server address="/rest">
    <jaxrs:serviceBeans>
        <ref bean="personService" />
    </jaxrs:serviceBeans>
    <jaxrs:providers>
        <bean class="org.codehaus.jackson.jaxrs.JacksonJaxbJsonProvider" />
    </jaxrs:providers>
</jaxrs:server>

<bean id="personService" class="de.griesser.rest.services.PersonServiceImpl" />
```

web.xml

Every request is delegated to CXFServlet.
```xml
<web-app>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/beans.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>CXFServlet</servlet-name>
        <display-name>CXF Servlet</display-name>
        <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>CXFServlet</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>

</web-app>
```
