+++
title = "c/c++ 可变参数"
description = ""
tags = [
    "c",
    "c++"
]
date = "2024-10-26"
categories = [
    "Development",
    "c",
    "c++"
]
menu = "main"

+++

#### 概述

在C语言中，可变参数（variadic arguments）允许函数接收不定数量和类型的参数。这种功能通常用于实现如 `printf` 和 `scanf` 等函数，这些函数可以处理任意数量的输入或输出项。

在C++中，可变参数允许函数接受任意数量的额外参数。这通过在函数声明的参数列表末尾使用省略号（`...`）来表示。



#### 函数体内访问可变参数

在一个使用了可变参数的函数体内部，这些参数的值可以通过 `<cstdarg>` 库提供的设施来访问。具体包括以下宏：

- `va_start`: 启用对可变参数函数参数的访问。
- `va_arg`: 访问下一个可变参数函数的参数。
- `va_copy` (从C++11开始): 复制可变参数函数的参数。
- `va_end`: 结束对可变参数函数参数的遍历。
- `va_list`: 保存 `va_start`, `va_arg`, `va_end` 和 `va_copy` 所需的信息。

如果最后一个参数之前是引用类型，或者其类型与默认参数提升后的类型不兼容，则 `va_start` 宏的行为是未定义的。如果在 `va_start` 中使用了一个包展开或由lambda捕获产生的实体作为最后一个参数，程序将被视为错误，且无需诊断。（从C++11开始）




**C 语法：**

```c
#include <stdarg.h>
void va_start(va_list ap, last);
type va_arg(va_list ap, type);
void va_end(va_list ap);
void va_copy(va_list dest, va_list src);
```



**C++ 示例：**

```c++
#include <stdio.h>
#include <cstdarg>

void print_numbers(int count, ...) {
    va_list args;
    va_start(args, count);

    for (int i = 0; i < count; i++) {
        int num = va_arg(args, int);
        printf("%d ", num);
    }

    va_end(args);
    printf("\n");
}

int main() {
    print_numbers(3, 1, 2, 3);
    print_numbers(5, 10, 20, 30, 40, 50);
    return 0;
}
```






#### 替代方案

- **可变模板**：也可以用来创建接受可变数量参数的函数。它们通常是更好的选择，因为它们不对参数类型施加限制，不执行整数和浮点数的提升，并且类型安全。
- **初始化列表**：如果所有可变参数共享一个共同的类型，`std::initializer_list` 提供了一种方便的机制（尽管语法不同）来访问可变参数。然而，在这种情况下，参数不能被修改，因为 `std::initializer_list` 只能提供指向其元素的常量指针。（从C++11开始）

#### 注意事项

- 在C语言直到C23标准，至少需要有一个命名参数出现在省略号参数之前，因此 `R printz(...);` 直到C23才是有效的。而在C++中，即使传递给此类函数的参数不可访问，这种形式也是允许的，并且通常用于SFINAE中的回退重载，利用省略号转换在重载解析中的最低优先级。
- 这种可变参数的语法是在1983年的C++中引入的，当时省略号前没有逗号。当C89从C++采用函数原型时，它将语法替换为要求逗号的形式。为了兼容性，C++98接受C++风格的 `f(int n...)` 和C风格的 `f(int n, ...)`。
- **类型安全**：可变参数函数不是类型安全的。调用者必须确保传递的参数类型与函数内部的 `va_arg` 调用匹配。
- **默认参数提升**：在传递给 `va_arg` 之前，整型和浮点型参数会进行默认的参数提升。例如，`char` 和 `short` 会被提升为 `int`，而 `float` 会被提升为 `double`。
- **带作用域的枚举**：传递带作用域的枚举（scoped enumeration）给可变参数函数是有条件支持的，具体行为由实现定义。
- **非POD类参数**：传递非POD（Plain Old Data）类参数给可变参数函数是有条件支持的，具体行为由实现定义。



