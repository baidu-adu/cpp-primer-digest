# Google C++ Style Digest

Note that the main purpose of style guid is **READABILITY**. The other
beneficial side effects:

*   less likely to introduce bug
*   better performance
*   short code
*   etc.

## Header Files

1.  **Guard**
    *   Google style guide
        requires
        [header guards](https://google.github.io/styleguide/cppguide.html#The__define_Guard).
        We could use `#pragma once` instead.
    *   The drawbacks of `#pragma once` are that it is not **part of
        the standard**, and they works differently in the subtle case
        when we intend to *include either one of them*. However, I
        would argue that `#pragma once` is now very well supported,
        and for the latter case, it usually suggests that you need a
        better design.
    *   On the other hand `#pragma once` reduces the possiblity of
        bugs (when moving files around), and is argurably more
        readable.
    *   **Decision**: Prefer `#pragma once` but allow header guards?
2.  **Inline Functions**
    *   **Rule of thumb**: do not inline a function if it is more than 10
        lines long. Period.
3.  **#include order**
    ```c++
    #include "foo/server/fooserver.h"  // corresponding header

    #include <sys/types.h>  // c system headers
    #include <unistd.h>

    #include <hash_map>  // c++ system headers
    #include <vector>

    #include "base/basictypes.h" // other headers
    #include "base/commandlineflags.h"
    #include "foo/server/bar.h"
    ```
    
    All in alphabetic order.
    
## Namespaces

1.  **Unamed Namespace**
    *   Use them in .cc file to hide functions or variables you do not
        want expose.
    *   **Do not** use them in .h files.
    
## Classes

1.  **Copy Constructor and Move Constructor**
    *   If you provide copy constructor and/or move constructor,
        provide the corresponding `operator=` overload as well.
2.  **Inheritance**
    *   All inheritance should be **public**. If you want to do
        private inheritance, you should be including an instance of
        the base class as a member instead.
    *   Do not overuse implementation inheritance. Composition is
        often more appropriate. Try to restrict use of inheritance to
        the "is-a" case: Bar subclasses Foo if it can reasonably be
        said that Bar "is a kind of" Foo.
    *   If you find yourslef wanting to use multiple inheritance,
        **THINK TWICE**.
3.  **Access Control**
    *   Data members are **private**, except when they are static
        const.
    *   For technical reasons, we allow data members of a test fixture
        class to be protected when using Google Test).
4.  **Declaration Order**
    *   `public`, `protected` and then `private`.
    *   In each section, group simular declarations together, prefer
        the order:
            *   `typedef` and `using`
            *   `struct` and `class`
            *   factory functions
            *   contructors
            *   `operator=`
            *   destructors
            *   methods
            *   data members
            
## Functions

1.  **Parameters**
    *   Ordering: Input and then Output.
    *   All **references** must be **const**, and this is the recommended
        way to pass parameters.
    *   For output parameters, pass pointers.
2.  **Default Argument**
    *   Not recommended for readablity issue. **Do not use** unless
        you have to.
3.  **Trailing Return Type Syntax**
    *   When in lambda. Period.

## Exceptions

1.  **DO NOT** throw exceptions. Returns error code, and let the upper
    level caller handles them.
    *   The benefit is that we will be forced to handle every error,
        and be free of **surprise** exits. Such exits are especially
        bad in multi-thread or multi-process programs.
2.  **Constructors** should avoid any code that can **fail**.
    *   As stated above, we need to return error code on failure.
        However, constructors can not do that.
    *   **Solution**: Use factory functions, or use `Init()` function
        to do the lift.
        
## Naming

I think for naming we should stick
to
[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Naming).

## Other

1.  Use `const` whenever it makes sense. With C++11, `constexpr` is a
    better choice for some uses of `const`
2.  `<stdint.h>` defines types like `int16_t`, `uint32_t`, `int64_t`,
    etc. You should always use those in preference to short, unsigned
    long long and the like, when you need a guarantee on the size of
    an integer.
3.  Macros damage readability. **Do not use** them unless the benefit
    is huge enough to compensate the damage.
4.  `auto` is permitted when it promotes readability.
5.  `switch` cases may have scopes:
    ```c++
    switch (var) {
      case 0: {  // 2 space indent
        ...      // 4 space indent
        break;
      }
      case 1: {
        ...
        break;
      }
      default: {
        assert(false);
      }
    }
    ```
