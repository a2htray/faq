I guess that i had found a method for programming with Unix

```c
#include <stdio.h>
#include <stdlib.h>
#include <malloc.h>

typedef int ElemType;

typedef struct Node {
    ElemType data;
    struct Node *next;
    int length;
} Node, *PtrLink;


int initLink(PtrLink *L) {
    (*L)->next = (Node*)malloc(sizeof(Node));

    if (!(*L)->next) {
       return 0;
    }

    (*L)->length = 0;

    return 1;
}


#define _p_init_stack initLink

typedef PtrLink Stack;

int initStack(Stack *S) {
    return _p_init_stack(S);
}

int main() {
    PtrLink L;

    initLink(&L);

    return 0;
}

```
