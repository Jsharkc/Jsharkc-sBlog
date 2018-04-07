---
title: Go注意点总结
date: 2016-11-15 21:34:10
tags: Golang
thumbnail: /img/gallery/GolangStudy.png
author: Jsharkc
---

# Go >注意点< 总结

#### 1. __布尔类型__

​     布尔类型 *不能* 接受其他类型的赋值，*不支持* 自动或者强制的类型转换

```Go
     var b bool
     b = 1  // 编译错误
     b = bool(1) // 编译错误
```
------

#### 2. int与int32

​     **注意：**int和int32在Go语言里被认为是两种不同的类型，编译器也不会帮你自动做类型转换

```Go
   var value2 int32
   value1:= 64   // value1将会被自动推导为int类型
   value2 = value1   // 编译错误
   // 可用强制转换解决这个编译错误：
   value2 = int32(value1)   // 编译通过
```

------
#### 3. 整数值比较
​     两种 **不同** 类型的整型数 **不能** 直接比较，比如 int8 类型的数和 int 类型的数不能直接比较，但各种类型的整型变量都可以直接与*字面常量* ( literal ) 进行比较

------
#### 4. ^x  表示对 x 取反 
----
#### 5. 字符串

​    字符串的内容可以用类似 _数组下标_ 的方式获取，但不能在初始化后被修改


```Go
   eg.
   	str := “Hello world”  
   	str[0] = ‘X’    //编译错误
```
----
#### 6. 一种特殊的switch 
```Go
   for k,v := range m{
        switch vv := v.(type) {
         case string:
             fmt.Println(k, "is string", vv)
         case int:
             fmt.Println(k, "is int", vv)
         case float64:
             fmt.Println(k,"is float64",vv)
         case []interface{}:
             fmt.Println(k, "is an array:")
             for i, u := range vv {
                 fmt.Println(i, u)
             }
         default:
             fmt.Println(k, "is of a type I don't know how to handle")
         }
   }
```
----
#### 7. make() 函数创建数组切片

```Go
   // 创建一个初始元素个数为5的数组切片，元素初始值为0:
   mySlice1 := make([]int, 5) // 创建一个初始元素个数为5的数组切片，
   // 元素初始值为0，并预留10个元素的存储空间:
   mySlice2 := make([]int, 5, 10) 
   // 直接创建并初始化包含5个元素的数组切片:
   mySlice3 := []int{1, 2, 3, 4, 5} 
   // 直接创建并初始化包含5个元素的数组切片 :
   mySlice3 := []int{3:4, 5}  // [0 0 0 4 5]
```


   当然，事实上还会有一个匿名数组被创建出来，只是不需要我们来操心而已。

----
#### 8. 数组切片追加

​    将一个数组切片 **追加** 到另一个数组切片的末尾：

```Go
   mySlice2 := []int{8,9,10}
   mySlice = append(mySlice,mySlice2...)
```


   **注意：**在第二个参数后面加三个点，不加会有编译错误，加省略号表示把mySlice2包含的元素打散后传入

----
#### 9. 判断能否从 map 中获取一个值的做法
```Go
    value, ok := map[key]
    if ok{  //找到了
       //处理找到的value
    }
```
----
#### 10.  创建并初始化类型的对象实例

```Go
rect1 := new(Rect)

rect2 := &Rect{}

rect3 := &Rect{0,0,100,200}

rect4 := &Rect{width:100,height:200}
```
----
#### 11.未显示初始化

​     Go语言中，未 **显式初始化** 的变量都会被初始化为该类型的 **零值** ，**bool** 类型零值为 **false** , **Int** 类型的零值为 **0**，**string** 类型的零值为 **空字符串**, **接口**或**引用类型**（包括 **slice**、**map**、**chan** 和函数）变量对应的零值是**nil**

----
#### 12. 无构造函数

​    Go语言中 **没有构造函数** 的概念，对象的创建通常交由一个全局的创建函数来完成，以NewXXX来命名，表示“构造函数”

----
#### 13.  go中最好用的超时机制——> select-case

​    `select`默认是阻塞的，只有当监听的 channel 中有发送或接收可以进行时才会运行，当多个 channel 都准备好的时候，select 是随机的选择一个执行的。

```Go
timeout := make(chan bool, 1)
go func() {
    time.Sleep(1e9) // 等待1秒钟
    timeout <- true 
}()

   // 然后我们把timeout这个channel利用起来 
   
select {
    case <-ch:
    // 从ch中读取到数据
    case <-timeout:
    // 一直没有从ch中读取到数据，但从timeout中读取到了数据
    }
```
**另一种实现方法**
```Go
func main() {
    c := make(chan int)
    o := make(chan bool)
    go func() {
        for {
            select {
                case v := <- c:
                    println(v)
                case <- time.After(5 * time.Second):
                    println("timeout")
                    o <- true
                    break
            }
        }
    }()
    <- o
}
```
​    在 select 里面还有 default 语法，select 其实就是类似switch 的功能，default 就是当监听的 channel 都没有准备好的时候，默认执行的（ select 不再阻塞等待 channel ）。

----

#### 14.  单向channel变量的声明

```go
var ch1 chan int        //ch1是一个正常的channel，不是单向的
var ch2 chan<- float64  //ch2是单向channel，只用于写float64数据
var ch3 <-chan int      //ch3是单向channel，只用于读取int了数据
```
----
#### 15. 关闭和判断channel
```go
  // 关闭 ch channel
  close(ch)
  // 判断一个channel是否关闭
  x,ok := <-ch
```
----
####16. runtime goroutine

Runtime 包中有几个处理 goroutine 的函数

* Goexit

  退出当前执行的 goroutine , 但是 defer 函数还会继续调用

* Gosched

  让出当前 goroutine 的执行权限， 调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复运行

* NumCPU

  返回 CPU 核数量

* NumGroutine

  返回正在执行和排队的任务总数

* GOMAXPROCS

  用来设置可以并行计算的CPU核数的最大值，并返回之前的值


----

#### 17. 一些链接

Golang学习思维导图
[http://yougg.github.io/static/gonote/GolangStudy.html](http://yougg.github.io/static/gonote/GolangStudy.html)

----

#### 18. 函数中局部变量 

​        在Go语言中，返回 **函数中局部变量** 的地址也是 **安全** 的。例如下面的代码，调用f函数时创建局部变量v，在局部变量地址被返回之后依然有效，因为指针p依然引用这个变量。

```go
var p = f()

func f() *int {
    v := 1
    return &v
}
```

----

#### 19. flag 包小例子

​	早些的echo版本中，就包含了两个可选的命令行参数：`-n`用于忽略行尾的换行符，`-s sep`用于指定分隔字符（默认是空格）。

```go
gopl.io/ch2/echo4
// Echo4 prints its command-line arguments.
package main

import (
    "flag"
    "fmt"
    "strings"
)

var n = flag.Bool("n", false, "omit trailing newline")
var sep = flag.String("s", " ", "separator")

func main() {
    flag.Parse()
    fmt.Print(strings.Join(flag.Args(), *sep))
    if !*n {
        fmt.Println()
    }
}
```

​	调用 **flag.Bool** 函数会创建一个新的对应布尔型标志参数的变量。它有三个属性：第一个是的命令行标志参数的名字“n”，然后是该标志参数的默认值（这里是false），最后是该标志参数对应的描述信息。如果用户在命令行输入了一个无效的标志参数，或者输入`-h`或`-help`参数，那么将打印所有标志参数的名字、默认值和描述信息。类似的，调用flag.String函数将于创建一个对应字符串类型的标志参数变量，同样包含命令行标志参数对应的参数名、默认值、和描述信息。程序中的`sep`和`n`变量分别是指向对应命令行标志参数变量的指针，因此必须用`*sep`和`*n`形式的指针语法间接引用它们。

​	当程序运行时，必须在使用标志参数对应的变量之前调用先flag.Parse函数，用于更新每个标志参数对应变量的值（之前是默认值）。对于非标志参数的普通命令行参数可以通过调用flag.Args()函数来访问，返回值对应对应一个字符串类型的slice。如果在flag.Parse函数解析命令行参数时遇到错误，默认将打印相关的提示信息，然后调用os.Exit(2)终止程序。

​	运行一些echo测试用例： 

```go
$ go build gopl.io/ch2/echo4
$ ./echo4 a bc def
a bc def
$ ./echo4 -s / a bc def
a/bc/def
$ ./echo4 -n a bc def
a bc def$
$ ./echo4 -help
Usage of ./echo4:
  -n    omit trailing newline
  -s string
        separator (default " ")
```

---

#### 20. Rune

```go
import "unicode/utf8"

s := "Hello, 世界"
fmt.Println(len(s))                    // "13"
fmt.Println(utf8.RuneCountInString(s)) // "9"
```

---

#### 21.内置函数

| - 名称             | - 说明                                     |
| ---------------- | ---------------------------------------- |
| close            | 用于管道通信                                   |
| len/cap          |                                          |
| new/make         | new 和 make 均是用于 **分配内存** ： new(T) 分配类型 T 的零值并返回其 **地址**，也就是指向类型 T 的指针。它也可以被用于基本类型：`v := new(int)`。make(T) 返回类型 T 的初始化之后的 **值** |
| copy/append      | 用于复制和连接切片                                |
| panic/recover    | 两者均用于错误处理机制                              |
| print/println    | 底层打印函数，在部署环境中建议使用 fmt 包                  |
| complex/realimag | 用于创建和操作复数                                |

---

#### 22. for-range 结构 

一般形式为：`for ix, val := range coll { }`

要注意的是，`val` 始终为集合中对应索引的值拷贝，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值（**译者注：如果 val 为指针，则会产生指针的拷贝，依旧可以修改集合中的原值**）。一个字符串是 Unicode 编码的字符（或称之为 `rune`）集合，因此您也可以用它迭代字符串：

```
for pos, char := range str {
...
}
```

每个 rune 字符和索引在 for-range 循环中是一一对应的。它能够自动根据 UTF-8 规则识别 Unicode 编码的字符。

---

#### 23. 从字符串生成字节切片

​	假设 s 是一个字符串（本质上是一个字节数组），那么就可以直接通过 `c := []byte(s)` 来获取一个字节的切片 c。另外，还可以通过 copy 函数来达到相同的目的：`copy(dst []byte, src string)`。

​	可以通过代码 `len([]int32(s))` 来获得字符串中字符的数量，但使用 `utf8.RuneCountInString(s)` 效率会更高一点。

​	还可以将一个字符串追加到某一个字符数组的尾部：

```
var b []byte
var s string
b = append(b, s...)
```

---

#### 24. 常量

**常量** 中的 **数据类型** 只可以是 **布尔型**、**数字型**（**整数型**、**浮点型**和**复数**）和**字符串型**。

常量的定义格式：`const identifier [type] = value`，例如：

```
const Pi = 3.14159
```

在 Go 语言中，你可以省略类型说明符 `[type]`，因为编译器可以根据变量的值来推断其类型。

- 显式类型定义： `const b string = "abc"`
- 隐式类型定义： `const b = "abc"`

**注：常量的值必须是能够在编译时就能够确定的**

---

#### 25. 方法

​	**类型** 和 **作用在它上面定义的方法** 必须在 **同一个包里** 定义，这就是为什么不能在 int、float 或类似这些的类型上定义方法。试图在 int 类型上定义方法会得到一个编译错误

---

#### 26. 反射

​	变量的最基本信息就是类型和值：反射包的   `Type` 用来表示一个 Go 类型，反射包的 `Value` 为 Go 值提供了反射接口。

​	两个简单的函数，`reflect.TypeOf` 和 `reflect.ValueOf`，返回被检查对象的 **类型** 和 **值** 。例如，x 被定义为：`var x float64 = 3.4`，那么 `reflect.TypeOf(x)` 返回 `float64`，`reflect.ValueOf(x)` 返回 `<float64 Value>`

* Value 有一个 Type 方法返回 reflect.Value 的 Type。

​        Type 和 Value 都有 Kind 方法返回一个常量来表示类型：Uint、Float64、Slice 等等 。

```
const (
	Invalid Kind = iota
	Bool
	Int
	Int8
	Int16
	Int32
	Int64
	Uint
	Uint8
	Uint16
	Uint32
	Uint64
	Uintptr
	Float32
	Float64
	Complex64
	Complex128
	Array
	Chan
	Func
	Interface
	Map
	Ptr
	Slice
	String
	Struct
	UnsafePointer
)
```

​	变量 v 的    `Interface()` 方法可以得到还原（接口）值，所以可以这样打印 v 的值：`fmt.Println(v.Interface())`。

​	通过 Type() 我们看到 v 现在的类型是   `*float64` 并且仍然是不可设置的。

要想让其可设置我们需要使用 `Elem()` 函数，这间接的使用指针：`v = v.Elem()`

-----

#### 27. 一种限制速率的方法

`time.Tick()` 函数声明为 `Tick(d Duration) <-chan Time`，当你想返回一个通道而不必关闭它的时候这个函数非常有用：它以 d 为周期给返回的通道发送时间，d是纳秒数。如果需要像下边的代码一样，限制处理频率

```go
import "time"

rate_per_sec := 10
var dur Duration = 1e9 / rate_per_sec
chRate := time.Tick(dur) // a tick every 1/10th of a second
for req := range requests {
    <- chRate // rate limit our Service.Method RPC calls
    go client.Call("Service.Method", req, ...)
}
```