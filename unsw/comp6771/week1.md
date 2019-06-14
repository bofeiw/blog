# UNSW COMP6771 (Advanced C++) Week 1

# C++ vs C
```C++
// helloworld.cpp
#include <iostream>

int main() {
  std::cout << "Hello, world!\n";
  return 0;
}
```
- most things are syntactically same in comparison to C
- `#include` with no `.h` and library differs
- namespace and output stream
- compile: `g++ -std=c++17 -o helloworld helloworld.cpp`

## "\n" vs std::endl
```C++
// The following two are equivalent
 
std::cout << "\n" << std::flush;
std::cout << std::endl;
```   
When streaming things to `stdout`, things are stored in a buffer. things are sent to terminal when the buffer is flushed. `std::endl` sends a `\n` and also do a flush. For devices with limited computational power, `\n` is better as it gives better performance.

# Runtime error
```C++
#include <string>

int main() {
  std::string s = "";
  s[0];
}
```
Runtime error is compiler-dependent. This error might crash and also might be ignored.

# `const`
```C++
#include <iostream>
#include <vector>

int main() {
    const int i = 0; // i is an int
    i++; // not allowed
    std::cout << i << '\n'; // allowed
    
    const std::vector<int> vec;
    vec[0]; // allowed
    vec[0]++; // not allowed
    vec.push_back(0); // not allowed
}
``` 
- The value cannot be modified
- Make everything const unless you know it will be modified
- You can know that a function won't try and modify something just from the signature
- Immutable objects are easier to reason about
- The compiler **may** be able to make certain optimisations
- Immutable objects are **much** easier to use in multithreading
- Don't have to worry about race conditions between threads
- Prevents bugs from changing things you shouldn't

# References
```C++
int i = 1;
int j = 2;

int& k = i;
k = j; // This does not make k reference j instead of i. It just changes the value.
std::cout << "i = " << i << ", j = " << j << ", k = " << k << '\n';
```
Reference is similar to C, but without the need of using `->` to access the value. It is like an alias to the original object. Reference is always `const` and is not null.

## References to `const`
```C++
int i = 1;
const int& ref = i;
std::cout << ref << '\n';
i++; // This is fine
std::cout << ref << '\n';
ref++; // This is not
```

## References to literal value
```C++
const int &i = 1; // this works
int &i = 1; // this does not
```
When references to literal value, it must be `const`.

```
const int j = 1;
const int& jref = j; // this is allowed
int& ref = j; // not allowed
```
The value of the object cannot be modified through the `const` reference but the original object can still be modified. Reference to `const` must also be `const`.

# Class templates
`std::optional<T>`, `std::vector<T>`, `std::unordered_map<KeyT, ValueT>` for example.
```C++
#include <unordered_map>
#include <vector>

// The following items are all function DECLARATIONS

// Not allowed - type templates are not types
std::vector GetVector();

// std::vector<int> and std::vector<double> are valid types.
std::vector<int> GetIntVector();
std::vector<double> GetDoubleVector();

// So is combining types
std::vector<std::unordered_map<int, std::string>> GetVectorOfMaps();
```
- Not class
- Syntactically similar to Java's generics, but **not** the same
- `std::vector<int>` and `std::vector<string>` are 2 different types, unlike Java

# `auto`
```C++
auto i = 0; // i is an int

std::vector<int> fn();
auto j = fn(); // j is std::vector<int>

// Pointers
int i;
const int *const p = i;
auto q = p; // const int*
auto const q = p;

// References
const int &i = 1; // int
auto j = i; // int
const auto k = i; // const int
auto &r = i; // const int&
```
Compiler will deduct the actual type of `auto`. Take exactly the type on the right-hand side but strip off the top-level const and &.

# Functions
## Default function values
```C++
string Rgb(short r = 0, short g = 0, short b = 0);
Rgb();// rgb(0, 0, 0);
Rgb(100);// Rgb(100, 0, 0);
Rgb(100, 200);     // Rgb(100, 200, 0)
Rgb(100, , 200);   // error
```
Default arguments are like Python, but arguments cannot be specified when calling it. And if one parameter uses default argument, all other parameters also need to have a default value. So `void test(int a = 0, int b);` does not work.
## Pass by reference
```C++
void swap(int& x, int& y) {
  int tmp;
  tmp = x;
  x = y;
  y = tmp;
}
```
With the pass by reference, to use it, just need to call it without pointer styles:
```C++
swap(i, j);
```
Pass by reference is useful when:
- The argument has no copy operation
- The argument is large  

Pass by reference is not compatible with C as it does not support it.

## Lvalue and Rvalue
- rvalues may only appear on the RHS of an assignment
- lvalues may appear on both the LHS and RHS
### Call by value
- The rvalue of an actual argument is passed 
- Cannot access/modify the actual argument in the callee
### Call by reference
- The lvalue of an actual argument is passed
- May access/modify directly the actual argument
- Eliminates the overhead of passing a large object

# Function Overloading
```C++
void Print(double d);      // (1)
int  Print(std::string s); // (2)
void Print(char c);        // (3)
Print(3.14);               // call (1)
Print("hello World!");     // call (2)
Print('A');                // call (3)
```
C++ function overloading is very similar to Java function overloading. Return types are ignored when overloading.  
Top-level `const` is ignored but low-level `const` is not.
```C++
// Top-level const ignored
Record Lookup(Phone p);
Record Lookup(const Phone p); // redefinition

// Low-level const not ignored
Record Lookup(Phone &p); (1)
Record Lookup(const Phone &p); (2)

Phone p;
const Phone q;
Lookup(p); // (1)
Lookup(q); // (2)
```
 
# `constexpr`
```C++
// Beats a #define any day.
constexpr int max_n = 10;

// This can be called at compile time, or at runtime
constexpr int ConstexprFactorial(int n) {
  return n <= 1 ? 1 : n * ConstexprFactorial(n - 1);
}
constexpr int tenfactorial = ConstexprFactorial(10);

// This may not be called at compile time
int Factorial(int n) {
  return n <= 1 ? 1 : n * Factorial(n - 1);
}
// This will fail to compile
constexpr int ninefactorial = Factorial(9);
```
A variable that can be calculated at compile time is calculated at compile time. A function whose inputs are known at compile time will run at compile time.  
As the computations are done at compile time, the run time of the program will be faster and less computation will be needed at run time.

# Build Systems (bazel)
```bazel
// path/to/BUILD

cc_library(
  name = "hello_world",
  srcs = ["hello_world.cpp"],
  hdrs = ["hello_world.h"],
  deps = []
)

cc_library(
  name = "printer",
  srcs = ["printer.cpp"]
  hdrs = ["printer.h"],
  deps = [
    # If it's declared within the same build
    # file, we can skip the directory
    ":hello_world"
  ]
)
```
Like make, bazel is another open source build system.
## build rules
- name
- list of sources `srcs`
- lsit of headers `hdrs`
- list of dependencies `deps`
## types of build rules
- `cc_library` A piece of code that can't run on its own, but can be depended upon by other files
- `cc_binary` The srcs should have a main function, has no headers, cannot be tested
- `cc_test` Works very similar to a binary, semantic difference
