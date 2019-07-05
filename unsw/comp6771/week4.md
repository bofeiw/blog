# UNSW COMP6771 (Advanced C++) Week 3 Operator Overloading

This week, various topics in operator overloading are discussed.

- Advantages:
  - Reuse existing code semantics
  - No verbosity required for simple operations
- Disadvantages:
  - Lack of context on operations

C++ gives freedom to overload operators, but that does not mean every operator should be overloaded. Only overlaod if they add meannings to codes.

## Operator Overload Design

<table><tbody>
<tr>
<th style="width: 426px;">Type</th>
<th style="width: 274px;">Operator(s)</th>
<th>Member / friend</th>
</tr>
<tr>
<td style="width: 426px;">I/O</td>
<td style="width: 274px;">&lt;&lt;, &gt;&gt;</td>
<td>friend</td>
</tr>
<tr>
<td style="width: 426px;">Arithmetic</td>
<td style="width: 274px;">+, -, *, /</td>
<td>friend</td>
</tr>
<tr>
<td style="width: 426px;">Relational, Equality</td>
<td style="width: 274px;">&gt;, &lt;, &gt;=, &lt;=, ==, !=</td>
<td>friend</td>
</tr>
<tr>
<td style="width: 426px;">Assignment</td>
<td style="width: 274px;">=</td>
<td>member (non-const)</td>
</tr>
<tr>
<td>Compound assignment</td>
<td>+=, -=, *=, /=</td>
<td>member (non-const)</td>
</tr>
<tr>
<td style="width: 426px;">Subscript</td>
<td style="width: 274px;">[]</td>
<td>member (both)</td>
</tr>
<tr>
<td>Increment/Decrement</td>
<td>++, --</td>
<td>member (non-const)</td>
</tr>
<tr>
<td style="width: 426px;">Arrow, Deference</td>
<td style="width: 274px;">-&gt;, *</td>
<td>member (both)</td>
</tr>
<tr>
<td>Call</td>
<td>()</td>
<td>member</td>
</tr>
</tbody></table>

- Use members when the operation is called in the context of a particular instance
- Use friends when the operation is called without any particular instance, even if they don't require access to private details

### I/O `<<` and `>>`

```cpp
#include <iostream>
#include <ostream>
#include <istream>
class Point {
  public:
    Point(int x, int y) : x_{x}, y_{y} {};
    friend std::ostream& operator<<(std::ostream& os, const Point& type);

  private:
    int x_;
    int y_;
};

std::ostream& operator<<(std::ostream& os, const Point& p) {
  os << "(" << p.x_ << "," << p.y_ << ")";
  return os;
}

int main() {
  Point p{1,2};
  std::cout << p << p << p << '\n';
  operator<<(operator<<(operator<<(std::cout, p), p), p);
}
```

The stream reference `os` and `is` should be the return value. Before the lecutre, I thought that this adds overhead to codes, but later I found that this is actually useful.

`std::cout << p` is actually a syntax sugar of `operator<<(std::cout, p)`, which makes code more readable. More importantly, returning the stream reference allows you to write chaining code. `std::cout << p << p << p;` is equivalent to `operator<<(operator<<(operator<<(std::cout, p), p), p);`. The returned stream reference is passed in the second call of output function, which is very handy.

### Compound assignment `+=` `-=` `*=` `/=` `*=`

```cpp
class Point {
  public:
    Point& operator+=(const Point& p);
    Point& operator-=(const Point& p);
    Point& operator*=(const Point& p);
    Point& operator/=(const Point& p);
    Point& operator*=(const int& i);

  private:
    int x_;
    int y_;
};

Point& Point::operator+=(const Point& p) {
  this->x_ += p.x_;
  this->y_ += p.y_;
  return *this;
}
Point& Point::operator-=(const Point& p) { /* Should we do this one? */ }
Point& Point::operator*=(const Point& p) { /* Should we do this one? */ }
Point& Point::operator/=(const Point& p) { /* Should we do this one? */ }
Point& Point::operator*=(const int& p) { /* Should we do this one? */ }
```

Before overloading operators, think about the context. Overloading addition operator is meaningful, but what does multiply or divide a point mean?

### Relational & Equality `==` `!=` `<` `>` `<=` `>=`

```cpp
// Point.h:
class Point {
  public:
    // hidden friend - preferred
    friend bool operator==(const Point& p1, const Point& p2) {
      return p1.x_ == p2.x_ && p1.y_ == p2.y_;
      // return std::tie(p1.x_, p1.y_) == std::tie(p2.x_, p2.y_);
    }
    friend bool operator!=(const Point& p1, const Point& p2) {
      return !(p1 == p2);
    }
    friend bool operator<(const Point& p1, const Point& p2) {
      // Do we want this? Alternatives?
    }
    friend bool operator<=(const Point& p1, const Point& p2);
    friend bool operator>(const Point& p1, const Point& p2);
    friend bool operator>=(const Point& p1, const Point& p2);

  private:
    int x_;
    int y_;
};
```

In most cases, `==` operator just checks if the data in the instances are equal, `!=` returns the opposite value of equal operator.

### Assignment `=`

```cpp
#include <iostream>
class Point {
  public:
    Point() = default;
    Point& operator=(const Point& p);
    Point& operator=(std::ostream &is);

  private:
    int x_;
    int y_;
};

Point& Point::operator=(const Point& p) {
  this->x_ = p.x_;
  this->y_ = p.y_;
  return *this;
}
Point& Point::operator=(std::ostream &is) {
    is << 3;
    return *this;
}

int main() {
    Point a;
    a = a;
    a = std::cout;
}
```

The parameter passed in does not always be a Point reference. It can be anything, even `std::ostream`.

### Subscript `[]`

```cpp
class Point {
  public:
    int& operator[](int i);       // setting via []
    int  operator[](int i) const; // getting via []

  private:
    int x_;
    int y_;
};

// Point.cpp:
#include <cassert>
int& Point::operator[](int i) {
  assert(i == 0 || i == 1);
  if (i == 0) return this->x_;
  else return this->y_;
};
int Point::operator[](int i) const {
  assert(i == 0 || i == 1);
  if (i == 0) return this->x_;
  else return this->y_;
};
```

Subscript usually does not check for validation. assert is good because it can be stripped out of optimisation builds. 

There are const and non-const types. const getter can be used if the point class is defined const like `const Point p;`, but non-const setter cannot be used in this situation.

### Increment/Decrement `++` `--`

```cpp
#include <iostream>
class RoadPosition {
  public:
    RoadPosition(int km) : km_from_sydney_(km) {}
    RoadPosition& operator++();      // prefix
    // This is *always* an int, no
    // matter your type.
    RoadPosition operator++(int);   // postfix
    void tick();
    int km() { return km_from_sydney_; }

  private:
    void tick_();
    int km_from_sydney_;
};

RoadPosition& RoadPosition::operator++() {
  this->tick_();
  return *this;
}
RoadPosition RoadPosition::operator++(int) {
  RoadPosition rp = *this;
  this->tick_();
  return rp;
}
void RoadPosition::tick_() {
  ++(this->km_from_sydney_);
} 

int main() {
  RoadPosition rp{5};
  std::cout << rp.km() << '\n';
  int val1 = (rp++).km();
  int val2 = (++rp).km();
  std::cout << val1 << '\n';
  std::cout << val2 << '\n';
}
```

To distinguish from frefix `++x` and postfix `x++`, we add an int at the function header of postfix, which is only to be used to tell compiler that it is postfix and should not be named and used.

### Arrow & Dereferencing `->` `*`

```cpp
#include <iostream>
class StringPtr {
  public:
    StringPtr(std::string *p) : ptr{p} { }
    ~StringPtr() { delete ptr; }
    std::string* operator->() { return ptr; }
    std::string& operator*() { return *ptr; }
  private:
    std::string *ptr;
};

int main() {
  std::string *ps = new std::string{"smart pointer"};
  StringPtr p{ps};
  std::cout << *p << std::endl;
  std::cout << p->size() << std::endl;
}
```

- Classes exhibit pointer-like behaviour when -> is overloaded
- For -> to work it must return a pointer to a class type or an object of a class type that defines its own -> operator
- Interesting example: std::optional
- `*` is the dereferencing operator in C, used to retrieve values from a pointer. In C++, it is used to retrieve the data encapsulated under the class.

### Type Casting `static_cast`

```cpp
// Point.h:
#include <vector>
class Point {
  public:
    Point(int x, int y) : x_(x), y_(y) {}
    operator std::vector<int>() {
      std::vector<int> vec;
      vec.push_back(x_);
      vec.push_back(y_);
      return vec;
    }

  private:
    int x_;
    int y_;
};

// Point.cpp:
#include <iostream>
#include <vector>
int main() {
  Point p{1,2};
  std::vector<int> vec = static_cast<std::vector<int>>(p);
  std::cout << vec[0] << '\n';
  std::cout << vec[1] << '\n';
}
```

To allow `static_cast` on customised class, just overload that type.

Full list of operator overloading: https://en.cppreference.com/w/cpp/language/operators