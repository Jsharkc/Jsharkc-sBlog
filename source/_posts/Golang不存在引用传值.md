---
title: Golang 中不存在引用传参
date:  2017-05-03 13:27:43
tags:  Golang
thumbnail: /img/gallery/gopherokno.png
author: Jshakrc
---
## Golang 中不存在引用传参

​															[原文链接](https://dave.cheney.net/2017/04/29/there-is-no-pass-by-reference-in-go)

​															翻译：Jsharkc

### 什么是引用变量 ?

在 C++ 语言中，你可以为已存在的变量声明一个别名，这就是引用变量：

```c++
#include <stdio.h>

int main() {
        int a = 10;
        int &b = a;
        int &c = b;

        printf("%p %p %p\n", &a, &b, &c); // 0x7ffe114f0b14 0x7ffe114f0b14 0x7ffe114f0b14
        return 0;
}
```

可以看到，a、b、c 三个变量的地址是相同的，也就是说它们是同一个内存地址的变量，只不过有三个别名。就好比你有一个大名，一个小名，不管别人叫大名还是小名叫的都是你，如果改变 a 变量，b、c 变量也会跟着变。当你声明一个引用变量在不同的函数作用域中这是非常有用的。

### Golang 没有引用变量

与 C++ 不同，在 Golang 中声明的每个变量都只能占用不同的内存空间的。

```go
package main

import "fmt"

func main() {
        var a, b, c int
        fmt.Println(&a, &b, &c) // 0x1040a124 0x1040a128 0x1040a12c
}
```

在 Golang 项目中不可能存在两个变量共享一块内存，但是可以创建两个变量指向相同的内存地址，但这和两个变量共享一块内存是不一样的。

```go
package main

import "fmt"

func main() {
        var a int
        var b, c = &a, &a
        fmt.Println(b, c)   // 0x1040a124 0x1040a124
        fmt.Println(&b, &c) // 0x1040c108 0x1040c110
}
```

### Map 和 Channel 是引用吗？

不，Map 和 Channel 不是引用，如果是的话下面这个程序会打印 false。

```Go
package main

import "fmt"

func fn(m map[int]int) {
        m = make(map[int]int)
}

func main() {
        var m map[int]int
        fn(m)
        fmt.Println(m == nil)
}
```

如果 `map m` 是和 C++ 风格一样的引用变量的话，那么 main 函数里声明的 m 与 fn 函数声明的 m 在内存中占用的应该是同一块内存。但是在 fn 中对 m 分配的值并没有影响到 main 中的 m，所以我们知道 map 和 channel 不是引用变量。

### 结论

Golang 没有引用传参，因为 Golang 不存在引用变量。

