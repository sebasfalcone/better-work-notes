# Notes about C++

## Static
The meanings changes depending on context and we have three scenarios:
- **Static Variables**: 
	- In a function.
	- In a class.
- **Static methods in a Class**

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

##### Two stages if static variable initialization
- Variables  with *Static storage duration* must be initialized once before the program start and destroyed after execution terminates. Initialization could happen in two consecutive stages:
	- Static initialization:
		- Happens first and usually at compile time. If possible, initial values for static variables are evaluated during compilation and burned into the data section of the executable.
		- Zero runtime overhead, early problem diagnosis and safety are advanteges of this called [constant initialization](https://en.cppreference.com/w/cpp/language/constant_initialization).
	- Dynamic initialization.



### Static methods
- They don't depend on the objects of the class. We are allowed to invoke static methods using the object and the `.` or `->` operators, but its recommended to invoke static methods using the class name and the scope resolution operator `::`.

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
## Classes


## Smart pointers
