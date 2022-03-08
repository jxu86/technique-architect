

```
type _defer struct {
	siz     int32      //参数和结果的内存大小
	started bool
	heap    bool       //是否是堆上分配
	openDefer bool     // 是否经过开放编码的优化
	sp        uintptr  //栈指针
	pc        uintptr  // 调用方的程序计数器
	fn        *funcval // 传入的函数
	_panic    *_panic  // panic that is running defer
	link      *_defer  //defer链表
	fd   unsafe.Pointer // funcdata for the function associated with the frame
	varp uintptr        // value of varp for the stack frame
	framepc uintptr
}
```

## golang中defer和return的顺序

`return其实并不是一个原子操作，其包含了下面几步：`  
`将返回值保存在栈上->执行defer函数->函数返回。`


```
例1：
func f1() (i int) {
　　i = 1
　　defer func(){i++}()
　　return i
}
```
return是分为两步执行的，第一步赋值给返回值，第二步真正的返回到函数外部。而defer是在第一步之后执行。
所以，例1中，“return i”其实是把 i赋值给返回值i（当然，这里return的值就是i，所以其实没有赋值），此时i=1,然后再执行 defer，i=2,返回的i最终值是2。
```
例2：
func f2() int {
    i := 1
    defer func(){i++}()
    return i
}
```
例2这里，“return i”把i=1赋值给返回值，但是这里的返回值没有显示声明，会生成一个临时变量，假设叫‘tmp’，即tmp=1。然后，执行defer的时候，i=2。但是这个和'tmp'没关系。所以最终返回的是1。
```
例3：
func f2() *int {
    i := 1
    defer func(){i++}()
    return &i
}
```
例3这里，“return &i”把i的地址赋值给返回值，同样是临时变量‘tmp *int’。即，真正返回的是i的地址*tmp。然后，执行defer的时候，对tmp指针没操作，但是tmp指向的那个值（即i），修改成了2。所以，如果对返回的指针取值，结果是2。和传址参数一样理解。    
`综上，如果是显式命名的返回值，则defer中可以对其操作。如果是非显式命名的返回值，则返回时会新定义一个返回变量，defer操作不到。当然，指针变量。。。为所欲为！`



参考:       
[深入 Go 语言 defer 实现原理](https://www.luozhiyun.com/archives/523)