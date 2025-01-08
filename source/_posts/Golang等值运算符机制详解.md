---
title: Golang等值运算符机制详解
tags:
  - nil
categories:
  - Golang
  - 基础
cover: 'https://s2.loli.net/2022/12/15/mugAGU9jBbTWs4Q.jpg'
abbrlink: 2463
date: 2023-02-01 09:07:22
---

# 背景
这篇文章源于代码中关于nil判断的一个问题。先从一段代码看起，下面这段代码是将传入的对象转换成`JSON string`并返回，其中，如果判断`i==nil`时，会返回`""`。
```go
func ToJSONString(i interface{}) string {
   if i == nil {
      return ""
   }
   bytes, _ := json.Marshal(i)
   return string(bytes)
}
```
这段代码初看并没有太大的问题，但实际上这里隐含了一个不容易被发现的问题，在说明这个问题之前，我们先看下这段代码在什么情况下会出现问题：
```go
type Data struct {
   V int `json:"v"`
}

func TestToJSONString(t *testing.T) {
   a := assert.New(t)

   a.Equal("", ToJSONString(nil))

   var data *Data
   a.Equal("", ToJSONString(data))

   var k *int
   a.Equal("", ToJSONString(k))
}
```
这段测试代码中三个断言分别返回：
```
PASS
FAIL, Expect: "", Actual: "null"
FAIL, Expect: "", Actual: "null"
```
这里很让人疑惑，data应该是nil，似乎data==nil的结论是false，为了确认这个问题，我们引入一段代码。
```go
var a *int = nil
var b interface{} = nil

fmt.Println("a == nil:", a == nil)
fmt.Println("b == nil:", b == nil)
fmt.Println("a == b:", a == b)
```
结果为：
```
a == nil: true
b == nil: true
a == b: false // 重点
```
这个例子比较简单，让我们看一个很相似，但稍微复杂些的例子
```go
var a *int = nil
var b interface{} = a

fmt.Println("a == nil:", a == nil)
fmt.Println("b == nil:", b == nil)
fmt.Println("a == b:", a == b)
```
结果为：
```
a == nil: true
b == nil: false // 重点
a == b: true
```
# 到底发生了什么？
首先我们要了解的是：在Go中，每个指针都有2个基本信息，指针的类型和指针的值，后续我将会使用`(type, value)`这样的形式来体现。也就是说，`a := nil`这样的代码无法通过编译，因为它缺失了`type`，所以需要这样`var a *int = nil`。
我们可以在fmt中输出这些指针的`type`：
```go
var a *int = nil
var b interface{} = nil

fmt.Printf("a.type:%T\n", a) // a.type:*int
fmt.Printf("b.type:%T\n", b) // b.type:<nil>
```
我们现在已经了解了`interace{}`的默认`type`为`nil`，下面我们不使用硬编码的方式，而是将`a`传递给`b`，然后再来看下结果：
```go
var a *int = nil
var b interface{} = a

fmt.Printf("a.type:%T\n", a) // a.type:*int
fmt.Printf("b.type:%T\n", b) // b.type:*int
```
也就是说，`b`现在有了一个新的类型`*int`。目前你已经了解了关于类型的一些基本机制，但还有一些问题还没有被解答.

# 当执行`==`时，发生了什么？
```go
var a *int = nil
var b interface{} = nil

fmt.Printf("a=(%T, %v)\n", a, a)
fmt.Printf("b=(%T, %v)\n", b, b)
fmt.Println("a == nil:", a == nil)
fmt.Println("b == nil:", b == nil)
fmt.Println("a == b:", a == b)
```
结果：
```
a=(*int, <nil>)
b=(<nil>, <nil>)
a == nil: true
b == nil: true
a == b: false
```
这段代码乍一看似乎不可能，像是在说：a==nil、b==nil，但a!=b。实际情况是，类型判断不仅仅只判断二者的值，还会判断其类型。
```
a == nil // 等价于：(*int, nil) == (*int, nil)
b == nil // 等价于：(nil, nil) == (nil, nil)
a == b   // 等价于：(*int, nil) == (nil, nil)
```
当我们用这样的方式写出来时，我们可以很轻易的明白，a!=b是成立的，但这些信息并不会在代码中体现出来，这也正是容易出现误解的地方。在这里，如果你想要判断二者是否都是nil，你可以这样写
```go
if a == nil && b == nil {
  // do something
}
```
接下来我们再来看看前面那个令人困惑的例子，相关结果直接跟在了代码后面
```go
var a *int = nil
var b interface{} = a

fmt.Printf("a=(%T, %v)\n", a, a)   // a=(*int, <nil>)
fmt.Printf("b=(%T, %v)\n", b, b)   // b=(*int, <nil>)
fmt.Println("a == nil:", a == nil) // a == nil: true
fmt.Println("b == nil:", b == nil) // b == nil: false
fmt.Println("a == b:", a == b)     // a == b: true
```
现在我们再来看b==nil这段代码会发现明朗许多：
```go
b == nil // 等价于：(*int, nil) == (nil, nil)
```
这里可能会疑惑，为什么等式右侧的`nil`，其`type`为`nil`呢？这是因为当`b`指定为`interface{}`类型的时候，无法确定其真实的类型，随着程序的运行，其类型可能会不断改变，所以其类型默认为`nil`。
更进一步，我们可以看一个硬编码的数字是如何进行比较判断的。数字会根据上下文来推断自己的类型，一个具体的例子如下：
```go
var a int = 12
var b float64 = 12
var c interface{} = a

fmt.Println("a==12:", a == 12) // true  => (int, 12) == (int, 12)
fmt.Println("b==12:", b == 12) // true  => (float64, 12) == (float64, 12)
fmt.Println("c==12:", c == 12) // true  => (int, 12) == (int, 12)
fmt.Println("a==c:", a == c)   // true  => (int, 12) == (int, 12)
fmt.Println("b==c:", b == c)   // false => (float64, 12) == (int, 12)
```
一个需要注意的点是，当`12`与一个`interface{}`进行比较时，会默认转换为`(int, 12)`，类似的`interface{}`也会被强制转换为`(nil, nil)`，如下代码展示了这个过程:
```go
var b float64 = 12
var c interface{} = b

fmt.Println("c==12:", c == 12) // c==12: false
fmt.Printf("c=(%T,%v)\n", c, c) // c=(float64,12)
fmt.Printf("hard-coded=(%T,%v)\n", 12, 12) // hard-coded=(int,12)
```

# 回到最开始的`nil`判断
让我们回到最开始出现问题的代码，现在再来看会清晰许多
```go
func ToJSONString(i interface{}) string {
   if i == nil {
      return ""
   }
   bytes, _ := json.Marshal(i)
   return string(bytes)
}

func TestToJSONString() {
  // ...
  var data *Data
  a.Equal("", ToJSONString(data))
}
```
第2行中，`i==nil`的判断相当于`(*Data, nil) == (nil, nil)`，显而易见，这样的等式并不会成立。当然，我们有其他的办法处理这样的等式判断，这需要用到`reflect`包，如下：
```go
func ToJSONString(i interface{}) string {
   if i == nil || (reflect.ValueOf(i).Kind() == reflect.Ptr && reflect.ValueOf(i).IsNil()) {
      return ""
   }
   bytes, _ := json.Marshal(i)
   return string(bytes)
}
```
