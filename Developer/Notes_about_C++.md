# Table of contents

- [Table of contents](#table-of-contents)
- [Notes about C++](#notes-about-c)
  - [The stack and the heap](#the-stack-and-the-heap)
  - [The static](#the-static)
    - [Characteristics](#characteristics)
    - [The stack](#the-stack)
    - [The heap](#the-heap)
  - [Function pointers](#function-pointers)
  - [Lambdas](#lambdas)
  - [lvalues and rvalues](#lvalues-and-rvalues)
    - [What is this all for?](#what-is-this-all-for)
  - [Initialization](#initialization)
      - [Conclusion](#conclusion)
  - [Static](#static)
    - [Static Variables](#static-variables)
      - [Initialization of Static Variables](#initialization-of-static-variables)
        - [Two stages of static variable initialization](#two-stages-of-static-variable-initialization)
      - [The green Zone - Constant initialization](#the-green-zone---constant-initialization)
        - [Force Const Initialization - *constexpr*](#force-const-initialization---constexpr)
        - [Your Second Line of Defense - *constinit*](#your-second-line-of-defense---constinit)
      - [The Yellow Zone - Dynamic Initialization](#the-yellow-zone---dynamic-initialization)
      - [The Red Zone - Static Initialization Order Fiasco](#the-red-zone---static-initialization-order-fiasco)
        - [Solving The Static Initialization Order Fiasco](#solving-the-static-initialization-order-fiasco)
    - [Summary](#summary)
    - [Static methods](#static-methods)
- [Casting](#casting)
  - [Dynamic casting](#dynamic-casting)
- [Exception handling](#exception-handling)
  - [Re-throwing exceptions](#re-throwing-exceptions)
    - [Throw a new exception](#throw-a-new-exception)
    - [The wrong way](#the-wrong-way)
    - [The right way](#the-right-way)
- [Good practices](#good-practices)
  - [Pointers](#pointers)
    - [Passing objects](#passing-objects)
    - [Smart pointers](#smart-pointers)
      - [RAII (Resource acquisition is initialization)](#raii-resource-acquisition-is-initialization)
      - [std::unique\_ptr](#stdunique_ptr)
      - [Raw pointers](#raw-pointers)
      - [std::shared\_ptr](#stdshared_ptr)
      - [std::weak\_ptr](#stdweak_ptr)
      - [boost::scoped\_ptr](#boostscoped_ptr)
      - [std::auto\_ptr](#stdauto_ptr)
- [Optimizations](#optimizations)
  - [++prefix forms are preferred against postfix++](#prefix-forms-are-preferred-against-postfix)
  - [Return Value Optimization (RVO)](#return-value-optimization-rvo)
  - [Name Return Value Optimization (NRVO)](#name-return-value-optimization-nrvo)
  - [Clearer interfaces](#clearer-interfaces)
    - [What is optional?](#what-is-optional)
- [Standard Algorithms](#standard-algorithms)
  - [Move semantics](#move-semantics)
- [Interesting implementations](#interesting-implementations)
  - [Defer](#defer)

# Notes about C++

## The stack and the heap
C++ has several types of memories that correspond to different parts of the physical memory:
- The static
- The stack
- The heap

## The static
Region where objects with static storage duration are stored. Objects in this region have their lifetime managed by the runtime system and exist for the entire duration of the program.

### Characteristics

1. **Global Scope**: Variables declared outside of any function (global variables) are stored in the static region. These variables are accessible throughout the entire program.
  ```C++
  int globalVar = 10; // Stored in the static region
  ```

2. **Static Variables**: Variables declared with the static keyword within functions or classes also reside in the static region. These variables retain their value between function calls or across different instances of a class.
  ```C++
  void exampleFunction() {
      static int counter = 0; // Stored in the static region
      counter++;
      std::cout << counter << "\n";
  }
  ```

3. **Initialization**: Static variables are initialized only once, the first time control passes through their declaration. If not explicitly initialized, they are zero-initialized.
  ```C++
  static int uninitializedVar; // Zero-initialized to 0
  static int initializedVar = 42; // Initialized to 42
  ```

4. **Lifetime**: The lifetime of static variables spans the entire execution of the program. They are created before the main function starts and destroyed after the main function ends.

5. **Storage**: Static storage is not part of the stack or heap. It has a fixed location in memory that remains constant throughout the program's execution.

### The stack
The default way to store objects in C++:
```C++
int f(int a)
{
    if (0 < a)
    {
        std::string s = "a positive number";
        std::cout << s << "\n";
    }
    return a;
}
```

Here **a** and **s** are stored (pushed) on the stack, also they are next to one another in memory, hence the name stack. The most important part is that **objects allocated on the stack are automatically destroyed when they go out of scope**.

The scope is defined by a pair of **{}**, except those used to initialize objects: 
```C++
std::vector<int> v = {1, 2, 3}; // this is not a scope

if (0 < v.size())
{ // this is the beginning of a scope
    ...
} // this is the end of a scope
```

There are three ways for objects to go out of scope:
- Encountering the next closing bracket **}**.
- Encountering a return statement.
- Having an exception thrown inside the current scope that is not caught inside the current scope.

### The heap
The heap is where dynamically allocated objects are stored, that is to say, **objects that** are allocated with a call to new**, which returns a pointer. In reality, the heap is the memory allocated by **malloc**, **calloc** and **realloc**, and new stores it in the **free store** (but the term is used anyways).

Objects stored on the heap are **not destroyed automatically** but when **delete** is called. This offers the advantage of keeping them longer than the end of a scope, and without incurring any copy except those of pointers (which are cheap). 

> NOTE: Pointers allow to manipulate objects polymorphically. A pointer to a base class can point to objects of any derived class.

The price for this flexibility is that the developer is in charge of the deletion of these objects also called deallocation. Deleting an object on the heap is not trivial, because delete has to be called **once and only once**, doing so leads to *undefined behavior*. If the memory is not deallocated, then this memory space is not reusable, this is called a *memory leak*.

As you can see, this is a problem. Later we will see the concept of [smart pointers](#smart-pointers) which will help us with this task.

## Function pointers
A way to assign a function to a variable.

## Lambdas
Whenever you have a function pointer, you can instead use a lambda, but without defining a function.

## lvalues and rvalues
https://www.fluentcpp.com/2018/02/06/understanding-lvalues-rvalues-and-their-references/
In C++, every expression is either an lvalue or an rvalue:
- An **lvalue** denotes an object whose resources cannot be reused. 
  - This includes expressions that designate objects directly by their *names*, but not only.
    - In `int y = f(x)`, x and y are objects names and are lvalues).
    - In the expression `myVector[0]` is also an lvalue.
- An **rvalue** denotes an object whose resources can be reused.
  - This typically includes *temporary objects* as they can't be manipulated at the place they are created and are soon to be destroyed.
    - In the expression `g(MyClass())`, `MyClass()` designates a temporary object that `g` can modify without impacting the code surrounding the expression.

- An **lvalue reference** is a reference that binds to an lvalue. They are marked with one ampersand.
- An **rvalue reference** is a reference that binds to an rvalue. They are marked with two ampersands.

> NOTE: There is one exception, it can be an lvalue const reference binding to an rvalue. 

### What is this all for?
rvalue references add the possibility to express a new intention in code: **disposable objects**. Passing objects as a reference means *you no longer care about them*.
```C++
void f(MyClass&& x)
{
  ...
}
```

The message is "the object that `x` binds to is YOURS. Do whatever you like with it". It's a bit like giving a copy to f, but without the copy.

This can be interesting for two purposes:
- Improving performance (see move constructors).
- Taking over ownership (since the object the reference binds to has been abandoned by the caller).

Note that this could not be achieved with lvalue references. For example:
```C++
void f(MyClass& x)
{
  ...
}
```

You can modify the value of the object that x binds to, but since it is an lvalue reference, it means that somebody probably cares about it at the call site.

//TODO

## Initialization
If an initializer is specified for an object, that initializer determines the initial value of an object. We can use one of the four syntactic styles:
```C++
X a1 {v};
X a2 = {v};
X a3 = v;
X a4(v);
```

Of these, only the first one can be used in every context and is less error-prone.

```C++
void fun(double val, int val2) 
{

    int x2 = val;    // if val == 7.9, x2 becomes 7 (bad)

    char c2 = val2;  // if val2 == 1025, c2 becomes 1 (bad)

    int x3 {val};    // error: possible truncation (good)

    char c3 {val2};  // error: possible narrowing (good)

    char c4 {24};    // OK: 24 can be represented exactly as a char (good)

    char c5 {264};   // error (assuming 8-bit chars): 264 cannot be 
                     // represented as a char (good)

    int x4 {2.0};    // error: no double to int value conversion (good)
}
```

#### Conclusion
Prefer {} initialization over alternatives unless you have a strong reason not to.

## Static
The meanings change depending on context and we have three scenarios:
- **Static Variables**: 
  - In a function.
  - In a class.
- **Static methods in a Class**.

### Static Variables
- In a function:
  - The variable gets allocated for the lifetime of the program. 
  - Their scope is internal, and only visible in the translation unit.

- In a class:
  - Are initialized only once and shared between all instances of the objects (of the same class). 
  - You need to initialize static variables outside of the class definition. Static variables can not be **initialized** using constructors or inline.
  - This variable is no longer associated with a specific object but with the class. Useful when we need to describe the whole population of classes (for example amount of instances).
  - It's a good idea to implement static methods to access static variables.

#### Initialization of Static Variables
- There is a category of variables that can (and should) be initialized before the program starts:
  - Static variables.
  - Global (namespace) variables.
  - Static class members.
- They live for the entire execution of the program and must be initialized before `main()` is run and destroyed after execution is finished, this is called *Static storage duration*.

##### Two stages of static variable initialization
Variables with *Static storage duration* must be initialized once before the program start and destroyed after execution terminates. Initialization could happen in two consecutive stages:
- Static initialization:
  - Happens first and usually at compile time. If possible, initial values for static variables are evaluated during compilation and burned into the data section of the executable.
  - Zero runtime overhead, early problem diagnosis and safety are advantages of this called [constant initialization](https://en.cppreference.com/w/cpp/language/constant_initialization). Ideally, all static variables are *const-initialized*.
  - If the initial value of the static variable can't be evaluated at compile time, the compiler will perform *zero-initialization*. So, during static initialization, all static variables are either *const-initialized* or *zero-initialized*.
- Dynamic initialization:
  - Takes place after static initialization. Happens at runtime for variables that can't be evaluated at compile time. Here, static variables are initialized every time the executable is run and not just during compilation.

#### The green Zone - Constant initialization
Is ideal, and the compiler will try to perform it whenever it can. This is the case when your variable is initialized by a [constant [expression](https://en.cppreference.com/w/cpp/language/constant_expression), which is an expression that can be evaluated at compile time.

```C++
struct MyStruct
{
    static int a;
};
int MyStruct::a = 67; 
```

##### Force Const Initialization - *constexpr*
It's not always clear if a variable is being initialized at compile time or at runtime. To make sure variables are initialized at compile time by declaring them `constexpr`.

```C++
struct Point
{
    float x{};
    float y{};
};

constexpr Point movePoint(Point p, float x_offset, float y_offset)
{
    return {p.x + x_offset, p.y + y_offset};
}

// OK const-initialized, enforced by compiler
constexpr auto P2 = movePoint({3.f, 4.f}, 1.f, -1.f);

// still const-initialized, but not enforced by compiler
auto P1 = P2;

struct Line
{
   Line(Point p1, Point p2)
   {
   }
   // ...
};

// Compile error, l can't be const-initialized (no constexpr constructor)
constexpr Line l({6.7f, 5.7f}, {5.4, 3.2});
```

`constexpr` must be your first choice when declaring global variables (assuming you need a global state, to begin with). `constexpr` variables are not just initialized at compile time, but constexpr implies const and immutable state is always the right way.

##### Your Second Line of Defense - *constinit*
Is a keyword introduced in the **C++20 standard**. Works just as `constexpr`, as it forces the compiler to evaluate a variable at compile time, but it **doesn't imply const**.

Variables declared `constinit` are always const-zero or const-initialized, but can be mutated at runtime.

```C++
constexpr auto N1{54}; // OK const-init ensured
constinit auto N2{67}; // OK const-init ensured, mutable

int square(int n)
{
  return n * n;
}
constinit auto N3 = square(N2); // Compilation error square(N2) not constant expression

int main()
{
  ++N2; // OK
  ++N1; // Compilation error
  return EXIT_SUCCESS;
}
```

#### The Yellow Zone - Dynamic Initialization
Imagine you need an immutable global `std::string` to store the software version. You don't want this object to be instantiated every time the program runs, but rather create it once and embed it into the executable as read-only memory. In other words, you want a `constexpr`:

```C++
constexpr auto VERSION = std::string("3.1.4");
```

The problem is that the compiler will tell you:

> error: constexpr variable cannot have non-literal type

The compiler complains because `std::string` defines a non-trivial destructor. This means that `std::string` is allocating some resources that must be freed on destruction, memory in this case. In other words, `std::string("3.1.4")` is not a constant expression. A workaround is the following:

```C++ 
const auto VERSION = std::string("3.4.1");
```

The compiler doesn't complain but at the cost of moving the initialization to runtime. Now `VERSION` must be part of dynamic initialization, not static. This approach is less efficient and less safe than static initialization (not that bad either).

#### The Red Zone - Static Initialization Order Fiasco
The problem with *dynamic initialization* is that the order in which variables are initialized at runtime is not always well defined.

Within a single compilation unit, static variables are initialized in the same order as they are defined in the source (*Ordered Dynamic Initialization*). 

Across compilation units, the order is undefined, and this is an issue if the initialization of a variable in `a.cpp` depends on another defined in `b.cpp`. This is called [Static Initialization Order Fiasco](https://isocpp.org/wiki/faq/ctors#static-init-order):

```C++
// a.cpp
int duplicate(int n)
{
    return n * 2;
}
auto A = duplicate(7); // A is dynamic-initialized

// b.cpp
#include <iostream>

extern int A;
auto B = A; // dynamic initialized

int main()
{
  std::cout << B << std::endl;
  return EXIT_SUCCESS;
}
```

This program is ill-formed. It may print `14` or `0` (all static variables are at least zero-initialized), depending if the dynamic initialization of `A` happens before `B` or not.

**NOTE**: This can only happen during the dynamic initialization phase. Is impossible to access values defined in another compilation unit on static initialization (and this is the safety part that we talk before).

##### Solving The Static Initialization Order Fiasco
Encountering this problem is often a symptom of poor software design. The best way to solve it is to refactor the code to break the initialization dependency of global across compilation units. Modules should be self-contained and strive for constant initialization.

If refactoring is not an option, one solution is the [initialization on first use](https://isocpp.org/wiki/faq/ctors#static-init-order-on-first-use). The basic idea is to create static variables that are initialized at runtime when it is the first time they are accessed. 

> Side note: This idea is used on the [singleton](https://refactoring.guru/design-patterns/singleton).

With this strategy it is possible to control the time when static variables are initialized at runtime, avoiding use-before-init.

```C++
// a.cpp
int duplicate(int n)
{
    return n * 2;
}

auto& A()
{
  static auto a = duplicate(7); // Initiliazed first time A() is called
  return a;
}


// b.cpp
#include <iostream>
#include "a.h"

auto B = A();

int main()
{
  std::cout << B << std::endl;
  return EXIT_SUCCESS;
}
```

This program will always consistently print `14`, as it is guaranteed that `A` will always be initialized before `B`.

### Summary
- Static variables must be initialized before the program starts.
- Variables that can be evaluated at compile time (those initialized by a constant expression) are const-initialized.
- All other static variables are zero-initialized during static initialization.
- `constexpr` forces the evaluation of a variable as a constant expression and implies const.
- `constinit` forces the evaluation of a variable as a constant expression and doesn't imply const.
- After static initialization, dynamic initialization takes place, which happens at runtime before main().
- Within a compilation unit static variables are initialized in the order of declaration.
- The order of initialization of static variables is undefined across compilation units.
- You can use the *Initialization On First Use Idiom* to circumvent the *static initialization order fiasco*.


### Static methods
They don't depend on the objects of the class. We are allowed to invoke static methods using the object and the `.` or `->` operators, but it's recommended to invoke static methods using the class name and the scope resolution operator`::`.

```C++
class User
{
    static int users_ammout;
      
    public:
        User()
        {
            std::cout << "\nNew user added - " << "User amount " << ++users_ammout;
        }
          
        ~User()
        {
            std::cout << "\nUser deleted   - " << "User amount " << --users_ammout;
        }
        
        static void count_users()
        {
           std::cout << "\nUser amount " << users_ammout;
        }
};
int User::users_ammout = 0;
  
int main()
{
    User::count_users();
    User * usr = new User();
    usr->count_users();
    delete usr;
}
```

# Casting

## Dynamic casting


# Exception handling
An exception is a problem that arises at runtime.

Exceptions provide a way to transfer control from one part of a program to another. C++ exception handling is built upon three keywords:
- **throw**: 
  - Throws an exception in any code block. 
  - The operand could be any type of expression.
- **try**:
  - Encloses a protected block of code.
- **catch**:
  - Follows a **try** block.
  - You can specify what type of exception you want to catch.
  - You could have different catch statements for different exception types.
 
```C++
#include <iostream>

void fun(auto val)
{
    throw val;
}

int main()
{
    try {
        fun(1.2);
    }
    catch (const char* err) { // Could catch fundamental types
        std::cout << "const char * " << err;
    }
    catch (std::string err) { // Could catch class types
        std::cout << "std::string " << err;
    }
    catch (int) { // No need to grab the throw value
        std::cout << "int";
    }
    catch (...) { // Catch any exception
        std::cout << "unknown type";
    }

    return 0;
}
```

## Re-throwing exceptions
Occasionally you may run into a case where you want to catch an exception but do not want to (or have the ability to) fully handle it at the point where you catch it. This is common when you want to log an error, but pass the issue along to the caller to handle it.

### Throw a new exception

```C++
void fun()
{
  try
  {
    insideFun(); // throws int exception on failure
  }
  catch (int exception)
  {
    logError("error in fun");
    
    throw "q"; // throw char exception "q" up the stack to be handled by caller of fun()
  }
}
```
- The exception from the catch block can be an exception of any type. It doesn't need to be the same type as the one that was caught.

### The wrong way

```C++
void fun()
{
  try
  {
    insideFun(); // throws int exception on failure
  }
  catch (int exception)
  {
    logError("error in fun");
    
    throw exception;
  }
}
```
Although this works, this method has a couple of downsides. 

- This doesn't throw the same exception as the one that is caught, but rather a copy-initialized copy (this could be less performant).

- But the worst scenario arises here:

```C++
void fun()
{
  try
  {
    insideFun(); // throws int exception on failure
  }
  catch (Base& exception)
  {
    logError("error in fun");
      
    throw exception; // Danger: this throws a Base object, not a Derived object
  }
}
```

In this case, `insideFun()` throws a **Derived object**, but the catch block is getting a **Base reference**. When we re-throw the exception it's doing so with a copy-initialized exception of type Base (not derived). Our Derived object has been sliced!

```C++
#include <iostream>

class Base
{
public:
    Base() {}
    virtual void print() { std::cout << "Base"; }
};

class Derived: public Base
{
public:
    Derived() {}
    void print() override { std::cout << "Derived"; }
};

int main()
{
    try
    {
        try
        {
            throw Derived{};
        }
        catch (Base& b)
        {
            std::cout << "Caught Base b, which is actually a ";
            b.print();
            std::cout << "\n";
            throw b; // the Derived object gets sliced here
        }
    }
    catch (Base& b)
    {
        std::cout << "Caught Base b, which is actually a ";
        b.print();
        std::cout << "\n";
    }

    return 0;
}
```

> OUTPUT: \
Caught Base b, which is actually a Derived \
Caught Base b, which is actually a Base 


### The right way
C++ provides a way to re-throw the same exception that was caught:

```C++
#include <iostream>

class Base
{
public:
    Base() {}
    virtual void print() { std::cout << "Base"; }
};

class Derived: public Base
{
public:
    Derived() {}
    void print() override { std::cout << "Derived"; }
};

int main()
{
    try
    {
        try
        {
            throw Derived{};
        }
        catch (Base& b)
        {
            std::cout << "Caught Base b, which is actually a ";
            b.print();
            std::cout << "\n";
            throw b; // the Derived object gets sliced here
        }
    }
    catch (Base& b)
    {
        std::cout << "Caught Base b, which is actually a ";
        b.print();
        std::cout << "\n";
    }

    return 0;
}
```

```bash
OUTPUT:
Caught Base b, which is actually a Derived
Caught Base b, which is actually a Derived
```

And this has the advantage that no copies are made. So no performance-killing copies or slicing.

# Good practices

## Pointers
Pointers have many advantages:
- Much smaller than objects.
- Used on dynamic memory allocation.
And disadvantages.
- Double free.
- Free on de-referenced pointer.
- De-reference on invalid memory address (0x00000001 -> null check won't help).
If we talk about raw pointers, we should avoid their use entirely and use some C++ idiomatic features.

### Passing objects 
C++ introduce the concept of reference, which by design is _not null_*, that allows passing large objects with minimal cost. And returning objects by value benefits from the [NRVO](#name-return-value-optimization-nrvo) and [move semantics](#move-semantics) to allow minimal cost in many cases. 

### Smart pointers
They essentially encapsulate all of the memory management, including the need to call delete at all.

#### RAII (Resource acquisition is initialization)
Is a very idiomatic concept in C++, that takes advantage of the property of the stack to simplify the memory management of objects on the heap. 

The principle is simple: wrap a resource (pointer for instance) into an object, and dispose of the resources in its destructor. This is what smart pointers do:

```C++
// This is just to grasp the concept of RAII. 
// Is not the complete interface of a smart pointer.

template <typename T>
class SmartPointer
{
public:
    explicit SmartPointer(T* p) : p_(p) {}
    ~SmartPointer() { delete p_; }

private:
    T* p_;
};
```

You can manipulate *smart pointers* as objects allocated on the stack, and the compiler takes care of automatically calling the destructor when out of scope.

Syntactically, a *smart pointer* behaves like a raw pointer, it can be dereferenced with operators `*` or `->` and you can test for nullity. The underlying pointer itself is accessible with a **.get()**.

One of the most important aspects is that the above interface doesn't deal with a copy! A **SmartPointer** copied also copies the underlying pointer, so the following code has a bug:
```C++
{
    SmartPointer<int> sp1(new int(42));
    SmartPointer<int> sp2 = sp1; // now both sp1 and sp2 point to the same object
} // sp1 and sp2 are both destroyed, the pointer is deleted twice!
```

How do we deal with a copy? Well, the various types of *smart pointers* differ in this. But this lets us express our intentions in code a lot better.

The various types of pointers are there to **express a design** in the code. Here are the types of pointers sorted by decreasing order of usefulness (according to someone):
- std::unique_ptr
- raw pointer
- std::shared_ptr
- std::weak_ptr
- boost::scoped_ptr
- std::auto_ptr

#### std::unique_ptr
This pointer is the sole owner of a memory resource. It will hold a pointer and delete it in its destructor (unless you customize it).

This allows you to express your intentions in an interface. Consider the following function:

```C++
std::unique_ptr<House> buildAHouse();
```

It gives you a pointer to a house, of which you are the owner. *No one else will delete this pointer*, except the unique_ptr returned from the function. 

> NOTE: std::unique_ptr is the preferred pointer to return from a **factory** function.
  

Note though that even when you receive a unique_ptr, you're not guaranteed that no one else has access to this pointer. If another context keeps a copy of the pointer inside your unique_ptr, then modifying the pointed object through the unique_ptr object will impact this other context. If you don't want this to happen, you can express it in the following way:

```C++
std::unique_ptr<const House> buildAHouse(); // for some reason, I don't want you
                                            // to modify the house you're being passed
```

To ensure only one unique_ptr owns a memory resource, std::unique_ptr cannot be copied. But the ownership can be transferred by moving a unique_ptr into another one.

```C++
std::unique_ptr<int> p1 = std::make_unique(42);
std::unique_ptr<int> p2 = move(p1); // now p2 hold the resource
                                    // and p1 no longer hold anything
```

#### Raw pointers
They share a lot with references, but the latter is preferred (except in some cases). The important thing to focus on is that **raw** pointers and references represent access to an object, but not ownership**. And this is the default way of passing objects to functions and methods.

This is relevant when you hold an object with an unique_ptr and want to pass it to an interface. You don't pass the unique_ptr, nor a reference to it, but a reference to the pointed object.

```C++
std::unique_ptr<House> house = buildAHouse();
renderHouse(*house);
```

#### std::shared_ptr
A single memory resource can be held by several std::shared_ptr at the same time. Internally maintains a count of how many of them are holding the same resource and when the last one is destroyed, it deletes the memory resource. Therefore allows copies, but with a reference-counting mechanism. 

They should not be used by default, because having several simultaneous holders of a resource:
- Makes for a more complex system.
- Makes thread safety harder.
- Makes the code counter-intuitive when an object is not shared in terms of the domain.
- Can produce performance costs, both memory and time.

One good case for using it is when objects are **shared in the domain**. 

![Shared_ptr](./Resources/Notes_about_C++/Shared_ptr.jpg)

#### std::weak_ptr
Can hold a reference to a shared object, but they don't increment the reference count. If no std::shared_ptr is holding an object, it will be deleted even if some weak pointers still point to it.

For this reason, weak_ptr needs to check if an object is still alive. For this it has to be copied into a shared_ptr:

```C++
void useMyWeakPointer(std::weak_ptr<int> wp)
{
    if (std::shared_ptr<int> sp = wp.lock())
    {
        // the resource is still here and can be used
    }
    else
    {
        // the resource is no longer here
    }
}
```

A typical use case for this is about breaking shared_ptr circular references. Consider the following code:

```C++
struct House
{
    std::shared_ptr<House> neighbour;
};

std::shared_ptr<House> house1 = std::make_shared<House>();
std::shared_ptr<House> house2 = std::make_shared<House>();;
house1->neighbour = house2;
house2->neighbour = house1;
```

None of the houses ends up being destroyed at the end of this code, because the shared_ptrs points into one another. But if one is a weak_ptr instead, there is no longer a circular reference.

#### boost::scoped_ptr
It disables the copy and even the move construction. It is the sole owner of a resource, and its owners cannot be transferred. Therefore, a scoped_ptr can only live inside a scope or as a data member of an object.

#### std::auto_ptr
Deprecated in C++11 and removed in C++17.

# Optimizations

## ++prefix forms are preferred against postfix++
This is due to the overhead presented on the former:

```C++
T& T::operator++()                T& T::operator--()      // the prefix form:
{                                 { 
  // perform increment              // perform decrement  // - do the work
  return *this;                     return *this;         // - always return *this;
}                                 }

T T::operator++(int)              T T::operator--(int)    // the postfix form: 
{                                 { 
  T old( *this );                   T old( *this );       // - remember old value
  ++*this;                          --*this;              // - call the prefix version
  return old;                       return old;           // - return the old value
}                                 } 
```

## Return Value Optimization (RVO)
This happens when two conditions are met: 
- The caller allocates space on the stack for the return value and passes the address to the callee.
- The Callee constructs the result **directly** in that space.

> NOTE: at the assembly level, the compiler always passes at least one parameter, this is the return address.

RVO can still apply even when the function has several return statements, as long as the returned objects are created on the return statements, therefore this object does not have a name:

```C++
T f()
{
  if (...)
  {
    return T(...);
  }
  else
  {
    return T(...);
  }
}
```

## Name Return Value Optimization (NRVO)
It can remove the intermediary objects even if the returned object has a name and is therefore not constructed on the return statement. So this object can be constructed before the return statement, like in the following example:

```C++
T f()
{
  T result(...);
  ...
  return result;
}
```

But, like with the RVO, the function still needs to return a unique object (which is the case in the above example), so that the compiler can determine which object inside of `f` it has to construct at the memory location of t (outside of f).

For example, the NRVO can still be applied in the following case, because only one object (result) can be returned from the function.

```C++
T f()
{
  T result(...);
  if (...)
  {
    return result;
  }
  ...
  return result;
}
```

**FINAL NOTE**
You can always try to facilitate RVO and NRVO by **returning only one object** from all the return paths of your functions, and by **limiting the complexity** in the structure of your functions.

## Clearer interfaces
Sometimes we need to represent a value that is "empty", "null" or "not set" on the return of a function. There are many approaches to this:
- **Return a special value**, like -1 where a positive integer is expected or "". 
  - This is brittle because these values may actually be meaningful, now or later, or be set by accident.
- **Returning a boolean or an **error code** indicating whether the function has succeeded and the result is passed through a function parameter.
  - This is brittle and clumsy because nothing enforces that the caller checks the return value and makes the surrounding code harder to write and read.
- **Throwing an exception**. This is good but not always usable because the surrounding code has then to be exception-safe.

One alternative is the use of **optional<T>**.

### What is optional?
For a given type T, optional<T> represents an object that can be:
- A value of type T
- An empty value

This way a new value is added to the possible values that T can hold (replacing the use of -1 or "") to represent "empty" or "not set".

An example of use could be:

```C++
#include <iostream>
#include <optional>
#include <string>

std::optional<std::string> fun(int control)
{
    if (0 == control)
    {
        return "Success";
    }

    return std::nullopt;
}

int main()
{
  auto var = fun(0);
  if (var)
  {
    std::cout << var.value() << std::endl;
  }
  else
  {
    std::cout << "not set" << std::endl;
  }
}
``` 

# Standard Algorithms


## Move semantics

# Interesting implementations

## Defer
When writing [RAII-compliant](#raii-resource-acquisition-is-initialization) code, you don’t have to worry about the de-allocation of the resource. Take for example `std::ifstream`, you’ve created an instance, and tried to read some data from the file and whether succeeded or failed miserably, you don’t have to do anything else! No need to close anything, no need to free anything, everything is handled for you!

But, when programming in C or C++, many system APIs will hand you a resource, that you are then responsible for closing/deallocating/etc. Understandably, this, just like any other task we have to do manually, is easily forgotten, making it prone to bugs, leaks and whatnot.

For example, this code block has a problem.
```C++
int update_startup_time() 
{
    unsigned startup;
    int fd = open(".startup_time", O_RDWR);
    if (fd < 0) 
    {
        fprintf(stderr, "open failed\n");
        return -1;
    }
    
    if (read_startup_time(fd, &startup) != 0) 
    {
        close(fd);
        fprintf(stderr, "read current startup time failed\n");
        return -1;
    }
    
    fprintf(stdout, "last startup time was %u\n", startup);
    
    startup = get_startup_time();
    if (write_startup_time(fd, startup) != 0) 
    {
        fprintf(stderr, "write current startup time failed\n");
        return -1;
    }
    
    close(fd);
    return 0;
}
```

If we manage to open the file, and then `read_current_startup()`, but then fail while trying to `write_current_startup()`, we will never close the `fd`. This is of course a resource leak, and may well result in exhausting all the file descriptors on a Linux system if it goes unnoticed for too long.

```C++
/*
 * Copyright (c) 2020-present Daniel Trugman
 *
 * Permission is hereby granted, free of charge, to any person obtaining a copy
 * of this software and associated documentation files (the "Software"), to deal
 * in the Software without restriction, including without limitation the rights
 * to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
 * copies of the Software, and to permit persons to whom the Software is
 * furnished to do so, subject to the following conditions:
 *
 * The above copyright notice and this permission notice shall be included in all
 * copies or substantial portions of the Software.
 *
 * THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
 * IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
 * FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
 * AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
 * LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
 * OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
 * SOFTWARE.
 */

#include <functional>

#define var_defer__(x) defer__ ## x
#define var_defer_(x) var_defer__(x)

#define ref_defer(ops) defer var_defer_(__COUNTER__)([&]{ ops; }) // Capture all by ref
#define val_defer(ops) defer var_defer_(__COUNTER__)([=]{ ops; }) // Capture all by val
#define none_defer(ops) defer var_defer_(__COUNTER__)([ ]{ ops; }) // Capture nothing

class defer
{
public:
    using action = std::function<void(void)>;

public:
    defer(const action& act)
        : _action(act) {}
    defer(action&& act)
        : _action(std::move(act)) {}

    defer(const defer& act) = delete;
    defer& operator=(const defer& act) = delete;

    defer(defer&& act) = delete;
    defer& operator=(defer&& act) = delete;

    ~defer()
    {
        _action();
    }

private:
    action _action;
};
```

The implementation is rather simple. It uses a lambda expression to create a function object that is executed once the object goes out of scope. And by taking advantage of lambda’s capture group, we can use the local members inside the cleanup function.

And how will that look with our original example? As beautiful as it gets:

```C++

int update_startup_time() 
{
    unsigned startup;
    int fd = open(".startup_time", O_RDWR);
    if (fd < 0) 
    {
        fprintf(stderr, "open failed\n");
        return -1;
    }
    
    defer close_fd([fd]{ close(fd); });
    
    if (read_startup_time(fd, &startup) != 0) 
    {
        fprintf(stderr, "read current startup time failed\n");
        return -1;
    }
    
    fprintf(stdout, "last startup time was %u\n", startup);
    
    startup = get_startup_time();
    if (write_startup_time(fd, startup) != 0) 
    {
        fprintf(stderr, "write current startup time failed\n");
        return -1;
    }
    
    return 0;
}
```

> Reference: https://gist.github.com/dtrugman/d3b10ad0a91b2f069f07f9311d24932a
