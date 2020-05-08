---
title: "看看go的sync.once"
date: 2020-05-07T20:45:47+08:00
draft: flase
toc: true
images:
tags: 
  - go
---
## 用途
once保证传入内部的函数只会执行一次
## once源码
```
type Once struct {
	// done indicates whether the action has been performed.
	// It is first in the struct because it is used in the hot path.
	// The hot path is inlined at every call site.
	// Placing done first allows more compact instructions on some architectures (amd64/x86),
	// and fewer instructions (to calculate offset) on other architectures.
	done uint32
	m    Mutex
}
func (o *Once) Do(f func()) {
	// Note: Here is an incorrect implementation of Do:
	//
	//	if atomic.CompareAndSwapUint32(&o.done, 0, 1) {
	//		f()
	//	}
	//
	// Do guarantees that when it returns, f has finished.
	// This implementation would not implement that guarantee:
	// given two simultaneous calls, the winner of the cas would
	// call f, and the second would return immediately, without
	// waiting for the first's call to f to complete.
	// This is why the slow path falls back to a mutex, and why
	// the atomic.StoreUint32 must be delayed until after f returns.

	if atomic.LoadUint32(&o.done) == 0 {
		// Outlined slow-path to allow inlining of the fast-path.
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```
源码并不长，struct内部也只有两个属性
1. 互斥锁
2. 计数器

从源码能看到，会先对计数器`done`进行原子性的判断是否为0，才会调用`doslow`方法，由于锁的存在，即便同时多个`goroutine`进入`doslow`方法,也只有一个goroutine能够拿到锁，其他的会阻塞在`lock`位置    
等执行完业务函数f后，将计数器置为1，再释放锁，这个时候之前阻塞在`lock`位置的`goroutine`能够接着往下执行了，但此时`o.done`已经被改成1了，于是无法继续往下执行f函数
## 小结
`mutex`保证了同一时间只有一个goroutine能够执行f   
`done`保证了在一个goroutine执行完f之后，其他的goroutine无法再调用f   
## 后续
对于once源码中出现的`Mutex`和`aotmic`还未进行深入的了解，放在后面在看吧   
peace
