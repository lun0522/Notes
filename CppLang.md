# The C++ Programming Language

## Chapter 6 Types and Declarations

`int` is the same to `signed int`, but `char` can either be `signed char` or `unsigned char`, which depends on implementation. Still prefer plain `char` because some functions in the standard library don't accept `signed char` or `unsigned char`.

Use `alignof()` to get the alignment of the argument type, and `alignas()` to specify alignment for declaration:

```cpp
auto ac = alignof(char);
alignas(atype) char buffer[1024];
```

Postfix declarator operators bind tighter than prefix ones, so `char* kings[]` is an **array** of `char` pointers, while `char (*kings)[]` is a **pointer** to a `char` array.

Use `::` to refer to the hidden global name:

```cpp
int x = 1;    // global x

void f() {
  int x = 2;  // local x
  ::x = 3;    // assign to the global x
  for (int x = 0; x < 4; ++x) {...}
  // this x is local to the scope of the loop
  // no way to refer to the "local x" inside of this loop
  int x = 5;  // error: cannot define x twice in the same scope
}
```

A variable declared after a `case` label within `switch (...) {}` can be seen in all other `case` labels after its declaration, so we have to add curly braces to set the scope, otherwise the compiler will say the initialization of it is bypassed in later `case` labels:

```cpp
switch (...) {
   case ...:
       auto u = ...;
       break;
   case ...:
       // u can still be seen here and it is not initialized!
}

```

Local variables (excluding global, namespace, `static`, etc) of built-in types are **not** initialized to `{}`:

```cpp
int x{};  // 0
int y;    // no well-defined value

int a1[100]{};  // all elements initilized to 0
int a2[100];    // can be anything

vector<int> v1{};
vector<int> v2;  // the same to v1
```

To use the default initializer we have to use `{}` rather than `()`, since `X a();` is function calling instead of initialization.

Note that when we use `auto`, for `auto x1{1};` and `auto x2{1, 2, 3};`, both `x1` and `x2` are deduced to be `initializer_list<int>`. Use `=` instead of `{}` with `auto` when we don't want a list.

Use `decltype()` to deduce the type of an initialized variable. This is useful for generic programming:

```cpp
template<typename T, typename U>
auto operator+(const T& t, const U& u) -> decltype(T{} + U{}) {
    return t + u;
}
```

## Chapter 7 Pointers, Arrays, and References

Array types are not assignable (a non-modifiable lvalue). For example, this is not valid:

```cpp
int a[10], b[10];
a = b;  // error
```
String literals are of type `const char[]`, and we cannot assign it to `char *` since we should not modify the string literal itself:

```cpp
char *p1 = "how";        // error
const char *p2 = "are";  // ok
char a[] = "you";        // will make a copy, also ok
```

Multi-line string and raw string:

```cpp
char a2[] = "long "
            "string";  // the same to "long string"
string s1{R"("long" "string")"}  // the same to ""long" "string""
string s2{R"(long
string)"};  // the same to "long\nstring"
```

If `char[]` is passed as an argument, it will be treated as `char*`, and thus `sizeof(arr)` will return the size of the pointer itself and we cannot use it to calculate the size of the array. Besides, `for (char c : arr) {..}` won't compile since the size is unknown. We may have to pass in the size of the array as well.

Read from right to left to find out what does `const` decorate:

```cpp
char a[] = "hello";
const char* p1 = a;  // pointer to const char
char const* p2 = a;  // pointer to const char
char *const p3 = a;  // const pointer to char
const char *const p4 = a;  // const pointer to const char
```

For a `const` lvalue reference, the original object can be rvalue or of different type, in which case implicit type conversion is performed if necessary, the resulting value is **copied** to a temporary object, and a reference is created referencing to it. Although we cannot say `int& x = 2;`, we can say `const int& x = 2.0;` since a temporary object (a lvalue) holds 2.

We can also use rvalue reference (`&&`) for temporary objects (so-called *destructive read*) (we do **not** use `const` here since moving implies modifying):

```cpp
string s1{"hello"};
string func();

string&& rc1{s1};       // error, cannot initialize with a lvalue
string&& rc2{"world"};  // ok
string&& rc3{func()};   // ok
// actually rc2 and rc3 are lvalue now
// && only means they came from rvalue

string s2 = static_cast<string>(rc2);  // rc2 will be destructed
string s3 = move(rc3);  // equivalent to static_cast
```

## Chapter 8 Structures, Unions, and Enumerations

Some `class` and `struct` are **Plain Old Data** types, which don't have user-defined constructors, destructors or virtual member functions. They are stored in contiguous sequence of bytes, so we can simply use `memcpy` to copy an array of them. We can use `is_pod` to predicate whether a type is POD.

To optimize the memory usage of `struct`, we can reorder elements in it to reduce the memory waste caused by alignments. Readability is usually considered more important than this improvement.

Prefer `enum class` to plain `enum`:

- We can specify the underlying data type: `enum class Color : char { red, green, blue };`
- Enumerators are in the scope (namespace) of `enum class`, so we cannot directly use `red`, but `Color::red`
- Enumerators cannot be implicitly converted to other types (eg: `int`) unless we use `static_cast`

## Chapter 9 Statements

To avoid a misuse of a variable, we can initialize a variable in the condition of `if`:

```cpp
if (double d = 3.0) {  // value of d is converted to boolean
  // do something
} else {
  // d is also valid in this branch
}
```

To make the range-based `for` loop (eg: `for (int x : v)`) work on a custom class `T`, we can either:

1. define `T::begin()` and `T::end()`, or
2. define `begin(&T)` and `end(&T)` in the enclosing scope (same namespace)

and overload the prefix operator `++` for the iterator if necessary. We can use `std::begin()` and `std::end()` to retrieve iterators of builtin arrays and STL containers.

## Chapter 10 Expressions

If a temporary object is not bound to a `const` lvalue reference or used to initialize a named object, it will be destroyed at the end of the full expression:

```cpp
string s1{"hello"};
string s2{"world"};
size_t length = strlen((s1 + s2).c_str());  // ok
const char* cs = (s1 + s2).c_str();  // using `cs` may cause error
```

`constexpr` must be of literal types (including integral, float-point, enumerator and arrays of them). We may use functions with `constexpr` return type to initialize it:

```cpp
constexpr int add(int x, int y) { return x + y; }
const int a = 1, b = 2;
constexpr int c = add(a, b);
```

Note that parameter types of `add` cannot be `constexpr`. Removing `constexpr`/`const` in the declaration of `add` or `a` and `b` will cause error. We can also use references with `constexpr` as long as the referenced object is constant:

```cpp
constexpr int minus(const int& x, const int& y) { return x - y; }
const int a = 1, b = 2;
constexpr int c = minus(a, b);
```

String literals are allocated on the staic memory, so:

```cpp
void f() {
  constexpr const char* p1 = "hello";  // ok
  constexpr const char* p2 = new char(5);  // error, it is allocated on the free store
  static const char a[] = "world";
  constexpr const char* p3 = a;  // ok
}
```

Some user-defined types are also literal types. Their constructors may have `constexpr` for the return type. See the standard for the requirement of literal types.

When considering conversions between numerical values, use `numeric_limits` to get the max representable number of a type, and use `{}` for initialization to prevent narrowing.

## Chapter 11 Select Operations

A branch of the ternary expression can throw an exception:

```cpp
int i = (p) ? *p : throw runtime_error{"unexpected nullptr"};
```

If `new` cannot allocate enough space for the object, it will throw `std::bad_alloc`. We can use `std::set_new_handler` to deal with it. We can also pass `std::nothrow` to avoid throwing exceptions, in which case `nullptr` will be returned if there is no enough space:

```cpp
int* p = new(nothrow) int[10000];  // p may be nullptr
if (!p) {...}
operator delete(p, nothrow);
```

The `new` operator can be overloaded. We can have a memory management class:

```cpp
class Arena {  // memory manager
 public:
  void* alloc(size_t sz);
  void free(void* p);
};

class X {
 public:
  X(int v) : val(v) {}
  void* operator new(size_t sz, Arena* a) { return a->alloc(sz); }
 private:
  int val;
};

extern Arena* storage;

void f() {
  X* p = new(storage) X{3};
  // use p
  p->~X();
  storage->free(p);
}
```

`new` actually contains two steps:

1. allocate enough space (don't care about the object type)
2. cast `void*` to `X*` and call the constructor

What we overload is the first step, while the second step is done as usual. In the line `X* p = new(storage) X{3};`, `X` is used to calculate the first argument `sz`, so we only have to pass one argument of type `Arena*` to `new`, which is called placement `new`. After that, `3` is sent to the constructor.

Note that we cannot use `delete` to deallocate the object since it was not allocated on the free store but by the manager, so we have to call the destructor explicitly and let the manager free the space. We can overload `delete` just like what we did to `new`, but we **cannot** call it by ourselves. It is actually used to handle the case where the constructor throws excpetions. See Wiki.

`new X{...}` will construct the object on the free store, while `X{...}` will contruct in the local scope. 

To eliminate ambiguity:

```cpp
struct X { int a, b; };
struct XX { double a, b; };
void f(X);
void f(XX);     // overloaded

void g() {
  f({1, 2});    // error, ambiguous
  f(X{1, 2});   // ok
  f(XX{1, 2});  // ok
}
```

To create a `std::initializer_list`, elements between curly braces are copied (and converted to the target format if necessary) to the underlying array of `initializer_list`. It is **immutable**, and thus when we use it to initialize another object (eg: `std::vector`), elements are copied rather than moved to the new object. If we directly use a `{}`-list for constructor arguments or to initilize an aggregate, elements won't be copied (unless as by-value arguments):

```cpp
int v1{1};     // direct initialization
int v2 = {2};  // copy initialization
```

We can use `initializer_list` to deal with immutable homogenous lists with varying lengths:

```cpp
int sum(initializer_list<int> val) {
  int s = 0;
  for (auto v : val) s += v;
  return s;
}

sum({1, 3, 5, 7, 9});
sum({1.0, 3, 5, 7, 9.0});  // error, not homogenous
```

Note that the compiler does not deduce the type of the entire list. We still have to specify the container type because of the language restriction:

```cpp
template<typename T>
void f1(T);

f1({1, 2, 3});  // won't compile

template<typename T>
void f2(const initializer_list<T>&);

f2({1, 2, 3});  // ok

template<typename T>
void f3(const vector<T>&);

f3({1, 2, 3});  // error
f3(vector<int>{1, 2, 3});  // ok
```

Forms of capture lists of lambda expressions:

- `[]`: no capture
- `[&]`: capture all by reference
- `[=]`: capture all by value
- `[capture-list]`: those preceded by `&` captured by reference, while others captured by value; it can contain `this` and names followed by `...`
- `[&, capture-list]`: capture those in the list by value and others by reference; the list can contain `this`
- `[=, capture-list]`: capture those in the list by reference (all of them should be preceded by `&`) and others by value; the list cannot contain `this` since we never capture it by reference (`this` is a pointer, just capture it by value)

If we capture `this`, all member variables are implicitly captured by reference (**no** need to put them in the capture list). For multi-threading, we'd better capture by value to avoid data race.

To modify the state of a variable captured by value, we can declare `mutable` before the body. However, different from capturing by reference, this modification is done on the local copy in the closure, so it is invisible to the outside world:

```cpp
int x = 10;
auto func1 = [&x] () { x += 10; };
func1();  // x becomes 20
auto func2 = [x] () mutable { x += 10; };
func2();  // x becomes 30, but still 20 to the outside world
func2();  // x becomes 40, but still 20 to the outside world
```

The return type of lambda expressions can be deduced by the compiler. If a lambda does not take parameters, we can omit `()`, but if we specify a return type with `->` or declare `mutable`, `()` can not be ommited. We can specify the type of lambda as `std::function<return-type (arguments-list)>`. This is necessary for recursion, because if we use `auto`, the type of itself has not been deduced inside of its body, and thus we cannot call itself recursively. Example for recursion:

```cpp
function<int (int, int)> gcd = [&gcd] (int a, int b) {
  if (b == 0) return a;
  return gcd(b, a % b);
};
```

Usage of casts (avoid explicit type conversion; always prefer `T{v}` casts and these named casts to C-style casts):

- `static_cast`: conversion between related types, may be of the same class hierachy (eg: `int` -> `double`)
- `reinterpret_cast`: conversion between unrelated types (eg: `int` -> `char*` or `int*` -> `char*`)
- `const_cast`: conversion between types that only differ by `const` or `volatile`
- `dynamic_cast`: runtime checked conversion of pointers and references into a class hierachy

For numerical values, `T{v}` only performs well-behaved conversions. If we have to use `static_cast`, we can compare the source and resulting numbers, and reject (throw exceptions) if the difference is intolerable.

## Chapter 12 Functions

A function that does not return the control flow to the caller (eg: `exit()`) can be marked `[[noreturn]]`.

`constexpr` functions are intended to be simple: no loops, no local variables, and no side effects. Recursions and conditional expressions are allowed (may increase compilation time).

Initialization of `static` variables wouldn't lead to data race unless we do recursive calls during the initialization or deadlocks happen. This is done with some lock-free construct.

To pass a reference to an array:

```cpp
void f(int(&r)[4]);

int a1[] = {1, 2, 3, 4};
int a2[] = {1, 2};
f(a1);  // ok
f(a2);  // error, different length
```

To be more flexible, we can put the length in template, which may create too many definitions for different lengths:

```cpp
template<typename T, int N>
void f(T(&r)[N]) {}

f(a1);  // ok
f(a2);  // ok
```

When we cannot specify the number and types of parameters of a function, in addition to variadic templates and `initializer_list`, we can also put `...` at the end of the parameter list to indicate there are more parameters to come (of unknown types):

```cpp
#include <cstdarg>  // declares all macros with prefix `va_`

void error(int severity ...) {  // we know nothing about other params except severity
  va_list ap;
  // specify the name of the last known parameter as the second parameter
  va_start(ap, severity);

  while (true) {
    char* p = va_arg(ap, char*);  // assume following params are char*
    if (!p) break;
      cerr << p << ' ';
  }

  cerr << endl;
  va_end(ap);  // paired with va_start
  if (severity) exit(severity);
} 
```

This is error-prone. Prefer to use `initalizer_list` or `const vector<string>&`.

To call a constructor from another constructor (alternative way is to use default parameters):

```cpp
class complex {
 public:
  complex(double r, double i) : re{r}, im{i} {}
  complex() : complex{0, 0} {}
 private:
  double re, im;
};
```

The compiler only makes sure the default value has the right type. Its value can be changed. For example, we may use a `static` value:

```cpp
struct X {
  int val;
  static int config;
  X(int v = config) : val{v} {}
};

int X::config = 1;

void f() {
  X x1{};  // `x1.val` should be 1
  x1.config = 2;  // or `X::config = 1;`
  X x2{}:  // `x2.val` should be 2
}
```

Note that the space for default pointer-type parameters matters. For example, `int f(char* = nullptr);` is good, while `int f(char*= nullptr)` has a syntax error because of `*=`.

For function overloading, the return type is not considered for overload resolution. Overloading across class or namespace is not enabled by default. For example, in a derived class, a method in the base class won't be considered for overload resolution. If we want that overloading happen, we have to use the `using`-directives or argument dependent lookup.

To take a pointer to a function, `&` is optional; to deference such a pointer, `*` is optional:

```cpp
void error(string s) {}
void (*fct)(string);

void f() {
  fct = &error;
  fct = error;   // also works
  (*fct)("hello");
  fct("world");  // also works
}
```

A pointer to function must reflect the linkage (`extern`) of it, while reflection of exception (`noexcept`) is optinal (will lose information if we omit it):

```cpp
extern "C" void error(string s) noexcept {}
extern "C" void (*fct1)(string) = &error;  // lose useful information
void (*fct2)(string) = &error;  // mismatch
```

Note that we **cannot** reflect the linkage or exception if we use the `using`-directives:

```cpp
using P1 = void(int);
using P2 = extern "C" void(int);  // error
using P3 = void(int) noexcept;    // error
```

Avoid macros. We may use some pre-defined ones:

```cpp
cout << __func__ << "() in file " << __FILE__ << " on line " << __LINE__ << endl;
```

## Chapter 13 Exception Handling

Two ways to avoid mamory leaks during exception handling:

1. **RAII**: Put the memory releasing code in the destructor. When we create local objects (or `unique_ptr`/`shared_ptr` if we need pointers), the destructor will always be called no matter the function is quitted normally or an exception is thrown. Note that this is important for constructors: if an exception is thrown inside of a constructor, objects that have been **completely** created in this constructor will be destructed automatically, and thus the caller won't have to worry about it

2. **"Finally"**: C++ does not support the keyword `finally`, but we can create a lambda expression to wrap the code for releasing resources, and let another local object hold it. When that object goes out of scope, resources are also released

An exception may be copied several times before it is caught, so don't put too much data in it. If the thrown object has a move constructor, it will be moved, which is not expensive. Otherwise it will be copied.

We may subclass a standard library exception (including `runtime_error`, `out_of_range`, etc.) and override the virtual function `what()` to provide description. We may also use `catch (...)` to catch any type of exception:

```cpp
struct MyError1 : std::runtime_error {
  const char* what() const noexcept { return "error 1"; }
};

struct MyError2 : std::out_of_range {
  const char* what() const noexcept { return "error 2"; }
};

void f() {
  try {...}
  catch (const std::exception& e) {  // catch MyError1 or MyError2
    cerr << e.what() << endl;
  }
  catch (...) {  // for all other types of exceptions, clean up and rethrow
    // clean up
    throw;
  }
}
```

When an exception is caught, we can rethrow it. Note that the type of rethrown exception can be different. In the above example, if we put `throw;` in the `catch`-block, the rethrown exception is exatly the previously caught one (of type `MyError1` or `MyError2`); but if we put `throw e;`, the type of the thrown exception will be `std::exception`.

`noexcept` **specifier** can be conditional. We can put an expression that evaluates to `true` or `false`:

```cpp
template<typename T>
void f(T& x) noexcept(is_pod<T>()) {}
```

We may also use a `noexcept` **operator** inside:

```cpp
template<typename T>
void g(vector<T>& v) noexcept(noexcept(f(v[0]))) {}
```

This **operator** does not run the expression (`f(v[0])` in this case), but only checks whether that expression is `noexcept`, and then report to the outer `noexcept` **specifier**.

We can use a `try`-block as the body of a function. This may be useful for constructors:

```cpp
struct X {
  int vi, vs;
  X(int v1, int v2) try
    : vi{v1}, vs{v2} {...}
  catch (const std::exception& e) {...}
};
```

Note that all completely constructed objects will have already been destructed before any `catch`-block is called, so we cannot use them or try to repair the new object. Usually we just throw another exception in the `catch`-block.

To handle an exception on a different thread, use `current_exception` to transfer the exception.

## Chapter 14 Namespaces

Argument-dependent lookup means when a function is called, the compiler don't just lookup the implementation of it in the current namespace, but also namespaces of its auguments:

```cpp
namespace chrono {

class Date {...};
bool operator==(const Date&, const std::string&);

}

void f(chrono::Date d, std::string s) {
  if (d == s) {...}  // implemented in namespace Chrono
}
```

Rules of associated namespaces (AN):

- If an argument is a class member, then AN is the class itself and the enclosing namespaces
- If an argument is a member of a namespace, the AN is the enclosing namespaces
- If an argument is a built-in type, no AN 

Namespaces are open, so we can add functions to namespaces in another place, or separate the interface with the implementation:

```cpp
namespace mine {

void f();  // declaration

}

namespace mine {

void f() {...}  // implementation
void g();  // add `g()` to `Mine`

}

Mine::g() {...}
```

For large projects, we may want to separate it into a user interface (which is stable) and an implementer interface (which may change a lot):

```cpp
namespace mine {

void f();

}

namespace mine_impl {

void f();
void helper1();
void helper2();

}
```

We can use alias to represent namespaces whose names are too long:

```cpp
namespace foundation_library_v2r11 {...}
namespace fib = foundation_library_v2r11;
```

If an explicitly qualified name (like `B::String` below) cannot be found in the namespace (like `B`), the compiler will look for `using`-directives within that namespace, and go to the mentioned namespaces (like `A`) to find the name:

```cpp
namespace A {

class String {...};
void f();

}

namespace B {

using namespace A;

}

void g(B::String);  // this works
void B::f() {...}   // this won't work! should use void A::f()
```

We can use this to extend an exisitng namespace, since everything in `A` is inherited by `B`. Note this doesn't work when we are to define something.

Use both `using`-**directives** (use the entire namespace) and `using`-**declarations** (use something within a namespace) to avoid name clashes:

```cpp
namespace A {

class String {...};
class Vector {...};

}

namespace B {

class String {...};
class List {...};

}

namespace C {

using namespace A;
using namespace B;
using B::String;
class List {...};
// here we can use `Vector` in A, `String` in B, 
// and `List` in C without stating namespaces

}
```

Namespaces can be nested. For example, inner namespaces might be the different version of the same library, in which case we can use `inline` to specify which version is used by default:

```cpp
namespace mine {

namespace v1 {

void f();

}
  
inline namespace v2 {

void f();

}

}

using namespace mine;
void g() {
  f();      // calls `Mine::V2::f()`
  V1::f();  // calls `Mine::V1::f()`
}
```

If we have variables and funtions that should be "global" only in the current file (*i.e.* `static`), we can put them in an **unnamed** namespace to avoid polluting the global namespace and name clashes:

```cpp
namespace {
  void f() {...}
}

// the unnamed namespace above is implicitly used here
f(); // ok
```

## Chapter 15 Source Files and Programs

`static`, `const`, `constexpr` and type aliases imply default internal linkage:

```cpp
static int x1 = 1;
const char x2 = 'a';
```

To make them accessible from other files, we either put them in header files, or:

```cpp
// .h
extern int x1;
extern const char x2;

// .cpp
int x1 = 1;
const char x2 = 'a';
```

If an `inline` function is used in different .cpp files, we cannot only define it in one .cpp file and try to use it in another file. They must be defined in all those files and have exactly the same implementation, because the compiler only works on a single translation unit (.cpp file) and replace each usage of that function with the implementation before invoking the linker. We can put the implementation in the header file instead so that it can be shared easily.

Avoid to use global variables. If really necessary, put them in unnamed namespaces or declare as `static`.

Do **not** put these in header files:

- Ordinary function definitions: `char get(char* p) { return *p++; }`
- Data/Aggregate definitions: `int a; int arr[] = { 1, 2, 3 };`
- Unnamed namespaces: `namespace {...}`
- `using`-directives: `using namespace std;`

Use `extern "C"` to tell the compiler not to mangle the function name of legacy C code:

```cpp
extern "C" int a1;  // declaration
extern "C" void f1();
extern "C" {
  int a2;  // definition
  extern int a3;  // declaration
  void f2();
  #include <string.h>
}
```

Another way to enclose declarations of C functions is to modify the C header file:

```cpp
#ifdef __cplusplus
extern "C" {
#endif
  int strcmp(const char*, const char*);
  int strlen(const char*);
#ifdef __cplusplus
}
#endif
```

In this way the file can still be compiled by the C compiler. We can also put the declaration in a namespace (such as `std::printf`) so that the global namespace will not be polluted.

Previously we tried to separate the interface from implementation by creating two namespaces (where the second one has suffix `_impl`). We can do something similar with header files:

```cpp
// parser.h
namespace parser {   // interface for the user
  double expr(bool get);
}

// parser_impl.h
#include "parser.h"  // let the compiler check consistency

namespace parser {   // interface for the implementer
  double expr(bool get);
  double term(bool get);
  double expr(bool get);
}
```

There is no guaranteed order of initialization of global variables in different translation units, and we cannot catch exceptions in the translation, so we have to minimize the usage of global variables and **not** to initialize with external variables.

Putting a global variable in a function so that it is not initialized until used for the first time, and thus compilation time is reduced:

```cpp
int& use_count() {
  static int uc = 0;
  return uc;
}
```

To terminate a program on purpose:

- Call `abort()`. It does **not** do any cleaning
- Call `quick_exit()`. It does **not** do any cleaning either, but we can set handlers (which will be invoked after `quick_exit()`) with `at_quick_exit()`
- Call `exit()`. Destructors of **static** objects will be called (destructors of **local** objects will **not** called). We can set handlers with `atexit()`

Throwing exceptions ensures destructors of local objects are called, so it is preferred than calling `exit`.

## Chapter 16 Classes

Copy constructor (eg: `X x{y};`) and assignment (eg: `X x = y;`) are provided by default (memberwise copy). Move constructor and assignment are also provided as memberwise move.

Use `explicit` to avoid implicit conversion, especially for signle-argument constructors:

```cpp
class X {
 public:
  explicit X(int v);
 private:
  int x;
};

X::X(int v) : x{v} {}  // no need to specify `explicit` again

X x1{1};      // ok
X x2 = X{1};  // ok. move constructor may be called
X x3 = {1};   // error
X x4 = 1;     // error
```

The member function defined within the class definition is implicitly inlined. It should be small, rarely modified and frequently used.

If a member variable is marked `mutable`, it can be modified even if it is in a `const` object. Another way is to store a pointer to that variable, so that its mutability is independent of the object.

`static` member variables must be declared within the definition of the class and then defined somewhere else even if it is marked `private`, **unless** their are `const` integral or enumeration types, or `constexpr` literal types:

```cpp
class Date {
 public:
  Date(int dd = 0, int mm = 0, int yy = 0) {
    d = dd ? dd : default_date.d;
    m = mm ? mm : default_date.m;
    y = yy ? yy : default_date.y;
  }
 private:
  int d, m, y;
  static const int x1 = 1;      // fine
  static const float x2 = 0.1;  // error
  static Date default_date;     // only declaration
};

Date Date::default_date{1, 1, 1970};  // definition
void f(Date);
void g() {
  f({});  // equivalent to passing a copy of Date::default_date
}

```

A member class (also called nested class) can access types and `static` member variables of its enclosing class. When given an instance of the enclosing class, the nested class can access members of it even if it is `private`. However, given an instance of the nested class, its enclosing class cannot access `private` members of it.

If a function is associated with a class, but does not need to directly access `private` members, instead of making it a member function, we can put it outside of the class definition but within the same enclosing namespace, so that the interface of the class is simpler and we can still use those functions via argument-dependent lookup:

```cpp
namespace chrono {
  class Date {
   public:
    Date (int=0, int=0, int=0);
   private:
    int d, m, y;
  };

  bool operator==(const Date& a, const Date& b);
  const Date& default_date() {  // lazy static variable
    static Date d{1, 1, 1970};
    return d;
  }

  // constructor should be able to see `default_date()`
  // so it must be defined outside of the class denition
  Date::Date(int dd, int mm, int yy) {
    d = dd ? dd : default_date().d;
    m = mm ? mm : default_date().m;
    y = yy ? yy : default_date().y;
  }
}
```

## Chapter 17 Construction, Cleanup, Copy and Move

We can mark a destructor `= delete` to prevent from allocation on stack. However, in that case we won't be able to call `delete` on instances allocated on free store anymore. An alternative way is to mark the destructor `private`:

```cpp
class X {
public:
  void destroy() { this->~X(); }
private:
  ~X();
};

void f() {
  X* x = new X();  // cannot allocate on stack anymore
  x->destroy();    // cannot `delete x` anymore
}
```

If any method of a class is `virtual`, probably this class will be subclassed. It might be necessary to mark its destructor `virtual` as well so that the destructor of derived class will always be called.

For local variables and free-store objects, if we don't use any constructor, members of class types will still be initialized while members of built-in types are left uninitialized (might be useful for performance critical programs):

```cpp
class Y {
  char buf[1024];
  Date date;
};

Y y0;  // static object, fully initialized
       // `buf` is filled with 0

void f() {
  Y y1{};  // constructor is called, fully initialized
  Y y2;    // local variable, `buf` is uninitialized 
           // while `date` is set to default date
}
```

A default memberwise constructor is provided unless some members are `const` or references. If we define a custom constructor that requires arguments, the default one will disappear, but the default assignment and copy constructors still exist.

Containers are able to use elements in initilizer lists to construct their content:

```cpp
struct X { X(string); };

void f() {
  X a1[10];  // error, cannot initlize
  X a2[] {string{"alpha"}, string{"beta"}};  // constructs two elements
  vector<X> v1;  // fine, no element
  vector<X> v2{string{"alpha"}, string{"beta"}};  // constructs two elements
}
```

If a class has multiple constructors, resolution rules are (always prefer the simpler constructor):

- If both a default constructor and a an `initializer_list` constructor might be invoked, prefer the default constructor
- If both an `initializer_list` constructor and an ordinary constructor might be invoked, prefer the `initializer_list` constructor

We can pass `initializer_list` by value since it is a simple wrapper of the underlying array and doesn't take much space. It provides methods `begin()`, `end()` and `size()` but doesn't support subscripting. It is immutable, so we **cannot** modify any element of it or apply a move constructor.

The constructor can initialize the base, but **cannot** initialize the base of base:

```cpp
struct X { X(int); };
struct XX: X { XX(int); };
struct X1: XX { X1(int a):  X(a) {} };  // error
struct X2: XX { X2(int a): XX(a) {} };  // ok
```

The constructor can also delegate to another constructor, but it **cannot** explicitly initialize a member at the same time:

```cpp
struct X {
  int a, b;
  X(int aa, int bb): a{aa}, b{bb} {}
  X(int aa): X{aa, 0} {}
  X(int aa): X{aa, 0}, a{aa} {}  // error
};
```

Bases are initialized in the order of declaration. They are initialized before members of the derived class, and destroyed after members. Members are also constructed in the order declared in the class definition, and destructed in the reversed order:

```cpp
struct X1 { X1(int x); };
struct X2 { X2(int x); };
struct XX: X1, X2 {
  int a1;
  int a2;
  XX(int x1, int x2, int a1, int a2)
    : X2{x2}, X1{x1}, a2{a2}, a1{a1} {}
};
// construction: X1 -> X2 -> a1 -> a2
//  destruction: a2 -> a1 -> X2 -> X1
```

**Copy constructors** usually acquire resources so they may throw exceptions. **Copy assignments** may throw as well even if some of them don't have to acquire resources. Imagine if a matrix is copied to another one but it happens that they have different dimensions, we may throw an exception. **Move operations** transfer ownership of existing resources so they shouldn't throw exceptions.

We should be especially careful about copy/move operations if a class contains pointer members. A shallow copy of an object containing pointers results in shared resources between objects. We may declare `shared_ptr` to wrap those pointers so that the user of the code will notice the shallow copy. We can either use a boolean flag to do copy-on-write:

```cpp
class Image {
 public:
  Image(const Image&);
  void write_block();
 private:
  bool shared;
  Data* data;  // expensive resource
  Data* clone();
};

Image::Image(const Image& img)  // shallow copy
    : shared{true}, data{img.data} {}

void write_block() {
  // deep copy only when we need 
  // to write to a shared resource
  if (shared) {
    data = clone();
    shared = false;
  }
  // write to own copy of data
}
```

When copy/assign an object of the derived class to an object of the base class, the copy constructor/assignment of the **base** class will be called, which is called **slicing**. To avoid it:

- Mark the copy constructor/assignment of the base class `= delete`
- Make the base class a `private` or `protected` base in the definition of derived class

If any constructor is declared, the default constructor will not be generated. For copy/move operations and the destructor, if any of them is declared, all others will not be generated. Sometimes the default implementation is more appropriate. For example, the default copy constructor guarantees to copy all member variables, while we may omit some of them in our own implementation. To get the default version back:

```cpp
struct X {
  ~X() {}  // no default copy/move operation provided anymore
  X(const X&) = default;  // restore default implementation
};
```

Usage of `delete`:

```cpp
struct X {
  X(int) = delete;  // eliminate undesired conversion
  void* operator new(size_t) = delete;  // prevent allocation on free store
};

template<class T>
T* clone(T* p) { return new T{*p}; }

X* clone(X*) = delete;  // eliminate a specification
```

## Chapter 18 Operator Overloading

Passing small objects by value might be more efficient since directly accessing an object is always faster than a reference.

It is recommended to define operators that needs direct access to the representation of class within the class definition, and those simply produce a new value be defined as nonmember functions. Note that only for nonmember functions, the left-hand side can be other types that are convertable to the argument type:

```cpp
struct Complex {
  double re, im;
  Complex(double r = 0, double i = 0): re{r}, im{i} {}
  Complex& operator+=(Complex a) {
    re += a.re;
    im += a.im;
    return *this;
  }
};

Complex operator+(Complex a, Complex b) {
  return a += b;
}

Complex f() {
  return 1 + Complex{2, 3};  // LHS is not Complex
}
```

The above overloading of `operator+` passes parameters by value, so we don't have to explicitly create a new object. Function `f` relies on constructors to convert `int` -> `double` -> `Complex`, so that we don't have to implement different versions of this function for different input types, and both LHS and RHS don't have to be `Complex`.

Relying on constructors to convert types has some flaws:

- Cannot convert to built-in types since they are not class
- Cannot avoid to modify the definition of destination class

Another way is to define a **user-defined conversion** function `X::operator T()`, where `X` is source type and `T` is destination type. Note that narrowing should be avoided. This function can be marked `explicit`, with which we may need to use `static_cast` to do explicit conversion:

```cpp
struct X {
  int x;
  X(int xx): x{xx} {}
  operator int() const { return x; }
  explicit operator string() const { return to_string(x); }
};

void f() {
  X one{1}, two{2};
  int i = two - one;
  one = two - one;
  string s1 = x;  // error
  string s2 = static_cast<string>(one);  // ok
}
```

Conversion to `bool` is a bit different. Other types can be "contextually converted to `bool`" without a cast:

```cpp
struct X {
  bool b;
  explicit operator bool() const { return b; }
};

void f() {
  X x{true};
  if (x)         // ok. contextual convertion
    bool b = x;  // error
}
```

For overload resolution, only one level of user-defined conversion will be considered, and built-in conversions are preferred over user-defined ones:

```cpp
struct X { X(int); };
struct Y { Y(X); };

void f(Y);
void g(double);
void g(X);

void h() {
  f(1);  // error. int -> X -> Y requires two steps
  g(1);  // g(double) will be called instead of g(X)
}
```

## Chapter 19 Special Operators

`->` is a unary postfix operator:

```cpp
class Ptr {
  X* operator->();
};

void f(Ptr p) {
  X* x = p.operator->();
  p->m = 1;  // (p.operator->())->m = 1
}
```

The assignment is **redirected** to a member of the underlying object of type `X`. The return type should be a pointer of an object that we can apply `->` since an `->` is implicitly inserted. This is important for implementing smart pointers and iterators.

Overloading increment and decrement ():

```cpp
class Ptr {
  Ptr& operator++();    // prefix
  Ptr operator++(int);  // postfix

  Ptr& operator--();    // prefix
  Ptr operator--(int);  // postfix
};
```

The parameter for postfix is a dummy placeholder to differentiate itself from prefix. Avoid to use postfix since in that case we need to create a temporary object to store the previous value.

Overloading allocation and deallocation:

```cpp
class Ptr {
  void* operator new(size_t);
  void operator delete(void*, size_t);

  void* operator new[](size_t);
  void operator delete[](void*, size_t);
}
```

They are implicitly `static` so they cannot access `this`. The size to be (de)allocated is passed as a parameter, which is determined by the type of the object following `delete`, so that we don't have to store it in a variable. Note that if this class is subclassed, we should mark these operators `virtual`, otherwise if a pointer of the base type points an object of a derived type, `delete` may receive a wrong size.

Use **user-defined literals** to enable expressions like `1.2_km`. Note that the standard library reserves all suffixes not starting with an `_`, and we'd better put these literals in a separate namespace for selective use:

```cpp
long double operator"" _km(long double x) {
  return x * 1000;
}

string operator"" _s(const char* p, size_t n) {
  return string{p, n};
}

void f() {
  long double distance = 1.2_km;
  cout << distance << endl;   // 1200
  cout << "Hello"_s << endl;  // converted to a `string`
}
```

There are four kinds of literals that can be defined:

- Integer literal (taking `unsigned long long` or `const char*`)
- Floating-point litral (taking `long double` or `const char*`)
- String literal (taking a <`const char*`, `size_t`> pair)
- Character literal (taking `char`, `wchar_t`, `char16_t` or `char32_t`)

With `friend` declarations within the definition of a class, it can grant access to private fields to other function/class:

```cpp
void f() {...};

class ListIterator {...};

class List {
 public:
  friend void ::f();
  friend class ListIterator;
  // following function is a non-member friend function
  friend ostream& operator<<(ostream& os, const List& list);
 private:
  // accessible for f() and all methods in ListIterator
  size_t size;
  int* data;
};
```

A friend function/class must be declared before the class to which they are friend, or in the same enclosing namspace:

```cpp
class C1;  // ok

namespace {

class CX {
public:
friend class C1;
friend class C2;
friend class C3;
}
  
class C2;  // ok

}  // namespace

class C3;  // error: not a friend of CX
```