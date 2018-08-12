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