


## 底层结构
```
// 非空接口
type iface struct {
	tab  *itab
	data unsafe.Pointer
}

// 空接口
type eface struct {
	_type *_type
	data  unsafe.Pointer
}
```
```
type itab struct {
	inter *interfacetype // 接口自身的元信息
	_type *_type         // 具体类型的元信息
	hash  uint32         // copy of _type.hash. Used for type switches. // _type 里也有一个同样的 hash，此处多放一个是为了方便运行接口断言
	_     [4]byte
	fun   [1]uintptr // variable sized. fun[0]==0 means _type does not implement inter. // 函数指针，指向具体类型所实现的方法
}
```
```
type _type struct {
	size       uintptr // 类型大小
	ptrdata    uintptr // size of memory prefix holding all pointers 前缀持有所有指针的内存大小
	hash       uint32  // 数据 hash 值
	tflag      tflag
	align      uint8 // 对齐
	fieldAlign uint8 // 嵌入结构体时的对齐
	kind       uint8 // kind 有些枚举值 kind 等于 0 是无效的
	// function for comparing objects of this type
	// (ptr to object A, ptr to object B) -> ==?
	equal func(unsafe.Pointer, unsafe.Pointer) bool
	// gcdata stores the GC type data for the garbage collector.
	// If the KindGCProg bit is set in kind, gcdata is a GC program.
	// Otherwise it is a ptrmask bitmap. See mbitmap.go for details.
	gcdata    *byte
	str       nameOff
	ptrToThis typeOff
}
```


* eface表示不含 method 的 interface 结构，或者叫 empty interface






参考:    
[接口](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/)    