In Matrix_API.h

```c
#pragma once

#ifdef __cplusplus
// always add this guard for C++ include
extern "C" {
#endif

typedef void* Paddle_C_MatrixPtr;

typedef struct tagPaddle_C_Matrix_API {
  void (*zero) (Paddle_C_MatrixPtr* self);
  void (*assign) (Paddle_C_MatrixPtr* self, float value);
} Paddle_C_Matrix_API;

extern Paddle_C_Matrix_API paddle_c_getMatrixAPI();


#ifdef __cplusplus
}
#endif
```
In Matrix_API.cpp

```cpp
#include "Matrix_API.h"

extern "C" {
static void zero_impl...;

Paddle_C_Matrix_API paddle_c_getMatrixAPI() {
  Paddle_C_Matrix_API api;
  api.zero = zero_impl;
  api.assign = assign_impl;
  return api;
}
}

```
