# C函数取变长参数

## 引入库

```c
#include <stdarg.h>
```

## 定义方法

```c
Status InitArray(Array *A, int dim, ...);
```

## 方法实现

```c
Status InitArray(Array *A, int dim, ...) {
    if (dim < 1 || dim > MAX_ARRAY_DIM) return ERROR;

    A->bounds = (int*)malloc(sizeof(int) * dim);
    if (! A->bounds) exit(OVERFLOW);

    int total = 1;

    va_list ap;
    va_start(ap, dim);

    for (int i = 0; i < dim; ++i) {
        A->bounds[i] = va_arg(ap, int);
        if (A->bounds[i] < 0) return ERROR;
        total *= A->bounds[i];
    }

    va_end(ap);

    A->base = (Array_ElemType*)malloc(sizeof(Array_ElemType) * total);
    if (! A->base) exit(OVERFLOW);

    A->constants = (int*)malloc(sizeof(int) * dim);
    if (! A->constants) exit(OVERFLOW);

    A->constants[dim - 1] = 1;
    for (int i = dim - 2; i >= 0; --i) {
        A->constants[i] = A->bounds[i + 1] * A->constants[i + 1];
    }

    return OK;
}
```