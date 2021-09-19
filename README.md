# determine_c_or_cpp

Determine programatically C from C++ as well as various versions

- In C the comma operator [forces an array-to-pointer conversion](https://t.co/jFGCC1PtPv) while in C++ the result of the comma operator is the [same value type as the RHS](https://t.co/tAcPqto738):

    ```cpp
    int f() {
     char arr[100];
     return sizeof(0, arr); // returns 8 in C and 100 in C++
    }
    ```

    I used this example in a [#CppPolls](https://twitter.com/shafikyaghmour/status/1193787548176248832)

- C uses struct, union and enum tags as a primitive form of namespace [see reference](https://stackoverflow.com/a/21793332/1708801). So the following code will give sizeof int for C and sizeof struct T for C++

    ```cpp
    #include <stdio.h> 

    extern int T; 

    int size(void) { 
        struct T { int i; int j; }; 
        return sizeof(T); 
    } 

    int main() { 
        printf( "%d\n", size() );     // sizeof(int) in C and sizeof(struct T) in C++
        
        return 0 ;
    }
    ```

- Character literals are treated different in C and C++. An *integer character constant* has type **int** in C and a *character lterals* with a single char has type **char**. So the following code will likely produce **4** for C and **1** for C++

    ```cpp
    #include<stdio.h>
    int main()
    {
       printf("%zu",sizeof('a'));    // sizeof(int) in C and sizeof(char) in C++
       return 0;
    }
    ```
    
- C90 does not have **//** style comments, we can differentiate C90 from C99, C11 and C++:

    ```cpp
    #include <stdio.h>

    int main() {
       int i = 2 //**/2
        ;
        printf( "%d\n", i ) ;    // 1 in C90
                                 // 2 in C99, C11 and C++
        return 0 ;
    }
    ```
    
- C99 *if statement* implicitly defines a block scope above and below it (h/t [this tweet](https://twitter.com/rep_stosq_void/status/1370123144514699274):

   ```cpp
    #include <stdio.h>

    struct s { long long m; };

    int main(void) {
      int x = 0;
      if (x)
        x = sizeof(struct s { char c;});
      else  
        printf("%zu\n", sizeof (struct s));   // 1 in C89
                                              // 8 in C99, assuming LP64
      return 0;
    }
   ```
    
- K&R C used the *unsigned preserving* approach to integer promotions, which means when we mix unsigned and signed integers they are promoted to *unsigned*. Where as ANSI C and C++ use *value preserving* approach, which means when mixing *signed* and *unsigned* integers the are promoted to *int* if *int* can represent all values. 

    ```cpp
    #include <stdio.h>

    int main() {
       if( (unsigned char)1 > -1 ) {    // false for K&R C
                                        // true for ANSI C and C++
           printf("yes\n" ) ;  
       } 
    }

    ```
    
- In C++ *true* and *false* are treated differently during preprocessing then the rest of the keywords and [are not replaced with 0](http://eel.is/c++draft/cpp#cond-11) and [subepxressions of type *bool* are subject to integral promotions](http://eel.is/c++draft/cpp#cond-12.sentence-8). In C *true* and *false* are not keywords but are macros with values `1` and `0` respetively if [`stdbool.h` is included](https://port70.net/~nsz/c/c11/n1570.html#7.18):

    ```cpp
    #if true
    #define ISC 0
    #else
    #define ISC 1
    #endif
    ```

- The Stackoverflow question [Can C++ code be valid in both C++03 and C++11 but do different things?](https://stackoverflow.com/q/23047198/1708801) contains several examples of code that generate different results for C++03 and C++11:
    - New kinds of string literals [...] Specifically, macros named R, u8, u8R, u, uR, U, UR, or LR will not be expanded when adjacent to a string literal but will be interpreted as part of the string literal. The following code will print **abcdef** in C++03 and **abc** in C++11:

        ```cpp
        #include <cstdio>

        #define u8 "abc"

        int main()
        {
           const char *s = u8"def";
    
           printf( "%s\n", s ) ;     // abcdef in C++03 and abc in C++11
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
          std::cout << (Y<X<1> >::c >::c>::c) << '\n';  // 0 in C++03 
                                                        // 0 in C++11
          std::cout << (Y<X< 1>>::c >::c>::c) << '\n';  // 3 in C++03
                                                        // 0 in C++11
        }
        ```
        
- We can identify if we are C++17 and greater by attempting to use trigrpahs, [which were removed](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2014/n4086.html) in [C++17](http://eel.is/c++draft/diff.cpp14.lex#1):

    ```cpp
    int iscpp17orGreater(){
      //??/
      return 1; // trigraph on previous line will be ignored in C++17
      return 0; // trigraphs two lines above will form continuation and comment out the previous line
    }
    ```
        

- How to tell C++20 from previous versions of C++. In C++20  introduced the spaceship operator <=> (🛸) and part of this change was rewritten candidates for equality and others see [\[over.match.oper\]p3.4](http://eel.is/c++draft/over.match.oper#3.4). The following code will print `0` in C++20 and `1` in previous versions of C++:

    ```cpp
    #include <iostream>

    struct A {
     operator int() {return 1;}
    };
    bool operator==(A, int){return 0;}

    What is the result of:

    int main() {
       std::cout <<(1 == A{})<<"\n"; // 0 in C++20
                                     // 1 in previous versions of C++
    }

    ```

- C++14 added the [single quote as a digit seperator](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2013/n3781.pdf) this means that prior to C++14 [a comma in single quotes would be treated as a seperator in macros but C++14 onwards would not](http://eel.is/c++draft/diff.cpp11#lex-1). Assuming we can indentify C++03, C++17 and C++20 we can use the following method to then differentiate C++11 and C++14. 

    ```cpp
    #define M(x, ...) __VA_ARGS__
    int x[2] = { M(1'2,3'4, 5) };
    // macro CPP03 is 1 is we are C++03
    // macro CPP17 is 1 if we are C++17
    // macro CPP20 is 1 is we are C++20
    int CPP14 = x[0] == 34 && !CPP17 && !CPP20;
    int CPP11 = x[0] == 5 && !CPP03 ;
    ```
