

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





参考:       
[深入 Go 语言 defer 实现原理](https://www.luozhiyun.com/archives/523)