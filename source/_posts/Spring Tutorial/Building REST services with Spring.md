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

> I am getting frustrated (沮丧) by the number of people calling any HTTP-based interface a REST API. Today's example is the socialsite (社交网站) REST API. That is RPC. It screams RPC. There is so much coupling on display that it should be given an X rating.
> What needs to be done to make the REST architectural (建筑) ssyle clear on the notion that hypertext is a constraint? 