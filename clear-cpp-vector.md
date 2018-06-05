# Clearing a C++ vector (and memory it has allocated)

I was recently having a hard time tracking down a memory leak in one of my applications. The documentation for [clear](http://www.cplusplus.com/reference/vector/vector/clear/) is not very _clear_ about the fact that after calling clear, the compiler is not guaranteed to cleanup the space allocated. 

There are two ways to handle cleaning up the memory allocated by a vector in this case:

```cpp
// Shrink method
my_vec.clear();
my_vec.shrink_to_fit(); // Capacity is now 0, memory cleared

// Swap method
vector< int >().swap( my_vec ); // Capacity is now 0
```
