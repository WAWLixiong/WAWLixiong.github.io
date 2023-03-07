---
title: c
description: ""
date: 2023-02-26
tags:
  - 202302
  - c
categories:
  - c
menu: main
---

## 类型声明

```c
char** args;
char **ap, *args[max];
ap = args;
```

`char *args[max];` args是个数组, 数组有max个元素, 元素类型是指针类型, 指向char类型元素

<!--more-->

## c语言关键字多重含义

Symbol|Meaning
--|--
static | 1. Inside a function, retains its value between calls </br> 2. At the function level, visible only in this file
extern | 1. Applied to a function definition, has global scope (and is redundant) </br> 2. Applied to a variable, defined elsewhere
void | 1. As the return type of a function, doesn't return a value </br> 2. In a pointer declaration, the type of a generic pointer </br> 3. In a parameter list, takes no parameters
\* | 1. The multiplication operator </br> 2. Applied to a pointer, indirection </br> 3. In a declaration, a pointer
& | 1. Bitwise AND operator </br> 2. Address-of operator
= </br> == | 1.Assignment operator </br> 2. Comparison operator
<= </br> <<= | 1. Less-than-or-equal-to operator </br> 2. Compound shift-left assignment operator
< </br> < | 1. Less-than operator </br> 2. Left delimiter in #include directive
() | 1. Enclose formal parameters in a function definition </br> 2. Make a function call </br> 3. Provide expression precedence </br> 4. Convert (cast) a value to a different type </br> 5. Define a macro with arguments </br> 6.Make a macro call with arguments </br> 7.Enclose the operand of the sizeof operator when it is a typename

## 优先级

Precedence problem | Expression | What People Expect | What They Actually Get
--|--|--|--
.的优先级高于*</br>->操作符用于消除这个问题|\*p.f|p所指向对象的字段f (*p.f)|对p取f偏移, 作为指针, 然后进行解除引用操作 *(p.f)
[]高于\*|int \*ap[]| ap是个指向int数组的指针 int(*ap)[]|ap是个元素为int指针的数组 int *(ap[])
函数()高于\*|int \*fp()|fp是个函数指针, 所指函数返回int 即:int(\*fp)()|fp是个函数, 返回 int\* 即:int \*(fp())
==和!=高于位操作运算符|(val & mask != 0)|(val & mask) != 0| val & (mask != 0)
==和!=高于赋值符|c=getchar() != EOF |(c=getchar()) != EOF|c = (getchar() != EOF)
算数运算符高于位运算符|msb << 4 + lsb|(msb << )4 + lsb|msb << (4 + lsb)
逗号运算符在所有运算符中优先级最低|i=1, 2|i=(1, 2)|(i=1), 2 其中2被丢弃

## 指针的思考

1. 指针通常不知道有多长，所以函数一般还需要传递一个长度参数
2. 字符数组和字符串的主要不同是有没有\0的区别
3. 想要创建什么类型的数组，那就是创建什么类型的指针
4. 指向指针的指针: 即指向指针数组的指针

## 优先级思考

1. *p++ = 1 先赋值,后++

## 求值顺序

1. c语言只有4个运算符(&&、||、?: 和 ,)规定的运算顺序
2. 注意: f(x, y) 中 x, y的求值顺序是未知的

## 内存布局

1. `int i, a[10]` 在内存由高位向地位分配时, i在高位, a[0]在最低位, 内存连续分配的, a[10]与i的地址相同, 具体内存分配由编译器实现不同而不同
