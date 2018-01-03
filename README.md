# determine_c_or_cpp

Determine programatically C from C++ as well as various versions

- C uses struct, union and enum tags as a primitive form of namespace [see reference](https://stackoverflow.com/a/21793332/1708801). So the following code will give sizeof int for C and sizeof struct T for C++

    ```cpp
    #include <stdio.h> 

    extern int T; 

    int size(void) { 
        struct T { int i; int j; }; 
        return sizeof(T); 
    } 

    int main() { printf( "%d\n", size() ); }
    ```

- Character literals are treated different in C and C++. An *integer character constant* has type **int** in C and a *character lterals* with a single char has type **char**. So the following code will likely produce **4** for C and **1** for C++

    ```cpp
    #include<stdio.h>
    int main()
    {
       printf("%zu",sizeof('a'));
       return 0;
    }
    ```

- The Stackoverflow question [Can C++ code be valid in both C++03 and C++11 but do different things?](https://stackoverflow.com/q/23047198/1708801) contains several examples of code that generate different results for C++03 and C++11:
    - New kinds of string literals [...] Specifically, macros named R, u8, u8R, u, uR, U, UR, or LR will not be expanded when adjacent to a string literal but will be interpreted as part of the string literal. The following code will print **abcdef** in C++03 and **abc** in C++11:

        ```cpp
        #include <cstdio>

        #define u8 "abc"

        int main()
        {
           const char *s = u8"def";
    
           printf( "%s\n", s ) ;
           return 0;
        }
        ```

   - Using >> to close multiple templates is no longer ill-formed but can lead to code with different results in C++03 and C+11.
   
        ```cpp
        #include <iostream>
        template<int I> struct X {
          static int const c = 2;
        };
        template<> struct X<0> {
          typedef int c;
        };
        template<typename T> struct Y {
          static int const c = 3;
        };
        static int const c = 4;
        int main() {
          std::cout << (Y<X<1> >::c >::c>::c) << '\n';
          std::cout << (Y<X< 1>>::c >::c>::c) << '\n';
        }
        ```
