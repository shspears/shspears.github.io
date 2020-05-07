---
layout: "post"
title: "move_semantics"
date: "2020-05-06 19:25"
---

# Move Semantic notes

> Most of the notes here were taken from Udemy's [Complete Modern C++](https://www.udemy.com/course/beg-modern-cpp/) course, authored by Umar Lone. Great course!

> - Move semantics were introduced in C++11 and improved upon in 14/17.
> - Move semantics essentially provide a way to move (or transfer) ownership of resources stored in memory from variable (object) to another, without the need to copy those resources in memory.
> - This greatly reduces the computational load required with copying data (such as a copy constructor, or passing objects as function arguments), especially in large objects/ data sets
> - It is also handy when trying to pass a non-copyable object (such as a std::unique_ptr) as a function argument
> - ***Keep in mind*** that the original object is no longer usable after a move and should not be resued afterwards
---
- ToC




---

### L-Value, R-Value, and R-Value References
- To understand move semantics (introduced in C++11) it's important to understand what l values, r values, and r value references are.
- r value references were introduced in C++11, so that the move constructor and move assignment operator could be implemented within classes
-
![lvalue_rvalue](https://github.com/shspears/shspears.github.io/blob/master/images/lvalue_and_rvalue.PNG)

![rvalue_references](https://github.com/shspears/shspears.github.io/blob/master/images/rvalue_reference.PNG)

```cpp
    #include <iostream>

    //Returns r-value
    int Add(int x, int y) {
    	return x + y;
    }
    //Return l-value
    int & Transform(int &x) {
    	x *= x;
    	return x;
    }

    void Print(int &x) {
    	std::cout << "Print(int&)" << std::endl;
    }
    void Print(const int &x) {
    	std::cout << "Print(const int&)" << std::endl;

    }
    void Print(int &&x) {
    	std::cout << "Print(int &&)" << std::endl;
    }
    int main() {
    	//x is lvalue
    	int x = 10;

    	//ref is l-value reference
    	int &ref = x ;
    	//Transform returns an l-value
    	int &ref2 = Transform(x) ;
    	//Binds to function that accepts l-value reference
    	Print(x);


    	//rv is r-value reference
    	int &&rv = 8 ;

    	//Add returns a temporary (r-value)
    	int &&rv2 = Add(3,5) ;
    	//Binds to function that accepts a temporary, i.e. r-value reference
    	Print(3);
    	return 0;
    }




```



### Move Constructor
- The traditional "Rule of Three" in C++, was that if you were creating a new class and you implemented either the copy constructor, copy assignment operator, or the destructor, you should implement all three!
- In Modern C++ (C++11/14/17/20), the Rule of Three is now the "Rule of Five". The Five being:

           1. Constructor constructor
           2. Copy assignment operator
           3. Destructor
           4. Move Constructor
           5. Move assignment Operator






### Move Assignment Operator



### Perfect Forwarding and std::forward()



### std::move()
