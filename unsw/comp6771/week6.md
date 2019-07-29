# UNSW COMP6771 (Advanced C++) Week 6 Function Templates, Class Templates

## Function Templates

```c++
template <typename T>
T min(T a, T b) {
  return a < b ? a : b;
}
 
int main() {
  std::cout << min(1, 2) << "\n"; // calls int min(int, int)
  std::cout << min(1.0, 2.0) << "\n"; // calls double min(double, double)
}
```

After compilation, each different types of the function signatures used will generate one piece of codes. It runs faster.

```c++
#include <iostream>
 
template <typename T, int size>
T findmin(const std::array<T, size> a) {
  T min = a[0];
  for (int i = 1; i < size; ++i) {
    if (a[i] < min) min = a[i];
  }
  return min;
}
 
int main() {
  std::array<int, 3> x{ 3, 1, 2 };
  std::array<double, 4> y{ 3.3, 1.1, 2.2, 4.4 };
  std::cout << "min of x = " << findmin(x) << "\n";
  std::cout << "min of x = " << findmin(y) << "\n";
}
```

- Type parameter: Unknown type with no value
- Nontype parameter: Known type with unknown value

Code explosion is an issue we need to take care of. Generating many different pieaces of codes will increase the binary size.

## Class Templates

```c++
// stack.h
#ifndef STACK_H
#define STACK_H
 
#include <iostream>
#include <vector>
 
template <typename T>
class Stack {
 public:
  friend std::ostream& operator<<(std::ostream& os, const Stack& s) {
    for (const auto& i : s.stack_) os << i << " ";
    return os;
  }
  void push(T&);
  void pop();
  T& top();
  const T& top() const;
 private:
  vector<T> stack_;
};
	
template <typename T>
void Stack<T>::push(const T &item) {
  stack_.push_back(item);
}
 
template <typename T>
void Stack<T>::pop() {
  stack_.pop_back();
}
 
template <typename T>
T& Stack<T>::top() {
  return stack_.back();
}
 
template <typename T>
const T& Stack<T>::top() const {
  return stack_.back();
}
 
template <typename T>
bool Stack<T>::empty() const {
  return stack_.empty();
}

#endif // STACK_H
```