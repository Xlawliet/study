# Go

## 基本命令

### `go build`

编译写好的.go文件，使之成为可以执行的exe文件

### go install

包含了两步

1. 通过 go build 将文件进行编译
2. 将编译后的文件放入gopath下的bin目录下



## 交叉编译

Go支持跨平台编译

在win平台编译一个能在linux平台执行的文件

```cmd
SET CGO_ENABLED = 0 #禁用CGO
SET GOOS=linux #目标平台是linux
SET GOARCH=amd64 #目标处理器架构是amd64
```



由包构成 import "xxx"

包中导出的变量都是**首字母大写**



函数

接受没有参数或多个参数

参数类型相同的情况下，只需要对最后一个变量进行声明如：

a,b,c,d int,s1,s2 string



可以拥有任意个返回值

func fun1(a,b int,s1,s2 string)(int, string){}



返回值可以被命名，会被视作定义在函数顶部的变量

没有参数的return语句返回已命名的返回值，但这样的写法会影响代码可读性，只允许在短函数中使用。如：

func split(sum int) (x,y int){x = sum * 4 /9   y=sum-x  return*}



## 变量

 基本变量类型：bool,int,float32,float64,string,uint,byte=uint8,rune=int32

complex64,complex128



## 声明变量

var 用于声明变量列表，类型在最后

var a,b,c,d int

声明变量可以包含初始值，不赋值则是默认值

int = 0  bool = false string = ""

在函数中声明变量可以使用简洁声明 :=



## 类型转换

T(v) 如：var i int = 32     var f float32 = float32(i)



## 常量声明 const



## 循环for

for i:= 0;i<10;i++{ sum+=i }

初始化语句和后置语句是可选的

for ; i<10 ;{ i++ }

for去掉分号后变成while

for  i<1000 {i+=i}

无限循环

for {/*代码*/}



## 条件语句

#### if - else

无需小括号，大括号必须

## switch

从上至下依次匹配case的值，匹配成功时停止。

Go自动提供了case中的break语句，分支会自动终止，除非以fallthrought语句结束。

case 无需为常量，取值不必为整数。

## **没有条件的swtich**

没有条件的swtich同 switch true一样，能将一长串if-then-else写的清晰

如：

```go
import (
	"fmt"
	"time"
)

func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}
```



## defer

该语句会将函数推迟到外层函数返回之后执行

推迟的函数调用会被压入一个栈中，被推迟的函数会按照后进先出的顺序调用。

```go
import "fmt"

func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}
```



## 指针

指针保存了值的内存地址，因为Go是值传递，所以要引用传递时需要使用指针。（间接引用）

*T是指向T类型的指针，其零值为nil



## 结构体

`type xxx struct{}`

```go
type Vertex struct{
		X int
    Y int
}
func main(){
  fmt.Println(Vertex(1,2))
}
```

其中的字段使用点号来访问

​	`v.X`

## 结构体文法

当没有对结构体中的某一字段赋值时，该字段会取变量类型的默认值



## 数组

var a []string  这个数字的大小是固定的不能改变

## 切片

数字的大小是固定的，切片为元素提供动态的、灵活的视角

```go
primes := [6]int{2,3,4,5,6,7}
var s []int = primes[1:4] //s包含primis中下标1到3点元素
```

切片中元素的更改同时也会修改原数组上的元素，相当于数组的引用

切片文法类似没有长度的数组文法

```go
[3]bool{true,true,false}

[]bool{true,true,false} //这样的创建方式相当于创建了上面数组的切片
```

```go
a[0:10]
a[:10]
a[0:]
a[:]
//对于数组而言，上面的切片都是等价的
```

切片拥有**长度len()**和**容量cap()**

长度为创建切片时的下标与上标之差

容量是创建切片时，数组长度与切片上标之差



## make

使用make语句来创建切片，同时也是创建动态数组的方式

`a:=make`

```go
	a := make([]int,5)
	fmt.Println(len(a),cap(a),a) //5,5 [0,0,0,0,0]

	b := make([]int,0,5)
	fmt.Println(len(b),cap(b),b) //0,5 []  为什么此时len为0？？？
```

## append

当需要向切片中添加新的元素时所使用

当切片底层的数组大小不足以继续添加元素时，会创建一个更大的新的数组，将切片指向这个新的数组



## 遍历切片

for循环中使用range遍历切片或映射

遍历切片时，每次迭代都会返回两个值，第一个值为元素的下标，第二个值为对应元素的副本

```go
for i,v:=range pow{
	fmt.Println(i,v)
}
```

可以将下标或值赋予_来忽略它

`for i,_:=range pow`   或 `for _,i:=range pow`

只需要索引时忽略第二个变量即可



## 映射

零值为nil

通过make创建和初始化

映射的文法与结构体相似，但是必须有键名

```
var m = map[string]Value{
	"a":Value,
	"b":value,
}
```

修改映射

```go
func main(){
  m := make(map[string]int) //键为string 值为int
  
  m["Answer"] = 42
  fmt.Println("The value:", m["Answer"])
  
  //修改值
  m["Answet"] = 48
  fmt.Println("The value:", m["Answer"])
  
  //删除元素
  delete(m,"Answer")
  fmt.Println("The value:", m["Answer"])
  
  //通过双赋值检测某个键是否存在
  elem,ok = m[key]
  //若键存在 ok为true,否则为false
}
```

练习

```go
func WordCount(s string) map[string]int {

	res := make(map[string]int)
	words := strings.Fields(s)
	
	for _,s:=range words{
		//res[s]+=1
		elem,ok := res[s]
		if ok {
			res[s] = elem+1
		}else{
			res[s] = 1
    }
	}
	return res
	//return map[string]int{"x": 1}
}
```



## 函数值

函数值也是值，可以作为参数或者返回值

```go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))
  //fmt.Println(hypot)  //此行会报错
	fmt.Println(compute(hypot))     //
	fmt.Println(compute(math.Pow))
}
```



## 函数的闭包

Go函数可以是一个闭包，闭包是一个函数值，引用函数体之外的变量。该函数可以访问并赋予其引用的变量的值，函数与这些变量绑定在一起

```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

//其中pos中绑定着自己的sum，neg也绑定着自己的sum
//todo 但是每次运行adder时都给sum重新赋值了啊？
func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
} 
```

定义函数作为返回值时，返回的函数只能被定义为匿名函数

## panic和recover

由于在老版本的go中没有异常机制，只有使用panic/recover模式来处理错误

```go
func funcA() {
	fmt.Println("func A")
}
func funcB() {
	defer func() {
		err := recover()
		//如果程序出出现了panic错误,可以通过recover恢复过来
		if err != nil {
			fmt.Println("recover in B")
		}
	}()
	panic("panic in B")
}
func funcC() {
	fmt.Println("func C")
}
func main() {
	funcA()
	funcB()
	funcC()
}
```

1. `recover()`必须搭配`defer`使用。
2. `defer`一定要在可能引发`panic`的语句之前定义。

## 方法

