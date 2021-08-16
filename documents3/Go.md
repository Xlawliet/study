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



## 函数

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

在函数中声明变量可以使用简洁声明 :=    这是Go的语法糖



如果使用变量的短声明，还可以在后面对其重新声明，但值的类型必须与之前声明时的一样。



### 重声明

前提是类型在其初始化时已经确定。只有在使用短变量声明时才会发生，否则无法通过编译。被“声明并赋值”的变量必须时多个，并且其中有一个是新的变量，这时才可以说对其中的旧变量进行了重声明。

```go
var err error
n, err := io.WriteString(os.Stdout, "Hello, everyone!\n")
```

此处使用了短变量声明对新变量n和旧变量err进行了“声明并赋值”，也是对后者的重声明。



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

### 构造函数

Go的结构体没有构造函数，但可以自己实现

```go
type person struct {
	name, city string
	age int8
}
func newPerson(name, city string, age int8) *person {
	return &person{
		name: name,
		city: city,
		age:  age,
	}
}
func main() {
	p9:=newPerson("xzy","banna",18)
	fmt.Println(p9)
}
```



## 数组

var a [size]string  数组的大小是固定的不能改变

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
	fmt.Println(len(b),cap(b),b) //0,5 []  为什么此时len为0？  //因为此时b只是开辟了内存空间，但还没有元素
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
//todo 但是每次运行adder时都给sum重新赋值了啊？ //猜测在后续调用时，函数会直接go to到return的部分，而不会再次对sum进行初始化赋值
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

## 分配空间

### new

`func new(type) *Type`

Type表示类型，new函数只接受一个参数

*Type表示类型指针，new函数返回一个指向该类型内存地址的指针

```go
func main(){
  a := new(int)  //并赋予默认值0
  b := new(bool) //默认值false
}
```

### make

也可以用于内存分配，区别于new，make只用于slice、map、chan的内存创建

## 方法和接收者

方法（Method）是一种作用于特定类型变量的函数。这种特定类型变量叫做`接收者（Receiver）`。接收者的概念类似于java中的this。

```go
func (接收者变量 接收者类型) 方法名（参数列表）（返回参数）{
  函数体
}
```

- 接收者变量一般用类型名称首字母小写
- 接收者类型可以为指针或非指针
- 方法名、参数列表、返回值与函数定义相同

```go
type person struct {
	name, city string
	age int8
}
func newPerson(name, city string, age int8) *person {
	return &person{
		name: name,
		city: city,
		age:  age,
	}
}
func (p person) Dream() {
	fmt.Printf("%s在做梦",p.name)
}
func main() {
	p9:=newPerson("a","b",18)
	p10:=newPerson("c","d",20)
	p9.Dream()
	p10.Dream()
}
```

函数与方法的区别在于，函数不属于任何类型，方法属于特定类型。

### 指针类型的接收者

接收者由一个结构体的指针组成，由于指针的特性，调用方法修改指针的任意成员变量时，方法结束后修改依然有效。

```go
// SetAge 设置p的年龄
// 使用指针接收者
func (p *Person) SetAge(newAge int8) {
	p.age = newAge
}
func main() {
	p1 := NewPerson("小王子", 25)
	fmt.Println(p1.age) // 25
	p1.SetAge(30)
	fmt.Println(p1.age) // 30
}
```

### 值类型的接收者

与指针类型的接受者相比，方法结束后无法保存修改后的值，修改操作只针对副本。

## 结构体的匿名字段

结构体允许在声明时没有字段名只有类型，但要求字段名称必须唯一。

```go
//Person 结构体Person类型
type Person struct {
	string
	int
}
func main() {
	p1 := Person{
		"小王子",
		18,
	}
	fmt.Printf("%#v\n", p1)        //main.Person{string:"北京", int:18}
	fmt.Println(p1.string, p1.int) //北京 18
}
```

## 嵌套结构体

结构体内允许嵌套结构体，嵌套体内部存在相同字段名时，为避免歧义需要通过指定具体的内嵌结构体字段名。如：

```go
//Address 地址结构体
type Address struct {
	Province   string
	City       string
	CreateTime string
}
//Email 邮箱结构体
type Email struct {
	Account    string
	CreateTime string
}
//User 用户结构体
type User struct {
	Name   string
	Gender string
	Address
	Email
}
func main() {
	var user3 User
	user3.Name = "沙河娜扎"
	user3.Gender = "男"
	// user3.CreateTime = "2019" //ambiguous selector user3.CreateTime
	user3.Address.CreateTime = "2000" //指定Address结构体中的CreateTime
	user3.Email.CreateTime = "2000"   //指定Email结构体中的CreateTime
}
```

## 结构体可见性

结构体中字段大写开头表示可公开访问，小写表示私有（仅在定义当前结构体的包中可访问,其他包不能访问）。

## 结构体的继承

```go
//Animal 动物
type Animal struct {
	name string
}
func (a *Animal) move() {
	fmt.Printf("%s会动！\n", a.name)
}
//Dog 狗
type Dog struct {
	Feet    int8
	*Animal //通过嵌套匿名结构体实现继承
}
type Cat struct{
	Feet int8
	*Animal
}
func (d *Dog) wang() {
	fmt.Printf("%s会汪汪汪~\n", d.name)
}
func main() {
	d1 := &Dog{
		Feet: 4,
		Animal: &Animal{ //注意嵌套的是结构体指针
			name: "乐乐",
		},
	}
	d1.wang() //乐乐会汪汪汪~
	d1.move() //乐乐会动！
	c1 :=&Cat{
		4,
		&Animal{name: "喵喵"},
	}
	c1.move()
}
```

## 结构体的Json序列化

```go
//Student 学生
type Student struct {
	ID     int
	Gender string
	name   string
}
//Class 班级
type Class struct {
	Title    string
	Students []*Student
}
func main() {
	c := &Class{
		Title:    "101",
		Students: make([]*Student, 0, 200),
	}
	for i := 0; i < 10; i++ {
		stu := &Student{
			name:   fmt.Sprintf("stu%02d", i),
			Gender: "男",
			ID:     i,
		}
		c.Students = append(c.Students, stu)
	}
	//JSON序列化：结构体-->JSON格式的字符串
	data, err := json.Marshal(c)
	if err != nil {
		fmt.Println("json marshal failed")
		return
	}
	fmt.Printf("json:%s\n", data)
	//JSON反序列化：JSON格式的字符串-->结构体
	str := `{"Title":"101","Students":[{"ID":0,"Gender":"男","Name":"stu00"},{"ID":1,"Gender":"男","Name":"stu01"},{"ID":2,"Gender":"男","Name":"stu02"},{"ID":3,"Gender":"男","Name":"stu03"},{"ID":4,"Gender":"男","Name":"stu04"},{"ID":5,"Gender":"男","Name":"stu05"},{"ID":6,"Gender":"男","Name":"stu06"},{"ID":7,"Gender":"男","Name":"stu07"},{"ID":8,"Gender":"男","Name":"stu08"},{"ID":9,"Gender":"男","Name":"stu09"}]}`
	c1 := &Class{}
	err = json.Unmarshal([]byte(str), c1)
	if err != nil {
		fmt.Println("json unmarshal failed!")
		return
	}
	fmt.Printf("%#v\n", c1)
}
```

## 结构体注意点

因为slice和map两种数据类型都包含了指向底层数据的指针，所以需要赋值它们或修改时都需要特别注意。如：

```go

type Person struct {
	name   string
	age    int8
	dreams []string
}

func (p *Person) SetDreams(dreams []string) {
//p.dreams = dreams  //不合适，会修改底层数组data的值
	p.dreams = make([]string, len(dreams))
	copy(p.dreams, dreams)
}

func main() {
	p1 := Person{name: "小王子", age: 18}
	data := []string{"吃饭", "睡觉", "打豆豆"}
	p1.SetDreams(data)
	fmt.Println(p1)
	// 你真的想要修改 p1.dreams 吗
	p1.dreams[1] = "不睡觉"
	//data[1] = "不睡觉"
	fmt.Println(data)
	fmt.Println(p1.dreams)  // 
}
```



## 基础包

包中的init()函数会被程序自动调用，不能自己调用。



## 接口

interface是一种类型，一种抽象的类型

```go
type 接口名称 interface{}
```

只要实现了接口中的方法，就实现了这个接口。如：

```go
type Sayer interface{
  say()	
}
type dog struct{}
type cat struct{}
func (d dog) say(){
  fmt.Println("狗叫")
}
func (c cat) say(){
  fmt.Println("猫叫")
}
func main(){
  var x sayer
  a := cat{}
  b := dot{}
  x = a
  x.say() //猫叫
  x = b
  x.say() //狗叫
}
```

## 值接收者实现接口

```go
type Mover interface {
	move()
}
type dog struct {}
func (d dog) move() {
	fmt.Println("狗会动")
}
func main() {
	var x Mover
	var wangcai = dog{} // 旺财是dog类型
	x = wangcai         // x可以接收dog类型
	var fugui = &dog{}  // 富贵是*dog类型
	x = fugui           // x可以接收*dog类型
	x.move()
}
```

值接收者实现接口后，无论是dog结构体还是结构体指针*dog都可以复制给接口变量，这是Go的语法糖

## 指针接收者实现接口

```go
func (d *dog) move() {
	fmt.Println("狗会动")
}
func main() {
	var x Mover
	var wangcai = dog{} // 旺财是dog类型
	x = wangcai         // x不可以接收dog类型
	var fugui = &dog{}  // 富贵是*dog类型
	x = fugui           // x可以接收*dog类型
}
```

此时只能传递指针类型变量

## 空接口

空接口中没有任何方法的接口，因为任何类型都实现了空接口。

空接口可以存储任意类型的变量。

```go
func main() {
	// 定义一个空接口x
	var x interface{}
	s := "Hello 沙河"
	x = s
	fmt.Printf("type:%T value:%v\n", x, x)
	i := 100
	x = i
	fmt.Printf("type:%T value:%v\n", x, x)
	b := true
	x = b
	fmt.Printf("type:%T value:%v\n", x, x)
}
```

### 空接口的应用

#### 接受任意类型的函数参数

```go
func show(a interface{}){
	fmt.Printf("type:%T,value:%v\n",a,a)
}
```

#### 空接口作为map的值

可以保存任意值的字典

```go
var studentInfo = make(map[string]interface{})
studentInfo["name"] = "abs"
studentInfo["age"] = 18
studentInfo["married"] = false
fmt.Println(studentInfo)
```

## 类型断言

空接口可以存储任意类型的值，如何获取具体的值

### ==接口值==

一个接口的值由 `一个具体类型`和`具体类型的值`两部分组成。分别称为`动态类型`和`动态值`。

`x.(T)` 断言语法格式

```go
	var x interface{}
	x = "123"
	v,ok := x.(int)
	if ok{
		fmt.Println(v)
	}else{
		fmt.Println("错误断言")
	}
```

断言多次就需要多个`if`判断

```go
func justifyType(x interface{}) {
	switch v := x.(type) {
	case string:
		fmt.Printf("x is a string，value is %v\n", v)
	case int:
		fmt.Printf("x is a int is %v\n", v)
	case bool:
		fmt.Printf("x is a bool is %v\n", v)
	default:
		fmt.Println("unsupport type！")
	}
}
```



## 反射

Go的变量分为两部分

- 类型信息：预先定义好的原信息
- 值信息：程序运行过程中可动态变化的

reflect包支持在程序编译期获取到变量的字段名称、类型信息、结构体信息等，并有能力修改它们。

## reflect包

Go中内置的reflect包提供了`reflect.TypeOf`和`reflect.ValueOf`两个函数来获取任意对象的Value和Type。

### TypeOf()

使用reflect.TypeOf()函数可以获得任意值的类型对象，程序通过类型对象可以访问任意值的类型信息。

```go
func reflectType(x interface{}){
	v := reflect.TypeOf(x)
	fmt.Printf("type is %v\n",v)
}
func main() {
	var a float32 = 3.14
	reflectType(a)  //type is float32
	var b float64 = 10000
	reflectType(b)  //type is float64
}
```

### ValueOf

reflect.ValueOf() 返回的是 `reflect.value` 类型，其中包含了原始值的信息。`reflect.Value`与原始值之间可以互相转换。

通过反射获取值

```go
func reflectValue(x interface{}) {
	v := reflect.ValueOf(x)
	k := v.Kind()
	switch k {
	case reflect.Int64:
		// v.Int()从反射中获取整型的原始值，然后通过int64()强制类型转换
		fmt.Printf("type is int64, value is %d\n", int64(v.Int()))
	case reflect.Float32:
		// v.Float()从反射中获取浮点型的原始值，然后通过float32()强制类型转换
		fmt.Printf("type is float32, value is %f\n", float32(v.Float()))
	case reflect.Float64:
		// v.Float()从反射中获取浮点型的原始值，然后通过float64()强制类型转换
		fmt.Printf("type is float64, value is %f\n", float64(v.Float()))
	}
}
func main() {
	var a float32 = 3.14
	var b int64 = 100
	reflectValue(a) // type is float32, value is 3.140000
	reflectValue(b) // type is int64, value is 100
	// 将int类型的原始值转换为reflect.Value类型
	c := reflect.ValueOf(10)
	fmt.Printf("type c :%T\n", c) // type c :reflect.Value
}
```

### 通过反射设置变量的值

想要在函数中通过反射修改变量的值，需要注意函数参数传递的是值拷贝，必须传递变量地址才能修改变量值。而反射中使用专有的`Elem()`方法来获取指针对应的值。

```go
package main

import (
	"fmt"
	"reflect"
)

func reflectSetValue1(x interface{}) {
	v := reflect.ValueOf(x)  
	if v.Kind() == reflect.Int64 {
		v.SetInt(200) //修改的是副本，reflect包会引发panic
	}
}
func reflectSetValue2(x interface{}) {
	v := reflect.ValueOf(x)
	// 反射中使用 Elem()方法获取指针对应的值
  // v的kind（） 为 ptr 类型
	if v.Elem().Kind() == reflect.Int64 {
		v.Elem().SetInt(200)
	}
}
func main() {
	var a int64 = 100
	// reflectSetValue1(a) //panic: reflect: reflect.Value.SetInt using unaddressable value
	reflectSetValue2(&a)
	fmt.Println(a)
}
```



## isNil()和isValid()

### isNil()

`func (v Value) IsNil() bool`

v持有的值必须是通道、函数、接口、映射、指针、切片之一；否则IsNil函数会导致panic。

### isValid()

`func (v value) IsValid() bool`

返回v是否持有一个值。如果v是Value零值会返回假，此时v除了IsValid、String、Kind之外的方法都会导致panic。

```go
func main() {
	var a *int
	fmt.Println("var a *int IsNil:", reflect.ValueOf(a).IsNil())
	// nil值
	fmt.Println("nil IsValid:", reflect.ValueOf(nil).IsValid())
	// 实例化一个匿名结构体
	b := struct{}{}
	// 尝试从结构体中查找"abc"字段
	fmt.Println("不存在的结构体成员:", reflect.ValueOf(b).FieldByName("abc").IsValid())
	// 尝试从结构体中查找"abc"方法
	fmt.Println("不存在的结构体方法:", reflect.ValueOf(b).MethodByName("abc").IsValid())
	// map
	c := map[string]int{}
	// 尝试从map中查找一个不存在的键
	fmt.Println("map中不存在的键：", reflect.ValueOf(c).MapIndex(reflect.ValueOf("娜扎")).IsValid())
}
```

## 结构体反射

通过reflect.TypeOf()获得反射对象信息之后，如果是结构体，可以通过反射对象Field()等方法获取结构体成员的详细信息。

其中Field()方法会返回一个StructField()来描述其中的信息。

```go
type StructField struct {
    // Name是字段的名字。PkgPath是非导出字段的包路径，对导出字段该字段为""。
    // 参见http://golang.org/ref/spec#Uniqueness_of_identifiers
    Name    string
    PkgPath string
    Type      Type      // 字段的类型
    Tag       StructTag // 字段的标签
    Offset    uintptr   // 字段在结构体中的字节偏移量
    Index     []int     // 用于Type.FieldByIndex时的索引切片
    Anonymous bool      // 是否匿名字段
}
```

实例：

```go
type student struct {
	Name  string `json:"name"`
	Score int    `json:"score"`
}
func main() {
	stu1 := student{
		Name:  "小王子",
		Score: 90,
	}
	t := reflect.TypeOf(stu1)
	fmt.Println(t.Name(), t.Kind()) // student struct
	// 通过for循环遍历结构体的所有字段信息
	for i := 0; i < t.NumField(); i++ {
		field := t.Field(i)
		fmt.Printf("name:%s index:%d type:%v json tag:%v\n", field.Name, field.Index, field.Type, field.Tag.Get("json"))
	}
	// 通过字段名获取指定结构体字段信息
	if scoreField, ok := t.FieldByName("Score"); ok {
		fmt.Printf("name:%s index:%d type:%v json tag:%v\n", scoreField.Name, scoreField.Index, scoreField.Type, scoreField.Tag.Get("json"))
	}
}
```

## 并发与并行

Go天生支持并发。

Go的并发通过`goroutine` 实现。`goroutine`类似于线程，属于用户态的线程，可以根据需要创建成千上万个`goroutine`并发工作。`goroutine`是由Go的运行时(runtime)调度完成，而线程是由操作系统和CPU调度完成。

Go害提供`channel`在多个`goroutine`间进行通信。`goroutine`和`channel`是Go语言秉承的CSP并发模式的重要实现基础。

### goroutine

在调用函数时在前面加上`go`关键字，就可以为函数创建一个`goroutine`。

```go
func hello(){
	fmt.Println("hello！！！！")
}
func main() {
	go hello()
	fmt.Println("done!")
	time.Sleep(time.Second)
}
```

此时如果main()所在的`goroutine`先结束了，那么hello可能还未被调用就被结束了，所以加上sleep。

之所以会先完成done，是因为创建新的`goroutine`时需要花费一些时间。

#### 启动多个`goroutine`

```go
var wg sync.WaitGroup //类似java 中的 countdownLatch,为0时才能继续往下

func hello(i int) {
	defer wg.Done() // goroutine结束就登记-1
	fmt.Println("Hello Goroutine!", i)
}
func main() {
	for i := 0; i < 10; i++ {
		wg.Add(1) // 启动一个goroutine就登记+1
		go hello(i)
	}
	wg.Wait() // 等待所有登记的goroutine都结束
}
```

#### 可增长的栈

OS线程（操作系统线程）一般都有固定的栈内存（通常为2MB）,一个`goroutine`的栈在其生命周期开始时只有很小的栈（典型情况下2KB），`goroutine`的栈不是固定的，他可以按需增大和缩小，`goroutine`的栈大小限制可以达到1GB，虽然极少会用到这么大。所以在Go语言中一次创建十万左右的`goroutine`也是可以的。

Go语言中的操作系统线程和goroutine的关系：

1. 一个操作系统线程对应用户态多个goroutine。
2. go程序可以同时使用多个操作系统线程。
3. goroutine和OS线程是多对多的关系，即m:n。

### channel

单纯地将函数并发执行是没有意义的。函数与函数间需要交换数据才能体现并发执行函数的意义。

虽然可以使用共享内存进行数据交换，但是共享内存在不同的`goroutine`中容易发生竞态问题。为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题。

Go语言的并发模型是`CSP（Communicating Sequential Processes）`，提倡**通过通信共享内存**而不是**通过共享内存而实现通信**。

如果说`goroutine`是Go程序并发的执行体，`channel`就是它们之间的连接。`channel`是可以让一个`goroutine`发送特定值到另一个`goroutine`的通信机制。

Go 语言中的通道（channel）是一种特殊的类型。通道像一个传送带或者队列，总是遵循先入先出（First In First Out）的规则，保证收发数据的顺序。每一个通道都是一个具体类型的导管，也就是声明channel的时候需要为其指定元素类型。



### channel类型

是一种类型，一种引用类型。

`var 变量名 chan 元素类型`

### channel创建

创建channel，通道是引用类型，其空值是nil。

`var ch chan int` 

声明之后需要使用`make`进行初始化后才能使用。

`make(chan T,[buffer size])`

其中缓冲大小是可以选的。

### channal操作

通道有发送(send)、接收(receive)和关闭(close)三种操作。

发送和接收都通过 `<-` 符号。

关闭使用内置的 `close`函数来关闭通道。



关于关闭通道需要注意的事情是，只有在通知接收方goroutine所有的数据都发送完毕的时候才需要关闭通道。通道是可以被垃圾回收机制回收的，它和关闭文件是不一样的，在结束操作之后关闭文件是必须要做的，但关闭通道不是必须的。

关闭后的通道有以下特点：

1. 对一个关闭的通道再发送值就会导致panic。
2. 对一个关闭的通道进行接收会一直获取值直到通道为空。
3. 对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值。
4. 关闭一个已经关闭的通道会导致panic。



### 无缓冲通道

```go
func main(){
  ch := make(chan int)
  ch <- 10
  fmt.Println()
}
```

使用`ch := make(chan int)`创建的是无缓冲的通道，无缓冲的通道只有在有人接收值的时候才能发送值。否则会形成==死锁deadlock==。

无缓冲更多情况下，必须有一个接收者，才能通过chan传输数据

```go
func recv(c chan int) {
	ret := <-c
	fmt.Println("接收成功",ret)
}
func main() {
	ch := make(chan int)
	go recv(ch)  //创建一个goroutine 并创建一个接收者ret
	ch <- 10     //此时传递数据
	time.Sleep(time.Second)
}
```

### 有缓冲通道

在创建通道时初始化其容量。

```go
func main() {
	ch := make(chan int,1)
	ch <- 10
	ret := <- ch
	fmt.Println(ret)
	ch <- 20
	ret = <- ch
	fmt.Println(ret)
}
```

如果通道中容量不足但仍然继续传输数据，就会出现**deadlock**。

### for range从通道循环取值  (有疑问，怎么保证ch2能取到值)

```go
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	// 开启goroutine将0~100的数发送到ch1中
	go func() {
		for i := 0; i < 100; i++ {
			ch1 <- i
		}
		close(ch1)
	}()
	// 开启goroutine从ch1中接收值，并将该值的平方发送到ch2中
	go func() {
		for {
			i, ok := <-ch1 // 通道关闭后再取值ok=false
			if !ok {
				break
			}
			ch2 <- i * i
		}
		close(ch2)
	}()
	// 在主goroutine中从ch2中接收值打印
	for i := range ch2 { // 通道关闭后会退出for range循环
		fmt.Println(i)
	}
}
```

### 单项通道

限制通道只能接受数据或发送数据

```go
func counter(out chan<- int) {
	for i := 0; i < 100; i++ {
		out <- i
	}
	close(out)
}
func squarer(out chan<- int, in <-chan int) {
	for i := range in {
		out <- i * i
	}
	close(out)
}
func printer(in <-chan int){
	for i:= range in{
		fmt.Println(i)
	}
}
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go counter(ch1)
	go squarer(ch2,ch1)
	printer(ch1)
}
```

### select 多路复用

```go
func main() {
	ch := make(chan int, 1)
	for i := 0; i < 10; i++ {
		select {
		case x := <-ch:
			fmt.Println(x)
		case ch<-i:
		}
	}
}
```

- 可处理一个或多个channel的发送/接收操作。
- 如果多个`case`同时满足，`select`会随机选择一个。
- 对于没有`case`的`select{}`会一直等待，可用于阻塞main函数。

## 并发安全和锁

### 互斥锁Mutex

Go中使用`sync`包的`Mutex`类型来实现互斥类。使用互斥锁来避免并发问题。

```go
var x int64
var wg sync.WaitGroup
var lock sync.Mutex
func add() {
	for i := 0; i < 5000; i++ {
		lock.Lock()
		x = x + 1
		lock.Unlock()
	}
	wg.Done()
}
func main() {
	wg.Add(2)
	go add()
	go add()
	wg.Wait()
	fmt.Println(x)
}
```

使用互斥锁能够保证同一时间有且只有一个`goroutine`进入临界区，其他的`goroutine`则在等待锁；当互斥锁释放后，等待的`goroutine`才可以获取锁进入临界区，多个`goroutine`同时等待一个锁时，唤醒的策略是随机的。



### 读写锁RWMutex

```go
var (
	x int64
	wg sync.WaitGroup
	lock sync.Mutex
	rwlock sync.RWMutex
)

func write() {
	//lock.Lock()
	rwlock.Lock()
	x = x+1
	time.Sleep(10 * time.Millisecond)
	rwlock.Unlock()
	//lock.Unlock()
	wg.Done()
}
func read(){
	//lock.Lock()
	rwlock.RLock()
	time.Sleep(time.Millisecond)
	rwlock.RUnlock()
	wg.Done()
}
func main() {
	start := time.Now()
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go write()
	}
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go read()
	}
	wg.Wait()
	end := time.Now()
	fmt.Println(end.Sub(start))
}
```

### sync.WaitGroup

`Add(delta int),Down(),Wait()`

WaitGroup是一个结构体，传递的时候需要**传递指针**。

### sync.Once

某些操作在高并发的场景下只执行一次，如只加载一次配置文件、只关闭一次通道等。

`func (o *Once) Do(f func()){}`

*备注：如果要执行的函数`f`需要传递参数就需要搭配闭包来使用。*

### sync.Map

Go中内置的map不是并发安全的。如：

```go
var m = make(map[string]int)

func get(key string) int {
	return m[key]
}

func set(key string, value int) {
	m[key] = value
}

func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			set(key, n)
			fmt.Printf("k=:%v,v:=%v\n", key, get(key))
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

需要加锁来保证并发的安全性，`sync`包中提供了一个开箱即用的并发安全版map-sync.Map。不需要使用make函数初始化就能直接使用。同时内置了`Store`,`Load`,`LoadOrStore`,`Delete`,`Range`等操作。

```go
var m = sync.Map{}
func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			m.Store(key, n)
			value, _ := m.Load(key)
			fmt.Printf("k=:%v,v:=%v\n", key, value)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```





## context

被译作上下文，是一个抽象概念。可理解为程序单元的一个运行状态、线程或者快照。Go中的程序单元为Goroutine。

Goroutine在执行之前，都要先知道程序当前的执行状态，通常会将这些状态封装在一个Context变量中，用于传递给要执行的Goroutine中。在网络编程中收到一个请求Request并进行处理时，通常需要开启多个goroutine来获取数据与逻辑处理。

即一个请求Request会在多个goroutine中处理。而这些goroutine可能需要共享Request的一些信息，同时当request被取消或者超市时，所有从这个Request创建的所有goroutine也应该被结束。

### context包

context包不仅实现了程序单元之间共享状态变量的方法，也能通过简单的方法，使得在被调用的程序单元外部，通过一些方法设置ctx的变量值，将过期或撤销这些信号能够传递给被调用的程序单元。

在网络编程中，若存在A调用B，B再调用C时，如果A调用B取消，那么也应该取消B的调用，通过在A，B，C的调用API之间传递Context，以及判断其状态就可以很好的解决这个问题。这就是gRPC的接口中需要带上ctx context.Context参数的原因之一。

context包的核心接口Context定义如下：

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

- Deadline会返回一个超时时间，Goroutine获得超时时间后，一个例子是：可以对某些io操作设定超时时间。
- Done方法返回一个信道（channel），当Context被撤销或过期时，该信道是关闭的，即它是一个表示Context是否已关闭的信号。
- 当Done信道关闭后，Err方法表明Context被撤销的原因。
- Value可以让Goroutine共享一些数据，当然获取数据是协程安全的。但使用时需要注意同步，如对变量加上读写锁。

Context接口没有提供方法来设置其值和国旗事件，所以说没有提供方法直接将其自身销毁，就是说Context不能改变和撤销自身，那么该如何通过Context传递改变后的状态？

### context的使用

goroutine间的创建和调用关系总是层层进行的，更靠近顶部的goroutine应有办法主动关闭其下属的goroutine的执行（否则程序可能就失控了）。为了实现这种关系，Context也应该像一棵树，叶子节点总是需要通过父节点衍生。

要创建Context树，首先就是获取到根结点，context.Background函数的返回值就是根节点。

```go
func Background() Context
```

该函数返回空的Context，该Context一般由接受Request请求的第一个goroutine创建，进入请求的Context根结点，它不能被取消、没有值、也没有过期时间。它常常作为处理Request的顶层context存在。

有了根结点，如何创建其他的子节点，孙节点呢？context提供了多个函数来创建它们：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key interface{}, val interface{}) Context
```

上述函数都接受以一个Context类型的参数parent，并返回一个Context类型的值，通过这些方法层层创建出不同的节点。并且根据接受参数设定子节点的一些状态值，接着就可以将子节点传递给下层的goroutine了。

通过这些函数可以通过Context传递改变后的状态，子context通过WithCancel方法，从而获得cancel的权利。

调用`CancelFunc`对象将撤销对应的`Context`对象，这就是主动撤销`Context`的方法。在父节点的`Context`所对应的环境中，通过`WithCancel`函数不仅可创建子节点的`Context`，同时也获得了该节点`Context`的控制权，一旦执行该函数，则该节点`Context`就结束了，则子节点需要类似如下代码来判断是否已结束，并退出该Goroutine：

```go
select {
	case -<cxt.Done():
			//do some clean
}
```

`WithDeadline`函数的作用也差不多，它返回的Context类型值同样是`parent`的副本，但其过期时间由`deadline`和`parent`的过期时间共同决定。当`parent`的过期时间早于传入的`deadline`时间时，返回的过期时间应与`parent`相同。父节点过期时，其所有的子孙节点必须同时关闭；反之，返回的父节点的过期时间则为`deadline`。

`WithTimeout`函数与`WithDeadline`类似，只不过它传入的是从现在开始Context剩余的生命时长。他们都同样也都返回了所创建的子Context的控制权，一个`CancelFunc`类型的函数变量。

当顶层的Request请求函数结束后，我们就可以cancel掉某个context，从而层层Goroutine根据判断`cxt.Done()`来结束。

`WithValue`函数，它返回`parent`的一个副本，调用该副本的Value(key)方法将得到val。这样我们不光将根节点原有的值保留了，还在子孙节点中加入了新的值，注意若存在Key相同，则会被覆盖。



### 总结

`context`包通过构建树型关系的Context，来达到上一层Goroutine能对传递给下一层Goroutine的控制。对于处理一个Request请求操作，需要采用`context`来层层控制Goroutine，以及传递一些变量来共享。

- Context对象的生存周期一般仅为一个请求的处理周期。即针对一个请求创建一个Context变量（它为Context树结构的根）；在请求处理结束后，撤销此ctx变量，释放资源。
- 每次创建一个Goroutine，要么将原有的Context传递给Goroutine，要么创建一个子Context并传递给Goroutine。
- Context能灵活地存储不同类型、不同数目的值，并且使多个Goroutine安全地读写其中的值。
- 当通过父Context对象创建子Context对象时，可同时获得子Context的一个撤销函数，这样父Context对象的创建环境就获得了对子Context将要被传递到的Goroutine的撤销权。





## 标准库http/template

`html/template` 包实现了数据驱动的模板，用于生成可防止代码注入的安全的HTML内容。提供了和`text/template`包相同的接口。Go中输出HTML的场景都应使用`html/template`包



### Go的模版引擎

内置了文本模版引擎`text/template` 和用于HTML文档的`html/template`。它们的作用机制简单归纳如下：

1. 模版文件通常定义为`.tmpl`和`.tpl`为后缀，必须使用UTF8编码。
2. 模版





## 内存对齐

是处理器为了提高处理性能而对存取数据的起始地址所提出的一种要求。

保证尽可能减少对内存的访问次数以实现对数据结构进行更加高效的操作。





## 内存管理

程序启动的时候会申请一块内存（此时还只是虚拟的地址空间），并将其切分后进行管理。

申请到的内存被分配了三个区域。

![image-20210714150809591](/Users/didi/Downloads/Go.assets/image-20210714150809591.png)

`arena`就是所谓的堆区，Go动态分配的内存都是在这个区域，它把内存分割成`8kb`大小的页（page），组合起来称为`mspan`。

`bitmap区域`标识`arena`区域哪些地址保存了对象，并用`4bit`标志位表示对象是否包含指针、GC标记信息。





## Error处理

error是一个接口类型。这个接口的声明中只包含了一个方法Error。Error方法不接受任何参数，但是会返回一个string类型的结果。通常的使用方法是获取返回结果，判断其返回值是否"不为nil"，如果"不为nil"就进入错误处理流程。

其次因为该接口中只有一个方法Error，所以当需要创建一个新的错误类型时，实现该方法即可。

程序中往往会有多个错误类型，对于具体错误的判断，Go中如何处理？

由于error是一个接口类型，所以即使同为error类型的错误值，它们的实际类型也可能不同。或者问：怎么判断一个错误具体代表哪一类错误？

- 对于类型在已知范围内的一系列错误值，一般使用类型断言表达式或者类型swtich语句来判断即可。
- 对于已有相应变量且类型相同的一系列错误值，一般直接使用判等操作来判断。
- 对于没有相应变量且类型未知的一系列错误值，只能使用其错误信息的字符串表示形式来做判断。



构建错误体系的基本方式有两种：创建立体的错误类型体系和创建扁平的错误值列表



### 例子：

标准库的net代码包中有一个名为Error的接口类型。算是内建接口类型error的一个扩展接口，因为error是net.Error的嵌入接口。

net.Error接口除了拥有error接口的Error方法之外，还有两个自己声明的方法:Timeout和Temporary。

net包中有很多错误类型都实现了net.Error接口：

1. *net.OpError;
2. *net.AddrError;
3. net.UnknownNetworkError等

可以把这些错误类型想象成一棵树，内建接口error就是树的根，而net.Error接口就是一个在根上延伸的第一级非叶子节点。



**通过用类型建立起树形结构的错误体系，用统一字段建立起可追根溯源的链式错误关联。**



## 运行时恐慌panic

该异常只会在程序运行时被抛出。

比如当一个切片长度是5，但我们在程序中想通过索引5访问其中的元素值，就会在运行中被提示越界了。

当然不仅仅是个提示，程序还会打印出panic的详细情况之后，终止运行。

```go
panic: runtime error: index out of range

goroutine 1 [running]:
main.main()
 /Users/haolin/GeekTime/Golang_Puzzlers/src/puzzlers/article19/q0/demo47.go:5 +0x3d
exit status 2
```

首先其中的“runtime error”的含义是，这是一个runtime代码包中抛出的 panic。在该panic中包含了一个runtime.Error接口类型的值。

此外，panic 详情中，一般还会包含与它的引发原因有关的 goroutine 的代码执行信息。如详情中所示的

`goroutine 1[running]`，表示有一个ID为1的goroutine在此panic被引发的时候正在运行。

下一行，“main.main()”表明了这个 goroutine 包装的go函数就是命令源码文件中的那个main函数，也就是说这里的 goroutine 正是主 goroutine。再下面的一行，指出的就是这个 goroutine 中的哪一行代码在此 panic 被引发时正在执行。



### panic被引发到终止运行的过程

首先初始的panic详情会被建立起来，并且该程序的控制权会立即从此行代码转移至调用其所属的函数的那行代码上，也就是栈中的上一级。意味着该行代码所属的函数终止。并且控制权讲沿着调用栈一级一级传播至最外层。

随后程序崩溃并终止运行，承载程序这次运行的进程也会随之死亡并消失。



panic转化为error类型值，并将其作为函数的结果值返回给调用方。





## nuwa

### 安装

```shell
curl -L -s https://git.xiaojukeji.com/nuwa/nuwa/raw/master/tools/get | sh
```

### 创建项目

```shell
nuwa new -n [SERVICE_NAME]
```



## Go工程化实践学习笔记

主要内容围绕4个case展开

<img src="/Users/didi/Downloads/Go.assets/image-20210706191306991.png" alt="image-20210706191306991" style="zoom:50%;" />

### 处理错误的三种方式

处理错误的三种方式

1. 直观的错误返回：发生时立即进行处理
2. 屏蔽过程中的error：最后统一返回error，发生错误时先存入具体的结构体中，最后对整个结构体处理。
3. 利用函数式编程的延迟运行；



![image-20210706191831399](/Users/didi/Downloads/Go.assets/image-20210706191831399.png)



### 分层下的Error Handing

在一个常见的三层调用中

1. 分层会带来大量的日志，查错时需要耗费大量的时间
2. 拼接错误字符串很费力



#### 标准库的error实现

![image-20210706192419698](/Users/didi/Downloads/Go.assets/image-20210706192419698.png)

获得最后的字符串后，很难获取到原始的error信息。



#### 统一错误底层实现

![image-20210706194216696](/Users/didi/Downloads/Go.assets/image-20210706194216696.png)

#### 从依赖结构体到依赖接口

![image-20210706194256545](/Users/didi/Downloads/Go.assets/image-20210706194256545.png)



![image-20210706195040349](/Users/didi/Downloads/Go.assets/image-20210706195040349.png)



### 类型别名

![image-20210706200143929](/Users/didi/Downloads/Go.assets/image-20210706200143929.png)

