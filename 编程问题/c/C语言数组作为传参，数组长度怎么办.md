# C语言数组作为传参，数组长度怎么办

```c
{
    char chars[] = "abcdefg";
    
    printf("%d\n", sizeof(chars) / sizeof(char)); // 8
    printf("%d\n", strlen(chars)); // 8
}
```

变量`chars`可以作为数组头元素的地址指针，在传入函数作为参数后，就确确实实是被当作指针了，如

```c
Status StrAssign1(HString *S, char * chars) {
    S->ch = chars;
    S->length = strlen(chars);
    
    printf("sizeof(chars) / sizeof(chars[0]) = %d / %d = %d", sizeof(chars), sizeof(chars[0]), sizeof(chars) / sizeof(chars[0]));
    // sizeof(chars) / sizeof(chars[0]) = 4 / 1 = 4
    
    return OK;
}
```

计算出数组的长度为4，那么`chars`则被当作`char *`类型的地址指针，`sizeof(chars)`得到的是地址指针的长度，而不是数组的长度。

如下方法解决

```c
// 传入时指定长度
Status StrAssign(HString *S, char * chars, int len) {
    S->ch = chars;
    S->length = len;

    return OK;
}

void main() {
    char chars[] = "abcdefg";
    StrAssign(&S, chars, sizeof(chars) / sizeof(char));
}
```

上述中的`sizeof(chars)`得到的是正确的数组长度,8