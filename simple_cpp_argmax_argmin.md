# Simple C++ argmin/argmax for std::vector

```c++
template <typename T, typename A>
int arg_max(std::vector<T, A> const& vec) {
  return static_cast<int>(std::distance(vec.begin(), max_element(vec.begin(), vec.end())));
}

template <typename T, typename A>
int arg_min(std::vector<T, A> const& vec) {
  return static_cast<int>(std::distance(vec.begin(), min_element(vec.begin(), vec.end())));
}
```

Used as follows:

```c++
#include <vector>
#include <iostream>

void main() {
  std::vector<int> my_vec = { 1, 43, 2, 4, 2, 12 };
  int max_idx = arg_max(my_vec);
  std::cout << my_vec[max_idx] << std::endl;
}
```
