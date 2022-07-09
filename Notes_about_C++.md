# Table of contents

- [Notes about C++](#notes-about-c)
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
  - [Return Value Optimization](#return-value-optimization)
  - [Classes](#classes)
  - [Smart pointers](#smart-pointers)

# Notes about C++

## Static
The meanings changes depending on context and we have three scenarios:
- **Static Variables**: 
	- In a function.
	- In a class.
- **Static methods in a Class**.

### Static Variables
- In a function:
	- The variable gets allocated for the lifetime of the program. 
	- Their scope is internal, and only visible on the translation unit.

- In a class:
	- Are initialized only once and shared between all instances of the objects (of the same class). 
	- You need to initialize static variables outside of the class definition. Static variables can not be **initialized** using constructors or inline.
	- This variable is not longer associated with an specific object but with the class. Useful when we need to describe the whole population of classes (example: amount of instances).
	- Its a good idea to implement static methods to access static variables.

#### Initialization of Static Variables
- There is a category of variables that can (and should) be initialized before the program starts:
	- Static variables.
	- Global (namespace) variables.
	- Static class members.
- They live for the entire execution of the program and must be initialized before `main()` is run and destroyed after execution finished, this is called *Static storage duration*.

##### Two stages of static variable initialization
Variables  with *Static storage duration* must be initialized once before the program start and destroyed after execution terminates. Initialization could happen in two consecutive stages:
- Static initialization:
	- Happens first and usually at compile time. If possible, initial values for static variables are evaluated during compilation and burned into the data section of the executable.
	- Zero runtime overhead, early problem diagnosis and safety are advantages of this called [constant initialization](https://en.cppreference.com/w/cpp/language/constant_initialization). Ideally, all static variables are *const-initialized*.
	- If the initial value of static variable can't be evaluated at compile time, the compiler will perform *zero-initialization*. So, during static initialization, all static variables are either *const-initialized* or *zero-initialized*.
- Dynamic initialization:
	- Takes place after static initialization. Happens at runtime for variables that can't be evaluated at compile time. Here, static variables are initialized every time the executable is run and not just during compilation.

#### The green Zone - Constant initialization
Is ideal, and the compiler will try to perform it whenever it can. This is the case when your variable is initialized by a [constant expression](https://en.cppreference.com/w/cpp/language/constant_expression), that is an expression that can be evaluated at compile time.
```
struct MyStruct
{
    static int a;
};
int MyStruct::a = 67; 
```

##### Force Const Initialization - *constexpr*
Its not always clear if a variable is being initialized at compile time or at runtime. To make sure variables are initialized at compile time is by declaring them `constexpr`.
```
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
`constexpr` must be your first choice when declaring global variables (assuming you really need a global state to begin with). `constexpr` variables are not just initialized at compile time, but constexpr implies const and immutable state is always the right way.

##### Your Second Line of Defense - *constinit*
Is a keyword introduced in the **C++20 standard**. Works just as `constexpr`, as it forces the compiler to evaluate a variable at compile time, but it **doesn't imply const**.

Variables declared `constinit` are always const-zero or const-initialized, but can be mutated at runtime.

```
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
Imagine you need and immutable global `std::string` to store the software version. You don't want this object to be instantiated every time the program runs, but rather create it once and embed into the executable as read-only memory. In other words, you want a `constexpr`:

`constexpr auto VERSION = std::string("3.1.4");`

The problem is that the compiler will tell you:

`error: constexpr variable cannot have non-literal type`

The compiler complains because `std::string` defines a non-trivial destructor. This means that `std::string` is allocating some resources that must be freed on destruction, memory in this case. In other words, `std::string("3.1.4")` is not a constant expression. A workaround is the following:

`const auto VERSION = std::string("3.4.1");`

The compiler doesn't complain but at the cost of moving the initialization to runtime. Now `VERSION` must be part of a dynamic initialization, not static. This approach is less efficient and less safe that static initialization (not that bad either).

#### The Red Zone - Static Initialization Order Fiasco
The problem with *dynamic initialization* is that the order in which variables are initialized at runtime is not always well defined.

Withing a single compilation unit, static variables are initialized in the same order as they are defined in the source (*Ordered Dynamic Initialization*). 

Across compilation units, the order is undefined, and this is an issue if the initialization of a variable in `a.cpp` depends on another defined in `b.cpp`. This is called [Static Initialization Order Fiasco](https://isocpp.org/wiki/faq/ctors#static-init-order):

```
// a.cpp
int duplicate(int n)
{
    return n * 2;
}
auto A = duplicate(7); // A is dynamic-initialized
```
```
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

This program is ill-formed. It may print `14` or `0` (all static variables are at leas zero-initialized), depending if the dynamic initialization of `A` happens before `B` or not.

**NOTE**: This can only happen during dynamic initialization phase. Is impossible to access values defined in another compilation unit on static initialization (and this is the safety part that we talk before).

##### Solving The Static Initialization Order Fiasco
Encountering this problem is often symptom of poor software design. The best way to solve it is to refactor the code to break the initialization dependency of global across compilation units. Modules should be self-contained and strive for constant initialization.

If refactoring is not an option, one solution is the [Initialization On First User](https://isocpp.org/wiki/faq/ctors#static-init-order-on-first-use). The basic idea is to create static variables that are initialized at runtime, when it is the first time they are accessed. 

> Side note: This idea is used on the [singleton](https://refactoring.guru/design-patterns/singleton).

With this strategy it is possible to control the time when static variables are initialized at runtime, avoiding use-before-init.

```
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
```
```
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
- All other static variables are zero-initialized during static initialization
- `constexpr` forces the evaluation of a variable as a constant expression and implies const.
- `constinit` forces the evaluation of a variable as a constant expression and doesn't implies const.
- After static initialization dynamic initialization takes places, which happens at runtime before main().
- Within a compilation unit static variables are initialized in the order of declaration.
- The order of initialization of static variables is undefined across compilation units.
- You can use the *Initialization On First Use Idiom* to circumvent the *static initialization order fiasco*.


### Static methods
They don't depend on the objects of the class. We are allowed to invoke static methods using the object and the `.` or `->` operators, but its recommended to invoke static methods using the class name and the scope resolution operator `::`.
```
class User
{
    static int users_ammout;
      
    public:
        User()
        {
            std::cout << "\nNew user added - " << "User ammount " << ++users_ammout;
        }
          
        ~User()
        {
            std::cout << "\nUser deleted   - " << "User ammount " << --users_ammout;
        }
        
        static void count_users()
        {
           std::cout << "\nUser ammount " << users_ammout;
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

## Return Value Optimization

## Classes

## Smart pointers
