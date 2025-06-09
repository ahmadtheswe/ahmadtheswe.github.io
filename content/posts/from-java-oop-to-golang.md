+++
date = '2025-06-09T11:10:13+07:00'
draft = false
title = 'From Java OOP to Golang'
author = 'ahmad'
categories = ['Java', 'Go']
tags = ['java', 'go']
featuredImage = '/images/from-java-oop-to-golang/featured.png'
+++

As a Java developer, moving to Go can be a bit challenging, especially when it comes to understanding how Go handles object-oriented programming (OOP) concepts. In this article, I will represent some common OOP concepts in Java and how they can be implemented in Go. This will help you transition from Java to Go more smoothly (hopefully, lol).

## How To Make Go Classes in Go? (Compared to Java)

Let's say we have a class in Java like this:
```java
class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }
}
```
In Go, we can create a similar structure using structs. Here's how you can do it:
```go
type Person struct {
    Name string
    Age  int
}
```
Go is more similar to C in this regard, where we use structs to define data types. We can also create methods for the struct, but Go does not have classes in the same way Java does.


Another example, we can create a service class in Java like this:
```java
class PersonService {
    public Person createPerson(String name, int age) {
        return new Person(name, age);
    }
}
```

In Go, we can achieve a similar functionality using a struct and methods. Here's how you can do it:
```go
type PersonService struct{}
func (ps *PersonService) CreatePerson(name string, age int) Person {
    return Person{Name: name, Age: age}
}
```
Classes in Go are not defined in the same way as in Java. Instead, we use structs to define data types and methods to define behaviors associated with those types.


## How To Do Inheritance in Go (Compared to Java)

Inheritance in Java is very straightforward, where a class can `extend` another class. Here's an example in Java:
```java
class Animal {
    public void speak() {
        System.out.println("Animal speaks");
    }
}
class Dog extends Animal {
    @Override
    public void speak() {
        System.out.println("Dog barks");
    }
}
```

As Java developers, inheritance in Go is a bit weird at first. Go doesn't have traditional inheritance like Java, but it supports composition through **struct embedding**. Here's how we can achieve similar functionality in Go:
```go
type Animal struct{}
func (a Animal) Speak() {
    fmt.Println("Animal speaks")
}
type Dog struct {
    Animal // Embedding for inheritance
}
func (d Dog) Speak() {
    fmt.Println("Dog barks")
}
```

## How To Do Interfaces in Go (Compared to Java)

In Java, we define interfaces like this:
```java
interface Animal {
    void Speak();
}
class Dog implements Animal {
    @Override
    public void Speak() {
        System.out.println("Dog barks");
    }
}
```

In Go, we define interfaces similarly, but we don't use the `implements` keyword. Here's how you can do it:
```go
type Animal interface {
    Speak()
}
type Dog struct{}
func (d Dog) Speak() {
    fmt.Println("Dog barks")
}
```

## How To Do Dependency Injection in Go (Compared to Java)

In Java, we often use interfaces and constructor injection for dependency injection. Here's an example:
```java
class Service {
    private final Repository repository;

    public Service(Repository repository) {
        this.repository = repository;
    }

    public void doSomething() {
        repository.save();
    }
}
```

In Go, we can achieve dependency injection using interfaces and struct embedding. Here's how you can do it:
```go
type Repository interface {
    Save()
}
type Service struct {
    repo Repository
}
func NewService(repo Repository) Service {
    return Service{repo: repo}
}
func (s Service) DoSomething() {
    s.repo.Save()
}
```

That's it! You can see that Go has a different approach to OOP compared to Java, but it still allows you to implement similar concepts. The key differences are the use of structs instead of classes, the absence of traditional inheritance, and the way interfaces are defined and implemented.