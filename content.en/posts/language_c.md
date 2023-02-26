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
