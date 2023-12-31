# Import Cpp Function into C
如何在C語言當中去調用C++的函數，以下為簡單的C++ source code以及header組成

## Cpp header and source code

### func.h
```cpp
#ifndef FUNC_H
#define FUNC_H
#include <iostream>

int add(int a, int b);

#endif /* FUNC_H */
```

開頭為預處理器指令，防止相同的標頭檔在同一個編譯單元中重複引入

### func.cpp
```cpp
#include "func.h"

int add(int a, int b)
{
    return a + b;
}

```

此時， `func.cpp` 包含的 `add` 函式為C++接口，C中不能直接調用，必須經過一層封裝後才能使用

若是想將C++的庫封裝成C編譯器能夠識別的形式，需要透過增加一個中間層來實現

## C API includes C++ function 

### func_wrapper.h

```cpp
#ifndef FUNC_WRAPPER_H
#define FUNC_WRAPPER_H

#ifdef __cplusplus
extern "C" {
#endif

int call_cpp_add(int a, int b);

#ifdef __cplusplus
}
#endif

#endif /* FUNC_WRAPPER_H */
```

一開始指示詞當中的 `__cplusplus` 是一個C++規範規定的預定義宏，僅當編譯器被識別為C++編譯器時才觸發

`extern "C"` 的作用則是把範圍內的程式視為C程式碼，不對函式名稱進行C++的命名修飾(name mangling)

`func_wrapper.h` 提供了C相容的介面，使得C程式碼能夠與被封裝的C++功能進行交互作用

### func_wrapper.cpp

```cpp
#include "func_wrapper.h"
#include "func.h"

#ifdef __cplusplus
extern "C" {
#endif

int call_cpp_add(int a, int b)
{
    return add(a, b);
}

#ifdef __cplusplus
}
#endif
```

`func_wrapper.cpp` 就是所謂的中間層或是C++與C之間的interface，透過C API把C++的函式封裝成C可以識別的函式

### testing.c

```cpp
#include <stdio.h>
#include <stdlib.h>
#include "func_wrapper.h"

int main() {

    printf("hello, world\n");

    int a = 10;
    int b = 24;
    int c = call_cpp_add(a, b);
    printf("c = %d\n", c);

    return 0;
}
```

## CMakeLists.txt configuration

### file structure

```bash
project
|
|-- src
|   |-- func.cpp
|   |-- func_wrapper.cpp
|
|-- include
|   |-- func.h
|   |-- func_wrapper.h
|
|-- lib
|   |-- libfunc.so
|
|-- CMakeLists.txt
|-- build
|
|-- testing
|   |-- build
|   |-- CMakeLists.txt
|   |-- testing.c
```

再來利用CMakeLists.txt來編譯程式。首先，先單獨把C++函式庫編譯成動態庫(.so)

### project/CMakeLists.txt

```cmake
cmake_minimum_required( VERSION 3.1 )
project( func )

set(install_path ${CMAKE_SOURCE_DIR}/lib)
message(STATUS "install_path: ${install_path}")

set( SRC_FILES
  ${CMAKE_CURRENT_SOURCE_DIR}/src/func.cpp
)

add_library(${PROJECT_NAME} SHARED ${SRC_FILES})

target_include_directories(${PROJECT_NAME}
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

install (TARGETS ${PROJECT_NAME}
         DESTINATION ${install_path})

```

在編完之後會在 `project/lib` 當中生成 `libfunc.so` 就是C++的動態庫

再來編譯 `testing.c` 同時引入C++動態庫 `libfunc.so`

### project/testing/CMakeLists.txt

```cmake
cmake_minimum_required( VERSION 3.1 )
project( testing )

set(SRC_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/../src/func_wrapper.cpp
)

# create C API library
add_library(func_wrapper SHARED ${SRC_FILES})

# find existed C++ library by given path  
find_library(libfunc func ${CMAKE_CURRENT_SOURCE_DIR}/../lib)

# import C++ library
target_link_libraries(func_wrapper 
    PUBLIC  
        ${libfunc})

target_include_directories(func_wrapper
  PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/../include
)

add_executable(${PROJECT_NAME} testing.c)

target_link_libraries(${PROJECT_NAME}
  PUBLIC
    func_wrapper
)
```

## build code and run

1. build C++ shared library
```bash
cd project/build
cmake ..
make -j8 install
```

2. build testing
```bash
cd project/testing/build
cmake ..
make -j8
```
3. run code 
```bash
cd project/testing/build
./testing
```

4. result
```bash
hello, world
c = 34
```

## reference 
[[C++与C之间相互接口和库函数调用]](https://blog.csdn.net/gatieme/article/details/52730680)

[[C中調用C++與C++調用C]](https://jasonblog.github.io/note/c++/cyu_czhong_de_han_shu_hu_xiang_diao_yong_de_chu_li.html)

[[C 語言中的 extern "C"]](https://www.readfog.com/a/1651251482765398016)

[[How to mix C and C++]](https://isocpp.org/wiki/faq/mixing-c-and-cpp#call-cpp)
