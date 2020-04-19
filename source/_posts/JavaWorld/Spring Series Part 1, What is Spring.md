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