# C++ Code Style and Primer Digest

**Note: this is a work in progress.**

This is largely based on
[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html).
If you have time, you should read it. C++ programmer of any level can
learn from it. 

[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
has clearly stated its
[goals](https://google.github.io/styleguide/cppguide.html#Goals),
which provides the **justification** of the rules in the style guide.
[Dr. Titus Winters](http://alumni.cs.ucr.edu/~titus/) had a very good
[talk](https://www.youtube.com/watch?v=NOCElcMcFik&t=2481s) on this.
Some of the goals that worth mentioning, especially for the new C++
programmers are:

1.  Optimize for the **READER**, not the writer.

    Realize that most of our time will be spent on reading the code
    than writing it. 
2.  Avoid surprising or dangerous constructs.

    Black magics may look awesome, but they tend to increase the risk
    of bugs and incorrectness (not only when you are developing it,
    but also when you or someone else is maintaining it). "Don't be
    clever".
3.  Be **consistent** (with existing code).

    This contributes to both the readability and the possibility of
    having automation tools.
    
We will restrain ourselves from talking much about format such as
indentation. It is not as important, and seriously you should make
good use of your favorite code editor, and the
handy [clang-format](http://clang.llvm.org/docs/ClangFormat.html).
See [C++ Programmer's Toolbox](#c-programmers-toolbox) for details.


    
## Header Files

1.  **Header guards** vs **#pragma once**

    Google style guide requires
    [header guards](https://google.github.io/styleguide/cppguide.html#The__define_Guard).
    We should prefer `#pragma once` instead.
    *   `#pragma once` is not part of the standard. However, all
        mainstream compilers have been supporting it for years.
    *   Some codebase may require header guards for **consistency**,
        but new code base may not have that constraint.
    *   `#pragma once` reduces the possibility of bugs (e.g. when
        moving files around), and is arguably more readable.
    *   [Details and example](cases/pragma_once.md).
2.  **Forward Declarations**

    See [Google Style Guide](https://google.github.io/styleguide/cppguide.html#Forward_Declarations).

    An exception for this rule is:
    
    * Use a Private Data Class for exposed API, especially for shared library.
       * look more details in [Expose API/Shared Library].

3.  **Inline Functions**
    
    **Rule of thumb**: do not inline a function if it is more than 10
    lines long.
4.  **#include order**
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
    
    *   Easy to maintain/read. The maintainer/reader can find the
        corresponding header file quickly.
    *   We do not have to stick to this rule strictly, although we
        need to be **consistent** about the order in our project.
    
## Namespaces

1.  **Unnamed Namespace**
    *   ** Do not** Use them in `.cpp`/`.h` files. Using `static` for hidding
        symbols in source files. The anonymous namespace will generate a namespace
        when compiling, and could generate a different namespace name each time.
        It will breaks the ABIs, make dubug harder.
    *   Sometimes you may want to expose internal functions or
        variables just for testing purpose. In this case it is better
        to declare them in
        [internal-only namespaces](cases/internal_only_namespaces.md).
2.  Never do **using namespace foo;**
    
    *   This pollutes the namespace, and can lead to hard-to-resolve
        compiler errors and bugs.
    *   If you really have something like
        `a::really::long::nested::name::space`, you can probably use
        [namespace alias](http://en.cppreference.com/w/cpp/language/namespace_alias)
        in a `.cpp` file:
        
        ```
        namespace short_name = a::really::long::nested::name::space;
        ```
        
        Do not use namespace alias in header files except in
        [internal-only namespaces](cases/internal_only_namespaces.md).
        This is because such aliases will affect every file that
        includes this header file.
3.  Avoid nested namespaces that match well-known top-level namespaces.
    
    Namespace collision may happen if you use `util` to refer to
    `my_library::util` within `namespace my_library`, while there is
    an existing top-level namespace called `util`. Even if no
    collision happens, this confuses the reader. Therefore,
    
    *   Try not to name your namespace `my_library::util` in this
        case.
    *   Refer to the top-level `util` as `::util` to explicitly say
        "top-level".
        
    Common top-level namespaces names that are prone to this are
    `util`, `base`, `aux`, etc.
    
## Classes

1.  **Copy Constructor and Move Constructor**
    *   If you provide copy constructor and/or move constructor,
        provide the corresponding `operator=` overload as well.
    *   Class should be disable copy/move by default.
    
2.  **Inheritance**
    *   All inheritance should be **public**. If you want to do
        private inheritance, you should be including an instance of
        the base class as a member instead.
    *   Do not overuse implementation inheritance. Composition is
        often more appropriate. Try to restrict use of inheritance to
        the "is-a" case: Bar subclasses Foo if it can reasonably be
        said that Bar "is a kind of" Foo.
    *   If you find yourself wanting to use multiple inheritance,
        **THINK TWICE**.
3.  **Access Control**
    *   Data members are **private**, except when they are static
        const.
    *   For technical reasons, we allow data members of a test fixture
        class to be protected when using Google Test).
4.  **Declaration Order**
    *   `public`, `protected` and then `private`.
    *   In each section, group similar declarations together, prefer
        the order:
            *   `typedef` and `using`
            *   `struct` and `class`
            *   factory functions
            *   constructors
            *   `operator=`
            *   destructors
            *   methods
            *   data members
            
## Functions

1.  **Parameters**
    *   Ordering: Input and then Output.
    *   Either use const references or pointer for non-fundamental types. The stl types are not fundamental types.
    *   Use **const reference** for read only parameter. Use **pointer** for writable parameter.
    
2.  **Default Argument**
    *   Not recommended for readability/ABI issue. **Do not use** unless
        you have to.

3.  **Trailing Return Type Syntax**
    *   When in lambda. Period.
    *   When using template, and the output type should be deducted by input types.
    
4.  **Return Value of Complicated Type**
    
    Returning values of simple types such as `int`, `int64_t`, `bool`
    may involves **copy**, but we hardly care. However it is rather
    important to **avoid copy** when returning values of complicated
    types because it can be rather expensive, e.g. `std::vector<int>`.
    
    One of the common pattern is to pass in a `std::vector<int>`
    pointer as parameter and construct it in place as below:
    
    ```c++
    void DoWork(..., std::vector<int> *result) {
        ...
        result->push_back(...);
        ...
        result->push_back(...);
        ...
    }
    ```
    
    Also we could return a smart pointer for complex type, such as unique_ptr<std::vector<int>>.
    
    I think this pattern should be discouraged in favor
    of
    [return value optimization](cases/return_value_optimization.md),
    because the latter is not only less error-prone, but also **much
    more readable**.

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
        
## Tricks

1.  **Thread-safe Local Static Initialization**

    In the following function:
    
    ```cpp
    void SomeFunction() {
      static SomeType some_static_variable();
      ...
    }
    ```
    
    If multiple control flows enter the function concurrently, is
    there a risk of **race** condition?
    
    The answer is **no**,
    if
    [compiled with C++11](http://stackoverflow.com/questions/8102125/is-local-static-variable-initialization-thread-safe-in-c11).
    
    In fact, if control enters the **declaration** concurrently while
    the variable is being initialized, the concurrent execution shall
    wait for **completion** of the initialization.
    
    This property can be taken advantage of to do more complicated
    object (lazy) initialization in a thread-safe way, together with
    the
    [lambda functions](http://en.cppreference.com/w/cpp/language/lambda).
    For example, the following code initializes a local static vector
    in a thread-safe way.
    
    ```cpp
    void SomeFunction() {
      static std::unique_ptr<vector<int>> my_list([]() {
        // Note that this lambda function is called within the declaration,
        // therefore is thread-safe.
        vector<int> *tmp_list = new vector<int>();
        tmp_list.push_back(1);
        tmp_list.push_back(2);
        ...
        return tmp_list;
      }());
    }
    ```
        
## Naming

I think for naming we should stick
to
[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html#Naming).


## Shared Library/API

* Please use C-API as the exposed API for most library.
   * Because C-API has a standard ABI in many operating system. All standard C compiler
     will compile same C function/variable to same symbol in asm.
   
   * Try to use void\* for C++ object instance. Try to use function pointer simulate
     member function.
     
   * [Example](cases/c_api_for_cpp.md)
   
* If you really want to expose CPP API, the following rules should be followed.
   * Do not use virtual method.
     * The virtual method will change the method name to an offset in vtable. And the offset is
       easily changed if the header could be changed. Then break the ABI compatibility.
     * Another guide for virtual method in API is that always creates new virtual method at the END OF CLASS declaration. And do not change older declarations. It will get method as `create`, `createEx`, `createEx2`, `...`, and it seems the virtual method is not necessary for this situations.
   * Use [private data class](https://en.wikipedia.org/wiki/Private_class_data_pattern), instead of directly declare member field in private.


## C++ Programmer's Toolbox

There are many available tools for C++, and it is always good to have
them in your pocket. Some of them help enforce the rules
automatically. Use them to free yourself from worrying about the
format and focus on more important stuff.

*   [clang](http://clang.llvm.org/)
*   [clang-format](http://clang.llvm.org/docs/ClangFormat.html)
*   [asan](https://github.com/google/sanitizers/wiki/AddressSanitizer) (Address Sanitizer)
*   [rtags](https://github.com/Andersbakken/rtags)

## Other

1.  Use the ones in standard library rather than the third-party
    implementation (e.g. boost) if you have a choice. This helps
    reduce both runtime dependencies and compile-time dependencies,
    and it promotes readability.

1.  `<stdint.h>` defines types like `int16_t`, `uint32_t`, `int64_t`,
    etc. You should always use those in preference to short, unsigned
    long long and the like, when you need a guarantee on the size of
    an integer.

1.  Macros damage readability. **Do not use** them unless the benefit
    is huge enough to compensate the loss.

1.  `auto` is permitted when it promotes readability.

1.  `switch` cases may have scopes:

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

1. Use smart_pointer as much as you can. Try unique_ptr rather than shared_ptr.

   * Smart Pointer is a better way to handle memory alloc/free then manually new/delete.
      * especially in multi-thread situation.
   * unique_ptr is much lighter than shared_ptr.
      * shared_ptr maintain a mutex, a reference count. Create and destory a shared_ptr could be a little bit slower than unique_ptr.
      * unique_ptr is better for multi-thread condition. It ensures that there are no other thread using this data right now.
   * Use unique_ptr<int[] > rather than vector<int> for array that will not grow.
      * unique_ptr is easier to return.
   * Use weak_ptr to avoid circle reference in shared_ptr.

1. Use **const** as much as you can. With C++11, `constexpr` is a
    better choice for some uses of `const`


   * It will let compiler to check whether program is correct or not, rather than die in runtime.
   * Not just for parameter, but also for member function.
     * **mutalbe** key word may be used in this situation.

1. Use standard library as much as you can.

   * **Do not** write a sort algorithm except you know what you are doing. For most situation, **std::algorithm** is your friend. And the implementation in **std** is good in general.
   * Use **std::thread** rather than pthread. Because C++ std library should be avaliable for most platform.
 
