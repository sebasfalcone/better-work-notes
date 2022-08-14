# The clean code

## Mantras
- If you want your code to be easy to write, make it easy to read.
- The cleanup doesn't have to be something big.
	- Change one variable name for the better.
	- Break up one function that's a little too large.
	- Eliminate one small bit of duplication
	- Clean up one composite `if` statement.

## Naming
- Use intention-revealing names:
	- If a name requires a comment, then the name does not reveal its intent:
		-  int d; // elapsed time in days ❌
		-  int elapsedTimeInDays; ✅

### Avoid disinformation:
- Don't leave false clues that obscure the meaning of code:
	- Refer to a grouping of accounts as `accountList` is a bad idea, unless it's an actual list.
	- Be careful using names which vary in small ways. For example, find the difference between `XYZControllerForEfficientHandlingOfStrings` and `XYZControllerForEfficientStorageOfStrings`
	- Spelling similar concepts similarly is *information*. Using inconsistent spellings is *disinformation*.

### Make meaningful distinctions:
- Write code solely to satisfy a compiler or interpreter is a big no. Because you can't use the same name to refer to two different things in the same scope, you might be tempter to change one name in an arbitrary way (misspell a word, add numbers, noise words, etc).

### Use pronounceable names:
- Take the following function as an example:
```
void copyChars(char a1[ ], char a2[ ]) 
{
	for (int i = 0; '\0' != a1[i]; i++) {
		a2[i] = a1[i];
	}
}
```
- This function reads much better when `source` and `destination` are used for the argument names.

- Or take this other example and tell me which is easier to understand:
```
class DtaRcrd102 {
	private Date genymdhms;
	private Date modymdhms;
	private final String pszqint = "102";
	/* ... */
};

======================== VS ========================

class Customer {
	private Date generationTimestamp;
	private Date modificationTimestamp;;
	private final String recordId = "102";
	/* ... */
};	
```

### Noise words are redundant:
- The word `variable` should never appear in a variable name, or the word `table` in a table name. 
- How is `nameString` better than `name`? Could a name ever be a float? If so, it breaks the rule of disinformation.
- Here is a perfect example of redundant words that take obscure the usage of some functions:
```
getActiveAccount();
getActiveAccounts();
getActiveAccountInfo();
```

### Use searchable names:
- Its a good practice to avoid the use of magic numbers, not only it makes the code much more explicit but makes it searchable.
- Likewise, the name `e` is a poor choice for any variable for which the programmer might need to search. A good rule is that single-letter names can **only** be used as local variables inside short methods / functions.
- If a variable is to be used in multiple places in a body of code, it is imperative to give it a search-friendly name:
```
void fun(void)
{
	for (int j=0; j<34; j++) {
		s += (t[j]*4)/5;
	}
}

======================== VS ========================

void fun(void)
{
	int sum = 0, realTaskDays, realTaskWeeks; 
	for (int j=0; j < NUMBER_OF_TASKS; j++) {
		realTaskDays = taskEstimate[j] * REAL_DAYS_PER_IDEAL_DAY;
		realTaskWeeks = (realTaskDays / WORK_DAYS_PER_WEEK);
		sum += realTaskWeeks;
	}
}
```
### Note that sum is not a particularly useful name but at least is searchable. 
- Yes, the intentionally named code makes for a longer function, but also it will be much easier to find the associated constants.

### Class names:
- **Classes and objects should have noun or noun phrase names** like `customer`, `wikiPage`, `account`, `addressParser`, etc. Avoid words like `manager`, `processor`, `data` in the name of a class. A class should not be a verb.

### Method names:
- **Methods should have verb or verb phrase names** like `postPayment`, `deletePage`, `save`, etc. 
- When constructors are overloaded, use static factory methods with names that describe the arguments.

`Complex fulcrumPoint = Complex.FromRealNumber(23.0);` is better than `Complex fulcrumPoint = new Complex(23.0);`

### Don't be cute:
- If naves are too clever, they will be memorable only to the people who share author's sense of humor, and only as long as these people remember the joke. Will they (or even you) what the function `holyGreanade` is supposed to do? Maybe a better name would be `deleteItems`.

### Pick one word per concept:
- It's confusing to have `fetch`, `retrieve` and `get` as equivalent methods of different classes / modules. How would you remember wich method name goes with which class? (and don't even talk about searchability).
- Likewise, It's confusing to have `controller`, `manager` and `driver` in the same code base. What is the difference between `deviceManager` and `protocolController`? Why are both not `controllers` or both `managers`. The naming leads you to expect that two objects have very different type as well as having different classes.
- Don't use the same word for two purposes.

### Context:
- Suppose you have a method called `add` and the outcome is that a value is added to a list... Should be better if we called the method `insert`, `append` or `push`?

### Use solution domain names:
- Remember that people who read your code will be programmers. Go ahead and user computer science terms like algorithm names, patter names, math terms, etc. 
- The name `accountVisitor` means a great deal to programmer who is familiar with the `VISITOR` patters. What programmer would not know what a `jobQueue` is?

### Add meaningful context:
-  When names are not meaningful by themselves, you need to add context by enclosing them in well-named classes, functions or namespaces. When all else fails, then prefixing the name may be necessary.
- Imagine that you have variables named `firstName`, `lastName`, `street`, `city`, `state` and `zipcode`. Taking together it's pretty clear that they form an address. But what if you just saw the `state` variable being used alone in a method? Would you automatically infer that is was part of an address?\
You can add context by using the prefix `addr`. Of course, the best solution is to create a class named `address`.

### Don't add gratuitous context
- If we are developing and app for "Gas Station Deluxe", its bad idea to prefix every class with `GSD`. You type G and press the completion key to be rewarded with a mile-long list of every class in the system.

## Functions
### Make it small!
- The first rule of functions is that they should be small. The second is that they should be smaller than that.

### Blocks and indenting
- Blocks within `if`, `else`, `while`, etc. should be one line long. Probably a function call. This keeps the enclosing function small, but adds documentary value because the function called withing the block can have a nice and descriptive name.
- This implies that the function should not be large enough to hold nested structures. Therefore, the indent level of a function should not be greater than one or two.

### Do one thing
- **Functions should do one thing. They should do it well. They should do it only.**
- The idea behind a function is to decompose a larger concept into a set of steps, this function could (and should if necessary) be further split.

### One level of abstraction per function
- To make sure out functions are doing one thing, all statements within our function need to be at the same level of abstraction. We can identify many levels of abstraction, for example:
	- Level 1: function call. The function has a really descriptive name, so we don't have to dive deep into it.
	- Level 2: access to attributes of a class. We need to start thinking.
	- Level 3: data processing in a for loop. We really need to read the code and interpret.

### Reading code from top to bottom: The stepdown rule
- We want every function to be followed by those at the next level of abstraction, so that we can read the program, descending one level of abstraction at a time. 

### Switch statements
- 