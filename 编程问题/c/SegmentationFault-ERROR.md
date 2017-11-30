# 运行报错Segmentation fault

> 错误信息

```c
Thread 1 received signal SIGSEGV, Segmentation fault.
0x00403fad in HString_Concat (T=0x28fe88, S1=..., S2=...) at D:\workspace\datastructure\lstring.c:43
43	        *(T->ch + i) = S1.ch[i];
```

原因在于访问了没有权限的地址，报错代码如下：

```c
Status HString_Concat(HString *T, HString S1, HString S2) {

    for (int i = 0; i < S1.length; ++i) {
        *(T->ch + i) = S1.ch[i]; // 这里报错了
    }

    for (int j = 0; j < S2.length; ++j) {
        *(T->ch + S1.length + j) = S2.ch[j];
    }

    T->length = S1.length + S2.length;

    return OK;
}
```

因为`T->ch`没有`malloc`一个`char*`类型的地址指针，修正后代码如下：

```c
Status HString_Concat(HString *T, HString S1, HString S2) {
    if (!(T->ch = (char*)malloc(sizeof(char)))) {
        return ERROR;
    }

    for (int i = 0; i < S1.length; ++i) {
        *(T->ch + i) = S1.ch[i];
    }

    for (int j = 0; j < S2.length; ++j) {
        *(T->ch + S1.length + j) = S2.ch[j];
    }

    T->length = S1.length + S2.length;

    return OK;
}
```