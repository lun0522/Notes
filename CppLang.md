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
int x = 1;  // global x

void f() {
    int x = 2;  // local x
    ::x = 3;  // assign to the global x
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
int x {};  // 0
int y;  // no well-defined value

int a1[100] {};  // all elements initilized to 0
int a2[100];  // can be anything

vector<int> v1 {};
vector<int> v2;  // the same to v1
```

To use the default initializer we have to use `{}` rather than `()`, since `X a();` is function calling instead of initialization.

Note that when we use `auto`, for `auto x1 {1};` and `auto x2 {1, 2, 3};`, both `x1` and `x2` are deduced to be `initializer_list<int>`. Use `=` instead of `{}` with `auto` when we don't want a list.

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
char *p1 = "how";  // error
const char *p2 = "are";  // ok
char a[] = "you";  // will make a copy, also ok
```

Multi-line string and raw string:

```cpp
char a2[] = "long "
            "string";  // the same to "long string"
string s1 {R"("long" "string")"}  // the same to ""long" "string""
string s2 {R"(long
string)"};  // the same to "long\nstring"
```

If `char[]` is passed as an argument, it will be treated as `char*`, and thus `sizeof(arr)` will return the size of the pointer itself and we cannot use it to calculate the size of the array. Besides, `for (char& c : arr) {..}` won't compile since the size is unknown. We may have to pass in the size of the array as well.

Read from right to left to find out what does `const` decorate:

```cpp
char a[] = "hello";
const char* p1 = a;  // pointer to const char
char const* p2 = a;  // pointer to const char
char *const p3 = a;  // const pointer to char
const char *const p4 = a; // const pointer to const char
```

For a `const` lvalue reference, the original object can be rvalue or of different type, in which case implicit type conversion is performed if necessary, the resulting value is **copied** to a temporary object, and a reference is created referencing to it. Although we cannot say `int& x = 2;`, we can say `const int& x = 2.0;` since a temporary object (a lvalue) holds 2.

We can also use rvalue reference (`&&`) for temporary objects (so-called *destructive read*) (we do **not** use `const` here since moving implies modifying):

```cpp
string s1 {"hello"};
string func();

string&& rc1 {s1};  // error, cannot initialize with a lvalue
string&& rc2 {"world"};  // ok
string&& rc3 {func()};  // ok
// actually rc2 and rc3 are lvalue now
// && only means they came from rvalue

string s2 = static_cast<string&&>(rc2);  // rc2 will be destructed
string s3 = move(rc2);  // equivalent to static_cast
```

## Chapter 8 Structures, Unions, and Enumerations

Some `class` and `struct` are Plain Old Data types, which don't have user-defined constructors, destructors or virtual member functions. They are stored in contiguous sequence of bytes, so we can simply use `memcpy` to copy an array of them. We can use `is_pod` to predicate whether a type is POD.

To optimize the memory usage of `struct`, we can reorder elements in it to reduce the memory waste caused by alignments. Readability is usually considered more important than this improvement.

Prefer `enum class` to plain `enum`. For `enum class`, enumerators are in the scope (namespace) of `enum class`, and they cannot be implicitly converted to other types (eg: `int`) unless we use `static_cast`. We can specify the underlying data type: `enum class Color : char { red, green, blue };`

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
string s1 {"hello"};
string s2 {"world"};
size_t length = strlen((s1 + s2).c_str()); // ok
const char* cs = (s1 + s2).c_str(); // using cs may cause error
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
    constexpr const char* p1 = "hello"; // ok
    constexpr const char* p2 = new char(5); // error, it is allocated on the free store
    static const char a[] = "world";
    constexpr const char* p3 = a; // ok
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
int* p = new(nothrow) int[10000]; // p may be nullptr
if (!p) {...}
operator delete(p, nothrow);
```

The `new` operator can be overloaded. We can have a memory management class:

```cpp
class Arena { // memory manager
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
    X* p = new(storage) X {3};
    // use p
    p->~X();
    storage->free(p);
}
```

`new` actually contains two steps:

1. allocate enough space (don't care about the object type)
2. cast `void*` to `X*` and call the constructor

What we overload is the first step, while the second step is done as usual. In the line `X* p = new(storage) X {3};`, `X` is used to calculate the first argument `sz`, so we only have to pass one argument of type `Arena*` to `new`, which is called placement `new`. After that, `3` is sent to the constructor.

Note that we cannot use `delete` to deallocate the object since it was not allocated on the free store but by the manager, so we have to call the destructor explicitly and let the manager free the space. We can overload `delete` just like what we did to `new`, but we **cannot** call it by ourselves. It is actually used to handle the case where the constructor throws excpetions. See Wiki.

`new X {...}` will construct the object on the free store, while `X {...}` will contruct in the local scope. We can do this to eliminate ambiguity:

```cpp
struct X { int a, b; };
struct XX { double a, b; };
void f(X);
void f(XX); // overloaded

void g() {
    f({1, 2}); // error, ambiguous
    f(X {1, 2}); // ok
    f(XX {1, 2}); // ok
}
```

To create a `std::initializer_list`, elements between curly braces are copied (and converted to the target format if necessary) to the underlying array of `initializer_list`. It is **immutable**, and thus when we use it to initialize another object (eg: `std::vector`), elements are copied rather than moved to the new object. If we directly use a `{}`-list for constructor arguments or to initilize an aggregate, elements won't be copied (unless as by-value arguments):

```cpp
int v1 {1}; // direct initialization
int v2 = {2}; // copy initialization
```

We can use `initializer_list` to deal with immutable homogenous lists with varying lengths:

```cpp
int sum(initializer_list<int> val) {
    int s = 0;
    for (auto v : val) s += v;
    return s;
}

sum({1, 3, 5, 7, 9});
sum({1.0, 3, 5, 7, 9.0}); // error, not homogenous
```

Note that the compiler does not deduce the type of the entire list. We still have to specify the container type because of the language restriction:

```cpp
template<typename T>
void f1(T);

f1({1, 2, 3}); // won't compile

template<typename T>
void f2(const initializer_list<T>&);

f2({1, 2, 3}); // ok

template<typename T>
void f3(const vector<T>&);

f3({1, 2, 3}); // error
f3(vector<int>{1, 2, 3}); // ok
```

Forms of capture lists of lambda expressions:

- `[]`: no capture
- `[&]`: capture all by reference
- `[=]`: capture all by value
- `[capture-list]`: those preceded by `&` captured by reference, while others captured by value; it can contain `this` and names followed by `...` (variadic template arguments)
- `[&, capture-list]`: capture those in the list by value and others by reference; the list can contain `this`
- `[=, capture-list]`: capture those in the list by reference (all of them should be preceded by `&`) and others by value; the list cannot contain `this` since we never capture it by reference (`this` is a pointer, just capture it by value)

If we capture `this`, all member variables are implicitly captured by reference (**no** need to put them in the capture list). For multi-threading, we'd better capture by value to avoid data race.

To modify the state of a variable captured by value, we can declare `mutable` before the body. However, different from capturing by reference, this modification is done on the local copy in the closure, so it is invisible to the outside world:

```cpp
int x = 10;
auto func1 = [&x] () { x += 10; };
func1(); // x becomes 20
auto func2 = [x] () mutable { x += 10; };
func2(); // x becomes 30, but still 20 to the outside world
func2(); // x becomes 40, but still 20 to the outside world
```

The return type of lambda expressions can be deduced by the compiler. If a lambda does not take parameters, we can omit `()`, but if we specify a return type with `->` or declare `mutable`, `()` can not be ommited. We can specify the type of lambda as `std::function<return type (arguments list)>`. This is necessary for recursion, because if we use `auto`, the type of itself has not been deduced inside of its body, and thus we cannot call itself recursively. Example for recursion:

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