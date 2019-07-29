# UNSW COMP6771 (Advanced C++) Week 7 Templates and Custom Iterators

## Inclusion Compilation Model

Templates implementation should be in .h files. We can use another .tpp file to implement, and #include it in the end of .h file.

*Stack.h*

```c++
#include <vector>

template <typename T>
class Stack {
 public:
  Stack() {}
  void pop();
  void push(const T& i);
 private:
  std::vector<T> items_;
}

#include "stack.tpp"
```

*Stack.tpp*

```c++
template <typename T>
void Stack<T>::pop() {
  items_.pop_back();
}

template <typename T>
void Stack<T>::push(const T& i) {
  items_.push_back(i);
}
```

*main.cpp*

```c++
int main() {
  Stack<int> s;
}
```

- Each template instantiation has it's own set of static members
- Each template instantiation has one unique instantiation of the friend

## Default Members in Class Templates

```c++
#include <vector>
#include <iostream>

template <typename T, typename CONT = std::vector<T>>
class Stack {
 public:
  Stack();
  ~Stack();
  void push(T&);
  void pop();
  T& top();
  const T& top() const;
  static int numStacks_;
 private:
  CONT stack_;
};

template <typename T, typename CONT>
int Stack<T, CONT>::numStacks_ = 0;

template <typename T, typename CONT>
Stack<T, CONT>::Stack() { numStacks_++; }

template <typename T, typename CONT>
Stack<T, CONT>:: ~Stack() { numStacks_--; }

int main() {
  Stack<float> fs;
  Stack<int> is1, is2, is3;
  std::cout << Stack<float>::numStacks_ << "\n";
  std::cout << Stack<int>::numStacks_ << "\n";
}
```

## Custom Iterators

*Minimum iterator class*

```c++
class Iterator {
 public:
  using iterator_category = std::forward_iterator_tag;
  using value_type = T;
  using reference = T&;
  using pointer = T*; // Not strictly required, but nice to have.
  using difference_type = int;

  reference operator*() const;
  Iterator& operator++();
  Iterator operator++(int) {
    auto copy{*this};
    ++(*this);
    return copy;
  }
  // This one isn't strictly required, but it's nice to have.
  pointer operator->() const { return &(operator*()); }

  friend bool operator==(const Iterator& lhs, const Iterator& rhs) { ... };
  friend bool operator!=(const Iterator& lhs, const Iterator& rhs) { return !(lhs == rhs); }
};
```

*Container requirements*

```c++
class Container {
  // Make the iterator using one of these by convention.
  class iterator {...};
  using iterator = ...;

  // Need to define these.
  iterator begin();
  iterator end();

  // If you want const iterators (hint: you do), define these.
  const_iterator begin() const { return cbegin(); }
  const_iterator cbegin() const;
  const_iterator end() const { return cend(); }
  const_iterator cend() const;
};
```

## Reverse Iterators

Reverse Iterators can be created by `std::reverse_iterator`.

```cpp
class Container {
  // Make the iterator using these.
  using reverse_iterator = std::reverse_iterator<iterator>;
  using const_reverse_iterator = std::reverse_iterator<const_iterator>;

  // Need to define these.
  reverse_iterator rbegin() { return reverse_iterator{end()}; }
  reverse_iterator rend() { return reverse_iterator{begin()}; }

  // If you want const reverse iterators (hint: you do), define these.
  const_reverse_iterator rbegin() const { return crbegin(); }
  const_reverse_iterator rend() const { return crend(); }
  const_reverse_iterator crbegin() const { return const_reverse_iterator{cend()}; }
  const_reverse_iterator crend() const { return const_reverse_iterator{cbegin()}; }
};
```

## Random Access Iterators

```cpp
class Iterator {
  ...
  using reference = T&;
  using difference_type = int;

  Iterator& operator+=(difference_type rhs) { ... }
  Iterator& operator-=(difference_type rhs) { return *this += (-rhs); }
  reference operator[](difference_type index) { return *(*this + index); }

  friend Iterator operator+(const Iterator& lhs, difference_type rhs) {
    Iterator copy{*this};
    return copy += rhs;
  }
  friend Iterator operator+(difference_type lhs, const Iterator& rhs) { return rhs + lhs; }
  friend Iterator operator-(const Iterator& lhs, difference_type rhs) { return lhs + (-rhs); }
  friend difference_type operator-(const Iterator& lhs, const Iterator& rhs) { ... }

  friend bool operator<(Iterator lhs, Iterator rhs) { return rhs - lhs > 0; }
  friend bool operator>(Iterator lhs, Iterator rhs) { return rhs - lhs < 0; }
  friend bool operator<=(Iterator lhs, Iterator rhs) { !(lhs > rhs); }
  friend bool operator>=(Iterator lhs, Iterator rhs) { !(lhs < rhs); }
}
```