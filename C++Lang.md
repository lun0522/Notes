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