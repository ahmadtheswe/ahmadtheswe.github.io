+++
date = '2023-06-23T23:52:06+07:00'
draft = false
title = 'Solid Principles in a Nutshell'
author = 'ahmad'
tags = ['java', 'solid', 'programming']
featuredImage = '/images/solid-principles-in-a-nutshell/featured.png'
+++

I think one of the most important sets of principles that should be understood by all software engineers is SOLID principles (not the only one, but the most basic). By understanding this principle, we assume that we'll be able to deliver good, cleaner and maintainable codes.

SOLID is an acronym that stands for Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, and Dependency Inversion. It's a best practice for developing a program using object-oriented programming.

There are so many articles that have discussed it. But, I think it is still a worthy topic to rediscuss about. Let's try to discuss this in the easiest way possible. This article also includes example codes in Java.

## Single Responsibility Principle

The main goal of this principle is a single class should only responsible for one purpose. That means we will violate this principle if we create a class that consists of multiple purposes.

By following this principle, it will give us benefits:

* Our code will be more modular.

* This can be easier for us to trace our code in the future as we already determine the specific purpose of each class.


### Wrong example

```java
public class Person {
    private String firstname;
    private String lastname;
    private String origin;
    private String age;
    private String sex;

    //constructor, getters and setters

    // methods that directly relate to the person properties
    public String returnFullName() {
        return firstname + " " + lastname;
    }
}
```

Then we add the `storeInDatabase` method.

```java
public class Person {
    private String firstname;
    private String lastname;
    private String origin;
    private String age;
    private String sex;

    //constructor, getters and setters

    // methods that directly relate to the person properties
    public String returnFullName() {
        return firstname + " " + lastname;
    }

    public void storeInDatabase(Person person) {
        // codes that perform database action
    }
}
```

What goes wrong :

* `storeInDatabase` is not directly related to or responsible for `Person` class.


### Correct example

To fix this, we can put `storeInDatabase` method in a new class, let say `PersonDatabase` class.

```java
public class PersonDatabase {

    public void storeInDatabase(Person person) {
        // codes that perform database action
    }
}
```

## Open-Closed Principle

This principle emphasizes in **<mark>a class should be open for extension but closed for modification</mark>**.

The benefit of following this principle is we don't modify the existing code which can probably cause new bugs. (But, it's an exception for bugs in existing code, you should fix it, not extend it :grin:).

### Example

Suppose that we have `Car` class like below.

```java
public class Car {
    private String brand;
    private String color;

    // constructors, getters, and setters
}
```

Then, we want a `Truck` class that has the same properties with `Car` class, with some additions. Instead of modifying `Car` class, we can extend it to `Truck` class like below.

```java
public class Truck extends Car {
    private int cargoVolume;

    // constructors, getters, and setters
}
```

By doing so, we don't need to modify the `Car` class that might be already used in some part of our program.

If you want to create another class like `Ambulance` class in the future, you can extend the `Car` class like you did with `Truck` class.

```java
public class Ambulance extends Car {
    private hasbloodPressureGague;
    private hasThermometer;

    // constructors, getters, and setters
}
```

## Liskov Substitution Principle

This principle is arguably the most complex principle among these 5 principles. <mark>The simple meaning of this principle may be a subclasses should be substitutable for their base classes</mark>.

Let say, `Toyota` class is a subclass of `Car` class. `Toyota` class should be able to replace `Car` class without disrupting the behavior of our program. Show this example :

```java
class Car {
    private String brand;

    public Car(String brand) {
        this.brand = brand;
    }

    public void startEngine() {
        System.out.println("Engine started.");
    }

    public void stopEngine() {
        System.out.println("Engine stopped.");
    }
}

class Toyota extends Car {
    public Toyota() {
        super("Toyota");
    }

    @Override
    public void startEngine() {
        System.out.println("Toyota engine started.");
    }

    @Override
    public void stopEngine() {
        System.out.println("Toyota engine stopped.");
    }

    public void engageAutopilot() {
        System.out.println("Autopilot engaged.");
    }
}
```

In this Java example, the `Car` class represents a generic car and a `Toyota` class represents a specific type of car (Toyota brand) that extends the behavior of the base class.

We can use the `Car` class and the `Toyota` class interchangeably, as demonstrated below:

```java
public class Main {
    public static void startCar(Car car) {
        car.startEngine();
    }

    public static void stopCar(Car car) {
        car.stopEngine();
    }

    public static void engageAutopilot(Car car) {
        if (car instanceof Toyota) {
            Toyota toyota = (Toyota) car;
            toyota.engageAutopilot();
        } else {
            System.out.println("Autopilot not available for this car.");
        }
    }

    public static void main(String[] args) {
        Car myCar = new Car("SomeBrand");
        startCar(myCar);    // Output: Engine started.
        stopCar(myCar);     // Output: Engine stopped.

        Toyota myToyota = new Toyota();
        startCar(myToyota);    // Output: Toyota engine started.
        stopCar(myToyota);     // Output: Toyota engine stopped.

        engageAutopilot(myCar);     // Output: Autopilot not available for this car.
        engageAutopilot(myToyota);  // Output: Autopilot engaged.
    }
}
```

The `startCar()` and `stopCar()` methods accept a `Car` object as a parameter. We can pass both a `Car` instance (`myCar`) and a `Toyota` instance (`myToyota`) to these methods without any issues.

The `engageAutopilot()` method checks if the provided Car object is an instance of `Toyota`. If it is, it casts it to a `Toyota` object and calls the `engageAutopilot()` method. Otherwise, it handles the case where autopilot functionality is not available for non-`Toyota` cars. This demonstrates the usage of the `Toyota` class as a subtype of the `Car` class without violating the Liskov substitution principle.

## Interface Segregation Principle

Segregation means keeping different things separate. This principle is about separating interfaces based on their purpose.

The principle states that many client-specific interfaces are better than one general-purpose interface. Clients should not be forced to implement a function they do not need.

```java
interface Worker {
    void work();
    void sleep();
}
```

Let's say that we have 2 classes that implement `Worker` interface, `Programmer` and `Robot` class.

```java
class Programmer implements Worker {
    public void work() {
        System.out.println("Programmer is working.");
        // Perform programming tasks
    }

    public void sleep() {
        System.out.println("Programmer is sleeping.");
        // Sleep at night
    }
}

class Robot implements Worker {
    public void work() {
        System.out.println("Robot is working.");
        // Perform robotic tasks
    }

    // Can't be implemented
    public void sleep() {
        // Robots don't sleep!
        throw new UnsupportedOperationException("Robots don't sleep!");
    }
}
```

As we see, sleep() method can be implemented by the Programmer class, but not by `Robot` class. To solve this, we can split `Worker` interface.

```java
interface Worker {
    void work();
}

interface LifeBeing {
    void sleep();
}
```

The `Programmer` class now implements all two interfaces, as it performs all the associated actions. However, the `Robot` class only implements the `Worker` interface, as it doesn't need to sleep.

```java
class Programmer implements Worker, LifeBeing {
    public void work() {
        System.out.println("Programmer is working.");
        // Perform programming tasks
    }

    public void sleep() {
        System.out.println("Programmer is sleeping.");
        // Sleep at night
    }
}

class Robot implements Worker {
    public void work() {
        System.out.println("Robot is working.");
        // Perform robotic tasks
    }
}
```

## Dependency Inversion Principle

This principle refers to the decoupling of the software modules. Dependency inversion is a design principle in object-oriented programming that states that high-level modules should not depend on low-level modules. Instead, both should depend on abstractions.

```java
// High-level module
class EmployeeTracker {
    private EmployeeService employeeService;

    public EmployeeTracker(EmployeeService employeeService) {
        this.employeeService = employeeService;
    }

    // other methods
    }
}

// Abstraction (interface) for employee service
interface EmployeeService {
    String getBranch(String employeeId);
    String getWage(String employeeId);
    String getPosition(String employeeId);
}

// Low-level module implementing the employee service
class EmployeeServiceImpl implements EmployeeService {
    public String getBranch(String employeeId) {
        // Logic to retrieve employee data
        return employee.getBranch();
    }
    public String getWage(String employeeId) {
        // Logic to retrieve employee data
        return employee.getWage();
    }
    public String getPosition(String employeeId) {
        // Logic to retrieve employee data
        return employee.getPosition();
    }
}
```

High-level module called `EmployeeTracker` needs to retrieve employee data. Instead of directly depending on the specific implementation, it depends on the `EmployeeService` abstraction/interface.

The `EmployeeServiceImpl` class is a low-level module that implements the `EmployeeService` interface. It contains the specific implementation details for retrieving employee data, such as making API calls.

`EmployeeTracker` class doesn't depend on these implementation details, promoting loose coupling and ensuring that the high-level module is not tightly coupled to specific low-level modules.

By applying this principle, <mark>we invert the dependency relationship between high-level and low-level modules. The high-level module depends on abstractions, while the low-level modules depend on those abstractions. This promotes modular and flexible code design</mark>.

## Conclusion

That's all a short explanation of SOLID principles. You can boost your code quality by applying these principles to your code. There are several benefits we'll get by following these principles :

* **Maintainability**: Easier code maintenance and modification.

* **Scalability**: Ability to add new features without modifying existing code.

* **Testability**: Facilitates effective unit testing and code verification.

* **Reusability**: Creation of reusable and interchangeable components.

* **Flexibility**: Greater adaptability and ability to switch implementations.

* **Readability and Understandability**: Clear, organized code structure for easier comprehension and collaboration.


## Additional Resources

* [A Solid Guide to SOLID Principles](https://www.baeldung.com/solid-principles) from Baeldung

* [The SOLID Principles of Object-Oriented Programming Explained in Plain English](https://www.freecodecamp.org/news/solid-principles-explained-in-plain-english) from freeCodeCamp

* [What are the SOLID principles in Java?](https://www.educative.io/answers/what-are-the-solid-principles-in-java) from educative