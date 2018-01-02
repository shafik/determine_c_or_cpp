# determine_c_or_cpp
Determine programatically C from C++ as well as various versions

- C uses struct, union and enum tags as a primitive form of namespace [see reference](https://stackoverflow.com/a/21793332/1708801). So the following code will give sizeof int for C and sizeof struct T for C++:

```cpp
#include <stdio.h> 

extern int T; 

int size(void) { 
    struct T { int i; int j; }; 
    return sizeof(T); 
} 

int main() { printf( "%d\n", size() ); }
```
