---
layout: "post"
title: "Lambda_expressions"
date: "2020-05-06 09:05"
---

# Lambda Expression Notes

> Most of the notes here were taken from Udemy's [Complete Modern C++](https://www.udemy.com/course/beg-modern-cpp/) course, authored by Umar Lone. Great course!
---
- Lambda Expression [Syntax](#lambda-expression-syntax)
- Basic [Examples](#examples)
- More complete (basic) [example](#more-complete-example)
- Capture list [example](#capture-lists)
- [Capture Lists](#capture-list)
     1. [Mutable keyword](#mutable-keyword) explaination
     2. Pass by [reference](#pass-by-reference)
     3. Pass by [value](#pass-by-value)
- New features added in [C++14](#generalized-capture) - creating new variables within the capture list
---
### Function Objects (Functors)

- Function objects are essentially classes (structures in most cases) that overload the function operator ().
- They have a state, where by global functions do not.
- Functors are preferred over function pointers for use with Callbacks. They are more efficient

> Functors
> 1. invoked through an object
> 2. static in nature
> 3. must specify at compile time
> 4. fast
> 5. can store state
> 6. Easy to optimize

> Function pointers
> 1. invoked through pointers
> 2. Dynamic in nature
> 3. can be specified at run time
> 4. slow
> 5. Cannot store state
> 6. Difficult to optimize

---
### Lambda expressions

- Even better than Functors for callbacks!
- Defines an 'anonymous' function object
- Allows to use the functionality of a Functor, without having to define a class / struct and without the need of overloading the () operator
- Can be passed as an argument of a function that accepts a functor.
- behaves like a function - can accept arguments and return value
- Typically Encapsulates a few lines of code
- can use **auto** to provide an explicit name

---
### Lambda Expression Syntax


```cpp
    [] (args) <mutable keyord> <exception specification> -> <return type>
```


[] - Lambda Introducer. Signifies the start of a Lambda. Contains a *Capture Clause*.

(args) - optional arguments for the Lambda

mutable - optional ability to specify mutable specification

exception specification - optional ability to use noexcept or noexcept(false)

-> return type - Optional - can explicitly specify trailing return type by using the -> (ie. -> int;)

---

### Examples
#### Most basic
```cpp
    [](){ std::cout << "Hello World" std::endl; }();
```

> The trailing () is used to *call* the lambda. Only executes once. Pretty useless!

#### Ability to call the lambda more than once!

- Still pretty basic though!
```cpp
    auto lambda1 = [](){std::cout << "Hello World" << std::endl; };
...
lambda1();
```

#### Generic Lambda (C++14)

- using **auto** here makes it fairly versitile.
```cpp
    auto sum = [](auto x, auto y){ return x + y; };
```
```cpp
    auto sum = [](auto x, auto y) noexcept { return x + y; };
```

- This is similar to the following Functor
```cpp
    template <typename T>
    struct sum
    {
      T operator(T x, T y)
      {
        return x + y;
      }
    };
```
---

#### More complete Example

```cpp
    #include <iostream>

    template <typename T, int size, typename Comparator>
    void Sort(T(&arr)[size], Comparator comp)
    {
        for(int i =0; i < size -1 ; i++)
        {
            for (int j = 0; j < size - 1; j++)
            {
                if (comp(arr[j], arr[j + 1]))
                {
                    T temp = std::move(arr[j]);
                    arr[j] = std::move(arr[j + 1]);
                    arr[j + 1] = std::move(temp);
                }
            }
        }
    }
    /*
    template <typename T>
    struct Comparator
    {
      T operator(T x, T y)
      {
        return x > y;
      }
    };
    */
    int main()
    {
        int arr[] = { 12,33,456,7745,8,42,89,05,889,0,3,2,6 };

        for (auto x : arr)
        {
            std::cout << x << ", ";
        }

        std::cout << std::endl;

        Sort(arr, [](auto x, auto y) {return x > y; });
        //Here, a lambda is used in replace of a comparator functor
        for (auto x : arr)
        {
            std::cout << x << ", ";
        }

        std::cout << std::endl;
        return 0;
    }
```

- By not using the Comparator functor, the Sort function call is easier to read
- By not using the Comparator functor, we avoid adding more to the global namespace

---
### Capture lists
#### Example

```cpp
    template <typename T, int size, typename Callback>
    void ForEach(T(&arr)[size], Callback operation)
    {
        for (int i = 0; i < size - 1; i++)
        {
            operation(arr[i]);
        }

    }

    int main()
    {
        int arr[] = { 12,33,456,7745,8,42,89,05,889,0,3,2,6 };

        //Now to use the ForEach function to print the elements of the array
        ForEach(arr, [](auto x) { std::cout << x << ", "; } );
        std::cout << std::endl;

        //Now, lets use the same function to add an offset to each element in the array
        int offset = 5;

        //We need to add the offset variable to the Capture List (it creates a copy of 'offset')
        ForEach(arr, [offset](auto &x) {  x += offset; } );
        //And print the results
        ForEach(arr, [](auto x) { std::cout << x << ", "; });
        std::cout << std::endl;

        //Unable to alter 'offset' in a non mutable lambda
        //ForEach(arr, [offset](auto& x) {  x += offset; offset++; });

        //mutable keyword removes the (default) const opeator on the capture list object / variables allowing them to get modified
        ForEach(arr, [offset](auto& x) mutable {  x += offset; offset++; });

        ForEach(arr, [](auto x) { std::cout << x << ", "; });
        std::cout << std::endl;

        std::cout << "Offset was 5, and now it is: " << offset << std::endl; //still 5!

        //When you pass a value by reference within the capture list - it isn't copied into the capture list,
        //therefore, the object in the capture list is intended to be altered and there is no need for the mutable keyword

        int sum{};
        ForEach(arr, [&sum](auto& x) {  sum += x; });

        return 0;
    }
```

- Here we are looking at capture lists - which allows passing of objects (/variables) to the Lambda for use in its implementation
- One function was created ( ForEach() ), and the Lambda is changed to demonstrate its versatility

```cpp
...
    //Print the elements in an array using ForEach() function
    ForEach(arr, [](auto x) { std::cout << x << ", "; } );
    std::cout << std::endl;
...
```

##### Capture List

- Summary:
![CL](https://github.com/shspears/shspears.github.io/blob/master/images/capture_list_summary.PNG)


- objects being passed to the Lambda using the [**capture list**]
```cpp
...
    //Now, lets use the same function to add an offset to each element in the array
    int offset = 5;

    //We need to add the offset variable to the Capture List (it creates a copy of 'offset')
    ForEach(arr, [offset](auto &x) {  x += offset; } );
...
```

##### Mutable Keyword
- Objects being passed via the Lambdas [**capture list**] are *copies* of the originals and are by default *const*
- To alter the state of the *copied* objects within the capture list, we need to explicitly use the **mutable** keyword
```cpp
    //Unable to alter 'offset' in a non mutable lambda
    //ForEach(arr, [offset](auto& x) {  x += offset; offset++; });

    //mutable keyword removes the (default) const opeator on the capture list object / variables allowing them to get modified
    ForEach(arr, [offset](auto& x) mutable {  x += offset; offset++; });
```

##### Pass by Reference
- When you pass a value by reference within the capture list - it isn't copied into the capture list,
  therefore, the object in the capture list is intended to be altered and there is no need for the mutable keyword
```cpp
    int sum{}; //lets sum all of the elements within the array
    ForEach(arr, [&sum](auto& x) {  sum += x; });
```
- It is possible to pass all variables / objects (within scope) to the lambda expression
- Although possible, it's probably not a good idea to do this from within the main function
```cpp
    ForEach(arr, [&](auto& x) {  sum += x; });
```

##### Pass by Value
- In the event you want to pass the entire scope to the lambda by **value**, you can do so by

```cpp
    ForEach(arr, [=](auto& x) {  sum += x; });
```

- It is important to note that objects / variables that are declated after the lambda **are not** within the scope of the expression and will not be included in the capture list [], wether by reference or value.

- For completeness:
```cpp
    //Capture sum by reference, and all other variables / objects by value
    ForEach(arr, [=, &sum](auto& x) {  ... });

    //Capture offset by value (copied) and all other variables / objects by reference. You may want to declare <mutable>
    ForEach(arr, [&, offset](auto& x) {  ... });
    //where "..." is the lambda implementation...!
```
###### **this** keyword
- If you use a lambda expression within a class' member function, you can include the **this** keyword within the capture list to include the class members..!!


#### A Few more Notes
- It is possible to have a lambda expression within a lambda expressions
 ```cpp
//lambda inside a lambda to take 5 and double it
  [](int x)
  {
    x *=2;
    [](int x)
    {
      std::cout << x << std::endl
    }(x);
  }(5);
```
- If a lambda **does not include a capture list**, it defaults (by the compiler) as a **function pointer**
    > This allows the use of lambdas for use with c funtions that require function pointers
```c
atexit([](){std::cout << "Program is exiting" << std::endl; });

---

### Generalized capture
- New feature added in C++14
- Allows the creation of new variables within the capture list
- The type of these variables is deduced from the type produced by the expressions
- Consequently, these variables must **always be initialized**
- If the initializer expression is a variable, the new variable can have the same (or different) namespace
- use the & operator before the variable name to create a Reference

```cpp
    [var = expression](args)
    [&var = expression](args)
```
```cpp
    std::ofstream out{"file.txt"};
    auto writeToFile = [out = std::move(out)](int intToWriteToFile) mutable
    {
      out << x;
    };
    ...
    write(100);
```
- But why would we use this?!
- In the event that the *writeToFile* lambda is the only place we use the ofstream object *out*, this makes sure that that reference is contained only within the lambda!
- ...Gotta love std::move(). It not only speeds things up, but it allows us to move non-copyable objects!!
- ***More Importantly***: it allows us to use std::unique_ptr<> objects inside a lambda express!!
