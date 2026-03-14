### Pipeline
`Pipeline`主要是一种网络优化。它本质上意味着客户端缓冲一堆命令并一次性将它们发送到服务器。这些命令不能保证在事务中执行。这样做的好处是节省呢每个命令的网络往返时间（RTT）。

示例：
```Go
pipe := rdb.Piprline
ctx := context.Background()

incr := pipe.Incr(ctx, "pipeline_counter")
pipe.Expire(ctx, "pipeline_counter", time.Hour)

_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```
上面的代码相当于将以下两个命令一次发个redis server端执行，与不使用`Pipeline`相比可以减少一次RTT。
```
INCR pipeline_counter
EXPIRE pipeline_counter 3600
```

也可以使用`Pipelined`：
```Go
var incr *redis.IntCmd
ctx := context.Background()
_, err := rdb.Pipelined(ctx, func(pipe redis.Pipeliner) error {
    incr = pipe.Incr(ctx, "pipelined_counter")
    pipe.Expire(ctx, "pipelined_counter", time.Hour)
    return nil
})
fmt.Println(incr,Val(), err)
```

### 事务
Redis是单线程的，因此单个命令始终是原子的，但是来自不同客户端的两个给定命令可以依次执行，例如在它们之间交替执行。但是，`MULTI/EXEC`能够确保在`MULTI/EXEC`两个语句之间的命令之间没有其他客户端正在执行命令。

在这种场景我们需要使用`TxPipeline`。`TxPipeline`总体上类似于`Pipeline`，但是它内部会使用`MULTI/EXEC`包裹排队的命令。例如：
```Go
pipe := rdb.TxPipeline()
ctx := context.Background()

incr := pipe.Incr(ctx, "tx_pipeline_counter")
pipe.Expire(ctx, "tx_pipeline_counter", time.Hour)

_, err := pipe.Exec()
fmt.Println(incr.Val(), err)
```
上面的代码相当于在一个RTT下执行了下面的redis指令：
```
MULTI
INCR pipeline_counter
EXPIRE pipeline_counter 3600
EXEC
```
`TxPipelined`方法与上述的`Pipelined`类似
```Go
var incr *redis.IntCmd
var ctx := context.Background()
_, err := rdb.TxPipelined(ctx, func(pipe redis.Pipeliner) error {
    incr = pipe.Incr(ctx, "tx_pipelined_counter")
    pipe.Expire(ctx, "tx_pipelined_counter", time.Hour)
    return nil
})
fmt.Println(incr.Val(), err)
```


### Watch
在某些场景下，我们除了要使用`MULTI/EXEC`命令外，还需要配合使用`WATCH`命令。在用户使用`WATCH`命令监视某个键之后，直到该用户执行`EXEC`命令的这段时间里，如果有其他用户抢先对被监视的键进行了替换、更新、删除等操作，那么当用户尝试执行`EXEC`的时候，事务将失败并返回一个错误，用户可以根据这个错误选择重试事务或者放弃事务。
```Go
Watch(fn func(*Tx) error, key ...string) error
```
Watch方法接收一个函数和一个或多个key作为参数。
```Go
	// 监视watch_count的值，并在值不变的前提下将其值+1
	key := "watch_count"
	err := rdb.Watch(context.Background(), func(tx *redis.Tx) error {
		n, err := tx.Get(context.Background(), key).Int()
		if err != nil && err != redis.Nil {
			return err
		}
		// 开始事务
		_, err = tx.TxPipelined(context.Background(), func(pipe redis.Pipeliner) error {
			// 业务逻辑
			time.Sleep(time.Second * 10)
			pipe.Set(context.Background(), key, n+1, 0)
			return nil
		})
		return err
	}, key)
	if err != nil {
		fmt.Printf("watch err:%v\n", err)
		return
	}
	fmt.Println("watch ok")
```