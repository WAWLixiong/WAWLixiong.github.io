---
title: rust
description: ""
date: 2023-02-26
tags:
  - 202302
  - rust
categories:
  - rust
menu: main
---

## struct 和 enum 类型

![rust_struct_enum](/imgs/rust_struct_enum.png)

## 分支

![rust_branch](/imgs/rust_branch.png)

## 项目组织结构

{{< columns>}}
![rust_project_layout](/imgs/rust_project_layout.png)
<--->
![rust_workspace](/imgs/rust_workspace.png)
{{< /columns >}}

## 单元测试

一般放在和被测试代码相同的文件中, 使用条件编译，只在测试环境下编译

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

集成测试一般放在 tests 目录下，和 src 平行。
和单元测试不同，集成测试只能测试 crate 下的公开接口，编译时编译成单独的可执行文件。

## 关联类型与泛型

关联类型和泛型参数都可以用来定义trait中的未指定类型。但是它们的作用和使用方式是不同的。

泛型参数是在定义trait的时候就指定的，而关联类型是在实现trait的时候才会确定具体的类型。泛型参数的类型必须在trait定义中已经确定，而关联类型的类型可以在实现trait的时候才确定。这使得关联类型更加灵活，可以用于更多的场景。

此外，泛型参数用于指定方法参数和返回值的类型，而关联类型用于指定trait内部的类型关系。这两个特性的作用和使用方式是不同的。

在一些特定的情况下，使用泛型参数可以替代关联类型。比如，如果一个trait只有一个方法，那么可以直接使用泛型参数来指定方法参数和返回值的类型。但是在复杂的trait中，使用关联类型可以使得代码更加通用和灵活。

因此，虽然泛型参数和关联类型都可以用来定义trait中的未指定类型，但它们的作用和使用方式是不同的，需要根据具体的情况来选择使用哪种特性

## clone和copy

在 Rust 中，Copy 和 Clone 是用于实现数据类型复制的 trait。它们是 Rust 语言中的两个核心 trait，都位于标准库的预定义 trait 中。

Copy trait表示该类型的值可以通过简单的内存复制来进行赋值操作，而不需要转移所有权。因此，当一个Copy类型的值被赋值给另一个变量时，它的值会被完整地复制到新的变量中，并且原来的变量仍然可以使用它的值。Copy trait通常适用于基本数据类型（如整数和布尔值）以及简单的数据结构（如元组和结构体），并且不能手动实现该trait。

Clone trait表示该类型的值可以通过克隆来进行赋值操作，并且在这个过程中，原始值的所有权被转移给新的变量。也就是说，当一个Clone类型的值被赋值给另一个变量时，它的值会被克隆一份，然后原来的变量将不再拥有该值的所有权。Clone trait需要手动实现，并且需要定义一个clone方法来执行克隆操作。

在内存方面，Copy类型的值在栈上复制，因此无论值的大小如何，复制的时间都是相同的。而对于Clone类型的值，由于它们可能包含指向堆上分配的内存的指针，因此进行克隆操作时可能需要分配新的堆内存，并将原始值的内容复制到新分配的内存中。

总之，Copy trait适用于基本数据类型和简单的数据结构，其值可以通过内存复制来传递，并且不需要手动实现该trait。而Clone trait则适用于需要转移所有权的复杂数据类型，并且需要手动实现该trait来定义如何克隆值

需要注意的是，如果一个类型实现了 Copy trait，则通常也应该实现 Clone trait。这是因为，虽然 Copy trait 可以直接通过简单的内存复制来复制一个类型，但在某些情况下（例如，如果一个类型包含指向堆内存的指针），需要执行自定义的克隆操作

## &pair.k

在 Rust 语法中，&pair.k 表示对一个名为 pair 的结构体的一个字段 k 进行引用（borrow），返回一个指向该字段的不可变引用（immutable reference）。

具体来说，&pair.k 的含义是获取 pair 结构体的字段 k 的不可变引用。这个引用可以用来读取 k 字段的值，但不能修改它。与此相对，如果要获取一个可变引用（mutable reference）则需要使用 &mut pair.k。

值得注意的是，对于结构体中的字段，Rust 中默认情况下是按值（by value）传递的。也就是说，如果在函数参数列表中传递一个结构体，那么实际上是将整个结构体拷贝一份进行传递。但是，使用 & 或 &mut 可以将结构体的字段以引用的方式传递，从而避免不必要的拷贝。

在某些情况下，还可以使用 .k 的方式来直接获取结构体的字段值，而不是使用引用。但这种方式会将整个字段的值拷贝一份，因此在需要频繁读取同一个字段的值时，使用引用通常更为高效

## ok_or 和 ok_or_else 的区别

ok_or 和 ok_or_else 都是 Rust 中 Result 类型的方法，它们用于将 Ok 变量转换为 Err 类型，但它们之间有一些差异。

ok_or 方法是一个立即求值的方法，它接受一个参数并返回一个包含该参数的 Err 类型的 Result。如果 Result 是 Ok 类型，则将其转换为一个包含该参数的 Err 类型的 Result，否则将保留原始的 Err 值

```rust
fn divide(a: i32, b: i32) -> Result<i32, &'static str> {
    if b == 0 {
        return Err("Cannot divide by zero!");
    }
    Ok(a / b)
}

let result1 = divide(5, 0).ok_or("Failed to divide by zero!");
assert_eq!(result1, Err("Failed to divide by zero!"));

let result2 = divide(10, 2).ok_or("Failed to divide by zero!");
assert_eq!(result2, Ok(5));

```

ok_or_else 方法与 ok_or 方法非常相似，但是它接受一个函数作为参数，该函数在需要时被调用来生成 Err 类型的值。如果 Result 是 Ok 类型，则该函数不会被调用，否则将使用该函数生成的值作为 Err 类型的值

```rust
fn divide(a: i32, b: i32) -> Result<i32, &'static str> {
    if b == 0 {
        return Err("Cannot divide by zero!");
    }
    Ok(a / b)
}

let result1 = divide(5, 0).ok_or_else(|| "Failed to divide by zero!");
assert_eq!(result1, Err("Failed to divide by zero!"));

let result2 = divide(10, 2).ok_or_else(|| "Failed to divide by zero!");
assert_eq!(result2, Ok(5));
```

## 实现不同trait的效果

1. `impl From<&ImageSpec> for String` String::from()
2. `impl TryFrom<&str> for ImageSpec` ImageSpec::try_from(), (&str).parse().unwrap()
3. `impl FromStr for KvPair` KvPair::from_str, (&str).parse().unwrap()

## Arc

Rust中的Arc是“原子引用计数（Atomically Reference Counted）”类型的缩写，它允许多个变量共享相同的数据，同时提供线程安全性。当多个变量需要共享相同的数据时，可以使用Arc。

以下是Arc的主要特点：

1. 原子性：Arc使用原子操作来确保线程安全，即使在多线程环境下，也可以安全地访问共享数据。
2. 共享所有权：Arc允许多个变量共享相同的数据，而不是每个变量都拥有自己的副本。这有助于减少内存使用量和复制数据的开销。
3. 线程安全：Arc可以安全地在线程之间共享，因为它实现了Sync和Send trait，分别表示它可以安全地在线程之间共享和安全地跨线程发送。
4. 引用计数：Arc跟踪指向数据的引用数量，当引用计数为0时，它会自动清除其内存。
5. 克隆：通过调用clone方法，可以创建Arc的新副本，从而增加引用计数。
6. 垃圾回收：由于Arc会自动清除其内存，因此不需要手动执行内存管理。

总之，Arc允许多个变量共享相同的数据，并提供线程安全和自动内存管理的功能
