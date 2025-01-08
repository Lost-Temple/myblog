---
title: Golang学习-知识点-1
tags:
  - channel
categories:
  - Golang
  - 基础
cover: 'https://s2.loli.net/2023/01/04/FCrhU5zdSo4IOvk.png'
abbrlink: 57201
date: 2023-01-04 08:58:34
---

# 前言
Golang名言：使用通信来共享内存，而不是使用共享内存来通信。Golang中有`sync`包提供了传统方式的锁机制。但更推荐使用`channel`来解决并发问题。

# channel的用法
## 什么是channel
`channel`是`goroutine`之间用来接收或发送消息的安全消息队列。`channel`就像是它的字面意思：通道。它就是两个`goroutine`之间用来同步资源的一个通道。
![channel示意图](Golang学习-知识点-1/1.jpg)

```go
func main() {
	ch := make(chan int, 1) // 创建一个类型为int,缓冲区大小为1的channel
	ch <-2 // 将2发送到channel中
	n, ok := <-ch // n从channel中接收数据
	if ok {
		fmt.Println(n) // 2
	}
	close(ch) // 关闭channel
}
```

## 使用时的注意点
- 向一个`nil channel`发送消息会一直阻塞

    Code:
    ```go
    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        var ch chan int

        go send(ch)
        <-ch
        time.Sleep(time.Second * 1)
    }

    func send(ch chan int) {
        fmt.Println("Sending value to channnel start")
        ch <- 1
        fmt.Println("Sending value to channnel finish")
    }
    ```
    Output:
    ```
    Sending value to channnel start
    fatal error: all goroutines are asleep - deadlock!
    goroutine 1 [chan receive (nil chan)]:
    goroutine 18 [chan send (nil chan)]:
    ```
    分析：上述代码只是声明了channel，它的默认初始值为nil, 在一个goroutine中向这个 nil channel 发送消息，这个goroutine 就会一直阻塞,而在main函数中(主routine)中想从channel中获取这个消息也是会被阻塞住，所以整个进行就相当于处理死锁状态。

- 向一个已关闭的`channel`发送消息会引发运行时`panic`
- `channel`关闭后不可以继续向它发送消息，但可以继续从它接收消息
- 当`channel`关闭后并且缓冲区为空时，继续从它接收消息会得到一个对应类型的零值

# unbuffered channels与buffered channels
- `unbuffered channel`是缓冲区大小为0的`channel`，这种`channel`的接收者会阻塞直至接收到消息；发送者会阻塞直至接收到消息，这种机制可以用于两个`goroutine`进行状态同步。
- `buffered channel`拥有缓冲区，当缓冲区已满时，发送者会阻塞，当缓冲区为空时，接收者会阻塞。
  
引用[The Nature Of Channels In Go](https://www.ardanlabs.com/blog/2014/02/the-nature-of-channels-in-go.html)中的两张图来说明`Unbuffered channels`与`Buffered channels`

## Unbuffered channels
![Unbuffered channels](Golang学习-知识点-1/2.png)


## Buffered channels
![Buffered channels](Golang学习-知识点-1/3.png)

### unbuffered channel示例
让我们构建一个示例程序，它使用四个 goroutine 和一个通道来模拟接力赛。比赛中的参赛者将被称为“ Goroutines”，通道将被用来在每个参赛者之间交换接力棒。这是一个典型的例子，说明了如何在 goroutines 之间传递资源，以及通道如何控制与之交互的 goroutines 的行为。

```go
func Runner(baton chan int) {
	var newRunner int

	// Wait to receive the baton
	runner := <-baton

	// Start running around the track
	fmt.Printf("Runner %d Running With Baton\n", runner)

	// New runner to the line
	if runner != 4 {
		newRunner = runner + 1
		fmt.Printf("Runner %d To The Line\n", newRunner)
		go Runner(baton)
	}

	// Running around the track
	time.Sleep(100 * time.Millisecond)

	// Is the race over
	if runner == 4 {
		fmt.Printf("Runner %d Finished, Race Over\n", runner)
		return
	}

	// Exchange the baton for the next runner
	fmt.Printf("Runner %d Exchange With Runner %d\n", runner, newRunner)
	baton <- newRunner
}

func main() {
	// Create an unbufferd channel
	baton := make(chan int)

	// First runner to his mark
	go Runner(baton)

	// Start the race
	baton <- 1

	// Give the runners time to race
	time.Sleep(500 * time.Millisecond)
}
```
运行结果：
```
Runner 1 Running With Baton
Runner 2 To The Line
Runner 1 Exchange With Runner 2
Runner 2 Running With Baton
Runner 3 To The Line
Runner 2 Exchange With Runner 3
Runner 3 Running With Baton
Runner 4 To The Line
Runner 3 Exchange With Runner 4
Runner 4 Running With Baton
Runner 4 Finished, Race Over
```

## channel的遍历
### 阻塞型的遍历方式
```go
func main() {
	ci := make(chan int, 5)
	for i := 1; i <= 5; i++ {
		ci <- i
	}
	close(ci)
	
	for i := range ci {
		fmt.Println(i)
	}
}
```
>值得注意的是：在遍历channel前，如果channel没有关闭，遍历会被阻塞，会出现deadlock的错误；如果在遍历时channel已经关闭，那么会遍历完数据后退出遍历；也就是说for range的遍历方式是阻塞型的遍历方式。

### 非阻塞型的遍历方式
`select`可用于非阻塞式消息发送、接收及多路选择
```go
func main() {
	ci := make(chan int, 2)
	for i := 1; i <= 2; i++ {
		ci <- i
	}
	close(ci)

	cs := make(chan string, 2)
	cs <- "hi"
	cs <- "golang"
	close(cs)

	ciClosed, csClosed := false, false
	for {
		if ciClosed && csClosed {
			return
		}
		select {
		case i, ok := <-ci:
			if ok {
				fmt.Println(i)
			} else {
				ciClosed = true
				fmt.Println("ci data fetching finished")
			}
		case s, ok := <-cs:
			if ok {
				fmt.Println(s)
			} else {
				csClosed = true
				fmt.Println("cs data fetching finished")
			}
		default:
			fmt.Println("wating...")
		}
	}
}
```
>`select`中有`case`代码块，用于`channel`发送或接收消息，任意一个`case`代码块准备好时，执行其对应内容；多个`case`代码块准备好时，随机选择一个`case`代码块并执行；所有`case`代码块都没有准备好，则等待；还可以有一个`default`代码块，所有`case`代码块都没有准备好时执行default代码块。这里的所谓的没有准备好，指的就是channel没有关闭。当执行了close()函数后，就是所谓的准备好了。