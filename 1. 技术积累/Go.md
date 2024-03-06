# Go

两种不同的定义变量方式，特点是`:=`只能用来定义变量，变量赋值时必须使用`=`

```Go
//方法一
var a int
var b []int
//方法二
a := 2
b := []int
```

定义struct

```go
type vertex struct{
	x, y int
}
```

slice 即切片，与python中lis切片操作类似，当定义array时，如果不定义长度，即`[]`，则此声明将定义slice而非array

```Go
//创建 slice 方法一:不指定count
letters := []string{"a", "b", "c", "d"}
//创建 slice 方法二:make function
func make([]T, len, cap) []T
a := make([]int, 5)  // len(a)=5
b := make([]int, 0, 5) // len(b)=0, cap(b)=5
```

```go
board := [][]string{
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
		[]string{"_", "_", "_"},
	}
```

func value

```go

import (
	"fmt"
	"math"
)

func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```

函数修饰器

```go

import "fmt"

func adder() func(int) int{
	sum := 0
	return func(x int) int{
		sum += x;
		return sum;
	}
}

func main(){
	pos, neg := adder(), adder()
	for i := 1; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2 * i),
		)
	}
}
```

```go

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```



go取消了class的概念，而是采用method方式将 func绑定到type上，实际上type的定义就如同class的定义，method就是成员函数

```go

import (
	"fmt"
	"math"
)

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}
```

这样有一些好处，比如可以将已有数据类型重新type，之后绑定func到type上，可以方便的扩展type的功能

```go

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}
/* 下面这样写只用到了func而未采用method概念
func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func Scale(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	Scale(&v, 10)
	fmt.Println(Abs(v))
}
*/
//使用func时是严格区分指针的，而只用methods时，二者皆可
```

## Interface 

Interface is a kind of type, which contains a set of methods that don't have specific implementation

```go

import (
	"fmt"
	"math"
)

type Abser interface {
	Abs() float64
}

func main() {
	var a Abser
	f := MyFloat(-math.Sqrt2)
	v := Vertex{3, 4}

	a = f  // a MyFloat implements Abser
	a = &v // a *Vertex implements Abser

	// In the following line, v is a Vertex (not *Vertex)
	// and does NOT implement Abser.
	// a = v will cause an error

	fmt.Println(a.Abs())
}

type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

type Vertex struct {
	X, Y float64
}

func (v *Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

先定义一个interface， 再定义多种type struct，对每一种struct type给出接口中method名称的具体实现，之后便可以定义接口并赋值type，接口便可以执行type中的具体定义

```go

import "fmt"

type I interface {
	M()
}

type T struct {
	S string
}

// This method means type T implements the interface I,
// but we don't need to explicitly declare that it does so.
func (t T) M() {
	fmt.Println(t.S)
}

func main() {
	var i I = T{"hello"}
	i.M()
}
```

这样便实现了由接口调用方法



当interface被赋值一个类型空指针时，经过method绑定后method仍然可以被调用，但是如果interface本身是空，那调用method时会导致run-time error

```go
func describe(i interface{}) {
    fmt.Printf("(%v, %T)", i, i)
}
```

***Type Assertion*** :interface可以直接用type定义，并且可以隐式地将type的值读取出来

```go
func main(){
    // 定义interface时后面的{}也是可选的
    var i interface{} = "hello"
    s := i.(string)
    fmt.Println(s)
    s, ok := i.(string)
    s, ok := i.(float) // return a false
    s := i.(float)  // cause a panic
}
```

Type switch:

```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

fmt 在Print时 是调用了相应对象的String接口

```go
package main

import "fmt"

type Person struct{
    Name string
    Age int
}
// func后面接的函数返回类型是可选的，前面接的绑定的type也是可选的
func (p Person)String() string{
    return fmt.Sprintf("%v (%v years)", p.Name, p.Age) 
    //注意Sprintf返回的是字符串 而Printf返回的是have (int, error)want (string)
}
func main(){
    a := Person{"Alice", 23}
    b := Person{"Bob", 21}
    fmt.println(a, b)
}
```

fmt内置的两个interface

```go
type error interface{
    Error() string
}
type Stringer interface {
    String() string
}
// 二者都可以将某一对象实现该接口 来实现某些功能
func (p Person)String() string{
    return fmt.Sprintf()
}
func (e *Myerror)Error() string{
    return fmt.Sprintf()  
}
```

关于自定义error错误消息提示：

```go
package main

import "fmt"
import "time"

type MyError struct{
	Now time.Time
	err string
}

func (e *MyError)Error() string{
	return fmt.Sprintf("Now is %v, %v", e.Now, e.err)
}

func run() error{
    // 这里关键要理解 为什么创建了MyError能作为一个error类型值返回
    // MyError本身只是一个结构体但由于该结构体实现了error接口中的Error方法，于是MyError可以被视为error
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main(){
	if err := run; err!=nil{
		fmt.Println(err)
	}
}
```

```go
package main

import (
	"fmt"
	"math"
)

type ErrNegativeSqrt float64

func Sqrt(f float64) (float64, error) {
	if f < 0 {
		return 0, ErrNegativeSqrt(f) //return value will be printed
	}
	return math.Sqrt(f), nil
}
func (e ErrNegativeSqrt)Error()string{
	if e < 0{
		return fmt.Sprintf("cannot Sqrt negative number: %v", float64(e))
	}
	return ""
}

func main() {
	fmt.Println(Sqrt(2))
	fmt.Println(Sqrt(-2))
}
```

关于Reader：

io package 中实现了具体的io.Reader interface

go的许多标准库中都实现了该接口，如files, network connections, compressions, ciphers 

io.Reader interface有一个Read方法：

```go
func (T) Read(b []byte) (n int, err error)
```

类型T可以通过实现Read方法拥有读取字符串的功能，读取后返回两个值 n和error

```go
package main

import (
	"fmt"
	"io"
	"strings"
)
// 先定义了一个strings.reader r，并输入字符串"Hello, Reader!"
// 之后创建了一个空的byte数组，并通过r.Read方法将字符串读取到b中
func main() {
	r := strings.NewReader("Hello, Reader!")
	fmt.Printf("%v, %T\n", r, r)
	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

```go
package main

import "golang.org/x/tour/reader"

type MyReader struct{}

// TODO: Add a Read([]byte) (int, error) method to MyReader.
// 上面的示例代码是通过strings中已经实现的Reader实现字符串读取
// 这里我们定义了一个结构体 MyReader，并实现该结构体的Read接口
// 这样MyReader的类型就成了reader
func (myread MyReader) Read (b []byte) (int, error){
	for x := range b{
		b[x] = 'A'
	}
	return len(b), nil
} 

func main() {
	reader.Validate(MyReader{})
}
```

定义模板函数

```go
func Index[T comparable](s []T, x T) int
```

```go
package main

import "fmt"

// Index returns the index of x in s, or -1 if not found.
func Index[T comparable](s []T, x T) int {
	for i, v := range s {
		// v and x are type T, which has the comparable
		// constraint, so we can use == here.
		if v == x {
			return i
		}
	}
	return -1
}

func main() {
	// Index works on a slice of ints
	si := []int{10, 20, 15, -10}
	fmt.Println(Index(si, 15))

	// Index also works on a slice of strings
	ss := []string{"foo", "bar", "baz"}
	fmt.Println(Index(ss, "hello"))
}

```

除了maps , slices还有 channel这个数据结构

`ch := make(chan int)`

通道可以在变量间传输数据，主要作用还是实现线程同步

```go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
```

channel可以由sender关闭

```go

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

也可以通过select语句选择channel

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
            case c <- x: // 当 c 通道中有数据可以被接收
                x, y = y, x+y
            case <-quit: // 当quit通道中接收到数据
                fmt.Println("quit")
                return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```

Time中对通道定义如下：

```go
func Tick(d Duration) <-chan Time
//d 是周期性时间间隔。
//返回值是一个只读通道 (<-chan Time)。
```

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	tick := time.Tick(100 * time.Millisecond)
	boom := time.After(500 * time.Millisecond)
	for {
		select { // 当下面三个case语句都满足条件时，随机执行，
		case <-tick:
			fmt.Println("tick.")
		case <-boom:
			fmt.Println("BOOM!")
			return
		default:
			fmt.Println("    .")
			time.Sleep(50 * time.Millisecond)
		}
	}
}
```

判断两棵二叉树的序列是否相同，用其他语言相当复杂

go中有 package tree

```go
type Tree struct {
    Left  *Tree
    Value int
    Right *Tree
}
```

go自带了互斥锁的实现，在使用多线程时，即用go启动多个线程时可以避免一些冲突

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

type SafeCounter struct{
	mu sync.Mutex
	v map[string]int
}

func (c *SafeCounter)Inc(key string){
	c.mu.Lock()
	c.v[key]++
	c.mu.Unlock()
}

func (c *SafeCounter)Value(key string)int{
	c.mu.Lock()
	defer c.mu.Unlock()
	return c.v[key]
}
func main(){
	c := SafeCounter{v:make(map[string]int)}
	for i := 0; i < 1000; i++ {
		go c.Inc("hello")
	}
	time.Sleep(time.Second)
	fmt.Println(c.Value("hello"))
}
```

