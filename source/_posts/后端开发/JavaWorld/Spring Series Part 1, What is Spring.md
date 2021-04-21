---
title: Spring Series Part 1, What is Spring

tag:
- The Spring Series

categories:
- JavaWorld

date: 2020-04-19 11:00:01
---
Spring is perhaps the best of the compoment-based frameworks that emerged (出现) at the turn of the 21st century. It vastly improves the way that developers write and deliver infrastructure code in Java-based applications. Since it inception (开端), Spring has been recognized as a leading framework for enterprise Java development. As an end-to-end application framework, Spring mirrors some of the Java EE capabilities, but it offers a combination (结合) of features and programming conventions (约定) you won't find elsewhere.

This article introduces Spring and its core programming philosophy (理念) and methodology (方法论): Inverstion of control and dependency injection. You'll also get started with Spring annotations and a couple of hands-on coding examples.

## Dependency injection and inversion of control
Spring's core idea is that instead of managing object relationships yourself, you offload them to the framework. Inversion of control (IOC) is the methodology used to manage object relationships. Dependency injection is the machanism (机制) for implementing IOC. Since these two concepts are related but different, let's consider them more closely:
- **Inversion of control (IOC)** does jsut what its name says: it inverst the traditional hierarchy of control for fulfilling object relationships. Instead of relying on application code to define how objects relate to each other, relationships are definded by the framework. As a methodology, IOC introduces consistency and predictability to object relations, but it does require you, as the developer, to give up some fine-grained control.
- **Dependency injection (DI)** is a machanism where the framework "injects" dependencies into your app. It's the practial implementation of IOC. Dependency injection hinges (关键) on polymorphism (多态性), in the sense that it allows the fulfillment (满足) of a reference type to change based on configurations in the framework. The framework injects variable references rather than having them manually fulfilled in application code.

>JSR-330
> Like much in the Java world, what began as an in-the-wild innovation, Spring, has been in part absorbed by standard specification. In this case, JSR-330 is the Java standard. This nice thing about the JSR-330 spec is you can use it elsewhere, and will see it in use elsewhere, beyond Spring. You can use it without using Spring. However, Spring brings a whole lot more to the table.

## Example 1: Spring dependency injection
Inversion of control and dependency injection are best understood by using them, so we'll start with a quick programming example.

Say you're modelling a car. If you're modeling in plain old Java, you might have an interface member on the Car class to reference an Engine interface, as shown in Listing 1.

### Listing 1. Object relations in plain old Java
```java
public interface Engine {

}

public class Car {
    private Engine engine;

    public Engine getEngine() {}

    public void setEngine(Engine engine){}
}
```

Listing 1 contains an interface for an Engine type, and a class for the concrete Car type, Which references the Engine. (Note that in a real programming scenario these would be in separate files.) Now you're creating a Car instance, you'd set the association as shown in Listing 2.

### Listing 2. Creating a Car with the Engine interface
```java
// ...
Car newCar = new Car();
Engine sixCylEngine = new InlineSixCylinderEngine();
newCar.setEngine(sixCylEngine);
// do stuff with the car
```

Note that you create the Car object first. You then create a new object that fulfills the Engine interface, and assign it manually to the Car object. That is how object associations work in plain old Java.

## Modeling classes and objects in Spring
Now let's look at the same example in Spring. Here, you could do something like what's shown in Listing 3. You start with the Car class, but in this case you add an annotation to it: `@Inject`.

### Listing 3. Example of using the @Inject annotation in Spring
```java
public class Car {
    @Inject
    private Engine engine;
    // ...
}
```

Using the `@Inject` annotation (or `@Autowired`, if you prefer) tells Spring to search the context and automatically inject an object into the reference, based on a set of rules.

Next, consider the `@Component` annotation, shown in Listing 4.

### Listing 4. @Component annotation
```java
@Component
public class InlineSixCylinderEngine implements Engine {
    // ...
}
```
Annotation a class with `@Component` tells Spring that it is available for fulfilling injections. In this case, the InlineSixCyEngine would be injected because it is availalbe and satisfies the interface requirement of the association. In Spring, this is called an "autowired" injection. (See below for more about Spring's `@Autowired` annotation.)

## Decoupling (解耦) as a design principle
Inversion of control with dependency injection removes a source of concrete dependency from your code. Nowhere in the program is there a hard-codede reference to the Engine implementation. This is an example of decoupling as a software design principle. Decoupling application code from implementation makes your code easier to manage and maintain. The application knows less about how its parts fit together, but it's much eaiser to make changes at any point in the application lifecycle.

> @Autowired vs @Inject
> @Autowired and @Inject do the same thing. However, @Inject is the Java standard annotation, whereas @Autowired is specific to Spring. They both serve the same purpose of telling the DI engine to inject the field or method with a matching object. You can use either one in Spring.

## Overview of the Spring framework
Now that you've seen some Spring code, let's take an overview of the framework and its components. As you can see, the framework consists of four main modules, which are broken into packages. Spring gives you a fair amount of flexibility with the modules you'll use: 
- Core container
    - Core
    - Bean
    - Context
    - Expression Language
- Aspect-oriented programming (AOP)
    - AOP
    - Aspects
    - Instrumentation
- Data access and integration
    - JDBC
    - JPA/ORM
    - JMS
    - Transactions
- Web
    - Web/REST
    - Servlet
    - Struts

Rather than cover everthing here, let's get started with two of the more commonly used Spring features.

## Starting up a new project: Spring Boot
We'll use Spring Boot to create an example project, which we'll use to demo Spring features. Spring Boot makes starting new projects much easier, as you'll seee for yourself. To begin, take a look at the main class shown below. In Spring Boot, we can take a main class with a `main()` method, and then choose to run it standalone, or package for deployment in a container like Tomcat.

### Listing 5. Main class with Spring Boot
```java
@SpringBootApplication
pulbic class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

Note two things about the above code: First, all of the work is abstracted into the framework. The main class boots up the app, but it doesn't konw anything about how the app works or delivers its functionality. Second, the `SpringApplication.run()` does the actual job of booting the app and passing in the Application class itself. Again, the work the app does is not apparent here.

The `@SpringBootApplication` annotation wraps up a few standard annotations and tells Spring to look at the package where the main calss exists for components. In our previous example, with the car and engine, this would allow Spring to find all classes annotated with `@Component` and `@Inject`. The process itself, called component scanning, is highly customizable.

You can build the app with the standard `mvn clane install`, and you can run it with the Spring Boot goal (`mvn spring-boot:run`). Before doing that, let's look at this application's pom.xml file.

### Listing 6. Starter pom.xml
```xml
<groupId>com.javaworld</groupId>
    <artifactId>what-is-spring</artifactId>
    <version>1.0.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
    </parent>

    <dependencies>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

Note two important features in the above code:
1. The partent element relies on the spring-boot-starter-parent proejct. This parent proejct defines a number of useful defaults, such as the default compiler level of JDK 1.8. For the most part, you can just trust that it knows what it's doing. As an example, you can omit the version number for many common dependencies, and SpringBootParent will set the versions to be compatible. When you bump up the parent's version number, the dependency versions and defaults will also change.
1. The spring-boot-maven-plugin allows for the executable JAR/WAR packging and in-plcae run (via the `mvn spring-boot:run` command).

## Adding Spring Web as a dependency
So far, we've been able to use spring-boot to limit how much work we put in to get an app up and running. Now let's add a dependency and see how quickly we cacn get something in a broswer.

### Listing 7. Adding Spring Web to a project
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

> Note
> Spring will automatically detect what files have changed and compile accordingly. You can just execute `mvn spirng-boot:run` to pickup changes.

Now that we've got a baisc project setup, we're ready for our two examples.

## Example 2. Building RESTful endpoints with Spring Web
We've used spring-boot-starter-web to bring in serveral dependencies that are useful for building web applications. Next we'll create a route handler for a URL path. Spring's web support is part of the Spring MVC (Model-View-Controller) module, but don't let that worry you: Spring Web has full and effective support for building RESTful endpoints, as well.

The class whose job it is to field URL requests is known as a controller, as shown in Listing 8.

### Listing 8. Spring MVC Rest controller
```java
@Controller
public class GreetingController {
    
    @RequestMethod(value = "/hi", method = RequestMethod.GET) 
    public String hi(@RequestParam(name = "name", required = false, defaultValue = "JavaWorld") String name, Model model) {
        return "Hello " + name;
    }
}
```

### The @Controller annotation
The @Controller annotation identifies a class as a controller. A class narked as a controller is also automatically identified as a component class, which makes it a candidate for auto-wiring. Wherever this controller is needed, it will be plugged into the framework. In this case, we'll plug it into the MVC system to handle requests.

The controller is a specialized kind of component. It support the @RequestMapping and @ResponseBody annotations that you see on the `hi()` method. These annotations tell the framework how to map URL requests to the app.

At this point, you can run the app with `mvn spring-boot:run`. When you hit the `/hi` URL, you'll get a response like "Hello JavaWorld".

Notice how Spring has taken the basics of autowiring components, and delivered a whole web framework. With Spring, you don't have to explicitly connect anything togetehr!

### The @Request annotations
The @RequestMapping allows you to define a handler for a URL path. Options include defining the HTTP method you want, which is what we've done in this case. Leaving RequestMethod off would instruct the program to handle all HTTP method types.

The @RequestParam argument annotation allows us to map the request parameters directly into the method signature, including requiring certain params and defining default values as we've done here. We can even map a request body to a class with the @RequestBody argument annotation.

### REST and JSON response
If you are creating a REST endpoint and you want to return JSON from the method, you can annotate the method with @ResponseBody. The response will then be automatically packaged as JSON. In this case you'll return an object from the method.

> Using MVC with Spring Web
> Similar to Struts, the Spring Web module can easily be used for a true model-view-controller setup. In this case, you would return a mapping in the given templating (like Thymeleaf), and Spring would resolve the mapping, provide the model you pass to it, and render the response. 

## Example #3: Spring with JDBC
Now let's do something more intertesting with our request hanlder: let's return some data from a database. For the purpose of this example, we'll use the H2 database. Thankfully, Spring Boot supports the in-memory H2 DB out of the box.

You can add the H2 DB to your app by including it in your pom.xml, as shown in Lisinting 9. We'll also add a dependency to spring-boot-starter-jdbc. This brings in what we need to control JDBC with Spring.

### Listing 9. Adding a Maven dependency to the H2 DB
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
</dependency>
```

Next, you'll want to configure the database. This is done with a spring.database.properties file, which is located in the `/resources` directory. Listing 10 shows how we can use H2 with the in-memory mode activated.

### Listing 10. H2 in-memory config
```properties
driverClassName=org.hsqldb.jdbc.JDBCDriver
url=jdbc:hsqldb:mem:myDb
username=sa
password=sa
```

### Service component classes
Now, we can start using the database. It's that easy. However, basic software design tells us never to access the data layer via the view layer. In this case, we don't want to access the JDBC support via the view controller. We need a service component. In Spring Web, we use the @Service annoation to create a service class. Like the @Controller annoation, using the @Service annotation designates a class as a kink of @Component. That means Spring will add it to the DI context, and you can autowire it into your controller.

> Annotating components
> Spring offers a few ways to annotate components. The most baisc way to indicate that a class is available for auto-wriring is via the @Component annotation. The @Service annotation does the same thing, but indicates a specific type of class. You could use the @Bean annotation to designate a method that would serve the purpose of creating a bean to be autowired.

Lising 11 shows a simple Service Component.

### Listing 11. Service component
```java
@Service("myService")
public class MyService {
  public String getGreeting(){
    return "Hey There";
  }
	public boolean addSong(String name) {
		if (name.length() > 15){
		  return false;
		}
		return true;
	}
	public List<String> getSongs() {
		return new ArrayList();
	}
}
```

Now we can access the service class from the controller. In listing 12, we'll injetc it.

### Listing 12. Injetcing MyService into the controller
```java
@Controller
public class GreetingController {
  @Inject
  private MyService myService;
    @RequestMapping(value = "/hi", method = RequestMethod.GET)
    public String hi(@RequestParam(name="name", required=false, defaultValue="JavaWorld") String name, Model model) {
        return myService.getGreeting() + name;
    }
}
```

Now the Controller is makring use of the Service class. Notice how Spring is allowing us to define a layered architecture using the same DI system. We can do the same in defining a data layer that the service class can use, and leverage Spring's support for a variety of datastores and datastore access approaches at the same time.

We can annotate our data layer class with @Repository, as seen in Listing 13, and the inject it into the service class. In the same way @Service allowed us to define the service layer, we are now defining the data layer in a decoupled way.

## The JdbcTemplate class
The data layer will require more than the service layer, because it will be talking to the database. Spring eases this primarily by providing the JdbcTemplate class.

### Listing 13. Repository data class
```java
@Repository
public class MyDataObject {
  public void addName(String name){
    jdbcTemplate.execute("DROP TABLE names IF EXISTS");
    jdbcTemplate.execute("CREATE TABLE names("id SERIAL, name VARCHAR(255))");
    jdbcTemplate.update("INSERT INTO names (name) VALUES (?)", name);
  }
}
```

Spring will automatically use the in-memory H2 DB we've configured. Notice how jdbcTemplate has eliminated all the boilerplate and error-handling code from this class. While this is a simplified example of accessing the database, it gives you an idea of how SPring works both to connect your application layers, and facilitates the use of other required services.

## Conclusion
Spring is one of the most advanced and compelete application development framewors for Java, bar none. It makes setting up an application easier, allows you to easily bring in the dependencies you need as the application grows, and is fully capable ofremping up the high-volume, production-grade use.

It's tough to argue using Spring in a new Java application. The Spring platform is maintained and advanced with vigor, and virtually any task you might need to undertake is doable with Spring. Using this platform will spare you considerable heavy lifting, and will help ensure your application design is robust and flexible. If you can use Spring to ease your development path, then do it.
