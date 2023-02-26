---
title: python
description: ""
date: 2023-02-26
tags:
  - 202302
  - python
categories:
  - python
menu: main
---


## contextvar

> Python 3.7 引入了 contextvars 模块，该模块提供了一种新的协程上下文变量机制，来代替之前的 threading.local()。相比于 threadlocal，contextvar 有以下区别：
> 范围：threadlocal 是在线程范围内存储变量，而 contextvar 是在协程范围内存储变量，因此 contextvar 可以在一个线程内有多个协程使用。
> 自动传播：contextvar 的值可以在协程之间自动传播。在协程中设置一个 contextvar 的值后，后续创建的协程都可以访问该值，即使它们在不同的异步任务中运行。
> 线程安全性：threadlocal 依赖于线程本地存储机制，不适用于异步编程，而 contextvar 可以安全地在异步程序中使用。
> 性能：contextvar 的性能通常优于 threadlocal，尤其在有大量协程的情况下，contextvar 的性能更卓越。
> 总之，contextvar 在协程的异步编程中更加安全、方便，具有更好的性能表现，并且可以跨越异步任务之间自动传播值，因此更适用于现代 Python 异步编程模式

```python
import asyncio
import contextvars

client_addr_var = contextvars.ContextVar('client_addr')
client_addr_var.set("niuniu")


def render_goodbye():
    # The address of the currently handled client can be accessed
    # without passing it explicitly to this function.

    client_addr = client_addr_var.get()
    return f'Good bye, client @ {client_addr}\n'.encode()


async def hello():
    client_addr = client_addr_var.get()
    return f'Hello, client @ {client_addr}\n'.encode()


async def handle_request(reader, writer):
    addr = writer.transport.get_extra_info('socket').getpeername()
    client_addr_var.set(addr)

    writer.write(await hello())

    # In any code that we call is now possible to get
    # client's address by calling 'client_addr_var.get()'.

    while True:
        line = await reader.readline()
        print(line)
        if not line.strip():
            break
        writer.write(line)

    writer.write(render_goodbye())
    writer.close()


async def main():
    srv = await asyncio.start_server(
        handle_request, 'localhost', 8083)

    async with srv:
        await srv.serve_forever()


asyncio.run(main())

# To test it you can use telnet:
#     telnet 127.0.0.1 8081

```
