---
title: Golang单元测试和基准测试
tags:
  - unit test
  - benchmark test
abbrlink: 29714
date: 2022-12-02 20:02:09
cover: https://s2.loli.net/2022/11/18/KvsLg6YXrU154aj.jpg
categories: 
  - Golang
  - 测试
---

# 单元测试
- 测试需要用到的断言，依赖
`github.com/stretchr/testify`

- 编写单元测试，文件命名带`_test.go`
`helloworld_test.go`
```go
import "testing"
func TestHelloWorld(t *testing.T) {
    t.Log("hello world")
}
```
- 测试所有用例
```shell
go test -v helloworld_test.go
```
- 指定用例
``` shell
go test -v -run TestHelloWorld helloworld_test.go
```
- 测试文件夹下所有的测试用例
```shell
go test -v ./directory
```

# 基准测试
- 非并发方式
`benchmark_test.go`
```go
func Benchmark_Add(b *testing.B) {
    var n int
    for i := 0; i < b.N; i++ {
        n++
    }
}
```
>**注：b.N 由基准测试框架提供。测试代码需要保证函数可重入性及无状态，也就是说，测试代码不使用全局变量等带有记忆性质的数据结构。避免多次运行同一段代码时的环境不一致，不能假设 N 值范围。**

- 并发方式
```go
func Benchmark_Dosomething(b *testing.B) {
	b.SetParallelism(8)
	b.RunParallel(func(pb *testing.PB) {
		for pb.Next() {
			Dosomething()
		}
	})
}
```
>**注：SetParallelism用来设置协程数量**

# 基准测试脚本编写
```shell
#!/bin/sh
go test -bench=. -run=none \
  -benchtime=300s \
  -benchmem -memprofile=mem.pprof \
  -cpuprofile=cpu.pprof \
  -cpu=8 \
  -timeout=0 \
  -blockprofile=block.pprof ./directory/
```

>**基准测试默认10分钟，如果测试总时长超过10分钟，测试的进程会被kill掉，所以指定timeout=0表示不限时长**

# go基准测试源码中`benchmark.go`
有这么一个核心方法
```go
// launch launches the benchmark function. It gradually increases the number
// of benchmark iterations until the benchmark runs for the requested benchtime.
// launch is run by the doBench function as a separate goroutine.
// run1 must have been called on b.
func (b *B) launch() {
	// Signal that we're done whether we return normally
	// or by FailNow's runtime.Goexit.
	defer func() {
		b.signal <- true
	}()

	// Run the benchmark for at least the specified amount of time.
	if b.benchTime.n > 0 { // 这里表示测试时是使用了次数限制，例如指定了参数-benchtime=1000x表示执行1000次
		// We already ran a single iteration in run1.
		// If -benchtime=1x was requested, use that result.
		// See https://golang.org/issue/32051.
		if b.benchTime.n > 1 {
			b.runN(b.benchTime.n)
		}
	} else { // 这个分支表示指定的是时间限制（区别于次数限制），例如指定了参数-benchtime=5s给示每个用例执行5s（这里会根据实际情况运行，而非精确的5s）
		d := b.benchTime.d
        // 注意下面循环条件中的n的值的上限
		for n := int64(1); !b.failed && b.duration < d && n < 1e9; {
			last := n
			// Predict required iterations.
			goalns := d.Nanoseconds()
			prevIters := int64(b.N)
			prevns := b.duration.Nanoseconds()
			if prevns <= 0 {
				// Round up, to avoid div by zero.
				prevns = 1
			}
			// Order of operations matters.
			// For very fast benchmarks, prevIters ~= prevns.
			// If you divide first, you get 0 or 1,
			// which can hide an order of magnitude in execution time.
			// So multiply first, then divide.
			n = goalns * prevIters / prevns
			// Run more iterations than we think we'll need (1.2x).
			n += n / 5
			// Don't grow too fast in case we had timing errors previously.
			n = min(n, 100*last)
			// Be sure to run at least one more than last time.
			n = max(n, last+1)
			// Don't run more than 1e9 times. (This also keeps n in int range on 32 bit platforms.)
			n = min(n, 1e9)
			b.runN(int(n))
		}
	}
	b.result = BenchmarkResult{b.N, b.duration, b.bytes, b.netAllocs, b.netBytes, b.extra}
}
```

>在基准测试时，如果设定的是时间限制，即每个测试用例执行多少时间。因为每次执行的时间都不可能完全一致，所以用例的执行次数到底应该为多少次，这个值是会被估算出来的，估算的过程就是在以下的循环中。
```go
for n := int64(1); !b.failed && b.duration < d && n < 1e9; {
    // 每一次循环，我们称之为以下过程的一轮，每一轮总的分为2个步骤
    // 1.估算一下n，n即是接下来用例需要执行多少次？
    // 2.估算好后，调用b.runN(int(n)) n的值为多少，用例这一轮就被调用了多少次
}
```

- 明确了大体的流程后，我们来分析一下细节：
  - n的上限问题
    >设定的用例执行时间还没到，但是用例执行次数已到，那就只有提前结束循环，也就意味着没法再驱动b.runN(int(n))的调用；也就会出现还没到设定时间，用例提前结束了。说明用例执行较快。这种情况换执行次数限定比较合适？

  - n的估算中有一个公式的问题

    `n = goalns * prevIters / prevns`

    - `goalns`是用例执行的目标时间，单位ns
    - `prevIters`是`上一轮`用例的执行次数
    - `prevens`是`上一轮`用例的执行时间，单位ns

    >那么问题来了：如果goalns的值较大（设置了比较长的用例测试时间，上一轮的测试次数也较大，但是上一轮的用例测试时间很短，那根据公式，估算出来的n可能很大，可能比int64的区间范围要大，溢出后就会变成一个负值。
    >变成负值之后,n的增长逻辑就不一样了，分析一下：
    >n += n / 5  -> n还是负值
    >n = min(n, 100*last) -> n的值还是n
    >n = max(n, last+1) -> 变为上一轮估算的值+1 == (last + 1)
    >n = min(n, 1e9) -> n == (last + 1)
    >同理，每一轮估算，估算值都是以1步进，问题就出现了：从n值溢出开始，假设这个时候last的值为 1e8，那n的值变化规率为100000001, 100000002, 100000003, ...,这样如果想让n达到1e9，要`1e9 - 1e8` 轮, 而且每轮要执行用例n次。注意：这只是一个估算的过程，还没真正开始做基准测试呢...这个估算执行次数的过程就已经很消耗时间了,最终有可能超过10min，因为go benchmark的默认执行时间就是10min，这个时候测试进程会被强杀，并提示执行超时的错误。

# 基准测试问题重现方法
- 写一个耗时很短的用例
- 设置一个较长的用例测试时间（不用太长，比如200s就行，只要想办法让n值溢出即可）
- 执行一下...就等着10min钟以后吧

# 解决办法
- 1. 上面提到的n溢出问题，虽然觉得benchmark.go里面的代码确实有点问题，可以把n的数据类型再改大一些如float64，在方法中增加判断溢出后进行特殊处理。
- 2. 出现这种情况，说明被测试的用例做长时间测试的必要性值得商榷

# 另外
>func (b *B) launch() 是非并行模式下会被调用的方法；同样在并行模式下，调用的是func (b *B) RunParallel(body func(*PB)) 应该同样存在类似问题。
