# Return Value Optimization and Copy Elision

## Problem

Suppose we wanted to write a function that returns a vector containing
number from `0` to `n - 1`. The simplest version is
**return-by-value**:

```c++
std::vector<int> Range(int n) {
    std::vector<int> result;
    result.reserve(n);
    for (int i = 0; i < n; ++i) {
        result.push_back(i);
    }
}
```

And we can call it like:

```c++
std::vector<int> x = Range(1000000);
```

A valid question is: does this involves a copy or move of the huge
``std::vector``?

## The Answer

If you are using a mordern compiler such
as [gcc](https://gcc.gnu.org/) and [clang](http://clang.llvm.org/),
the answer is **no** and **no**. A mechanism called **copy elision**
will be enforced here as a **return value optimization**. 

The standard has some detailed description on when **copy elision**
happens, but in general copy elision happens when one (or both) of the
conditions are met (There are other cases but are not important enough
to be highlighted here):

1.  Assigning a temporary to a variable of the same type.
2.  Returning a value right before it goes out of its scope.

When copy elision happens, the variable gets the value of the
temporary without calling the copy constructor nor the move
constructor. In fact it claims the temporary and becomes it.

## How about the Passing-A-Pointer Pattern

Another pattern that is widely used for this is pass-a-pointer:

```c++
void Range(int n, std::vector<int> *result) {
    result->reserve(n);
    for (int i = 0; i < n; ++i) {
        result->push_back(i);
    }
}
```

This is acceptable, but I would say it is not as good bcause:

1.  Do we assume the pointer `result` is initialized? A bad assumption
    can core dump here.
2.  It is far less readable, and it requires certain amount of mental
    work to realize that we are actually **returning** a vector.
    
## Conclusion

Stick to the copy elision approach and let the compiler lift the
weight. Do not try to outsmart the compiler by making your code less
readable.


