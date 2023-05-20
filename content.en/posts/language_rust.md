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

## 强制类型转换

### 强制转换

```rust
trait Trait {}

fn foo<X: Trait>(t: X) {}

impl<'a> Trait for &'a i32 {}

fn main() {
    let t: &mut i32 = &mut 0;
    foo(t);
}
```

```rust
error[E0277]: the trait bound `&mut i32: Trait` is not satisfied
 --> src/main.rs:9:9
  |
3 | fn foo<X: Trait>(t: X) {}
  |           ----- required by this bound in `foo`
...
9 |     foo(t);
  |         ^ the trait `Trait` is not implemented for `&mut i32`
  |
  = help: the following implementations were found:
            <&'a i32 as Trait>
  = note: `Trait` is implemented for `&i32`, but not for `&mut i32`
```

### 点运算的执行逻辑

1. value具有T类型, value.func() -> T::func(value),  
2. auto-reference, auto-mut-reference
3. Deref, DerefMut
4. ?Size

## 集合容器

1. 切片是视图，迭代器是数据访问操作
2. Vec 有额外的 capacity，可以增长；而 Box<[T]> 一旦生成就固定下来，没有 capacity，也无法增长; Box<[T]> 对数据具有所有权, 只能在堆上, &[T]没有所有权，可以在栈上或堆上; 注意 Box<[T]> 和 Box<[T; n]> 并不相同

## trubofish

```rust
// 它也可以用来在函数和方法中绑定泛型参数
let v = Vec::<bool>::new();
println!("{:?}", v);
```

## Eq, PartialEq

![eq_partial_eq](/imgs/eq_partial_eq.png)

## 结构体是值类型

```rust
use std::thread;

#[derive(Debug)]
struct Foo {
    data: i32,
}

fn main() {
    let foo = Foo { data: 42 };
    let handle = thread::spawn(move || {
        println!("Data from another thread: {}", foo.data);
    });
    handle.join().unwrap();
    println!("{:?}", foo); // 这里仍然可以使用
}
```

## 闭包

![closure](/imgs/closure.png)

The compiler prefers to capture a closed-over variable by immutable borrow, followed by unique immutable borrow (see below), by mutable borrow, and finally by move. It will pick the first choice of these that is compatible with how the captured variable is used inside the closure body.

unique immutable: The compiler prefers to capture a closed-over variable by immutable borrow, followed by unique immutable borrow (see below), by mutable borrow, and finally by move. It will pick the first choice of these that is compatible with how the captured variable is used inside the closure body
<https://doc.rust-lang.org/reference/types/closure.html#unique-immutable-borrows-in-captures>

```rust
let mut b = false;
let x = &mut b;
{
    let mut c = || { *x = true; };
    // The following line is an error:
    // let y = &x;
    c();
}
let z = &x;
```

If the move keyword is used, then all captures are by move or, for Copy types, by copy, regardless of whether a borrow would work

```rust
fn t1(){
    let mut name = String::from("hello");

    // 1.不可变引用，&name被存储在闭包c1里
    let c1 = || &name;
    // 可使用所有者变量name，且可多次调用闭包
    println!("{}, {:?}, {:?}", name, c1(), c1());

    // 2.可变引用，&mut name被存储在闭包c2里，调用c2的时候要修改这个字段，
    // 因此c2也要设置为mut c2
    let mut c2 = || {
        name.push_str(" world ");
    };
    // 可多次调用c2闭包
    // 但不能调用c2之前再使用name或引用name，因为&mut name已经存入c2里了
    // println!("{}", name);  // 取消注释将报错
    // println!("{}", &name); // 取消注释将报错
    c2();
    c2();

    // // 3.Move/Copy，将name移入到闭包c3中
    // let c3 = || {
    //     let x = name;
    //     // let y = name;  // 取消注释见报错，use of moved value
    // };
    // // println!("{}", name);  //取消注释将报错
}
```

## Deref

1. *解引用
2. 函数参数

Deref coercion is a convenience Rust performs on arguments to functions and methods, and works only on types that implement the Deref trait. It happens automatically when we pass a reference to a particular type’s value as an argument to a function or method that doesn’t match the parameter type in the function or method definition
<https://doc.rust-lang.org/book/ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods>

```rust
fn fun(a: impl Asref<str>){
  todo!()
}
fun(&str)
```

1. From &T to &U when T: Deref<Target=U>
2. From &mut T to &mut U when T: DerefMut<Target=U>
3. From &mut T to &U when T: Deref<Target=U>
4. todo: 不可能从 `不可变`转为`可变`

## Trait Object

![trait_object](/imgs/trait_object.png)

指向同一个数据的 trait object 其 ptr 地址相同
指向同一种类型的同一个 trait 的 vtable 地址相同

```rust
use std::fmt::{Debug, Display};
use std::mem::transmute;

fn main() {
    let s1 = String::from("hello world!");
    let s2 = String::from("goodbye world!");
    // Display / Debug trait object for s
    let w1: &dyn Display = &s1;
    let w2: &dyn Debug = &s1;

    // Display / Debug trait object for s1
    let w3: &dyn Display = &s2;
    let w4: &dyn Debug = &s2;

    // 强行把 triat object 转换成两个地址 (usize, usize)
    // 这是不安全的，所以是 unsafe
    let (addr1, vtable1): (usize, usize) = unsafe { transmute(w1) };
    let (addr2, vtable2): (usize, usize) = unsafe { transmute(w2) };
    let (addr3, vtable3): (usize, usize) = unsafe { transmute(w3) };
    let (addr4, vtable4): (usize, usize) = unsafe { transmute(w4) };

    // s 和 s1 在栈上的地址，以及 main 在 TEXT 段的地址
    println!(
        "s1: {:p}, s2: {:p}, main(): {:p}",
        &s1, &s2, main as *const ()
    );
    // trait object(s / Display) 的 ptr 地址和 vtable 地址
    println!("addr1: 0x{:x}, vtable1: 0x{:x}", addr1, vtable1);
    // trait object(s / Debug) 的 ptr 地址和 vtable 地址
    println!("addr2: 0x{:x}, vtable2: 0x{:x}", addr2, vtable2);

    // trait object(s1 / Display) 的 ptr 地址和 vtable 地址
    println!("addr3: 0x{:x}, vtable3: 0x{:x}", addr3, vtable3);

    // trait object(s1 / Display) 的 ptr 地址和 vtable 地址
    println!("addr4: 0x{:x}, vtable4: 0x{:x}", addr4, vtable4);

    // 指向同一个数据的 trait object 其 ptr 地址相同
    assert_eq!(addr1, addr2);
    assert_eq!(addr3, addr4);

    // 指向同一种类型的同一个 trait 的 vtable 地址相同
    // 这里都是 String + Display
    assert_eq!(vtable1, vtable3);
    // 这里都是 String + Debug
    assert_eq!(vtable2, vtable4);
}
```

## 生命周期

lieftime variance
<https://lifetime-variance.sunshowers.io/></br>
<https://doc.rust-lang.org/nomicon/subtyping.html></br>
<https://doc.rust-lang.org/reference/subtyping.html#variance></br>
<https://github.com/fucking-translation/blog/tree/main/src/lang/rust></br>
<https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#2-if-t-static-then-t-must-be-valid-for-the-entire-program></br>

### 生命周期及相关误解

1. **T** only contains owned types
   - T包含 &T, &mut T
   - &T 和 &mut T没有交集
2. if **T: &'static** then **T** must be valid for the entire program
3. **&'a T** and **T: 'a** are the same thing
   - &'a T 暗示至少T: 'a, T: 'a包含所有的&'a T，但是反过来不对
   - &'static Ref<'a, T>, compile error, T: 'a的，不可能再有'static的引用
4. my code isn't generic and doesn't have lifetimes
5. if it compiles then my lifetime annotations are correct
6. boxed trait objects don't have lifetimes
7. compiler error messages will tell me how to fix my program
8. lifetime can grow and shrink at run-time
   - 在编译时确定，采用最短的lifetime
9. downgrading mut refs to shared refs is safe
   - anti-pattern
   - re-borrowed shared ref 不能于其他 shared ref 共存
10. closures follow the same lifetime elision rules as functions

## tokio

1. task非常轻量, 64bytes
2. task必须是 'static bound 的(<https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#2-if-t-static-then-t-must-be-valid-for-the-entire-program>)
   1. T: 'static is some T that can be safely held indefinitely long, including up until the end of the program. T: 'static includes all &'static T however it also includes all owned types, like String, Vec, etc
3. task必须实现Send，因为需要由scheduler在多个线程中执行
4. ![tokio_conn](/imgs/tokio_conn.png)

```rust
use bytes::Bytes;
use mini_redis::client;
use tokio::sync::{mpsc, oneshot};

type Responser<T> = oneshot::Sender<mini_redis::Result<T>>;

#[derive(Debug)]
enum Command {
    Get {
        key: String,
        resp: Responser<Option<Bytes>>,
    },

    Set {
        key: String,
        val: Bytes,
        resp: Responser<()>,
    },
}


#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel(32);

    let tx2 = tx.clone();

    let manager = tokio::spawn(async move {
        let mut client = client::connect("localhost:49153").await.unwrap();

        while let Some(cmd) = rx.recv().await {
            match cmd {
                Command::Get { key, resp } => {
                    let res = client.get(&key).await;
                    // ignore error
                    let _ = resp.send(res);
                }
                Command::Set { key, val, resp } => {
                    let res = client.set(&key, val).await;
                    // ignore error
                    let _ = resp.send(res);
                }
            }
        }
    });

    let t1 = tokio::spawn(async move {
        let (resp_tx, resp_rx) = oneshot::channel();
        let cmd = Command::Get { key: "k1".to_string(), resp: resp_tx };
        if tx.send(cmd).await.is_err() {
            eprintln!("connection task shutdown");
            return;
        }
        let res = resp_rx.await;
        println!("Got (Get) = {:?}", res);
    });

    let t2 = tokio::spawn(async move {
        let (resp_tx, resp_rx) = oneshot::channel();
        let cmd = Command::Set { key: "k2".to_string(), val: "hello".into(), resp: resp_tx };
        if tx2.send(cmd).await.is_err() {
            eprintln!("connection task shutdown");
            return;
        }
        let res = resp_rx.await;
        println!("Got (Set) = {:?}", res);
    });

    t1.await.unwrap();
    t2.await.unwrap();
    manager.await.unwrap();
}
```

## 智能指针

1. 智能指针一定是胖指针，胖指针不一定是智能指针, String是指针指针，&str是普通胖指针
2. 智能指针都对数据有所有权，普通胖指针没有所有权
3. 实现了 Deref，DerefMut，Drop的都是智能指针, e.g. Box\<T\>, Vec\<T\>, Rc\<T\>, Arc\<T\>, PathBuf, Cow<'a, B>, MutexGuard\<T\>, RwLockReadGuard 和 RwLockWriteGuard
4. Box\<T\>目的：在堆上分配内存，类似与C/C++的malloc功能
5. Box::new() 是一个函数，所以传入它的数据会出现在栈上，再移动到堆上。如果是一个非常大的结构，就有可能stack overflow, `cargo run —release` 会inline代码所以不会出现问题
6. Cow<'a, B> 写时克隆, 包裹一个只读借用，但如果调用者需要所有权或者需要修改内容，那么它会 clone 借用的数据
   [Cow](/imgs/Cow.png)
   为何 Borrow 要定义成一个泛型 trait

   ```rust
   let s = "hello world!".to_owned();
   // 这里必须声明类型，因为 String 有多个 Borrow 实现 
   // 借用为 &String
   let r1: &String = s.borrow();
   // 借用为 &str
   let r2: &str = s.borrow();
   ```

   ```rust
    use serde::Deserialize;
    use std::borrow::Cow;

    #[derive(Debug, Deserialize)]
    struct User<'input> {
        #[serde(borrow)]
        name: Cow<'input, str>,
        age: u8,
    }

    fn main() {
        let input = r#"{ "name": "Tyr", "age": 18 }"#;
        let user: User = serde_json::from_str(input).unwrap();

        match user.name {
            Cow::Borrowed(x) => println!("borrowed {}", x),
            Cow::Owned(x) => println!("owned {}", x),
        }
    }
   ```

## 容器类型 Vec\<T\>, [T;n], &[T], Box<[T]>

1. 切片 只读&[T], 可写&mut[T]，分配在堆上Box<[T]>
2. 切片和数据的关系
    ![slice](/imgs/slice.png)
3. &[T] 和 &Vec\<T\>的关系
    ![slice_dif](/imgs/slice_dif.png)
4. [1, 2, 3] 虽然没有实现 AsRef 但是内建了其实现，所以可以用在需要AsRef Bound的地方, 其解引用就是&[T]
5. &str和普通的切片没有区别
    ![str_slice](/imgs/str_slice.png)
6. 字符数组和字符串的区别
    1. &[char] 不能与 &str直接对比, String和&str可以直接对比
    ![char_array](/imgs/char_array.png)
7. &[T], &mut[T]不具备所有权，Box<[T]> 具备所有权
    ![box_T](/imgs/box_T.png)
8. Box<[T]>与Vec\<T\>相似, Box<[T]> 没有capacity, 目前只能通过Vec\<T\>::into_boxed_slice()转换
9. Box<[T]> 有一个很好的特性是，不像 Box<[T;n]> 那样在编译时就要确定大小，它可以在运行期生成，以后大小不会再改变

![slice_transfer](/imgs/slice_transfer.png)

## 分发手段

1. 泛型静态分发
2. trait object动态分发
3. enum 的不同状态分发
