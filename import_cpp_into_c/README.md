# Import Cpp into C
如何在C語言當中去調用C++的函數
以下為簡單的C++ source code以及header組成

### func.h
```cpp
#ifndef FUNC_H
#define FUNC_H
#include <iostream>

int add(int a, int b);

#endif /* FUNC_H */
```

### func.cpp
```cpp
#include "func.h"

int add(int a, int b)
{
    return a + b;
}

```

