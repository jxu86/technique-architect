`go1.16.5`







```
// src/time/sleep.go
type Timer struct {
	C <-chan Time
	r runtimeTimer
}

type runtimeTimer struct {
	pp       uintptr
	when     int64
	period   int64
	f        func(interface{}, uintptr) // NOTE: must not be closure
	arg      interface{}
	seq      uintptr
	nextwhen int64
	status   uint32
}
```










参考:       
[Golang 定时器详解](https://studygolang.com/articles/32876)            
[👍Go中定时器实现原理及源码解析](https://www.luozhiyun.com/archives/458)