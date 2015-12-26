---
layout: post
title: Persistence layer with Spring Data JPA, EclipseLink and Derby 
---

Let's split data and logic layers. The data layer will be configured using Spring Data JPA which provides generic DAO implementation, EclipseLink as a reference implementation for the Java Persistence API and an embedded database Derby. I said configured, because there is almost no implementation.

[The complete source code is available here.](https://github.com/nadegegriesser/code-samples/tree/3.0.1)

Technologies used :

* Java SE Development Kit 8u66
* Eclipse IDE for Java Developers Version: Mars.1 Release (4.5.1)
* Maven 3.3.3 (comes with Eclipse)
* Spring Framework 4.2.3 (as Maven dependency)
* Spring Data JPA 1.9.2.RELEASE (as Maven dependency)
* EclipseLink 2.6.2 (as Maven dependency)
* Apache Derby 10.12.1.1 (as Maven dependency)


### pom.xml

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>4.2.3.RELEASE</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.2.3.RELEASE</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.2.3.RELEASE</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>1.9.2.RELEASE</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.eclipse.persistence</groupId>
    <artifactId>javax.persistence</artifactId>
    <version>2.1.1</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.eclipse.persistence</groupId>
    <artifactId>org.eclipse.persistence.jpa</artifactId>
    <version>2.6.2</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.apache.derby</groupId>
    <artifactId>derby</artifactId>
    <version>10.12.1.1</version>
    <scope>runtime</scope>
</dependency>
```

### Entity

The entity is quite similar to the resource, except that the id is a Long. We will let the database take care of generating the numeric id.

```java
@Entity
public class PersonEntity implements Serializable {

    private static final long serialVersionUID = 4382376373655477269L;

    @Id
    @GeneratedValue
    private Long id;
    private String lastName;
    private String firstName;

    protected PersonEntity() {
    }

    public PersonEntity(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
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

}
```

### Repository

Thanks to Spring Data JPA there is no need to implement the repository. All you need is to extend the CrudRepository interface and that's it : your repository can already find, save and delete entities. 

```java
@Transactional
@Repository
public interface PersonRepository extends CrudRepository<PersonEntity, Long> {
}
```

Following naming conventions, you could also add query methods to look for entities with specific field values. For example :

```java
List<PersonEntity> findByLastName(String lastName);
```


### Persistence unit

The following persistence unit will use EclipseLink for Object Relational Mapping and a in memory Derby database. The Derby database will be created under target/databases and we will let EclipseLink create the schema.

Setting exclude-unlisted-classes to false, will include all the classes annotated with @Entity.

Weaving is turned off, as we do not needed lazy loading, change tracking, fetch groups, and internal optimizations for the start.

To see the sql statements generated, we set eclipselink.logging.level.sql to FINE.

```xml
<persistence-unit name="persistenceUnit" transaction-type="RESOURCE_LOCAL">
    <provider>org.eclipse.persistence.jpa.PersistenceProvider</provider>
    <exclude-unlisted-classes>false</exclude-unlisted-classes>
    <properties>
        <property name="javax.persistence.jdbc.driver" value="org.apache.derby.jdbc.EmbeddedDriver" />
        <property name="javax.persistence.jdbc.url"
            value="jdbc:derby:${project.build.directory}/databases/db;create=true" />
        <property name="eclipselink.ddl-generation" value="create-tables" />
        <property name="eclipselink.ddl-generation.output-mode" value="database" />
        <property name="eclipselink.weaving" value="false" />
        <property name="eclipselink.logging.level.sql" value="FINE" />
    </properties>
</persistence-unit>
```

Normally this file is saved under src/main/resources/META-INF/persistence.xml but we will save it as src/main/resources/META-INF/derby-persistence.xml and enable resource filtering in the pom.xml to get the path for the database set.

### Spring configuration

The following spring configuration will enable annotation based configuration on addition to the XML configuration, create an entity manager factory using the persistence unit we previously created, create a transaction manager and enable annotation driven transaction management, create the repositories annotated with @Repository within the base-package.

```xml
<context:annotation-config />

<bean id="entityManagerFactory"
    class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    <property name="persistenceXmlLocation"
        value="classpath:META-INF/derby-persistence.xml" />
</bean>

<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="entityManagerFactory" />
</bean>

<tx:annotation-driven />

<jpa:repositories base-package="de.griesser.persistence.repositories" />
```

