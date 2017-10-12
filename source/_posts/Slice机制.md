---
title: Golang slice 切片原理
date: 2017-07-10 19:27:43
tags: Golang
thumbnail: https://blog.bluematador.com/assets/img/mini-guide-google-golang-why-its-perfect-for-devops/golang-gopher-laptop.png
---

# Golang slice 切片原理

​	golang 中的 slice 是比较好用的一种结构，能根据需求变长，相对于 array 的死板，slice 更加灵活也更加常用，有道说：知其然，知其所以然。现在，我们就看看 slice 到底是怎样一种结构。

### slice源码

```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

​	根据平常使用情况，我们推测 len 是 slice 长度，cap 是 slice 的容量，而 array 则是指向底层数组的指针。

​	有同学可能不知道 `unsafe.Pointer`是什么，那么我们一并在这讲解一下。

```go
package unsafe

type ArbitraryType int

//	- A pointer value of any type can be converted to a Pointer.
//	- A Pointer can be converted to a pointer value of any type.
//	- A uintptr can be converted to a Pointer.
//	- A Pointer can be converted to a uintptr.
type Pointer *ArbitraryType

func Sizeof(x ArbitraryType) uintptr
func Offsetof(x ArbitraryType) uintptr
func Alignof(x ArbitraryType) uintptr
```

`unsafe.Pointer`：一个指向 `int`类型的指针，通常用于转换不同类型的指针，go 语言中指针不能运算。

`uintptr`：内置类型，能存储指针的整型，底层类型是`int`，可以和`unsafe.Pointer`互相转换，因为就

​		是 `int`型，所以可以用来做运算，GC 不会把它当指针，所以不持有对象，会被回收。

上面的四句注释是什么意思呢，解释一下就是：

1. 任何类型的指针可转换成一个 Pointer 类型的值。
2. Pointer 类型可以转换成任何类型的指针。
3. uintptr 类型可转换成 Pointer 类型。
4. Pointer 类型可以转换成 uintptr 类型。

同学们有没有想到一些什么？就是说，go 语言中虽然没有指针运算，但是通过 unsafe 包，可以把指针转换成

unsafe.Pointer，再转换成 uintptr，之后就能和 C 语言指针运算类似的功能了。举个例子：

```go
package main

import (
	"fmt"
	"unsafe"
)

type Entity struct {
	a	byte
	b	byte
}

func main() {
  	entity := Entity{}
  	fmt.Println(entity)	

	p := unsafe.Pointer(&entity)	// 转换成通用指针 unsafe.Pointer 类型
	uintpa := uintptr(p)			// 保存结构体 entity 实例地址，偏移量为0
	pb := (*byte)(p)				// 将通用指针转换为 byte 型指针
	*pb = 10						// 给转换后的指针赋值
	fmt.Println(entity)				// 结构体内容跟着改变

	uintpb := uintpa + unsafe.Offsetof(entity.b)	// 偏移到 entity.b 字段开始的位置
	p = unsafe.Pointer(uintpb)		// 将偏移后的地址转换为通用指针 unsafe.Pointer 类型
	pb = (*byte)(p)					// 将通用指针转换为 byte 型指针
	*pb = 20						// 给转换后的指针赋值
	fmt.Println(entity)				// 结构体内容跟着改变
}

// 结果：
// {0 0}
// {10 0}
// {10 20}
```

好，unsafe.Pointer 就告一段落，继续讲我们的 slice。

![sliceInstance](/img/gallery/slice.png)

​						**[图片取自 the way to go ，侵权请告知，删]**

### 通过 make 创建切片

make 创建切片的源码：

```go
func makeslice(et *_type, len, cap int) slice {
	// 根据类型获取此类型能包含元素的最大长度
	maxElements := maxSliceCap(et.size)
  
  	// 比较容量和长度 若比零小或比最大值大，越界
	if len < 0 || uintptr(len) > maxElements {
		panic(errorString("makeslice: len out of range"))
	}
	
	if cap < len || uintptr(cap) > maxElements {
		panic(errorString("makeslice: cap out of range"))
	}

	// 向内存申请一块此类型 array 的空间
	p := mallocgc(et.size*uintptr(cap), et, true)
	// 将指针、长度、容量赋值并返回切片结构
	return slice{p, len, cap}
}
```

### 切片增长

通过 append 可以对切片扩容，源码看下面：

```go
func growslice(et *_type, old slice, cap int) slice {
	if raceenabled {
		callerpc := getcallerpc(unsafe.Pointer(&et))
		racereadrangepc(old.array, uintptr(old.len*int(et.size)), callerpc, funcPC(growslice))
	}
	if msanenabled {
		msanread(old.array, uintptr(old.len*int(et.size)))
	}

	if et.size == 0 {
		if cap < old.cap {
			panic(errorString("growslice: cap out of range"))
		}
		// 创建一个不为nil的切片
		return slice{unsafe.Pointer(&zerobase), old.len, cap}
	}

	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			for newcap < cap {
				newcap += newcap / 4
			}
		}
	}

	var lenmem, newlenmem, capmem uintptr
	const ptrSize = unsafe.Sizeof((*byte)(nil))
	switch et.size {
	case 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		newcap = int(capmem)
	case ptrSize:
		lenmem = uintptr(old.len) * ptrSize
		newlenmem = uintptr(cap) * ptrSize
		capmem = roundupsize(uintptr(newcap) * ptrSize)
		newcap = int(capmem / ptrSize)
	default:
		lenmem = uintptr(old.len) * et.size
		newlenmem = uintptr(cap) * et.size
		capmem = roundupsize(uintptr(newcap) * et.size)
		newcap = int(capmem / et.size)
	}

	if cap < old.cap || uintptr(newcap) > maxSliceCap(et.size) {
		panic(errorString("growslice: cap out of range"))
	}

	var p unsafe.Pointer
	if et.kind&kindNoPointers != 0 {
		p = mallocgc(capmem, nil, false)
		memmove(p, old.array, lenmem)
		memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
	} else {
		p = mallocgc(capmem, et, true)
		if !writeBarrier.enabled {
			memmove(p, old.array, lenmem)
		} else {
			for i := uintptr(0); i < lenmem; i += et.size {
				typedmemmove(et, add(p, i), add(old.array, i))
			}
		}
	}

	return slice{p, old.len, newcap}
}
```

切片在append的时候如果有额外的容量可用，append将可用的元素合并到切片的长度，然后对他进行赋值，如果没有可用的容量，append会创建新的底层数组，将现有的值复制到新的数组里再追加新的值。

### 切片复制

通过 copy 可以复制一个切片，源码如下：

```go
func slicecopy(to, fm slice, width uintptr) int {
	if fm.len == 0 || to.len == 0 {
		return 0
	}

	n := fm.len
	if to.len < n {
		n = to.len
	}

	if width == 0 {
		return n
	}

	if raceenabled {
		callerpc := getcallerpc(unsafe.Pointer(&to))
		pc := funcPC(slicecopy)
		racewriterangepc(to.array, uintptr(n*int(width)), callerpc, pc)
		racereadrangepc(fm.array, uintptr(n*int(width)), callerpc, pc)
	}
	if msanenabled {
		msanwrite(to.array, uintptr(n*int(width)))
		msanread(fm.array, uintptr(n*int(width)))
	}

	size := uintptr(n) * width
	if size == 1 { 
		*(*byte)(to.array) = *(*byte)(fm.array) // known to be a byte pointer
	} else {
		memmove(to.array, fm.array, size)
	}
	return n
}
```

copy切片会把源切片值(第二个参数值)中的元素复制到目标切片(第一个参数值)中，并返回被复制的元素个数，copy 的两个类型必须一致，并且实际复制的数量等于实际较短切片长度。