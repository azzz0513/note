**上下文（Context）可以理解为程序执行的背景环境，包含了在特定时刻程序所需的所有信息**。这些信息可以包括变量的值、函数的调用情况、执行的位置等。就好像是在一场戏剧中，演员需要了解剧本、舞台布景和其他演员的动作一样，程序也需要上下文来理解自身在何处、在做什么。
## 上下文的作用
上下文在编程中具有重要作用，它帮助程序管理状态、控制流以及执行过程。以下是上下文的关键作用：
1. 状态管理：上下文有助于跟踪和管理程序中的状态变化。在多线程，每个线程都有自己的上下文，以避免数据交叉干扰。
2. 函数调用：当一个函数被调用时，程序的上下文会切换到这个函数的上下文。函数的执行将在新的上下文中进行，而在函数执行完毕后，程序将返回到原来的上下文。
3. 内存管理：上下文有助于管理内存分配和释放。
4. 异常处理：在程序运行时，如果出现了异常情况，上下文将切换到异常处理的状态，以确保程序可以适当地应对异常。

## Context
在Go语言中，context 是一个核心并发控制工具，用于管理跨Goroutine的请求生命周期、传递元数据和实现协调式取消。
对服务器传入的请求应该创建上下文，而对服务器的传出调用应该接受上下文。它们之间的函数调用链必须传递上下文，或者可以使用WithCancel、WithDeadline、WithTimeout或WithValue创建的派生上下文。当一个上下文被取消时，它派生的所有上下文也被取消。

### Context的核心作用
- 取消信号：context可以在父goroutine中创建，并传递给子goroutine。当父goroutine完成时，可以通过调用CancelFunc取消子goroutine的执行。
- 超时控制：context可以设置一个超时时间，确保子goroutine在指定时间内完成任务，防止无限等待。
- 上下文信息：context可以携带一些请求相关的信息，比如用户ID、请求ID、开始时间等，方便在不同的goroutine中访问和使用。
在 Goroutine 构成的树形结构中对信号进行同步以减少计算资源的浪费是`context.Context`的最大作用。
如下图所示，我们可能会创建多个 Goroutine 来处理一次请求，而`context.Context`的作用是在不同 Goroutine 之间同步请求特定数据、取消信号以及处理请求的截止日期。
![[images/Pasted image 20250918205824.png]]
每一个`context.Context`都会从最顶层的Goroutine一层一层传递到最下层。`context.Context`可以在上层Goroutine执行出现错误时，将信号及时同步给下层。
![[images/Pasted image 20250918210002.png]]
如上图所示，当最上层的 Goroutine 因为某些原因执行失败时，下层的 Goroutine 由于没有接收到这个信号所以会继续工作；但是当我们正确地使用 `context.Context`时，就可以在下层及时停掉无用的工作以减少额外资源的消耗：
![[images/Pasted image 20250918210052.png]]


### Context接口
context.Context是一个接口，该接口定义了四个需要实现的方法。具体签名如下：
```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```
其中：
- `Deadline`方法需要返回当前Context被取消的时间，也就是完成工作的截止时间（deadline）；
- `Done`方法需要返回一个Channel，这个Channel会在当前工作完成或者上下文被取消之后关闭，多次调用Done方法会返回同一个Channel；
- `Err`方法会返回当前Context结束的原因，它只会在Done返回的Channel被关闭时才会返回非空的值；
    - 如果当前Context被取消就会返回Canceled错误；
    - 如果当前Context超时就会返回DeadlineExceeded错误；
- `Value`方法会从Context中返回键对应的值，对于同一个上下文来说，多次调用Value 并传入相同的Key会返回相同的结果，该方法仅用于传递跨API和进程间跟请求域的数据；

### 在Goroutine间传递Context
context的设计初衷就是在线性调用链中传递。当启动一个新的goroutine时，应将context传递给该goroutine。多个 Goroutine 同时订阅 `ctx.Done()` 管道中的消息，一旦接收到取消信号就立刻停止当前正在执行的工作。
```Go
package main
 
import (
    "context"
    "fmt"
    "time"
)
 
func worker(ctx context.Context) {
    select {
    case <-ctx.Done():
        fmt.Println("Worker: Context已取消")
    case <-time.After(5 * time.Second):
        fmt.Println("Worker: 完成任务")
    }
}
 
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
 
    go worker(ctx)
 
    fmt.Println("Main: 等待5秒后取消Context")
    time.Sleep(3 * time.Second)
    cancel()
    fmt.Println("Main: Context已取消")
}
```

### 默认上下文
`context`包中最常用的方法就是`context.Background`、`context.TODO`，这两个方法都会返回预先初始化好的私有变量`background`和`todo`，他们会在同一个Go程序中被复用。
```go
func Background() Context {
	return background
}

func TODO() Context {
	return todo
}
```
这两个私有变量都是通过 `new(emptyCtx)` 语句初始化的，它们是指向私有结构体`context.emptyCtx`的指针，这是最简单、最常用的上下文类型：
```go
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (*emptyCtx) Done() <-chan struct{} {
	return nil
}

func (*emptyCtx) Err() error {
	return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}
```
- `context.Background`是上下文的默认值，所有其他的上下文都应该从它衍生出来
- `context.TODO`应该仅在不确定应该使用哪种上下文时使用
在多数情况下，如果当前函数没有上下文作为入参，我们都会使用`context.Background`作为起始的上下文向下传递。
![[images/Pasted image 20250918212845.png]]

### 取消信号
`context.WithCancel`函数能够从`context.Context`中衍生出一个新的子上下文并返回用于取消该上下文的函数。一旦我们执行返回的取消函数，当前上下文以及其子上下文都会被取消，所有的Goroutine都会同步收到这一取消信号。
```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}
```
- `context.newCancelCtx`将传入的上下文包装成私有结构体`context.cancelCtx`
- `context.propagateCancel`会构建父子上下文之间的关联，当父上下文被取消时，子上下文也会被取消：
  ```go
func propagateCancel(parent Context, child canceler) {
	done := parent.Done()
	if done == nil {
		return // 父上下文不会触发取消信号
	}
	select {
	case <-done:
		child.cancel(false, parent.Err()) // 父上下文已经被取消
		return
	default:
	}

	if p, ok := parentCancelCtx(parent); ok {
		p.mu.Lock()
		if p.err != nil {
			child.cancel(false, p.err)
		} else {
			p.children[child] = struct{}{}
		}
		p.mu.Unlock()
	} else {
		go func() {
			select {
			case <-parent.Done():
				child.cancel(false, parent.Err())
			case <-child.Done():
			}
		}()
	}
}
```
1. 当 `parent.Done() == nil`，也就是 `parent` 不会触发取消事件时，当前函数会直接返回；
2. 当 `child` 的继承链包含可以取消的上下文时，会判断 `parent` 是否已经触发了取消信号；
    - 如果已经被取消，`child` 会立刻被取消；
    - 如果没有被取消，`child` 会被加入 `parent` 的 `children` 列表中，等待 `parent` 释放取消信号；
3. 当父上下文是开发者自定义的类型、实现了`context.Context`接口并在 `Done()` 方法中返回了非空的管道时；
    1. 运行一个新的 Goroutine 同时监听 `parent.Done()` 和 `child.Done()` 两个 Channel；
    2. 在 `parent.Done()` 关闭时调用 `child.cancel` 取消子上下文；

`context.propagateCancel`的作用是在`parent`和`child`之间同步取消和结束的信号，保证在`parent`被取消时，`child`也会收到对应的信号，不会出现状态不一致的情况。

`context.cancelCtx.cancel`：该方法会关闭上下文中的Channel并向所有的子上下文同步取消信号：
```go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
	c.mu.Lock()
	if c.err != nil {
		c.mu.Unlock()
		return
	}
	c.err = err
	if c.done == nil {
		c.done = closedchan
	} else {
		close(c.done)
	}
	for child := range c.children {
		child.cancel(false, err)
	}
	c.children = nil
	c.mu.Unlock()

	if removeFromParent {
		removeChild(c.Context, c)
	}
}
```

`context`中还有另外两个函数`context.WithDeadline`和`context.WithTimeout`也都能创建可以被取消的计时器上下文`context.timerCtx`：
```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}

func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		return WithCancel(parent)
	}
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
	propagateCancel(parent, c)
	dur := time.Until(d)
	if dur <= 0 {
		c.cancel(true, DeadlineExceeded) // 已经过了截止日期
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.err == nil {
		c.timer = time.AfterFunc(dur, func() {
			c.cancel(true, DeadlineExceeded)
		})
	}
	return c, func() { c.cancel(true, Canceled) }
}
```
- `context.WithDeadline`在创建`context.timerCtx`的过程中判断了父上下文的截止日期与当前日期，并通过`time.AfterFunc`创建定时器，当时间超过了截止日期后会调用`context.timerCtx.cancel`同步取消信号。
- `context.timerCtx`内部不仅通过嵌入`context.cancelCtx`结构体继承了相关的变量和方法，还通过持有的定时器`timer`和截止时间`deadline`实现了定时取消的功能。

### 传值方法
`context.WithValue`能从父上下文中创建一个子上下文，传值的子上下文使用`context.valueCtx`类型：
```go
func WithValue(parent Context, key, val interface{}) Context {
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}
```
`context.valueCtx`结构体会将除了`Value`之外的`Err`、`Deadline`等方法代理到父上下文中，它只会响应`context.valueCtx.Value`方法：
```go
type valueCtx struct {
	Context
	key, val interface{}
}

func (c *valueCtx) Value(key interface{}) interface{} {
	if c.key == key {
		return c.val
	}
	return c.Context.Value(key)
}
```

### 示例
```Go
package main

import (
	"context"
	"fmt"
	"time"
)

func step1(ctx context.Context) context.Context {
	child := context.WithValue(ctx, "name", "张三")
	return child
}

func step2(ctx context.Context) context.Context {
	child := context.WithValue(ctx, "age", 13)
	return child
}

func step3(ctx context.Context) {
	fmt.Printf("name %s\n", ctx.Value("name"))
	fmt.Printf("age %d\n", ctx.Value("age"))
}

func f1() {
	// 定时关闭管道
	// 所有的context都应该从context.Background()开始，这是整个上下文树的根节点
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	// 超时关闭之后触发ctx.Done，函数返回前调用cancel()放置内存泄露
	defer cancel()

	select {
	case <-ctx.Done():
		fmt.Println("timeout")
	}
}

func f2() {

	// 父context关闭会关闭其子context
	// 子context关闭并不会关闭其父context，只有父context自身的超时时间到期或者被显示Cancel，父context才会关闭
	parent, cancel1 := context.WithTimeout(context.Background(), time.Millisecond*1000)
	defer cancel1()
	t1 := time.Now()

	time.Sleep(time.Millisecond * 250)

	child, cancel2 := context.WithTimeout(parent, time.Millisecond*250)
	defer cancel2()
	t2 := time.Now()

	select {
	case <-parent.Done():
		fmt.Println("parent done")
		t3 := time.Now()
		fmt.Println(t3.Sub(t1).Milliseconds(), t3.Sub(t2).Milliseconds())
	case <-child.Done():
		fmt.Println("child done")
		t3 := time.Now()
		fmt.Println(t3.Sub(t1).Milliseconds(), t3.Sub(t2).Milliseconds())
	}
}

func f3() {
	// 显式调用CancelFunc关闭管道
	ctx, cancel := context.WithCancel(context.Background())
	t0 := time.Now()
	go func() {
		time.Sleep(time.Millisecond * 500)
		cancel()
	}()
	select {
	case <-ctx.Done():
		fmt.Println("ctx done")
		t1 := time.Now()
		fmt.Println(t1.Sub(t0).Milliseconds())
	}
}

func main() {
	//grandParent := context.Background()
	//father := step1(grandParent)
	//grandSon := step2(father)
	//step3(grandSon)
	//f1()
	f2()
	//f3()
}
```