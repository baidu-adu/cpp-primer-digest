We recommend you use C-API to expose your library to others when you are writing CPP library because the ABI of CPP library is very fragile and easily to break.

When we talk about breaking ABI, we mean when we upgrade our releasing shared library, the applications which depend on old library cannot be run with the new library, and the applications must be recompiled. It will be a disaster if our library is used everywhere. We could always use the static library, to let user always upgrade their applications by recompiling. But, sometimes only shared library can be used, such as multiple language bindings. Or we need user can update their shared library only, without recompiling their application.

There are several situations will break the CPP ABI, force users to recompile their applications, or will make binary releasing more difficult when using CPP.

## User Want to Change Compiler.

As we all know, the CPP standard didn't contain rules about [name mangling](https://en.wikipedia.org/wiki/Name_mangling). Different compilers would generate different symbols in assembly. It should not be a big thing in Linux/Unix, because the most widely used compilers in Linux/Unix, GCC and Clang, will generate the same symbols in assembly. But in Windows, MSVC use an entirely different strategy about name mangling. So, we should provide at least two binary packages when exposing CPP interface in Windows.

## Multiple language bindings

If we want to write our library in C/CPP, but let users can use whatever language(such as Python, Matlab, Julia, Golang) they like. Expose the library in C should be better because most of other languages can invoke C-API in their standard library or as a language built-in feature. C++, especially Modern C++ (C++ 11 and above) is not supported very well for every other language.

## Change the function definition.

Modify the definition of a function should break ABI. It is obviously, but it is hard to notice when using CPP.

Because of function overloading feature, add/remove the parameter of a function (no matter if default argument is added or not) will change the function name in assembly, and break ABI. Here is an example.

```cpp
// in library v1.0
void foo();

// in library v1.1(wrong)
void foo(bool someFlags=false);

// in library v1.1(correct)
void foo();
void fooWithBool(bool someFlags);
```

In header above, the calling code of function `foo`, could always be `foo();`, but when upgrading library to v1.1, all application should be recompiled, because the function `foo` is missing. `foo(bool)` is not `foo()` because they have a different name in assembly.

So never use function overloading and default argument when exposing shared library unless you know what you are doing.

