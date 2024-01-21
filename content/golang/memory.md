---
title: "内存管理"
description: "内存管理"
keywords: "内存"

date: 2023-12-27T23:05:02+08:00
lastmod: 2023-12-27T23:05:02+08:00

categories:
  - golang
tags:
  - 内存管理

# 原文作者
# Post's origin author name
#author:
# 原文链接
# Post's origin link URL
#link:
# 图片链接，用在open graph和twitter卡片上
# Image source link that will use in open graph and twitter card
#imgs:
# 在首页展开内容
# Expand content on the home page
expand: false
# 外部链接地址，访问时直接跳转
# It's means that will redirecting to external links
#extlink:
# 在当前页面关闭评论功能
# Disabled comment plugins in this post
#comment:
#  enable: false
# 关闭文章目录功能
# Disable table of content
#toc: false
# 绝对访问路径
# Absolute link for visit
#url: "memory.html"
# 开启文章置顶，数字越小越靠前
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

内存管理

```go
type _type struct {
    size       uintptr
    ptrdata    uintptr // size of memory prefix holding all pointers
    hash       uint32
    tflag      tflag
    align      uint8
    fieldAlign uint8
    kind       uint8
    // function for comparing objects of this type
    // (ptr to object A, ptr to object B) -> ==?
    equal func (unsafe.Pointer, unsafe.Pointer) bool
    // gcdata stores the GC type data for the garbage collector.
    // If the KindGCProg bit is set in kind, gcdata is a GC program.
    // Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
    gcdata    *byte
    str       nameOff
    ptrToThis typeOff
}
```

参考文章： https://www.luozhiyun.com/archives/506
```shell
[root@VM-8-3-centos test]# dlv debug test.go 
Type 'help' for list of commands.
(dlv) break main.main
Breakpoint 1 set at 0x49d106 for main.main() ./test.go:9
(dlv) c
> main.main() ./test.go:9 (hits goroutine(1):1 total:1) (PC: 0x49d106)
     4:	
     5:	type A struct {
     6:	    test string
     7:	}
     8:	
=>   9:	func main() {
    10:	    a := new(int)
    11:	    fmt.Println(a)
    12:	}
(dlv) n
> main.main() ./test.go:10 (PC: 0x49d114)
     5:	type A struct {
     6:	    test string
     7:	}
     8:	
     9:	func main() {
=>  10:	    a := new(int)
    11:	    fmt.Println(a)
    12:	}
(dlv) b break runtime.newobject
Breakpoint break set at 0x40d5e6 for runtime.newobject() /usr/local/go/src/runtime/malloc.go:1253
(dlv) c
> [break] runtime.newobject() /usr/local/go/src/runtime/malloc.go:1253 (hits goroutine(1):1 total:1) (PC: 0x40d5e6)
Warning: debugging optimized function
  1248:	}
  1249:	
  1250:	// implementation of new builtin
  1251:	// compiler (both frontend and SSA backend) knows the signature
  1252:	// of this function.
=>1253:	func newobject(typ *_type) unsafe.Pointer {
  1254:		return mallocgc(typ.size, typ, true)
  1255:	}
  1256:	
  1257:	//go:linkname reflect_unsafe_New reflect.unsafe_New
  1258:	func reflect_unsafe_New(typ *_type) unsafe.Pointer {
(dlv) p typ
(*runtime._type)(0x4a5140)
*runtime._type {size: 8, ptrdata: 0, hash: 3413333906, tflag: tflagUncommon|tflagExtraStar|tflagNamed|tflagRegularMemory (15), align: 8, fieldAlign: 8, kind: 2, equal: runtime.memequal64, gcdata: *0, str: 516, ptrToThis: 18688}
(dlv) 
```
执行查看newobject创建过程，创建一个64位int类型，注意在进入该断点时候在去break runtime.newobject 这样抓取到的才是int 堆类型的创建

golang struct内存对齐
CPU 访问内存时，并不是逐个字节访问，而是以字长（word size）为单位访问。  
简言之：合理的内存对齐可以提高内存读写的性能，并且便于实现变量操作的原子性。  
内存对齐: https://geektutu.com/post/hpg-struct-alignment.html


内存对齐是指，字节起始位置要是当前字段大小的整数倍，同时整体长度要是最大字段长度的整数倍。

内存对齐的原因 https://stackoverflow.com/questions/381244/purpose-of-memory-alignment  

总结来说两条规则：

长度要对齐: 结构体的长度要至少是内部最长的基础字段的整数倍。
地址要对齐: 字段的地址偏移要是自身长度的整数倍。

假设结构体如下
```go
type TestStruct struct {
	ptr uintptr
	f1  uint8
	f2  *uint8
	f3  uint32
	f4  *uint64
	f5  uint64
}
```
内存布局 | 8 | 1 - 7 | 8 | 4- 4| 8| 8 |
总共需要 48个字节
其中ptrdata 指向f4 最后一个字节，因此是40，指向f5开始的地方
```shell
(dlv) p typ
*runtime._type {
	size: 48,
	ptrdata: 40,
	hash: 4075663022,
	tflag: tflagUncommon|tflagExtraStar|tflagNamed (7),
	align: 8,
	fieldalign: 8,
	kind: 25,
	alg: *runtime.typeAlg {hash: type..hash.main.TestStruct, equal: type..eq.main.TestStruct},
	gcdata: *20,
	str: 28887,
	ptrToThis: 49504,}
```

hash值 暂时不清楚用途，类型hash值
tflag 类型实际是一个uint8 额外类型信息标志  
tflagUncommon 代表类型中含有指针  
tflagExtraStar rtype.str字段有*前缀  
tflagNamed 代表类型有个名字  
tflagRegularMemory 该类型是否内存连续  

align      uint8   //该类型变量对齐方式  在内存分配时候该字节的对齐方式
// Align returns the alignment in bytes of a value of
// this type when allocated in memory.
fieldalign uint8   //该类型结构字段对齐方式  该类型在结构体中应该对齐的方式，比如8字节
// FieldAlign returns the alignment in bytes of a value of
// this type when used as a field in a struct.

kind： 表示类型  runtime/typekind.go 中定义
```go
const (
	kindBool = 1 + iota
	kindInt
	kindInt8
	kindInt16
	kindInt32
	kindInt64
	kindUint
	kindUint8
	kindUint16
	kindUint32
	kindUint64
	kindUintptr
	kindFloat32
	kindFloat64
	kindComplex64
	kindComplex128
	kindArray
	kindChan
	kindFunc
	kindInterface
	kindMap
	kindPtr
	kindSlice
	kindString
	kindStruct
	kindUnsafePointer

	kindDirectIface = 1 << 5
	kindGCProg      = 1 << 6
	kindMask        = (1 << 5) - 1
)
```

```go
typeAlg 主要是类型相关的hash或者是是否相等算法
// typeAlg is 总是 在 reflect/type.go 中 copy或使用.
// 并保持他们同步.
type typeAlg struct {
	// 算出该类型的Hash
	// (ptr to object, seed) -> hash
	hash func(unsafe.Pointer, uintptr) uintptr
	// 比较该类型对象
	// (ptr to object A, ptr to object B) -> ==?
	equal func(unsafe.Pointer, unsafe.Pointer) bool
}
```

gcdata： 这是一个字节数组，存储的是一个指针类型位置的bitmap 
*20
20 转换成二进制数据就是 00010100  高位指向的是低地址 可以看出第三个和第五个8字节存储的是指针

```go
type A struct {
    test string
    t1 * int64
    t2 * uint64
    t3   int64
    t5   uint8
}
```

```shell
(dlv) p &a.p &a.ptr
Command failed: a has no member p
(dlv) p &a.Test
Command failed: a has no member Test
(dlv) p &a.test
(*string)(0xc000064150)
(dlv) p &a.t1
(**int64)(0xc000064160)
(dlv) p &a.t2
(**uint64)(0xc000064168)
(dlv) p &a.t3
(*int64)(0xc000064170)
(dlv) p &a.t5
(*uint8)(0xc000064178)
(dlv) 
```

可以看到该类型的gcdata为13  00001101  第一个字段string为指针类型 第二个字段int64为指针类型，第三个字段为指针类型
理论上是00000111 但是因为string占据了16个字节，需要占据两位，因此变成了00001101

nameOff 和 typeOff 类型是 int32 ，这两个值是链接器负责嵌入的，相对于可执行文件的元信息的偏移量。元信息会在运行期，
加载到 runtime.moduledata 结构体中。

TODO： 类型 interface底层实现 (https://blog.csdn.net/i6448038/article/details/82916330) 

raceenabled 竞态检测
https://blog.csdn.net/qq_25341531/article/details/119903651

<!--more-->
