---
layout: post
title: "Using Grails to throw-up a simple interface over JPA entities"
date: 2012-03-29 18:25
comments: true
categories:
- Kasabi
- JPA
- Grails
---

Over the last few weeks, I've been wanting to write a demo for
[Kasabi](http://kasabi.com) to show how to integrate data from a database using
object-relational mapping. The idea was that data in an existing database would
be polled and loaded in an ETL fashion. However, in order to build a compelling
demo, I wanted to generate a simple CRUD over my existing domain classes.

<!-- more -->

The scenario
------------

My entity classes followed a DB schema based on the classic suppliers-parts
database used by C. J. Date in his database textbooks.

The suppliers-parts database contains information about a business'
**Customers** and **Suppliers**, each of these parties will have an **Address**,
which in the relational model is decomposed into a separate database table. In
addition, our database will have a catalogue of **Parts** and a mapping
indicating which **Supplier** supplies which **Part**. Furthermore,
**Customers** will be able to place an **Order**, where an **Order** is an order
of a certain quantity of **Parts**, sourced from a given **Supplier**.

The datamodel
-------------

                                +---------+
        +--------------+        | Country |  +-----------+
        |   Address    |  *..1  |---------|  | Customer  |
        |--------------+--------+ name    |  |-----------|
        |              |        +---------+  | title     | 1..*
        | addressline1 |                     | firstname +------+
        | addressline2 |         1..*        | surname   |      |
        | addressline3 +---------------------+           |      |
        |              |                     +-----------+      |
        | city         |                                        |
        | postcode     |                            +-----------+---+
        |              |      +-----------+  *..1   | CustomerOrder |
        +---+----------+      | OrderItem +---------+---------------|
            |                 |-----------|         |               |
            |           1..*  |           |         +---------------+
            |          +------+ quanity   +-------+
            |          |      |           |  1..* |
            |          |      +-----------+       |
            |          |                    +-----+--+
            |       +--+-------+            | Part   |
            |       | Supplier |            |--------|
            |       |----------|            |        |
            |  1..* |          |    *..*    | name   |
            +-------+ name     +------------+ color  |
                    |          |            | weight |
                    +----------+            +--------+

JPA Entities
------------

It is assumed, for the purposes of this demo, that JPA entity classes already
exist for the domain. The easiest way to create entity classes is using the
JPA annotations. E.g.

```java  Customer.java
@Entity
public class Customer extends AuditableEntityBase {

    private String title;
            
    @NotNull
    private String firstname;

    @NotNull
    private String surname;

    @ManyToOne
    private Address address;

    ...
    
}
```

All of my classes have an inherited Id and Version field. These are
defined in **EntityBase**.

```java EntityBase.java
@MappedSuperclass
public class EntityBase {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column(name = "id")
    private Long id;
    
    @Version
    @Column(name = "version")
    private Integer version;
    
    ...
}
```

Furthermore, a number of classes also extend from **AuditableEntityBase** in
order to timestamped audit information. Automatic event handlers control the
capture of timestamp fields that indicate when a record was created or last
modified.

```java AuditableEntityBase.java
@MappedSuperclass
public class AuditableEntityBase extends EntityBase {

    @Temporal(javax.persistence.TemporalType.TIMESTAMP)
    @Column(name = "created")
    private Date dateTimeCreated;
    
    @Temporal(javax.persistence.TemporalType.TIMESTAMP)
    @Column(name = "modified")
    private Date dateTimeModified;
    
    ...
    
    @PreUpdate
    @PrePersist
    public void updateTimeStamps() {
        dateTimeModified = new Date();
        if (dateTimeCreated==null) {
            dateTimeCreated = new Date();
        }
    }
}
```

The entity classes for the suppliers-parts domain model are in a simple java
project, packaged up using maven2. See - [link...]()

Build and install the packages use maven

    mvn clean install


Creating a simple webapp using grails
-------------------------------------

The whole point of this demo is to show how relational data can be extracted
from an existing database. In reality, this existing database would be
continually updated by existing systems. And for the purposes of a demo, it
would be interesting to model this situation. In the following steps, we will
create a simple web-based CRUD application that allows you to create and update
records. We need something that will get us a working system with the bare
minimum of effort, so we're using SpringSource Grails to build a simple webapp
infront of our JPA database.

For the database we'll used a file-based H2 database, although in reality you
would probably be using a production-strength database.

###Create the application###

First, create a new grails app

    grails create-app suppliers-parts-webapp
    
Next, we need to bring our JPA classes into the project. The JPA classes are
a maven project, so as long as we've installed them into our local maven repo (
by running {{mvn install}} in the suppliers-parts-domain project),
we can pull these in via a dependency:

In the BuildConfig.groovy configuration, uncomment the
mavenLocal() repo. And add the following, new entry to the dependencies section:

``` plain grails-app/conf/BuildConfig.groovy
dependencies {
    compile "com.kasabi.demo.suppliers-parts:domain-model:0-1-3-SNAPSHOT"
}
```

Update the database to use a file-based development database (rather than the
default, in-mem db) in grails-app/conf/DataSource.groovy

``` plain grails-app/conf/DataSource.groovy
development {
  dataSource {
    dbCreate = "update"
    url = "jdbc:h2:file:/tmp/suppliersparts;MVCC=TRUE;AUTO_SERVER=TRUE"
  }
}
```    
Tell grails to use the JPA entity manager, and inform hibernate (Grails uses
Hibernate as its persistence implementation) about each entity class you want
to work with.

``` xml grails-app/conf/hibernate/hibernate.cfg.xml
<!DOCTYPE hibernate-configuration SYSTEM
  "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <mapping package="com.kasabi.demo.suppliersparts.domain" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.EntityBase" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.AuditableEntityBase" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.Part" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.Address" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.Country" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.Customer" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.CustomerOrder" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.Supplier" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.OrderItem" />
        <mapping class="com.kasabi.demo.suppliersparts.domain.Title" />
    </session-factory>
</hibernate-configuration>
```

Grails will generate scaffolding views and controllers for each entity class.
This is achieved using:

    grails generate-all <CLASS_NAME>
    
You'll now have all ui pages and web controllers for each entity. There seems
to be a bug with handling the audit and id fields (which aren't needed in the
web forms) so the following script (regenerateScaffolding.sh) has some manual
'sed' commands to remove these extra fields.

```bash generate-all.bash
#!/bin/bash

echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.Part 
echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.Address 
echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.Country 
echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.Customer
echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.CustomerOrder
echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.Supplier
echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.OrderItem
echo "a" | grails generate-all com.kasabi.demo.suppliersparts.domain.Title

for i in `find . -name _form.gsp`
  do sed -i -e "/field: 'id', 'error')}/,+6d" $i
     sed -i -e "/field: 'dateTimeModified', 'error')}/,+6d" $i
     sed -i -e "/field: 'dateTimeCreated', 'error')}/,+6d" $i
done
```

Finally, we need to configure Grails to process the audit fields. To do this, we
need to add a Hibernate event listener and register it. For those interested...
the code is [here].

###Run the application###

From the web-app project, it should be simple to start the app on port 8080
with the command:

    grails run-war
