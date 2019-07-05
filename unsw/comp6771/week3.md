# UNSW COMP6771 (Advanced C++) Week 3 OOP
This week is about Object Oriented Programming in C++.

## Object Scope & Lifetime 
- Oppose to first year courses where variables should be defined at the top, it is especially bad in C++
- Every variable is an object, but does not add overhead
- Lifetime starts when variable comes in scope (constructed)
- Lifetime ends when it goes out of scope (destructed)

### Construction
`()`: only for functions  
`{}`: either functions or objects, generally objects to minimise ambiguity
```cpp
int main() {
  std::vector<int> v11; // Calls 0-argument constructor. Creates empty vector.
  // There's no difference between these:
  // T variable = T{arg1, arg2, ...}
  // T variable{arg1, arg2, ...}
  std::vector<int> v12{}; // No different to first
  std::vector<int> v13 = std::vector<int>(); // No different to the first
  std::vector<int> v14 = std::vector<int>{}; // No different to the first

  std::vector<int> v3{v2.begin(), v2.end()}; // constructed with an iterator
  std::vector<int> v4{v3}; // Constructed off another vector

  std::vector<int> v51{5, 2}; // Initialiser-list constructor {5, 2}
  std::vector<int> v52(5, 2); // Count + value constructor (5 * 2 => {2, 2, 2, 2, 2})
}
```

For basic types, constructors works but default constructors need to be **manually** called.
```cpp
int main() {
  int n; // not constructed (memory contains previous value)
  int n2{}; // Default constructor (memory contains 0)
  int n3{5};

  // This version is nice because it gives us an error.
  int n4{5.5};
  // You need to explictly tell it you want this.
  int n6{static_cast<int>(5.5)};

  // Not so nice. No error
  int n5 = 5.5;
}
```

#### Default constructor (synthesized default constructor)
Unless we define our own constructors the compile will declare a default constructor
```text
for each data member in declaration order
  if it has an in-class initialiser
    Initialise it using the in-class initialiser
  else if it is of a built-in type (numeric, pointer, bool, char, etc.)
    do nothing (leave it as whatever was in memory before)
  else
    Initialise it using its default constructor
```
Cannot be generated when any data members are missing both in-class initialisers and default constructors
```cpp
class A {
  int a_;
};

class B {
  B(int b): b_{b} {}
  int b_;
};

class C {
  int i{0}; // in-class initialiser
  int j; // Untouched memory
  A a;
  // This stops default constructor
  // from being synthesized.
  B b;
};
```

#### Constructor initialiser list
Data members are constructed **in order of memebr declaration**.
```cpp
class NoDefault {
  NoDefault(int i);
}

class B {
  // Constructs s_ with value "Hello world"
  B(int& i): s_{"Hello world"}, const_{5}, no_default{i}, ref_{i} {}
  // Doesn't work - constructed in order of member declaration.
  B(int& i): s_{"Hello world"}, const_{5}, ref_{i}, no_default{ref_} {}
  B(int& i) {
    // Constructs s_ with an empty string, then reassigns it to "Hello world"
    // Extra work done (but may be optimised out).
    s_ = "Hello world";

    // Fails to compile
    const_ = "Goodbye world";
    ref_ = i;
    // This is fine, but it can't construct it initially.
    no_default_ = NoDefault{1};
  }

  std::string s_;
  // All of these will break compilation if you attempt to put them in the body.
  const int const_;
  NoDefault no_default_;
  int& ref_;
};
```

#### `default`
`<function declaration> = default;` tells the compiler to synthesize the function

#### Copy Constructor
- Constructs one object to be a copy of another
- The compiler-generated copy-constructor just calls each member's copy constructor in order of declaration
```cpp
class T {
  T(const T&);
};
```

#### Copy Assignment
- Like a copy constructor, but the destination is already constructed
- Requires destroying the old data, and constructing the new data
- Copy-and-swap idiom is an elegant way of doing this
    - It constructs then destructs. Since construction might fail, it should go first
    - Requires move assignment to be defined
- Takes in an lvalue
- Compiler-generated one performs memberwise copy-assignment operator
```cpp
class T {
  // A copy-assignment operator
  T& operator=(const T& original);

  // The copy-and-swap idiom
  // This is also a copy-assignment operator
  T& operator=(T copy) {
    std::swap(*this, copy);
    return *this;
  }
};
```
```cpp
MyClass base;
MyClass copy_constructed = base;

MyClass copy_assigned;
copy_assigned = base;
```

#### Rvalue Eeferences
- Rvalue references look like T&& (lvalue is T&)
- An lvalue denotes an object whose resource cannot be reused
- An rvalue denotes an object whose resources can be reused

```cpp
void inner(int&& value) {
  ++value;
  std::cout << value << '\n';
}

void outer(int&& value) {
  inner(value); // This fails? Why?
  std::cout << value << '\n';
}

int main() {
  f1(1); // This works fine.
  int i;
  f2(i); // This fails because i is an lvalue.
}
```
The caller (main) has promised that it won't be used anymore, and an rvalue reference parameter is an lvalue inside the function.

`std::move` converts something to an rvalue, which says "I don't care about this anymore", and all it does is to allow the compiler to use rvalue reference overloads.
```cpp
void inner(int&& value) {
  ++value;
  std::cout << value << '\n';
}

void outer(int&& value) {
  inner(std::move(value));
  // Value is now in a valid but unspecified state.
  // Although this isn't a compiler error, this is bad code.
  // Don't access variables that were moved from, except to reconstruct them.
  std::cout << value << '\n';
}

int main() {
  f1(1); // This works fine.
  int i;
  f2(std::move(i));
}
```

#### Move Constructor
- Always should be declared noexcept
- Unless otherwise specified, objects that have been moved from are in a valid but unspecified state
- Will likely be faster than the copy constructor
- Compiler-generated one performs memberwise move-construction
```cpp
class T {
  T(T&&) noexcept;
};
```

#### Move Assignment
- Always should be declared noexcept
- Like the move constructor, but the destination is already constructed
- Compiler-generated one performs memberwise move-assignment
```cpp
class T {
  T& operator=(T&&) noexcept;
};
```

### Destruction
`noexcept`: compiler wont generate recovery code. Exception thrown in `noexcept` function will terminate the program. `noexcept` tells caller there is no need to worry about exception handling.  
Destructors are called when objects goes out of scope. It is marked as `noexcept` because it is not part of controlled process. In practice, implicit destructors are noexcept unless the class is "poisoned" by a base or member whose destructor is noexcept(false).  

### RAII (Resource Acquisition Is Initialisation)
- Acquire the resource in the constructor​
- Release the resource in the destructor
- Recourse examples: memory, locks, files
- Every resource should be owned by either:
    - Another resource (eg. smart pointer, data member)
    - The stack
    - A nameless temporary variable
    
## Incomplete Types
An incomplete type may only be used to define pointers and references, and in function declarations (but not definitions). A class cannot have data members of its own.
```cpp
struct Node {
  int data;
  // Node is incomplete - this is invalid
  // This would also make no sense. sizeof(Node) is unknown
  Node next;
};

struct Node {
  int data;
  // this is fine
  Node* next;
};
```

## Member access control
```cpp
class Foo {
 public:
  // Members accessible by everyone
  Foo();

 protected:
  // Members accessible by members, friends, and subclasses

 private:
  // Accessible only by members and friends
  void PrivateMemberFunction();

  int private_data_member_;

 public:
  // May define multiple sections of the same name
};
```

## `class` vs `struct`
**Only** difference:
- All members of a struct are public by default
- All members of a class are private by default

Difference is semantic, not technical. `std::pair` and `std::tuple` are struct templates.

## `friend` functions
- they are non-member functions
- can access private parts of the class
- Generally bad but sometimes required
    - Non-member operator overloads
    - related classes
        - A Window class might have WindowManager as a friend
        - A TreeNode class might have a Tree as a friend
        - Container could have iterator_t<Container> as a friend
- use when
    - The data should not be available to everyone
    - There is a piece of code very related to this particular class
    
## Class Scope `::`
Anything declared inside the class needs to be accessed through the scope of the class, using `::`
```cpp
// foo.h

class Foo {
 public:
  // Equiv to typedef int Age
  using Age = int;

  Foo();
  Foo(std::istream& is);
  ~Foo();

  void MemberFunction();
};
```
```cpp
// foo.cpp
#include "foo.h"
 
Foo::Foo() {
}
 
Foo::Foo(std::istream& is) {
}
 
Foo::~Foo() {
}
 
void Foo::MemberFunction() {
  Foo::Age age;
}
```

## `this` pointer
- Member functions has `this` as a parameter implicitly
- The compiler treats an unqualified reference to a class member as being made through the this pointer.
- it is top-level const

## `static` members
- static data has global lifetime
- both function and data may be declared static
```cpp
// For use with a database
class User {
  static std::string table_name;
  static std::optional<User> query(const std::string& username);
  
  void commit();
  std::string username;
}

User user = *User::query("Alice");
user.username = "Bob"
User::commit(); // fails to compile (commit is not static)
user.commit();

std::cout << User::table_name;
std::cout << User::username; // Fails to compile
```

## `explicit` type conversions
If a constructor for a class has 1 parameter, the compiler will create an implicit type conversion from the parameter to the class, can be turned off by the keyword `explicit`.
```cpp
class Age {
  Age(int age);
};

// Explicitly calling the constructor
Age age{20};
// Attempts to use an integer
// where an age is expected.
// Implicit conversion done.
// This seems reasonable.
Age age = 20;
```
```cpp
class IntVec {
  // This one allows the implicit conversion
  IntVec(int length): vec_(length, 0);

  This one disallows it.
  explicit IntVec(int length): vec_(length, 0);

  std::vector<int> vec_;
};

// Explictly calling the constructor.
IntVec container{20};

// Implicit conversion.
// Probably not what we want.
IntVec container = 20;
```

## `namespace`
```cpp
// path/to/file.h

namespace path {
    namespace to {
        class MyClass {
        }
        
        void MyFn();
    } // namespace to
} // namespace path
```
```cpp
// main.cpp
#include "path/to/file.h"

int main() {
  path::to::MyClass myClass;
  path::to::MyFn();
}
```

### `using`
There are two ways to use
#### Declaration
```cpp
#include "path/to/file.h"

int main() {
  // Imports a single thing
  // into the current scope.
  using path::to::MyClass;
  MyClass myClass;
}
```
#### Type Alias
```cpp
// Far cleaner than typedef, and the
// arguments are the right way around!
int main() {
  using intvec = std::vector<int>;
  intvec vec;

  C::iterator_t it;
}

class IteratorC {
}

class C {
  using iterator_t = IteratorC;
}
```

## **Never** write `using namespace std;`
The most mind blowing thing this week is about `using namespace std;`. Previously I have seen a lot of examples make use of it and it worked fine. And it makes codes shorter. But in lecture we are told not to write it, which confused me. After discussion with lecturer, I completely changed my mind: `shorter != cleaner`. I thought that shorter code is cleaner to read. But it was wrong. `using namespace` will pollute namespaces and it makes code actually harder to read (You dont know `map` is `std::map` or `map` of from somewhere else). It sacrifices readability to gain shorter code, which is completely not worthy. Readability matters, and shorter decreases readability. Shorter code is never the goal to reach, code is for human to read.
