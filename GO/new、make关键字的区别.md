make 和 new 是两个用于内存分配的内置函数.

# new关键字

## 功能
1. 分配内存
2. 返回指向该内存的指针

##  先看源码

### 声明部分
```

// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.

//新的内置函数分配内存。第一个参数是一个类型，
//不是一个值，返回的值是一个指向new对象的指针
//分配该类型的零值。

func new(Type) *Type

```

```

```


# make 关键字

## 功能
1. 分配内存
2. 初始化
3. 返回初始化的内容本身

## 先看源码

### 声明部分
```
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
//
//   - Slice: The size specifies the length. The capacity of the slice is
//     equal to its length. A second integer argument may be provided to
//     specify a different capacity; it must be no smaller than the
//     length. For example, make([]int, 0, 10) allocates an underlying array
//     of size 10 and returns a slice of length 0 and capacity 10 that is
//     backed by this underlying array.
//   - Map: An empty map is allocated with enough space to hold the
//     specified number of elements. The size may be omitted, in which case
//     a small starting size is allocated.
//   - Channel: The channel's buffer is initialized with the specified
//     buffer capacity. If zero, or the size is omitted, the channel is
//     unbuffered.


// make内置函数分配并初始化一个类型为
// slice, map, or chan （only）。和new一样，第一个参数是类型，而不是a
/ /值。与new不同，make的返回类型与它的函数的返回类型相同
//参数，而不是指向它的指针。结果的规格取决于
//类型：
//
// - Slice: size指定长度。切片的容量为
//等于其长度。第二个整数参数可以提供给
//指定不同的容量；它必须不小于
/ /长度。例如，make（[]int, 0,10）分配一个底层数组
//返回一个长度为0，容量为10的切片
//由这个底层数组支持。
// - Map：一个空的Map被分配了足够的空间来容纳
//指定的元素个数。大小可以省略，在这种情况下
//初始大小较小。
// - Channel: Channel的buffer被初始化
//缓冲区容量。如果为零，或者省略大小，则通道为
/ /无缓冲的。
func make(t Type, size ...IntegerType) Type


```


## 源码解读
开发人员在源码里,还diss了一下字节的sonic包,通过linkname访问了不暴漏的函数.

slcie , map ,chan 等均为引用数据类型
数据 data 存储为指针,
返回的是slcie,map,chan等本身
但是其中的data 也是为指针

```
// makeslice should be an internal detail,
// but widely used packages access it using linkname.
// Notable members of the hall of shame include:
//   - github.com/bytedance/sonic
//
// Do not remove or change the type signature.
// See go.dev/issue/67401.
//
//go:linkname makeslice
func makeslice(et *_type, len, cap int) unsafe.Pointer {
	mem, overflow := math.MulUintptr(et.Size_, uintptr(cap))
	if overflow || mem > maxAlloc || len < 0 || len > cap {
		// NOTE: Produce a 'len out of range' error instead of a
		// 'cap out of range' error when someone does make([]T, bignumber).
		// 'cap out of range' is true too, but since the cap is only being
		// supplied implicitly, saying len is clearer.
		// See golang.org/issue/4085.
		mem, overflow := math.MulUintptr(et.Size_, uintptr(len))
		if overflow || mem > maxAlloc || len < 0 {
			panicmakeslicelen()
		}
		panicmakeslicecap()
	}

	return mallocgc(mem, et, true)
}
```


# 代码实践
```
func main() {
	// new
	stringPrt := new(string)
	fmt.Println(*stringPrt)
	fmt.Println(stringPrt)
	*stringPrt = "hello world"
	fmt.Println(stringPrt)
	fmt.Println(*stringPrt)

	// make
	stringSlice := make([]string, 0, 10)
	fmt.Println(stringSlice)
}

```

# 异同点

## 差异点

1. 值接受不同
 new接受的是一个类型,且没有限制特定类型.
 make接收的是map,slice,channel指定类型之下,且可指定容量大小,长度.
 设置合理的大小,可避免扩容,优化性能.

 2. 返回值不同
 new 返回的是类型的指针.
 make 返回的是初始化后的本身.