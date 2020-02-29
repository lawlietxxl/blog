---
title: go code snippets
date: 2020-02-26 21:37:52
tags: [go]
---
go go go
<!--more-->
# go tools

```bash
go build
go run
go fmt
go version
godoc -http=:6060
go doc fmt
go install
go get github.com/xx/xx
```

# 基础语法和库
+ 文件基本结构

```go
package main

import "fmt"

func main() {
	fmt.Println("123")
}
```
+ reflect.TypeOf(3) // int float64 bool string
+ 声明/赋值变量

```go
var quantity int
quantity = 2
var length, width float64 = 1, 2 // 一次赋值两个, 类型也可以省略
a, b := 1.1, 1.2 // 不用var声明，隐式声明，并且赋值
```
+ 开头大写的变量或者函数是exported的，例如 fmt.Println
+ go不会自动转换，需要显示进行conversion，否则会报错

```go
length := 1.2
width := 2
fmt.Println(length > float64(width))
```

短声明只要有一个名字不同就可以编译通过，新变量名会是声明，老变量名会是赋值

```go
grade1, err := strconv.ParseFloat("1.1", 64)
grade2, err := strconv.ParseFloat("1.1", 64)
```


+ 时间库

```go
var now time.Time = time.Now();
var year int = now.Year();
```
+ 字符串替换

```go
replacer := strings.NewReplacer("#","o")
fixed := replacer.Replace("1#2")
```

+ io

```go
reader := bufio.NewReader(os.Stdin)
input, error := reader.ReadString('\n')
file, err := os.Open("file.txt")
```

**go不允许声明了变量但是不用，否则会无法编译**

可以使用下滑线
```go
file, _ := os.Open("file.txt")
```

+ log 库

```go
log.Fatal(err) // 打印并且退出程序
```

+ strconv

```go
grade, err := strconv.ParseFloat(input, 64)
```

+ random

```go
import "math/rand"

rand.Seed(time.Now().Unix())
rand.Intn(100) + 1
```

+ 标准输入输出

```go
fmt.Println("%0.2f\n", 1.0/3/0)
s := fmt.Sprintln("%0.2f\n", 1.0/3/0) 
```
{% asset_img f0082-01.png %}
{% asset_img f0082-02.png %}

%后面的是最小宽度，点后面是小数点后面的长度
{% asset_img f0084-01.png %}

+ 常用命令

```go
go build
go build -o main
go install // 安装当前目录下的代码
go get github.com/headfirst/xxx // 下载到本地
go doc strconv // 给自己代码写注释
godoc -http=:6060 // 浏览器看api
```

+ os

```go
os.Args // slice
```

+ math

```go
math.Inf(-1) //最小值
```

# 条件语句

```go
if grade == 100 {
    fmt.Println()
} else if grade == 110 && a > 100 {

} else {

}
```

# 循环

```go
for x:=4; x <=6; x++ {}

for x:=4; x <=6; x+=2 {}
```

**初始化部分和循环后部分是可选的**

```go
for x<= 3 {
}

for true {
}
```

# 函数
+ error的新建

```go
import "errors"
err := errors.New("123")
// or
err := fmt.Errorf("abc %f", 1.1)
```

+ 返回多个值作为结果

```go
func mayReturns() (int, bool, string) {
    return 1, true, "hello"
}

// 可以给返回起名字，方便阅读
func floatParts(num float64) (integerPart int, fractionalPart float64) {
    //...
}

// 返回异常
func paintNeeded(width float64, height float64) (float64, error) {
    return 1.1, fmt.Errorf("error message")
}

// 永远要处理异常
amount, err := paintNeeded(1.1, -2)
if err != nil {
    log.Fatal(err)
}
//...
```

**go是一个pass-by-value的语言，函数结果得到的是值的copy**，所以函数需要指针

+ 指针类型

{% asset_img f0104-01.png %}

**go允许返回函数内某个变量的指针**

# 包

**import后面的字符串是按照文件夹名字来的**
**go文件中package的名字应该跟它所在文件夹一样（除了main）**

+ 包的常量

常量不能用短赋值
```go
const TriangleSides int = 3
```

# Array

```go
var notes [7]string
var dates [3]time.Time

// 声明赋值
var notes [7]string = [7]string{"1", "2"}
notes := [8]string{...}

fmt.Println(ntoes) // [1 2]
fmt.Println("%#v", notes) // [3]string{...}

len(notes)
```
如果取下标超过数组，会产生**panic**，这是动态错误

数组的for... range 用法
{% asset_img f0157-01.png %}

```go
for index, note := range notes {
    fmt.Println(index, note)
}
```

读文件例子
```go
file, _ := os.Open("xxx")
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    fmt.Println(scanner.Text())
}
```

**数组的长度必须是constant的**

```go
i := rand.intn(10)
var arr [i]string //这样是编译出错的！！！
```

# slices

```go
var myArr [5]int // array -- 写出size （同时编译器帮我们新建了数组）
var mySlice []int //slice -- 不写size （这时候需要我们使用make新建）
mySlice = make([]int, 7)
mySlice[0] = ...

// 或者
mySlice := make([]string, 3)
len(mySlice)
for index, value := range mySlice {
    // ...
}

// 或者
mySlice := []string{"1", "2"}
```

+ slice建立在array基础上的, slice是全部或者部分array的视图
```go
mySlice := myArray[1:3] // 通过array建立slice [arr[1], arr[2]]
mySlice := myArray[1:] //到结束 
mySlice := myArray[:3] // 从开始

// 但是不能通过array对slice进行赋值
```

因为slice是array的视图，所以改变array会改变slice。所以使用make或者 slice literal生成slice是比较好的。

+ append

```go
slice := []string{"a", "b"}
slice = append(slice, "c")
```

**潜在问题：当append的时候，array可能重新创建以获得更大的空间。这时候保留两个slice可能会造成问题。所以这里的方式是不停的替换老的slice。dont let 2 slices have the same underlying array**

+ 未赋值的slice，默认值是nil。但是很多函数特殊处理了nil的slice
```go
len(intSlice) //如果intSlice是nil，则返回0，不会造成npe
intSlice = append(intSlice, 1)

// 可变长参数函数，的入参是slice
func mix(nums ...int)

// 如果想要传slice到这个函数里面，需要用...
mix(mySlice...)
```

os.Args也是一个slice

赋值3d的slice，需要一个一个的赋值
```go
tiles = make([][][]*tile, x)

for i := range tiles {
    tiles[i] = make([][]*tile, y)
    for j := range tiles[i] {
        tiles[i][j] = make([]*tile, z)
    }
}
```

# map

```go
var myMap map[string]float64 // 声明
myMap = make(map[string]float64) // 赋值

// 或者
myMap := make(map[string]float64)

// map literals
myMap := map[string]float64{"a":1.1, "b":1.2}

// 使用
mapMap["abc"] = 1

// 检查某个key是否在map中

value, ok = myMap["abc"]
// ok = true, if "abc" in myMap

// 删除某个key
delete(myMap, "abc")

// 遍历
for key, value := range myMap {
    // ...
}
```

**go语言中的map是无序的**
```go
sort.Strings(names) //进行排序
```

# struct

+ 初级用法 -- 声明1个struct并为它赋值

{% asset_img f0235-01.png %}

把var替换为**type**关键字可以定义类型
例如
```go
type part struct {
    des string
    count int
}

var myPart part
myPart.des = ""
myPart.count = 2
```

+ 函数调用
go 是 pass-by-value的，所以需要传指针

```go
func applyDiscount(s *subscriber) {
    s.rate = 4.99 // 这里不需要*，因为.运算符可以作用于指针
    // 或者
    (*s).rate = 4.99 // 必须加括号，因为.优先级高于*
}
```
+ 大写的struct或struct下的域才能exported

```go
type MyStruct struct {
    name string
    Count int
}
```
+ struct literal

```go
subscriber := Subscriber{Rate:4.99, Name:"123"}
```

+ struct嵌套struct

```go
type Employee struct {
    HomeAddress Address
    // ...
}

type Address struct {
    // ...
    Zip int
}
```

匿名内部struct，隐去struct名字，只保留类型
{% asset_img f0258-02.png %}

> 需要了解：匿名内部struct的field是会提升到外部类可见的，比如 employee.Zip

# 自定义type和method

go不允许重写

```go
// 可以从underlying type转换成自定义type
type Gallons float64
g := Gallons(1.1)
g := Gallons(1.1) + 1.1


type myType int

func (m myType) methodExample(i int){}

// 使用指针
func (m *myType) methodExample(i int){
    *myType *= i
}

// 无论method怎么声明，调用写法一样
m.methodExample(3)

// 可以用指针调用 也可以用value调用
```

# 封装和嵌套

+ go不强制进行封装，如果不需要的话是可以直接export的
+ go封装的单位是package
+ go的通用做法是SetYear(), getter写法是 Year()，直接大写变量
+ go没有npe

```go
type dad struct {
	c child
}

type child struct{
	Name string
}

func main() {
	d := dad{}
	fmt.Println(d.c.Name) // 这不会报错
}
```

# interface 接口

myType自动成为实现myInterface的类型。interface内部方法的可见性与大小写一致
```go
type myInterface interface {
    method()
    method1(float64)
    method2() int
}

type myType int

(m *myType) method(){}
(m *myType) method1(){}
(m *myType) method2(){}
```

+ type assertion

```go
var robot Robot = interfaceNoiceMaker.(Robot) // interfaceInstance.(type)

// comma ok convention
recorder, ok := player.(TapeRecorder)
```

+ 字符串接口
{% asset_img f0342-03.png %}

+ 空接口 = anything

```go
type Anything interface{}

func AcceptAnything (i interface{}) {
}
```

# 异常处理 -- defer， recover()

```go
func getFloat() {
    file, err := OpenFile(filename)
    if err != nil {
        return // ...
    }
    defer CloseFile(file)
}
```

+ defer 后面只能调用 function或者method
+ defer 命令顺序是栈顺序，先写的后触发，最后写的先触发

+ 引发panic -- panic不是最好的处理异常的方式，因为会引起程序崩溃

```go
panic("abc")
```

+ error用于程序的bug，而panic用于处理那些不可能出现的情况：如果出现此情况，则程序就走不下去

+ recover()

```go
fmt.Println(recover()) // 对于没有panic的程序，recover()返回nil

func freakOut() {
    panic("abc")
    recover() // 不会执行到
}

// 所以用defer是可以执行到recover的

func main() {
    defer recover()
    panic("abc")
}
```
例子
{% asset_img f0372-02.png %}

+ 可以再度抛出panic，panic入参是空接口
{% asset_img f0374-02.png %}

**go语言建议少用panic和recover**

+ Most programs should panic only in the event of an unanticipated error. You should think about all possible errors your program might encounter (such as missing files or badly formatted data), and handle those using error values instead.

# 协程和信道

使用go语句，轻松开启协程

+ 与java不同，如果main协程结束，则main中开启的协程就直接结束了。如下程序，不会打印a
而java，子线程不结束，整个线程就不结束
```go
func a() {
	for i := 0; i < 10; i++ {
		fmt.Print("a")
	}
}

func main() {
	go a()
	fmt.Println("main")
}

// sleep 1s即可打印
func main() {
	go a()
    fmt.Println("main")
    time.Sleep(1*time.Second)
}
```

go语句不能得到返回，协程之间沟通需要用channel

```go
var myChannel chan float64 // 声明一个接收float64的channel
myChannel = make(chan float64)

// 或者 
myChannel := make(chan float64)

// send to a channel
myChannel <- 3.13

// receive from a channel
<- myChannel
```

**send操作阻塞send协程，receive操作阻塞receive协程**

+ channel使用struct value

```go
c := make(chan MyStruct)
```

# 测试

```go
go test github.com/xx/xx
```

xxx_test.go
写法convention
{% asset_img f0408-01.png %}
详细信息
{% asset_img f0410-02.png %}

# web apps -- http库

# 其他

```go
// initial statement of if
if x, err:=save(); err != nil {...}

// switch不需要break，命中一个就返回了
switch x {
    case 1:
        ...
    case 2:
        ...
    default:
        ...
}

// 其他类型
int8 int16 int32 int64 // 默认最好使用int
uint
uint8 uint16 ...
float32

// rune 类型是一个unicode字符 rune占不定长bytes（俄文2bytes，中文3bytes），ascII占1bytes
len("世界") == 4 // len函数返回的是byte数量
utf8.RuneCountInString("世界") == 2 //使用这个函数可以返回正确的字符长度
utf8.RuneCountInString("ab") == 2

bytes := ([]byte("世界你好吗"))[2:]
stirng(bytes) //乱码
bytes := ([]rune("世界你好吗"))[2:] // 用rune
stirng(bytes) //正常

// buffered channels

channel := make(chan int, 3)

```