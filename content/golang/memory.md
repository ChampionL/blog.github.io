---
title: "内存管理"
description: "内存管理"
keywords: "源码方式剖析内部管理方式，从基础类型出发，持续更新"

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
<!--more-->

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

内存分配流程
- runtime.newobject()(malloc.go) 新建对象  
- func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer (malloc.go) 申请内存对象
- mp := acquirem() // 获取当面goroutine的mcache
- c := getMCache(mp) //获取p的mcache      *mcache
- if size <= maxSmallSize 32kb
- if noscan && size < maxTinySize { // 不含有指针，小于16字节对象
- 当对象小于16字节时候，我们不能直接释放该对象，因为小于16字节对象会通过小对象物体来进行合并到一个16字节对象里
- tiny allocator 主要为小字符串和独立的转义字符，在json压力测试中，tiny allocator可以减少
  减少内存分配次数12% 减少堆内存分配20%
- off = alignUp(off, 8) //调整off成8倍数 调整off的位置使其为8的倍数 4的倍数 2的倍数
- if off+size <= maxTinySize && c.tiny != 0 { //如果off+size大小小于16字节，并且tiny模块有分配内存块，那么在该tiny块上分配
- x = unsafe.Pointer(c.tiny + off) //tiny加上偏移量就是需要分配内存
- c.tinyoffset = off + size
  c.tinyAllocs++
  小微对象分配的内存块,更新offset以及微小对象分配数量
- 如果tiny为空，或者off+size超过了当前tiny对象大小，那么申请分配一个新的内存块，
- span = c.alloc[tinySpanClass] // tiny对象需要分配16字节 tinySpanClass = 2<<1 + 1 等于 5(136) 从mcache的alloc中获取mspan
- v := nextFreeFast(span)       //从mspan中获取一个对象块
- v, span, shouldhelpgc = c.nextFree(tinySpanClass) //如果没有获取到，查看下一个内存块 TODO  
`tinyAllocs似乎是从0开始，首次分配完成是0 再次分配才是1，似乎如果是首次分配和正常块分配可以处理成一样`
- 此时申请到了就将该元素作为返回值处理，此时这个块的offset还只是部分
- 从mcache的alloc数组中查看是否已经有该类型的mspan申请下来，如果有，则在该mspan申请内存
- 如果该mspan为空，那么从mheap_的cental[spc]的mcentral中调用cacheSpan申请新的mspan
  将该span挂在mcache的alloc[spc]下
- s := mheap_.alloc(npages, c.spanclass)  如果mcentral中没有可以利用的mspan 两个spanset扫描也没有
- base, scav = c.alloc(npages)  从pagecache中分配小内存对象
- 
```go
type heapArena struct {
	bitmap [heapArenaBitmapWords]uintptr //用1bit表示其中8字节(64)的使用情况，4字节(非64位)的使用情况  整个heapArena区域64M 分成每个word是否在使用，表示一个word的使用情况

	// If the ith bit of noMorePtrs is true, then there are no more
	// pointers for the object containing the word described by the
	// high bit of bitmap[i].
	// In that case, bitmap[i+1], ... must be zero until the start
	// of the next object.
	// We never operate on these entries using bit-parallel techniques,
	// so it is ok if they are small. Also, they can't be bigger than
	// uint16 because at that size a single noMorePtrs entry
	// represents 8K of memory, the minimum size of a span. Any larger
	// and we'd have to worry about concurrent updates.
	// This array uses 1 bit per word of bitmap, or .024% of the heap size (for 64-bit).
	noMorePtrs [heapArenaBitmapWords / 8]uint8

	// spans maps from virtual address page ID within this arena to *mspan.
	// For allocated spans, their pages map to the span itself.
	// For free spans, only the lowest and highest pages map to the span itself.
	// Internal pages map to an arbitrary span.
	// For pages that have never been allocated, spans entries are nil.
	//
	// Modifications are protected by mheap.lock. Reads can be
	// performed without locking, but ONLY from indexes that are
	// known to contain in-use or stack spans. This means there
	// must not be a safe-point between establishing that an
	// address is live and looking it up in the spans array.
	spans [pagesPerArena]*mspan // pagesPerArena 每个arena有多少个page arena是 heapArenaBytes / pageSize  heapArenaBytes 是64位，pageSize是8kb  可能多个page页面指向的是同一个mspan指针

	// pageInUse is a bitmap that indicates which spans are in
	// state mSpanInUse. This bitmap is indexed by page number,
	// but only the bit corresponding to the first page in each
	// span is used.
	//
	// Reads and writes are atomic.
	pageInUse [pagesPerArena / 8]uint8

	// pageMarks is a bitmap that indicates which spans have any
	// marked objects on them. Like pageInUse, only the bit
	// corresponding to the first page in each span is used.
	//
	// Writes are done atomically during marking. Reads are
	// non-atomic and lock-free since they only occur during
	// sweeping (and hence never race with writes).
	//
	// This is used to quickly find whole spans that can be freed.
	//
	// TODO(austin): It would be nice if this was uint64 for
	// faster scanning, but we don't have 64-bit atomic bit
	// operations.
	pageMarks [pagesPerArena / 8]uint8

	// pageSpecials is a bitmap that indicates which spans have
	// specials (finalizers or other). Like pageInUse, only the bit
	// corresponding to the first page in each span is used.
	//
	// Writes are done atomically whenever a special is added to
	// a span and whenever the last special is removed from a span.
	// Reads are done atomically to find spans containing specials
	// during marking.
	pageSpecials [pagesPerArena / 8]uint8

	// checkmarks stores the debug.gccheckmark state. It is only
	// used if debug.gccheckmark > 0.
	checkmarks *checkmarksMap

	// zeroedBase marks the first byte of the first page in this
	// arena which hasn't been used yet and is therefore already
	// zero. zeroedBase is relative to the arena base.
	// Increases monotonically until it hits heapArenaBytes.
	//
	// This field is sufficient to determine if an allocation
	// needs to be zeroed because the page allocator follows an
	// address-ordered first-fit policy.
	//
	// Read atomically and written with an atomic CAS.
	zeroedBase uintptr
}
```
`bitmap`：该bitmap用一个bit来描述arenaHeap中一个word的使用情况
`noMorePtrs`:heapArena结构体中的noMorePtrs字段用于标记arena中是否还有指针未被扫描。它的作用是在并发垃圾回收过程中进行同步和优化

mallocinit  内存初始化
physPageSize&(physPageSize-1) != 0 判断是否是2的次幂


```go
type fixalloc struct {
	size   uintptr
	first  func(arg, p unsafe.Pointer) // called first time p is returned
	arg    unsafe.Pointer
	list   *mlink
	chunk  uintptr // use uintptr instead of unsafe.Pointer to avoid write barriers
	nchunk uint32  // bytes remaining in current chunk
	nalloc uint32  // size of new chunks in bytes
	inuse  uintptr // in-use bytes now
	stat   *sysMemStat
	zero   bool // zero allocations
}
```
fixalloc是固定size的内存分配，在开始时候指定为固定大小  
size表示要固定分配的内存大小  
first表示这个块第一次mmap过来时候需要执行的函数，f.arg 表示这个函数要执行的参数  
chunk表示当前可以分配的内存大小，可能会多次覆盖这个值，当内存不够时候回信申请内存块来覆盖，  
fixalloc固定大小为16k,按照size大小取整后 `f.nalloc = uint32(_FixAllocChunk / size * size) `  
nchunk表示当前这块内存块还剩下多少，表示大小。  
inuse表示当前的fixalloc有多少内存在使用  
f.stat 表示sysMemStat 分配了多少内存  
f.list 如果有不需要再使用了的size大小的内存，需要是否，则会将整个内存清空，只保留第一个字作为指向下一个释放块的大小 
所以f.list是指向空闲已释放的内存块，如果f.list有空闲对象，则是否出来，同时将其清零。  
TODO: 如果chunk被覆盖，说明内存已经被完全用完了，如果释放后会被放入list也不会放入chunk中，所以目前看新建chunk  
之前内存还是能够寻址到  

pageAlloc: https://zhuanlan.zhihu.com/p/603335718
用于存储256T内存中页的使用信息，bitmap表示了页的使用情况。用于寻找符合连续的几页是空闲的进行分配。
基数树:  https://ivanzz1001.github.io/records/post/data-structure/2018/11/18/ds-radix-tree

