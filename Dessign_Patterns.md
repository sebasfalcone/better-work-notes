# Table of contents

- [Table of contents](#table-of-contents)
- [Design patterns](#design-patterns)
- [Object Oriented Programming - OOP](#object-oriented-programming---oop)
  - [Basics of OOP](#basics-of-oop)
  - [Objects and classes](#objects-and-classes)
  - [Class hierarchies](#class-hierarchies)
  - [Pillars of OOP](#pillars-of-oop)
    - [Abstraction](#abstraction)
    - [Encapsulation](#encapsulation)
      - [C++ example](#c-example)
    - [Inheritance](#inheritance)
    - [Polymorphism](#polymorphism)
  - [Relations Between objects](#relations-between-objects)
    - [Dependency](#dependency)
    - [Association](#association)
    - [Aggregation](#aggregation)
    - [Composition](#composition)
  - [Summary](#summary)
- [Software design principles](#software-design-principles)
  - [Code reuse](#code-reuse)
- [Design patterns](#design-patterns)
  - [What's a Design Pattern?](#whats-a-design-pattern)
  - [What does the patter consist of?](#what-does-the-patter-consist-of)
  - [Classification of patterns](#classification-of-patterns)
- [Design principles](#design-principles)
  - [Encapsulate what varies](#encapsulate-what-varies)
    - [Encapsulation on a method level](#encapsulation-on-a-method-level)
    - [Encapsulation on a class level](#encapsulation-on-a-class-level)
  - [Program to an interface, not an implementation](#program-to-an-interface-not-an-implementation)
    - [Example](#example)
  - [Composition over inheritance](#composition-over-inheritance)
    - [Example](#example)
  - [SOLID principle](#solid-principle)

# Design patterns

# Object Oriented Programming - OOP

## Basics of OOP

- Is a paradigm.
- The idea is to wrap pieces of data and behavior related to that data into bundles called **objects**.
- The objects are a set of **"blueprints"** called **classes**.

## Objects and classes

- As a quick reminder of UML diagrams:

![UML_example](./Resources/UML_example.jpg)

- You have a cat named Oscar. Oscar is an object, an instance of the **cat** class.
>
- Every cat has standard attributes: name, sex, age, weight, etc. These are the **class's fields**.
- All cats behave similarly, they: sleep, eat, run, etc. These are the **class's methods**.
- Collectively, fields and methods can be referenced as the **members** of their class.
>
- Data stores inside the object's fields is often referenced as **state**, and all the object's methods define its **behavior**.

## Class hierarchies
- In a program we usually have more than one class. Some of these classes might be organized into **class hierarchies**.
- Now imagine a **Dog class**, turns out that dogs and cats have a lot in common: name, sex, age, etc. Dogs can sleep, eat and run, same as cats. So we can define the base **Animal class** that would list all the common attributes and behaviors. 
>
- A parent class is called **superclass** and its children are **subclasses**. 
- Subclasses inherit state and behavior from their parent, defining only attributes or behaviors that differ.
>
- In this example, the **cat class** would have the **meow method** and the **dog class** the **bark method**. 

![Hierarchy](./Resources/Hierarchy_cat_dog.jpg )

- We can go one level deeper and create a more general class for all living organisms. Then it will become a superclass for **animals** and **plants**. This pyramid of classes is a **hierarchy**.

![Hierarchy](./Resources/Hierarchy_organism.jpg)

- **Subclasses can override the behavior of methods** that they inherit from parent classes. 
- A subclass can either completely replace the default behavior or just enhance it with some extra stuff.

## Pillars of OOP
- Object oriented programming is based on four pillars:

### Abstraction
- Objects *model* attributes and behaviors of real objects in a specific context. For example, an **Airplane Class** could exist in a flight simulator and a flight booking application. But what they model is very different:

![Abstraction](./Resources/Abstraction.jpeg)

- **Abstraction** is a model of a real-world object or phenomenon, limited to a specific context, in which represents all details relevant to this context with high accuracy and omits all the rest.

### Encapsulation
- To start a car engine, you only need to turn a key or press a button. You've also have a steering wheel, a gear knob and some pedals. This illustrates how each object has an **interface** or in other words, a public part of an object open to interactions with other objects.
- *Encapsulation* is the ability of and object to hide parts of its state and behaviors from other objects. 

- *Encapsulate* something means to make it **private**, in other words accessible only from within the methods of its own class. There is a less restrictive mode called **protected** that makes a member of a class available to subclasses as well.

- Interfaces and abstract classes / methods are based on concepts of abstraction and encapsulation.

#### C++ example

- An interface describes the behavior or capabilities of a C++ class without committing to a particular implementation of that class. In C++, interfaces are implemented using abstract classes.

- A class is made abstract by declaring at least one of its functions as pure virtual function. A pure virtual function is specified by placing "= 0":

```
// Base class
class Shape {
   public:
      // pure virtual function providing interface framework.
      virtual int getArea() = 0;
      
      void setWidth(int w) {
         width = w;
      }
   
      void setHeight(int h) {
         height = h;
      }
   
   protected:
      int width;
      int height;
};
```
- The purpose of an **abstract class** (often referred to as an ABC) is to provide an appropriate base class from which other classes can inherit. Abstract classes cannot be used to instantiate objects and serves only as an **interface**. Attempting to instantiate an object of an abstract class causes a compilation error.

- Thus, if a subclass of an ABC needs to be instantiated, it has to implement each of the virtual functions, which means that it supports the interface declared by the ABC. Failure to override a pure virtual function in a derived class, then attempting to instantiate objects of that class, is a compilation error.

- Classes that can be used to instantiate objects are called **concrete classes**. Here is a continuation of the code above:

```
// Derived classes
class Rectangle: public Shape {
   public:
      int getArea() { 
         return (width * height); 
      }
};
```

- **Designing Strategy**
	- An object-oriented system might use an abstract base class to provide a common and standardized interface appropriate for all the external applications. Then, through inheritance from that abstract base class, derived classes are formed that operate similarly.

	- Imagine you have a **FlyingTransport** interface with a method **fly(origin, destination, passengers). When designing an air transportation simulation, you could restrict the **Airport** class to work only with objects that implement the **FlyingTransport** interface. After this, you can be sure that any object passed to an airport object would be able to arrive or depart from this type of airport.

	![Airport](./Resources/Airport.jpeg)

	- You could change the implementation of the **fly** method in these classes in any way you want (as long as the signature of the method remains the same as declared in the interface).

### Inheritance
- Is the ability to build new classes on top of existing ones. If you want to create a class that's slightly different from an existing one, you can extend the existing class and put extra functionality into a resulting subclass, which inherits fields and methods of the superclass (this avoids duplicating code).

- As a consequence you have the same interface as their parent class. 
	- You can't hide a method in a subclass if it was declared in the superclass. 
	- Also, you must implement all abstract methods, even if they don't make sense for tour subclass.

![Inheritance](./Resources/Inheritance.jpeg)

### Polymorphism
- Most **animals** can make sounds. We can anticipate that all subclasses will need to override the base **makeSound** method so each subclass can emit the correct sound.
- So we can declare it *abstract*, this let us omit any default implementation of the method in the superclass, but force all subclasses to come up with their own.

![Polymorphism](./Resources/Polymorphism.jpeg)

- Imagine the following scenario:
	- You have a bag with several cats and dogs, and after taking one out you don't know for sure what it is. But when it makes a sound, the animal emits a specific sound depending on its concrete class.
```
// Base class
class Animal 
{
   public:
    virtual void sound() = 0;// pure virtual function providing interface framework.
};
 
// Derived classes
class Cat: public Animal 
{
   public:
      void sound() 
      { 
         std::cout << "Meow!" << std::endl;
      }
};

class Dog: public Animal 
{
   public:
      void sound() 
      { 
         std::cout << "Woof!" << std::endl;
      }
};
 
int main(void) 
{
    std::vector<std::unique_ptr<Animal>> bag;
    bag.emplace_back(new Dog); bag.emplace_back(new Cat); bag.emplace_back(new Dog); 
    bag.emplace_back(new Cat); bag.emplace_back(new Dog); bag.emplace_back(new Dog);

    for (int i = 0; i < bag.size(); i++)
        bag[i]->sound();
        
    return 0;
}
```
```
Output:
Woof!
Meow!
Woof!
Meow!
Meow!
```
- The program doesn't know the concrete type of the object obtained, but thanks to a mechanism called *polymorphism*, the program can trace down the subclass of an object whose method is being executed and run the appropriate behavior.

- *Polymorphism* is the ability of a program to detect the real class of an object and call its implementation, even when its real type is unknown in the current context.

## Relations Between objects
In addition to *inheritance* and *implementation* there are other types of relations between objects. 

### Dependency
![Dependency](./Resources/Dependency.jpeg)
- There is a dependency between two classes if some changes to the definition of one class might result in modifications to another class. 
- Typically occur when you use concrete class names in your code. For example:
	- When you specify types in method signatures.
	- When instantiating objects via constructor calls.
- You can make a dependency weaker if you make your code dependent on interfaces or abstract classes instead of concrete classes.

### Association
![Association](./Resources/Association.jpeg)
- Is a relationship in which one objects uses or interacts with another. Is a specialized kind of dependency, where an object always has access to the objects with which interacts, whereas simple dependency doesn't establish a permanent link.

- Lets look a combined example to understand the differences between *association* and *dependency*. \
Imagine we have a `Professor` class:
```
#include <iostream>
#include <random>
#define MAX_GRADE 10

class Course 
{
    private:
        int m_performance;
        std::string m_name;
        
    public:
        Course(int performance, const char * name) : m_performance(performance), m_name(name) {}
        
        int test()
        {
            return rand()%(m_performance - 1);  
        }

        // This method is a consumer (doesn't modify fields of the class)
        // So const keyword must be used at the end        
        void printCourseName() const
        {
            std::cout << m_name << std::endl;    
        }
};

class Student 
{
    public:
        int takeTest(int performance)
        {
            return performance%(MAX_GRADE-1);   
        }
};

class Professor 
{
    Student student;

    public:
        void teach(Course& course) 
        { 
            std::cout << "You've got a " << student.takeTest(course.test()) << " on "; 
            course.printCourseName();
        }
};

int main()
{
    Professor professor;
    Course mathematics(1550, "Mathematics");
    Course geography(1700, "Geography");
    Course spanish(2000, "Spanish");
    
    professor.teach(mathematics);
    professor.teach(geography);
    professor.teach(spanish);
}
```
```
output:
You've got a 8 on Mathematics
You've got a 6 on Geography
You've got a 8 on Spanish
```

- Look at the `teach` method. It takes `Course` class as an argument, which then is used in the body of the method. If someone changes the `test` method of the `Course` class (alters its name, adds required parameters, etc), our code will break. This is called a dependency.

- Look at the `student` field and how it's used in the `teach` method. We can say that `Student` class is a dependency for professor. If `takeTest` changes, the professor's code will break. However, the `student` field is always accessible to any method of the `Professor`, the `Student` class is not just a dependency but also an association.

### Aggregation
![Aggregation](./Resources/Aggregation.jpeg)
- Is a specialized type of association that represents "one-to-many", "many-to-many" or "whole-part" relations between multiple objects.
- Usually, under aggregation, an object has a set of other objects and servers as a container or collection. The component can exist without the container and can be linked to several containers at the same time.

### Composition
![Composition](./Resources/Composition.jpeg)
- Is a specific kind of aggregation, whereas object is composed of one or more instances of the other. The distinction between this and other relations is that the component can only exist as part of the container.

## Summary
- **Dependency**: Class А can be affected by changes in class B.
- **Association**: Object А knows about object B. Class A depends
on B.
- **Aggregation**: Object А knows about object B, and consists of B.
Class A depends on B.
- **Composition**: Object А knows about object B, consists of B, and
manages B's life cycle. Class A depends on B.
B. Objects A can be treated as B. Class A depends on B.
- **Inheritance**: Class А inherits interface and implementation of
class B but can extend it. Objects A can be treated as B. Class
A depends on B

![Relations_summary](./Resources/Relations_summary.jpeg)

# Software design principles
Before going deep on design patterns, lets discuss the process of designing software architecture.

## Code reuse
Is one of the most common ways to reduce development costs. 

Making existing code work in a new context usually takes extra effort. You should avoid tight coupling between components, dependencies on concrete classes instead of interfaces, hard-coded operations.

Three levels of reuse:
- **Lowest level**: Classes
   - You reuse libraries, container and maybe some class "teams" like container/iterator.

- **Middle level**: Patterns
   - Smaller and more abstract that frameworks. They're a description about how a couple of classes can relate to and interact with each other.

- **High level**: Frameworks
   - They identify the key abstractions for solving a problem, represent them by classes and define relationships between them. 
   - Lets you define custom behavior and it will call you when it's your turn to do something.

What is good about the middle layer is that offers reuse but less risk. Building a framework is high-risk and a significant investment. Patterns let you reuse design ideas and concepts independently of concrete code.


# Design patterns

## What's a Design Pattern?
Are solutions to commonly occurring problems in software design. They are like pre-made blueprints that you can customize to solve a recurring design problem in your code.

The pattern is not an specific piece of code, but a general concept. You can follow the pattern details and implement a solution that suits the realities of your own program.

## What does the patter consist of?
- **Intent** of the patter briefly describes both the problem and the solution.
- **Motivation** further explains the problem and the solution the pattern makes possible.
- **Structure** of classes shows each part of the pattern and how they are related.
- **Code example** in one or more programming languages.

## Classification of patterns
Design patterns differ by their complexity, level of detail and scale of applicability to the entire system being designed.

- The most basic and low-level patterns are called *idioms* and they usually apply only to a  single programming language.

- The most universal and high-level patterns are called *architectural patterns*. These patterns can implement these patterns in virtually any language.

We can divide patterns in three groups:
- **Creational patterns** provide object creation mechanisms that increase flexibility and reuse of existing code.
- **Structural patterns** explain how to assemble objects and classes into larger structures, while keeping the structures flexible and efficient.
- **Behavioral patterns** take care of effective communication and the assignment of responsibilities between objects.

# Design principles

## Encapsulate what varies

> **Identify the aspects of your application that vary and separate them from what stays the same.**

The goal of this principle is to minimize the effect caused by changes. You can isolate parts of the program that vary in independent modules, protecting the rest of the code from adverse effects.

### Encapsulation on a method level
Imagine you're making an e-commerce website. And you implement a `getOrderTotal` method that calculates a grand total for the order, including taxes.

We can anticipate that tax-related code might need to change in the future. Overtime, the actual formula for tax calculation may change due to new laws or regulations. As a result, you'll need to change the `getOrderTotal` method quite often. But even the method's name suggests that it doesn't care about *how* taxes are calculated:

```
class Commerce 
{
    public:
        float getOrderTotal(Order& order)
        {
            float total = 0;
            float totalPreTax = order.sumAllProducts();
            
            if("EEUU" == order.country())
            {
                total = totalPreTax * 1.07;
            }
            else if ("EU" == order.country())
            {
                total = totalPreTax * 1.20;
            }
            else
            {
               total = totalPreTax;
            }
                
            return total;
        }
};

// Tax calculation is mixed with the rest of the method's code
```
You can extract the tax calculation logic into a separated method, hiding it from the original one:
```
class Commerce 
{
    public:
        float addTax(float preTaxValue, Order& order)
        {
            if("EEUU" == order.country())
            {
                total = totalPreTax * 1.07;
            }
            else if ("EU" == order.country())
            {
                total = totalPreTax * 1.20;
            }
            else
            {
                total = totalPreTax;    
            }
                
            return total;    
        }
        
        float getOrderTotal(Order& order)
        {
            float totalPreTax = order.sumAllProducts();
            
            return addTax(totalPreTax, order);
        }
};

// You can get the tax rate by calling a designation method.
```
> NOTE: see that we respect the [ROV](https://shaharmike.com/cpp/rvo/)

Tax-related changes become isolated inside a single method. Moreover, if the tax calculation logic becomes too complicated, it's now easier to move it to a separated class.

### Encapsulation on a class level
Over time you add more and more responsibilities to a method which used to do a simple thing. And this added behaviors eventually blur the primary responsibility of the containing class. Extracting everything to a new class might make things much more clear and simple.

![Encapsulation_class_level](./Resources/Encapsulation_class_level.jpeg)

## Program to an interface, not an implementation
You can tell that the design is flexible enough if you can easily extend it without breaking any existing code. A `Cat` that can eat any food is more flexible that one that can eat just sausages.

When you want to make two classes collaborate, you can start by making one of them dependent on the other, but a more flexible way is to set up collaboration between objects:
1. Determine what exactly one object needs from the other (Which methods does it execute?).
2. Describe these methods in a new interface or abstract class.
3. Make the class that is a dependency implement this interface.
4. Make the second class dependent on this interface rather than on the concrete class. You can make it work with objects of the original class, but the connection is now much more flexible.

![Interface_not_implementation_principle](./Resources/Interface_not_implementation/Principle.jpeg)

At first glance it seems that you only created a more complicated code, but when you or someone wants do add more functionality, this will come in handy.

### Example
Say you created a *software development company simulator*. You have different classes that represent various employee types:

![Interface_not_implementation_tightly_coupled](./Resources/Interface_not_implementation/Tightly_coupled.jpeg)

The `Company` class is tightly coupled to a concrete class of employees. Despite the difference in their implementation, we can generalize various work-related methods and extract a common interface for all employee classes.

Also, we can apply polymorphism inside the `Company` class, treating various employee objects via the `Employee` interface:

![Interface_not_implementation_polymorphism](./Resources/Interface_not_implementation/Polymorphism.jpeg)

The `Company` class remains coupled to the employee classes. This is bad, if we introduce new types of companies that work with other types of employees, we'll need to override most of the `Company` class instead of reusing code.

To solve this problem, we declare the method for getting employees as *abstract*. Each company will implement this method differently, creating only those employees that it needs.
![Interface_not_implementation_factory_method](./Resources/Interface_not_implementation/Factory_method.jpeg)

Now, the `Company` class become independent from various employee classes and can extend this class to introduce new types of companies and employees while still reusing a portion of the base company class. Extending the base company doesn't break any existing code that already relies on it.

> NOTE: This is a pattern! Refereed as [Factory method](https://refactoring.guru/design-patterns/factory-method) (more on that latter).

## Composition over inheritance
Inheritance is the most obvious and easy way of reusing code between classes. You have two classes with the same code, create a common base class for those two and move the similar code into it.

Unfortunately, inheritance comes with caveats that become apparent when you program has tons of classes:

- **A subclass can't reduce the interface of a supperclass**:
   - You have yo implement all abstract methods of the parent class, even if you won't use them.
- **When overriding methods you need to make sure that the new behavior is compatible with the base one**:
   - Objects of the subclass may be passed to any code that expects objects of the superclass and you don't want that code to break.
- **Inheritance breaks encapsulation of the superclass**:
   - Internal details of the parent class become available to the subclass.
- **Subclasses are tightly coupled to superclasses**:
   - Any change in a superclass may break the functionality of subclasses.
- **Trying to reuse code through inheritance can lead to creating parallel inheritance hierarchies**:
   - Inheritance usually takes place in a single dimension. But when there are two or more dimensions, you have to create lots of class combinations, bloating the class hierarchy.

An alternative to *inheritance* is *composition*.
   - Inheritance represent the "is a" of a relationship (a car *is a* transport).
   - Composition represent the "has a" of a relationship (a car *has an* engine).

> NOTE: this principle also applies to aggregation.

### Example
You need to create a catalog app for a car manufacturer. The company makes both cars and trucks; they can be either electrical or gas; all models have either manual control or autopilot.

![Composition_over_inheritance_inheritance](./Resources/Composition_over_inheritance/Inheritance.jpeg)

Each additional parameter results in multiplying the number of subclasses. There's a lot of duplicated code between subclasses, because you can't extend two classes at the same time.

With composition, instead of a car objects implementing a behavior on their own, they can delegate it to other objects.

The added benefit is that you can replace a behavior at runtime. You can replace an engine object linked to a car object just by assigning a different engine object to the car.

![Composition_over_inheritance_composition](./Resources/Composition_over_inheritance/Composition.jpeg)

> NOTE: this structure of classes resembles the [Strategy pattern](https://refactoring.guru/design-patterns/strategy).

## SOLID principle