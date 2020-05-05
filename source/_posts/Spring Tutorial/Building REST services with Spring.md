---
title: Building REST services with Spring

categories:
- Spring Tutorial

date: 2020-04-29
---
REST has quickly become the de-facto (事实上) standard for building web services on the web because they're easy to build and easy to consume (消费).

There're much larger discussion to be had about how REST fits in the world of microservices, but for this tutorial - let's just look at building RESTful services.

Why REST? REST embraces (拥抱) the precepts (方案) of the web, including its architecture (架构), benefits (优势), and everything else.  This is no surpise given its author, Roy Fielding, was involved (参与) in probably a dozen specs which govern (统治) how the web operates.

What benefits? The web and its core protocol, HTTP, provide a stack of features:
- Stiable (合适的) action (GET, POST, PUT, DELETE, ...)
- Caching
- Redirection and forwarding
- Security (encryption 加密 and authentication)

There are all critical factors (要害因素) on building resilient (有适应力的) services. But that is not all. The web is built out of lots of tiny specs, hence (因此) it's been able to evolve easily, without getting bogged down in "standards wars".

Developers are able to draw upon (凭借) 3rd party toolkits that implement these diverse (不同的) sepcs and instantly (立刻) have both client and server technology at their fingertips (指尖).

So  building on top of HTTP, REST APIs provide the means to build flexible APIs that can:
- Support backward compatibility (兼容)
- Evolvable (可展开的) APIs
- Scaleable services
- Securable services
- A spectrum (系列) of stateless (无状态) to stateful (有状态) servcie

What's important to realize is that REST, however ubiquitous (似乎无所不在的), is not a standard, per se (本质上), but an approach, a style, a set of constraints (约束) on your architecture that can help you build web-scale systems. In this tutorial we will use the Spring protfolio (作品集) to build a RESTful service while leveraging (对…施加影响) the stackless features of REST.

## Getting Started
As we work through this tutorial, we'll use Spring Boot. Go to Spring Initializr and select the follow:
- Web
- JPA
- H2
- Lombok

Then choose "Generate Project". A `.zip` will downloaded. Unzip it. Inside you'll find a simple, Maven-based project including a `pom.xml` build file (NOTE: You can use Gradle. The example in this tutorial will be Maven based.)

Spring Boot can work with any IDE. You can use Eclipse, IntelliJ IDEA, Netbeans, etc. The Spring Tool suite is an open-source, Eclipse-based IDE distribution (发行) that provides a superset of the Java EE distribution of Eclipse. It includes features that making working with Spring application even easier. It is, by no means, required. But consider it if you want that extra oomph (特质) for your keystrokes (按键). Here's a video demonstrating how to get started with STS and Spring Boot. THis is a general introduction to familiarize you with the tools.

If you pukc up IntelliJ IDEA as your IDE for this tutorial, you have to install lombok plugin. In order to see how we install plugin in IntelliJ please have a look at managing-pulginss. After this you have to ensure that "Enable annotation processing" checkbox is ticked under: Perferences -> Complier -> Annotation Processors, as it is described [here](https://stackoverflow.com/questions/14866765/building-with-lomboks-slf4j-and-intellij-cannot-find-symbol-log).

## The Story so Far...
Let's start off with the simplest thing we can construct. In fact, to make it as simple as possible, we can even leave out the concepts of REST. (Later on, we'll add REST to understand the differencne.)

our example models a simple payroll service that managers the employees of a company. Simply put, you need to store employee objects in an H2 in-memory database, and access them via JPA. This will be warpped with a Spring MVC layer to access remotely.

nonrest/src/main/java/payroll/Employee.java
```java
@Data
@Entity
class Employee {

  private @Id @GeneratedValue Long id;
  private String name;
  private String role;

  Employee() {}

  Employee(String name, String role) {
    this.name = name;
    this.role = role;
  }
}
```

Despite being small, this Java class contains much:
- @Data is a Lombok annotation to create all the getters, setters, equals, hash, and toString methods, based on the fields.
- @Entity is a JPA annotation to make this object ready for storage in a JPA-based data store.
- id, name, and role are the attribute for our domain object, the first being marked with more JPA annotations to indicate it's the primary key and automatically populated by the JPA provider.
- a custom constructor is created when we need to create a new instance, bu don't yet have an id.

With the domain object definition, we can now turn to Spring Data JPA to handle the tedious (冗长的) database interactions. Spring Data repositories are interfaces with methods supporting reading, updating, deleting, and creating records against a back end store. Some repositories also support data paging, and sorting, where appropriate (适当的). Spring Data synthesizes (整合) implementations based on conventions found in the naming of the methods in the interface.

> There are multiple repository implementations besides JPA. You can use Spring Data MongoDB, Spring Data GemFire, Spring Data Cassandra, etc. For this tutorial, we'll stick with JPA.

nonrest/src/main/java/payroll/EmployeeRepostiory.java
```java
interface EmployeeRepository extends JpaRepository<Employee, Long> {

}
```

This interface extends Spring Dat JPA's `JpaRepository`, specifying the domain type as Employee and the id types as Long. This interface, though empty on the surface, packs a punch (能击出有力的一拳) given it supports:
- Creating new instances
- Updating existing ones
- Deleting
- Finding (one, all, by simple or complex properties)

Spring Data's repository solution makes it possible to sidestep data store specifics and instead solve a majority or problems using domain-specific terminology.

Believe it or not, this is enough to launch an application! A Spring Data application is, at a minimum, a `public static void main` entry-point and the `@SpringBootApplication` annotation. This tells Spring Boot to help out, wherever possible.

```java
@SpringBootApplication
public class PayrollApplication {

  public static void main(String... args) {
    SpringApplication.run(PayrollApplication.class, args);
  }
}
```

`@SpringBootApplication` is a meta-annotation that pulls in component scanning, autoconfiguration, and property support. We won't dive into the details of Spring Boot in this tutorial, bu in essence, it will fire up a servlet container and serve up our service.

Nevertheless (然而), an application with no data isn't very interesting, so let's preload it. The follow class will get loaded automatically by Spring:
```java
@Configuration
@Slf4j
class LoadDatabase {

  @Bean
  CommandLineRunner initDatabase(EmployeeRepository repository) {
    return args -> {
      log.info("Preloading " + repository.save(new Employee("Bilbo Baggins", "burglar")));
      log.info("Preloading " + repository.save(new Employee("Frodo Baggins", "thief")));
    };
  }
}
```

What happens when it gets loaded?
- Spring Boot will run ALL CommandLineRunner beans once the application context is loaded.
- This runner will request a copy of the EmployeeRepository you just created.
- Using it, it wll create two entities and store them.
- @Slf4j is a Lombok annotation to autocreate an Slf4j-based LoggerFactory as log, allowing us to log these newly created "employees".

Right-click and Run PayRollApplication, and this is what you get:
```
# Fragment of console output showing preloading of dataW
...
2018-08-09 11:36:26.169  INFO 74611 --- [main] payroll.LoadDatabase : Preloading Employee(id=1, name=Bilbo Baggins, role=burglar)
2018-08-09 11:36:26.174  INFO 74611 --- [main] payroll.LoadDatabase : Preloading Employee(id=2, name=Frodo Baggins, role=thief)
...
```

This isn't the whole log, but just the key bits of preloading data. (Indeed, check out the whole console. It's glorious.)

## HTTP is the Platform
To warp your repository with a web layer, you must turn to Spring MVC. Thanks to Spring Boot, there is little in infrastructure (基础建设) to code. Instead, we can focus on actions:
```java
@RestController
class EmployeeController {

  private final EmployeeRepository repository;

  EmployeeController(EmployeeRepository repository) {
    this.repository = repository;
  }

  // Aggregate root

  @GetMapping("/employees")
  List<Employee> all() {
    return repository.findAll();
  }

  @PostMapping("/employees")
  Employee newEmployee(@RequestBody Employee newEmployee) {
    return repository.save(newEmployee);
  }

  // Single item

  @GetMapping("/employees/{id}")
  Employee one(@PathVariable Long id) {

    return repository.findById(id)
      .orElseThrow(() -> new EmployeeNotFoundException(id));
  }

  @PutMapping("/employees/{id}")
  Employee replaceEmployee(@RequestBody Employee newEmployee, @PathVariable Long id) {

    return repository.findById(id)
      .map(employee -> {
        employee.setName(newEmployee.getName());
        employee.setRole(newEmployee.getRole());
        return repository.save(employee);
      })
      .orElseGet(() -> {
        newEmployee.setId(id);
        return repository.save(newEmployee);
      });
  }

  @DeleteMapping("/employees/{id}")
  void deleteEmployee(@PathVariable Long id) {
    repository.deleteById(id);
  }
}
```

- @RestController indicates that the data returned by each will be written straight into the reponse body instead of rendering a template.
- An EmployeeRepository is injected by constructor into the controller.
- We have for each operations (`@GetMapping`, `@PostMapping`, `@PutMapping` and `@DeleteMpaaing`, corresponding to HTTP GET, POST, PUT, and DELETE calls). (NOTE: It's useful to read each method and understand what they do.)
- EmloyeeNotFoundException is an exception used to indicate when an employee is looked up but not found.

```java
class EmployeeNotFoundException extends RuntimeException {

  EmployeeNotFoundException(Long id) {
    super("Could not find employee " + id);
  }
}
```

When an EmployeeNotFoundException is thrown, this extra tidbit (花絮) of Spring MVC configuration is used to render an HTTP 404:
```java
@ControllerAdvice
class EmployeeNotFoundAdvice {

  @ResponseBody
  @ExceptionHandler(EmployeeNotFoundException.class)
  @ResponseStatus(HttpStatus.NOT_FOUND)
  String employeeNotFoundHandler(EmployeeNotFoundException ex) {
    return ex.getMessage();
  }
}
```

- **@ResponseBody** signals taht this advice is rendered straight (笔直地) into the response body.
- **@ExceptionHandler** configures the advice to only respond if an EmployeeNotFoundException is thrown.
- **@ResponseStatus** says to issue an **HttpStatus.NOT_FOUND**, i.e. an **HTTP 404**.
- The body of the advice generates the content. In this case, it gives the message of the exception.

To lanuch the application, either right-click the **public static void main** in **PayRollApplication** and select Run from your IDE, or:

Spring initializr uses maven warpper so type this:
```bash
$ ./mvnw clean spring-boot:run
```

Alternatively using your installed maven version type this:
```bah
$ mvn clean spring-boot:run
```

When the app starts, we can immediately interrogate (interrogate) it:
```bash
$ curl -v localhost:8080/employees
```

This will yield:
```
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /employees HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200
< Content-Type: application/json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Thu, 09 Aug 2018 17:58:00 GMT
<
* Connection #0 to host localhost left intact
[{"id":1,"name":"Bilbo Baggins","role":"burglar"},{"id":2,"name":"Frodo Baggins","role":"thief"}]
```

Here you can see the pre-loaded data, in a compacted format.

If you try and query a user that doesn't exist...

```bash
$ curl -v localhost:8080/employees/99
```

You get...

```bash
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /employees/99 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 404
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 26
< Date: Thu, 09 Aug 2018 18:00:56 GMT
<
* Connection #0 to host localhost left intact
Could not find employee 99
```

This message nicely shows an **HTTP 404** error with the custom message **Cound not find employee 99**.

It's not hard to show the currently coded interactions...

```bash
$ curl -X POST localhost:8080/employees -H 'Content-type:application/json' -d '{"name": "Samwise Gamgee", "role": "gardener"}'
```

Creates a new **Employee** record, and then sends the content back to us:
```
{"id":3,"name":"Samwise Gamgee","role":"gardener"}
```

You can alter (更改) the user:
```
$ curl -X PUT localhost:8080/employees/3 -H 'Content-type:application/json' -d '{"name": "Samwise Gamgee", "role": "ring bearer"}'
```

Updates the user:
```
{"id":3,"name":"Samwise Gamgee","role":"ring bearer"}
```

> Depending on how you construct your service can have significant impacts (有很大关系). In this situation, replace is a better description than update. For example, if the name was NOT provided, it would instead get nulled out.

And you can delete...

```
$ curl -X DELETE localhost:8080/employees/3
$ curl localhost:8080/employees/3
Could not find employee 3
```

This is all well and good, but do we have RESTful service yet? (IF you didn't catch the hint, the answer is no.)

What's missing?

## What makes something RESTful?
So far, you have a web-based service that handles the core operations involving employee data. But that's not enough to make things "RESTful".
- Pretty URLs like /employees/3 aren't REST.
- Merely using **GET**, **POST**, etc. aren't REST.
- Having all the CRUD operations laid out arent't REST.

In fact, what we have built so far is better described as **RPC** (Remote Procedure Call 远程过程调用). That's because there is no way to know how to interact with this service. If you published this today, you'd also have to write a document or host a developer's portal somewhere with all the details.

This **statement** (报告) of Roy Fielding may further lend a clue (线索) to the difference between **REST** and **RPC**:

> I am getting frustrated (沮丧) by the number of people calling any HTTP-based interface a REST API. Today's example is the socialsite (社交网站) REST API. That is RPC. It screams RPC. There is so much coupling (结合) on display that it should be given an X rating.

> What needs to be done to make the REST architectural (建筑) style clear on the notion that hypertext is a constraint (约束)? In other words, if the engine of application state (and hence the API) is not being driven by hypertext, then it cannot be RESTful and cannot be a REST API. Period. Is there some broken manual somewhere that needs to be fixed?

The side effect of NOT including hypermedia in our representationsis that clients MUST hard code URIs to navigate the API. This leads to the same brittle nature (脆弱性) that predated the rise of e-commerce (电子商务) on the web. It's a signal that our JSON output needs a little help.

Introducing Spring HEATEOAS, a Spring project aimed at helping you write hypermedia-driven outputs. To upgrade your service to being RESTful, add this to your build:
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

This tiny library will give us the constructs to define a RESTful service and then render it in an acceptable format for client consumption (消费).

A critical ingredient (关键成分) to any RESTful service is adding links to relvant (相关性) oprations. To make your controller more RESTful, add links like this:
```java
@GetMapping("/employees/{id}")
EntityModel<Employee> one(@PathVariable Long id) {

  Employee employee = repository.findById(id)
    .orElseThrow(() -> new EmployeeNotFoundException(id));

  return new EntityModel<>(employee,
    linkTo(methodOn(EmployeeController.class).one(id)).withSelfRel(),
    linkTo(methodOn(EmployeeController.class).all()).withRel("employees"));
}
```

This is very similar to what we had bedore, but a few things have changed:
- The return type of the method has changed from **Empoyee** to **EntityMode**. **EntityMode** is a generic container from Spring HATEOAS that includes not only the data but a collcection of links.
- **linkTo(MethodOn(EmployeeController.class).one(id).withSelfRel())** asks that Spring HATEOAS build a link to the **EmployeeController**'s `one()` method, and flag it as a self link.  
- **linkTo(methodOn(EmployeeController.class).all()).withRel("employees")** asks Spring HATEOAS to build a link to the aggregate (聚合) root, **all()**, and call it "empolyees".

What do we mean by "build a link"? One of Spring HATEOAS's core types is **Link**. It includes a **URI** and a **rel** (relation). Links are what empower (授权) the web. Before the World Wide Web, other document systems would render information or links, but it was the linking of documents WITH data that stitched (缝合) the web together.

Roy Fielding encourages bulding APIs with the same techniques that make the web successful, and links are one of them.

If you restart the application and query the employee record of **Bilbo**, you'll get a slightly different response than earlier:
```
# RESTful representation of a singe employee
{
  "id": 1,
  "name": "Bilbo Baggins",
  "role": "burglar",
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees/1"
    },
    "employees": {
      "href": "http://localhost:8080/employees"
    }
  }
}
```

This decompressed output shows not only the data elements you saw earlier (**id**, **name** and **role**), but also a **_links** entry containing two URIs. This entire document is formatted using **HAL**.

HAL is a lightweight mediatype that allows encoding not just data but also hypermedia controls, alerting consumers to other parts of the API they can navigate toward. In this case, there is a "self" link (kink of like a "this" statement in code) along with a link back to the **aggregate root**.

To make the aggregate root ALSO more RESTful, you want to include top level links while ALSO including any RESTful components within:
```java
// Getting an aggregate root resource
@GetMapping("/employees")
CollectionModel<EntityModel<Employee>> all() {

  List<EntityModel<Employee>> employees = repository.findAll().stream()
    .map(employee -> new EntityModel<>(employee,
      linkTo(methodOn(EmployeeController.class).one(employee.getId())).withSelfRel(),
      linkTo(methodOn(EmployeeController.class).all()).withRel("employees")))
    .collect(Collectors.toList());

  return new CollectionModel<>(employees,
    linkTo(methodOn(EmployeeController.class).all()).withSelfRel());
}
```

Wow! That method, which used to just be `repository.findAll()` has grown big! Let's unpack it.

**CollectionModel** is another Spring HATEOAS container aimed at encapsulating  (压缩) collections. it, too, also lets you include links. Don't let that first statement slip by (流逝). What does "encapsulating collections" mean? Collections of employees?

Not quite.

Since we're talking REST, it should encapsulate collections of **empoyee resources**.

That's why you fetch (取回) all the employees, but then tranform them into a list of **EmtityModel** objects. (Tanks Java 8 Stream API!!)

If you restart the appllcation and fetch the aggregate root, you can see what this looks like.

```java
# RESTful representation of a collection of employee resources
{
  "_embedded": {
    "employeeList": [
      {
        "id": 1,
        "name": "Bilbo Baggins",
        "role": "burglar",
        "_links": {
          "self": {
            "href": "http://localhost:8080/employees/1"
          },
          "employees": {
            "href": "http://localhost:8080/employees"
          }
        }
      },
      {
        "id": 2,
        "name": "Frodo Baggins",
        "role": "thief",
        "_links": {
          "self": {
            "href": "http://localhost:8080/employees/2"
          },
          "employees": {
            "href": "http://localhost:8080/employees"
          }
        }
      }
    ]
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees"
    }
  }
}
```

For this aggregate root, which serves up aa collection of employee resources, there is a top-level **"self"** link. The **"Collection"** is listed underneath (在底下) the "_embedded" section. This is how HAL represents collections.

And each individual member of the collection has their information as well as related links.

What is the point of adding all these links? It makes it possible to evolve (进化) REST services over time. Existing links can be maintained (维护) while new links are added in the future. Newer clients may take advantage (优势) of the new links, while legacy (遗产) clients can sustain (维持) themselves on the old links. This is especially helpful is services get relocated and moved around. As long as the link structure is maintained, clients can STILL find and interact with things.

## Simplifying Link Creation
Did you notice the repetition (重复) in single employee link creation? The code to provide a single link to an employee as well as an "employees" links to the aggregate root was shown twice. If that raised your concern (关心), good! There's a solution.

Simply put, you need to define a function that converts **Employee** objects to **EntityModel** objects. While you could easily code this method yourself, there are benefits down the road of implementing Spring HATEOAS's **RepresentationModelAssembler** interface.

```java
@Component
class EmployeeModelAssembler implements RepresentationModelAssembler<Employee, EntityModel<Employee>> {

  @Override
  public EntityModel<Employee> toModel(Employee employee) {

    return new EntityModel<>(employee,
      linkTo(methodOn(EmployeeController.class).one(employee.getId())).withSelfRel(),
      linkTo(methodOn(EmployeeController.class).all()).withRel("employees"));
  }
}
```

This simple interface has one method: **toModel()**. It is based on converting a non-resource object (**Employee**) into a resource-based object (**EntityModel**).

All the code you saw earlier in the controller can be moved into this class. And by applying Spring Framework's **@Conponent**, this component will be automatically created when the app starts.

> Spring HATEOAS's abstract base class for all resources is **RepresentationModel**. But for simplicity, I recomend using **EntityModel** as your mechanism to easily warp all POJOs as resources.

To leverage (生效) this assembler (装配工), you only have to alter the EmployeeController by injecting the assembler in the constructor.

```java
@RestController
class EmployeeController {

  private final EmployeeRepository repository;

  private final EmployeeModelAssembler assembler;

  EmployeeController(EmployeeRepository repository,
             EmployeeModelAssembler assembler) {

    this.repository = repository;
    this.assembler = assembler;
  }

  ...

}
```

From here, you can use it in the single-item employee method:
```java
@GetMapping("/employees/{id}")
EntityModel<Employee> one(@PathVariable Long id) {

  Employee employee = repository.findById(id)
    .orElseThrow(() -> new EmployeeNotFoundException(id));

  return assembler.toModel(employee);
}
```

This code is almost the same, except instead of creating the **EntityModel** instance here, you delegate it to the assembler. Maybe that doesn't look like much?

Applying the same thing in the aggregate root controller method is more impressive (引人注目的):
```java
@GetMapping("/employees")
CollectionModel<EntityModel<Employee>> all() {

  List<EntityModel<Employee>> employees = repository.findAll().stream()
    .map(assembler::toModel)
    .collect(Collectors.toList());

  return new CollectionModel<>(employees,
    linkTo(methodOn(EmployeeController.class).all()).withSelfRel());
}
```

The code is, again, almost the same, however you get to replace all that **EntityModel** creation logic with **map(assembler::toModel)**. Thanks to Java 8 method references, it's super easy to plug it in and simplify your controller.

> A key design goal of Spring HATEOAS is to make it easier to do The Right Thing. In this scenario, adding hypermedia to your service without hard coding a thing.

At this stage (阶段), you've created a Spring MVC REST controller that actually produces hypermedia-powered content! Clients that don't speak HAL can ignore the extra bits while consuming the pure data. Clients that DO speak HAL can navigate your empowered API.

But that is not the only thing needed to build a truly RESTful service with Spring.

## Evolving REST APIs
With one additional library and a few lines of extra code, you have added hypermedia to your application. But that is not the only thing needed to make your service RESTful. An important facet (方面) of REST if the fact that it's neither a technology stack nor a single standard.

REST is a collection of architectural constraints (约束) that when adopted (被采用的) make your application much more resilient (有弹性的). A key factor (因素) of resilience is that when you make upgrades to your services, your clients don't suffer from downtime.

In the "olden" days, upgrades were notorious (声名狼藉的) for breaking clients. In other words, an upgrade to the server required an update to the client. In this day and age, hours or even minutes of downtime spent doing an upgrade can cost milions in lost revenue (收益).

Some compaines requre that you present management with a plan to minimize downtime. In the past, you could get away with upgrading at 2:00 a.m. on a Sunday when load was at a minimum. But in today's Internet-based e-commerce with international coustomers, such strategies are not as effective.

SOAP-based services and CORBA-based services were incredibly brittle (难以置信的脆弱). It was hard to roll out (推出) a server that could support both old and new clients. With REST-based practices, it's much easier Especially using the Spring stack.

Imagine this design problem: You've rolled out a system with this **Employee**-based record. The system is a major hit. You've sold your system to countless enterprises. Suddenly, the need for an employee's name to be split into **firstName** and **lastName** arises.

Uh oh. Didn't think of that.

Before you open up the **Employee** class and replace the single field **name** with **firstName** and **lastName**, stop and think for a second. Will that break any clients? How long will it take to upgrade them. Do you even control all the clients accessing your services?

Downtime = lost money. Is management ready for that?

There is an old strategy that precedes REST by years.

> Never delete a column in a database.

You can always add columns (fields) to a database table. But don't take one away. The principle in RESTful services is the same. Add new fields to your JSON representations, but don't take any away. Like this:
```json
{
  "id": 1,
  "firstName": "Bilbo",
  "lastName": "Baggins",
  "role": "burglar",
  "name": "Bilbo Baggins",
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees/1"
    },
    "employees": {
      "href": "http://localhost:8080/employees"
    }
  }
}
```

Notice how this format shows **firstName**, **lastName**, AND **name**? While it sports duplication of information, the purpose is to support both old and new clients. That means you can upgrade the server without requiring clients upgrade at the same time. A good move that should reduce downtime.

And not only should you show this information in both the "old way" and the "new way", you should also process incoming data both ways.

How? Simple. Like this:
```java
@Data
@Entity
class Employee {

  private @Id @GeneratedValue Long id;
  private String firstName;
  private String lastName;
  private String role;

  Employee() {}

  Employee(String firstName, String lastName, String role) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.role = role;
  }

  public String getName() {
    return this.firstName + " " + this.lastName;
  }

  public void setName(String name) {
    String[] parts =name.split(" ");
    this.firstName = parts[0];
    this.lastName = parts[1];
  }
}
```

This class is very similar to the previous version of **Employee**. Let's go over the changes:
- Field name has been replaced by firstName and lastName. Lombok will generate getters and setters for those.
- A "virtual" getter for the old **name** property, **getName()** is defined. It uses the **firstName** and **lastName** fileds to produce a value.
- A "virtual" setter for the old **name** property is also defined, **setName()**. It parses an incoming string and stores it into the proper fields.

Of course not EVERY change to your API is as simple as splitting a string or merging two strings. But it's surely not impossible to come up with a set of transform for most scenarios (情景), ehh?

Another fine tuning is to ensure each of your REST methods returns a proper response. Update the POST method like this:
```java
@PostMapping("/employees")
ResponseEntity<?> newEmployee(@RequestBody Employee newEmployee) throws URISyntaxException {

  EntityModel<Employee> entityModel = assembler.toModel(repository.save(newEmployee));

  return ResponseEntity
    .created(entityModel.getRequiredLink(IanaLinkRelations.SELF).toUri())
    .body(entityModel);
}
```

- The new **Employee** object is saved as before. But the resulting object is wrapped using the **EmployeeModelAssembler**.
- Spring MVC's **responseEntity** is used to create an **HTTP 201 Created** status message. This type of response typically includes a **Location** response header, and we use the URI derived from the model's self-related link.
- Additionally, return the resource-based version of the saved object.

With this tweak (微调) in place, you can use the same endpoint to create a new employee resource, and use the legacy **name** field:
```bash
$ curl -v -X POST localhost:8080/employees -H 'Content-Type:application/json' -d '{"name": "Samwise Gamgee", "role": "gardener"}'
```

The output is shown below:
```
> POST /employees HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 46
>
< Location: http://localhost:8080/employees/3
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Fri, 10 Aug 2018 19:44:43 GMT
<
{
  "id": 3,
  "firstName": "Samwise",
  "lastName": "Gamgee",
  "role": "gardener",
  "name": "Samwise Gamgee",
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees/3"
    },
    "employees": {
      "href": "http://localhost:8080/employees"
    }
  }
}
```

This not only has the resulting object rendered in HAL (both **name** as well as **firstName** / **lastName**), but also the **Location** header populated with **http://localhost:8080/employees/3**. A hypermedia powered client could opt to "surf" to this new resource and proceed to interact with it.

The PUT controller method needs similar tweaks:
```java
@PutMapping("/employees/{id}")
ResponseEntity<?> replaceEmployee(@RequestBody Employee newEmployee, @PathVariable Long id) throws URISyntaxException {

  Employee updatedEmployee = repository.findById(id)
    .map(employee -> {
      employee.setName(newEmployee.getName());
      employee.setRole(newEmployee.getRole());
      return repository.save(employee);
    })
    .orElseGet(() -> {
      newEmployee.setId(id);
      return repository.save(newEmployee);
    });

  EntityModel<Employee> entityModel = assembler.toModel(updatedEmployee);

  return ResponseEntity
    .created(entityModel.getRequiredLink(IanaLinkRelations.SELF).toUri())
    .body(entityModel);
}
```

The **Employee** object built from the **save()** operation is then wrapped using the **EmployeeModelAssembler** into an **EntityModel** object. Using the **getRequiredLink()** method, you can retrieve (找回) the Link created by the **EmployeeModelAssembler** with a SELF rel. This method returns a Link which must be turned into a URI with the **toUri** method.

Since we want a more detailed HTTP response code than **200 OK**, we will use Spring MVC's **ResponseEntity** wrapper. It has handy static method **created()** where we can plug (补充) in the resource's URI. It's debatable (有争议的) if **HTTP 201 Created** carries the right semantics (语意) since we aren't necessarily "creating" a new resource. But it comes pre-loaded with a Location response header, so run with it. 

```
$ curl -v -X PUT localhost:8080/employees/3 -H 'Content-Type:application/json' -d '{"name": "Samwise Gamgee", "role": "ring bearer"}'

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> PUT /employees/3 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 49
>
< HTTP/1.1 201
< Location: http://localhost:8080/employees/3
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Fri, 10 Aug 2018 19:52:56 GMT
{
	"id": 3,
	"firstName": "Samwise",
	"lastName": "Gamgee",
	"role": "ring bearer",
	"name": "Samwise Gamgee",
	"_links": {
		"self": {
			"href": "http://localhost:8080/employees/3"
		},
		"employees": {
			"href": "http://localhost:8080/employees"
		}
	}
}
```

That employee resource has now been updated and the location URI sent back. Finally, update the DELETE operation suitably:
```java
@DeleteMapping("/employees/{id}")
ResponseEntity<?> deleteEmployee(@PathVariable Long id) {

  repository.deleteById(id);

  return ResponseEntity.noContent().build();
}
```

This return an **HTTP 204 No Content** response.

```
$ curl -v -X DELETE localhost:8080/employees/1

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> DELETE /employees/1 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 204
< Date: Fri, 10 Aug 2018 21:30:26 GMT
```

> Making changes to the fields in the **Employee** class will require coordination (协调) with your detabase team, so that they can properly (正确地) migrate existing content into the new columns.

You are now ready for an upgrade that will NOT disturb existing clients while newer clients can take advantage of the enhancements (增强)!

By the way, are you worried about sending too much information over the wire (电线)? In some systems where every byte counts, evolution of APIs may need to take a backseat (后座). But don't pursue such premature (未成熟的) optimization until you measure.